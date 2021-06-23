# 前言
平时并发编程，除了维护修改共享变量的场景，有时我们也需要为每一个线程设置一个私有的变量，进行线程隔离，java提供的ThreadLocal可以帮助我们实现，而讲到ThreadLocal则不得不讲讲java的四种引用，不同的引用类型在GC时表现是不一样的，引用类型Reference有助于我们了解如何快速回收某些对象的内存或对实例的GC控制

-	四种引用类型在JVM的生命周期
-	引用队列(ReferenceQueue)
-	ThreadLocal的实现原理和使用
-	FinalReference和finalize方法的实现原理
-	Cheaner机制

关注公众号，一起交流，微信搜一搜: 潜行前行
---

## 1 四种引用类型在JVM的生命周期 
### 强引用(StrongReference)
-	创建一个对象并赋给一个引用变量，强引用有引用变量指向时，永远也不会垃圾回收，JVM宁愿抛出OutOfMemory异常也不会回收该对象；强引用对象的创建，如
```java
Integer index = new Integer(1);
String name = "csc";
```
-	如果中断所有引用变量和强引用对象的联系（将引用变量赋值为null），JVM则会在合适的时间就会回收该对象

### 软引用(SoftReference)
-	和强用引用不同点在于内存不足时，该类型引用对象会被垃圾处理器回收
-	使用软引用能防止内存泄露，增强程序的健壮性。SoftReference的特点是它的一个实例保存对一个Java对象的软引用，该软引用的存在不妨碍垃圾收集线程对该Java对象的回收
-	SoftReference类所提供的get()方法返回Java对象的强引用。另外，一旦垃圾线程回收该对象之后，get()方法将返回null
```java
    String name = "csc";
    //软引用的创建
    SoftReference<String> softRef = new SoftReference<String>(name);
    System.out.println(softRef.get());
```
### 弱引用(WeakReference)
-	特点：无论内存是否充足，只要进行GC，都会被回收
```java

    static class User{
        String name;
        public User(String name){   this.name = name;   }
        public String getName() {  return name;  }
        public void setName(String name) {     this.name = name;  }
    }
    //弱引用的创建
    WeakReference<User> softRef = new WeakReference<User>(new User("csc"));
    System.out.println(softRef.get().getName()); //输出 csc
    System.gc();
    System.out.println(softRef.get()); //输出 null
    //弱引用Map
    WeakHashMap<String, String> map = new WeakHashMap<String, String>();

```
### 虚引用(PhantomReference)
-	特点：如同虚设，和没有引用没什么区别；虚引用和软引用、弱引用不同，它并不决定对象的生命周期。如果一个对象与虚引用关联，则跟没有引用与之关联一样，在任何时候都可能被垃圾回收器回收
-	要注意的是，虚引用必须和引用队列关联使用，当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会把这个虚引用加入到与之关联的引用队列中。程序可以通过判断引用队列中是否已经加入了虚引用，来了解被引用的对象是否将要被垃圾回收
```java
public static void main(String[] args) {
   
    ReferenceQueue<User> queue = new ReferenceQueue<>();  
    PhantomReference<User> pr = new PhantomReference<User>(new User("csc"), queue);  
    //PhantomRefrence的get方法总是返回null，因此无法访问对应的引用对象。
    System.out.println(pr.get());  // null
    System.gc();
    System.out.println(queue.poll()); //获取被垃圾回收的"xb"的引用ReferenceQueue
}
```


引用类型 | 被垃圾回收时间	| 场景 |  生存时间
---   | ---          |---    |   --- 
强引用 | 从来不会      | 对象的一般状态 | JVM停止运行时终止
软引用	| 当内存不足时   |	对象缓存|	内存不足时终止
弱引用 | 正常垃圾回收时 |	对象缓存|	垃圾回收后终止
虚引用	| 正常垃圾回收时 |	跟踪对象的垃圾回收	|垃圾回收后终止

## 2 引用队列(ReferenceQueue)
-	引用队列可以配合软引用、弱引用及虚引用使用；当引用的对象将要被JVM回收时，会将其加入到引用队列中
```java
  ReferenceQueue<String> queue = new ReferenceQueue<String>();  
  WeakReference<String> pr = new WeakReference<String>("wxj", queue); 
  System.gc();
  System.out.println(queue.poll().get()); // 获取即将被回收的字符串　wxj
```

## 3 ThreadLocal的原理和使用
### ThreadLocal 的实现原理
- 每个线程都内置了一个ThreadLocalMap对象
```java
public class Thread implements Runnable {
	/* 当前线程对于的ThreadLocalMap实例，ThreadLocal<T>作为Key, 
	 * T对应的对象作为value */
	ThreadLocal.ThreadLocalMap threadLocals = null;
```
- ThreadLocalMap作为ThreadLocal的内部类，实现了类似HashMap的功能，它元素Entry继承于WeakReference，key值是ThreadLocal，value是引用变量。也就是说jvm发生GC时value对象则会被回收
```java
public class ThreadLocal<T> {
	//ThreadLocal对象对应的hash值，使用一个静态AtomicInteger实现
	private final int threadLocalHashCode = nextHashCode();
    //设置value
	public void set(T value) {
        Thread t = Thread.currentThread();  
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
    //获取value
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
      　　　　//获取当前线程的ThreadLocalMap，再使用对象ThreadLocal获取对应的value
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
    ....
    //类似HashMap的类
    static class ThreadLocalMap {
    	//使用开放地址法解决hash冲突
        //如果hash出的index已经有值，通过算法在后面的若干位置寻找空位
    	private Entry[] table;
        ...
        //Entry 是弱引用
        static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;
            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
   }     
```

### ThreadLocal不保证共享变量在多线程的安全性
-	从ThreadLocal的实现原理可知，ThreadLocal只是为每个线程保存一个副本变量，副本变量的修改不影响其他线程的变量值，因此ThreadLocal不能实现共享变量的安全性

### ThreadLocal 使用场景
-	线程安全，包裹线程不安全的工具类，比如java.text.SimpleDateFormat类，当然jdk1.8已经给出了对应的线程安全的类java.time.format.DateTimeFormatter
-	线程隔离，比如数据库连接管理、Session管理、mdc日志追踪等。

### ThreadLocal内存泄露和WeakReference
-	ThreadLocalMap.Entry是弱引用，弱引用对象是不管有没有被引用都会被垃圾回收
-	发生内存泄漏一般是在线程池的线程，生命周期长，threadLocals引用会一直存在，当其存放的ThreadLocal被回收（弱引用生命周期短）后，它对应的Entity成了e.get()==null的实例。线程不死则Entity一直不会被回收，这就发生了内存泄漏
-	如果线程跨业务操作相同的ThreadLocal，还会造成变量安全问题
-	通常在使用完ThreadLocal最好调用它的remove()；在ThreadLocal的get、set的时候，最好检查当前Entity的key是否为null，如果是null就把Entity释放掉，value则会被垃圾回收

## 4 finalize方法的实现原理FinalReference
```java
final class Finalizer extends FinalReference<Object> { 
　　private static ReferenceQueue<Object> queue = new ReferenceQueue<>();
  	/* Invoked by VM */
   static void register(Object finalizee) {
        new Finalizer(finalizee);
   }
   private static class FinalizerThread extends Thread {
        ....
        public void run() {
            ...
            for (;;) {
                try {
                    Finalizer f = (Finalizer)queue.remove();
                    //这里会实现Object.finalize的调用
                    f.runFinalizer(jla);
           ....
    }
    
   static {
        ...
        Thread finalizer = new FinalizerThread(tg);
        ...　//执行Object.finalize的守护线程
        finalizer.setDaemon(true);
        finalizer.start();
    }
    
```
-	cpu资源比较稀缺的情况下FinalizerThread线程有可能因为优先级比较低而延迟执行finalizer对象的finalize方法
-	因为finalizer对象的finalize方法迟迟没有执行，有可能会导致大部分finalizer对象进入到old分代，此时容易引发old分代的gc，甚至fullgc，gc暂停时间明显变长

## 5 Cheaner机制
-	上一篇文章有介绍到jdk1.8的Cleaner[框架篇：ByteBuffer和netty.ByteBuf详解](https://juejin.cn/post/6939403878366445605)


欢迎指正文中错误
---
#  参考文章
-	[ThreadLocal原理及使用场景大揭秘](https://www.jianshu.com/p/5af663b35779)
-	[JDK源码分析之FinalReference完全解读](https://blog.csdn.net/zero__007/article/details/60146268)
-	[一次 Young GC 的优化实践](https://www.jianshu.com/p/79d4a0516f11)
-	[Netty资源泄露检测](https://blog.csdn.net/yangguosb/article/details/80138719)
-	[避免使用Finalizer和Cleaner机制](https://www.cnblogs.com/IcanFixIt/p/8133798.html)
-	[【JAVA Reference】Cleaner 源码剖析（三）](https://blog.csdn.net/Sword52888/article/details/101452260)
