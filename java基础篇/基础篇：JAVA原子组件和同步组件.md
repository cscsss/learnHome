# 前言
在使用多线程并发编程的时，经常会遇到对共享变量修改操作。此时我们可以选择ConcurrentHashMap，ConcurrentLinkedQueue来进行安全地存储数据。但如果单单是涉及状态的修改，线程执行顺序问题，使用Atomic开头的原子组件或者ReentrantLock、CyclicBarrier之类的同步组件，会是更好的选择，下面将一一介绍它们的原理和用法
-	原子组件的实现原理CAS
-	AtomicBoolean、AtomicIntegerArray等原子组件的用法、
-	同步组件的实现原理
-	ReentrantLock、CyclicBarrier等同步组件的用法

关注公众号，一起交流，微信搜一搜: 潜行前行
---

# 原子组件的实现原理CAS
- 	cas的底层实现可以看下之前写的一篇文章:[详解锁原理，synchronized、volatile+cas底层实现](https://juejin.cn/post/6854573210768900110)

## 应用场景
-	可用来实现变量、状态在多线程下的原子性操作
-	可用于实现同步锁(ReentrantLock)

# 原子组件
-	原子组件的原子性操作是靠使用cas来自旋操作volatile变量实现的
-	volatile的类型变量保证变量被修改时，其他线程都能看到最新的值
-	cas则保证value的修改操作是原子性的，不会被中断
### 基本类型原子类
```java
AtomicBoolean　//布尔类型
AtomicInteger　//正整型数类型
AtomicLong	  //长整型类型
```
-	使用示例
```java
public static void main(String[] args) throws Exception {
    AtomicBoolean atomicBoolean = new AtomicBoolean(false);
    //异步线程修改atomicBoolean
    CompletableFuture<Void> future = CompletableFuture.runAsync(() ->{
        try {
            Thread.sleep(1000); //保证异步线程是在主线程之后修改atomicBoolean为false
            atomicBoolean.set(false);
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    });
    atomicBoolean.set(true);
    future.join();
    System.out.println("boolean value is:"+atomicBoolean.get());
}
---------------输出结果------------------
boolean value is:false
```

## 引用类原子类
```java
AtomicReference
//加时间戳版本的引用类原子类
AtomicStampedReference
//相当于AtomicStampedReference，AtomicMarkableReference关心的是
//变量是否还是原来变量，中间被修改过也无所谓
AtomicMarkableReference
```
-	AtomicReference的源码如下，它内部定义了一个`volatile V value`，并借助VarHandle(具体子类是FieldInstanceReadWrite)实现原子操作，MethodHandles会帮忙计算value在类的偏移位置，最后在VarHandle调用Unsafe.`public final native boolean compareAndSetReference(Object o, long offset, Object expected, Object x)`方法原子修改对象的属性
```java
public class AtomicReference<V> implements java.io.Serializable {
    private static final long serialVersionUID = -1848883965231344442L;
    private static final VarHandle VALUE;
    static {
        try {
            MethodHandles.Lookup l = MethodHandles.lookup();
            VALUE = l.findVarHandle(AtomicReference.class, "value", Object.class);
        } catch (ReflectiveOperationException e) {
            throw new ExceptionInInitializerError(e);
        }
    }
    private volatile V value;
    ....
```
### ABA问题
- 线程X准备将变量的值从A改为B，然而这期间线程Y将变量的值从A改为C，然后再改为A；最后线程X检测变量值是A，并置换为B。但实际上，A已经不再是原来的A了
- 解决方法，是把变量定为唯一类型。值可以加上版本号，或者时间戳。如加上版本号，线程Y的修改变为A1->B2->A3，此时线程X再更新则可以判断出A1不等于A3 
-	AtomicStampedReference的实现和AtomicReference差不多，不过它原子修改的变量是`volatile Pair<V> pair;`，Pair是其内部类。AtomicStampedReference可以用来解决ABA问题
```java
public class AtomicStampedReference<V> {
    private static class Pair<T> {
        final T reference;
        final int stamp;
        private Pair(T reference, int stamp) {
            this.reference = reference;
            this.stamp = stamp;
        }
        static <T> Pair<T> of(T reference, int stamp) {
            return new Pair<T>(reference, stamp);
        }
    }
    private volatile Pair<V> pair;
```
-	如果我们不关心变量在中间过程是否被修改过，而只是关心当前变量是否还是原先的变量，则可以使用AtomicMarkableReference
-	AtomicStampedReference的使用示例
```java
public class Main {
    public static void main(String[] args) throws Exception {
        Test old = new Test("hello"), newTest = new Test("world");
        AtomicStampedReference<Test> reference = new AtomicStampedReference<>(old, 1);
        reference.compareAndSet(old, newTest,1,2);
        System.out.println("对象："+reference.getReference().name+";版本号："+reference.getStamp());
    }
}
class Test{
    Test(String name){ this.name = name; }
    public String name;
}
---------------输出结果------------------
对象：world;版本号：2
```
### 数组原子类
```java
AtomicIntegerArray	　//整型数组
AtomicLongArray		　//长整型数组
AtomicReferenceArray	//引用类型数组
```
-	数组原子类内部会初始一个final的数组，它把整个数组当做一个对象，然后根据下标index计算法元素偏移量，再调用UNSAFE.compareAndSetReference进行原子操作。数组并没被volatile修饰，为了保证元素类型在不同线程的可见，获取元素使用到了UNSAFE`public native Object getReferenceVolatile(Object o, long offset)`方法来获取实时的元素值
-	使用示例
```java
//元素默认初始化为0
AtomicIntegerArray array = new AtomicIntegerArray(2);
// 下标为０的元素，期待值是0，更新值是１
array.compareAndSet(0,0,1);
System.out.println(array.get(0));
---------------输出结果------------------
1
```

### 属性原子更新类
```java
AtomicIntegerFieldUpdater　
AtomicLongFieldUpdater
AtomicReferenceFieldUpdater
```
-	如果操作对象是某一类型的属性，可以使用AtomicIntegerFieldUpdater原子更新，不过类的属性需要定义成volatile修饰的变量，保证该属性在各个线程的可见性，否则会报错
-	使用示例
```java
public class Main {
    public static void main(String[] args) {
        AtomicReferenceFieldUpdater<Test,String> fieldUpdater = AtomicReferenceFieldUpdater.newUpdater(Test.class,String.class,"name");
        Test test = new Test("hello world");
        fieldUpdater.compareAndSet(test,"hello world","siting");
        System.out.println(fieldUpdater.get(test));
        System.out.println(test.name);
    }
}
class Test{
    Test(String name){ this.name = name; }
    public volatile String name;
}
---------------输出结果------------------
siting
siting
```

### 累加器
```java
Striped64
LongAccumulator
LongAdder
//accumulatorFunction：运算规则，identity：初始值
public LongAccumulator(LongBinaryOperator accumulatorFunction,long identity)
```
-	LongAccumulator和LongAdder都继承于Striped64，Striped64的主要思想是和ConcurrentHashMap有点类似，分段计算，单个变量计算并发性能慢时，我们可以把数学运算分散在多个变量，而需要计算总值时，再一一累加起来
-	LongAdder相当于LongAccumulator一个特例实现
-	LongAccumulator的示例
```java
public static void main(String[] args) throws Exception {
    LongAccumulator accumulator = new LongAccumulator(Long::sum, 0);
    for(int i=0;i<100000;i++){
        CompletableFuture.runAsync(() -> accumulator.accumulate(1));
    }
    Thread.sleep(1000); //等待全部CompletableFuture线程执行完成，再获取
    System.out.println(accumulator.get());
}
---------------输出结果------------------
100000
```
# 同步组件的实现原理
-	java的多数同步组件会在内部维护一个状态值，和原子组件一样，修改状态值时一般也是通过cas来实现。而状态修改的维护工作被Doug Lea抽象出AbstractQueuedSynchronizer(AQS)来实现
- 	AQS的原理可以看下之前写的一篇文章:[详解锁原理，synchronized、volatile+cas底层实现](https://juejin.cn/post/6854573210768900110)

# 同步组件
## ReentrantLock、ReentrantReadWriteLock
-	ReentrantLock、ReentrantReadWriteLock都是基于AQS(AbstractQueuedSynchronizer)实现的。因为它们有公平锁和非公平锁的区分，因此没直接继承AQS，而是使用内部类去继承，公平锁和非公平锁各自实现AQS，ReentrantLock、ReentrantReadWriteLock再借助内部类来实现同步
-	ReentrantLock的使用示例
```java
ReentrantLock lock = new ReentrantLock();
if(lock.tryLock()){
    //业务逻辑
    lock.unlock();
}
```
-	ReentrantReadWriteLock的使用示例
```java
public static void main(String[] args) throws Exception {
    ReentrantReadWriteLock lock = new ReentrantReadWriteLock();
    if(lock.readLock().tryLock()){ //读锁
        //业务逻辑
        lock.readLock().unlock();
    }
    if(lock.writeLock().tryLock()){ //写锁
        //业务逻辑
        lock.writeLock().unlock();
    }
}
```
## Semaphore实现原理和使用场景
-	Semaphore和ReentrantLock一样，也有公平和非公平竞争锁的策略，一样也是通过内部类继承AQS来实现同步
-	通俗解释：假设有一口井，最多有三个人的位置打水。每有一个人打水，则需要占用一个位置。当三个位置全部占满时，第四个人需要打水，则要等待前三个人中一个离开打水位，才能继续获取打水的位置
-	使用示例
```java
public static void main(String[] args) throws Exception {
    Semaphore semaphore = new Semaphore(2);
    for (int i = 0; i < 3; i++)
        CompletableFuture.runAsync(() -> {
            try {
                System.out.println(Thread.currentThread().toString() + " start ");
                if(semaphore.tryAcquire(1)){
                    Thread.sleep(1000);
                    semaphore.release(1);
                    System.out.println(Thread.currentThread().toString() + " 无阻塞结束 ");
                }else {
                    System.out.println(Thread.currentThread().toString() + " 被阻塞结束 ");
                }
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        });
    //保证CompletableFuture 线程被执行，主线程再结束
    Thread.sleep(2000);
}
---------------输出结果------------------
Thread[ForkJoinPool.commonPool-worker-19,5,main] start 
Thread[ForkJoinPool.commonPool-worker-5,5,main] start 
Thread[ForkJoinPool.commonPool-worker-23,5,main] start 
Thread[ForkJoinPool.commonPool-worker-23,5,main] 被阻塞结束 
Thread[ForkJoinPool.commonPool-worker-5,5,main] 无阻塞结束 
Thread[ForkJoinPool.commonPool-worker-19,5,main] 无阻塞结束 
```
-	可以看出三个线程，因为信号量设定为２，第三个线程是无法获取信息成功的，会打印阻塞结束
## CountDownLatch实现原理和使用场景
-	CountDownLatch也是靠AQS实现的同步操作
-	通俗解释：玩游戏时，假如主线任务需要靠完成五个小任务，主线任务才能继续进行时。此时可以用CountDownLatch，主线任务阻塞等待，每完成一小任务，就done一次计数，直到五个小任务全部被执行才能触发主线
-	使用示例
```java
public static void main(String[] args) throws Exception {
    CountDownLatch count = new CountDownLatch(2);
    for (int i = 0; i < 2; i++)
        CompletableFuture.runAsync(() -> {
            try {
                Thread.sleep(1000);
                System.out.println(" CompletableFuture over ");
                count.countDown();
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        });
    //等待CompletableFuture线程的完成
    count.await();
    System.out.println(" main over ");
}
---------------输出结果------------------
 CompletableFuture over 
 CompletableFuture over 
 main over 
```
## CyclicBarrier实现原理和使用场景
-	CyclicBarrier则是靠`ReentrantLock lock`和`Condition trip`属性来实现同步
-	通俗解释：CyclicBarrier需要阻塞全部线程到await状态，然后全部线程再全部被唤醒执行。想象有一个栏杆拦住五只羊，需要当五只羊一起站在栏杆时，栏杆才会被拉起，此时所有的羊都可以飞跑出羊圈
-	使用示例
```java
public static void main(String[] args) throws Exception {
    CyclicBarrier barrier = new CyclicBarrier(2);
    CompletableFuture.runAsync(()->{
        try {
            System.out.println("CompletableFuture run start-"+ Clock.systemUTC().millis());
            barrier.await(); //需要等待main线程也执行到await状态才能继续执行
            System.out.println("CompletableFuture run over-"+ Clock.systemUTC().millis());
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    });
    Thread.sleep(1000);
    //和CompletableFuture线程相互等待
    barrier.await();
    System.out.println("main run over!");
}
---------------输出结果------------------
CompletableFuture run start-1609822588881
main run over!
CompletableFuture run over-1609822589880
```
## StampedLock
-	StampedLock不是借助AQS，而是自己内部维护多个状态值，并配合cas实现的
-	StampedLock具有三种模式：写模式、读模式、乐观读模式
-	StampedLock的读写锁可以相互转换
```java
//获取读锁，自旋获取，返回一个戳值
public long readLock()
//尝试加读锁，不成功返回0
public long tryReadLock()
//解锁
public void unlockRead(long stamp) 
//获取写锁，自旋获取，返回一个戳值
public long writeLock()
//尝试加写锁，不成功返回0
public long tryWriteLock()
//解锁
public void unlockWrite(long stamp)
//尝试乐观读读取一个时间戳，并配合validate方法校验时间戳的有效性
public long tryOptimisticRead()
//验证stamp是否有效
public boolean validate(long stamp)
```
-	使用示例
```java
public static void main(String[] args) throws Exception {
    StampedLock stampedLock = new StampedLock();
    long stamp = stampedLock.tryOptimisticRead();
    //判断版本号是否生效
    if (!stampedLock.validate(stamp)) {
        //获取读锁，会空转
        stamp = stampedLock.readLock();
        long writeStamp = stampedLock.tryConvertToWriteLock(stamp);
        if (writeStamp != 0) { //成功转为写锁
            //fixme 业务操作
            stampedLock.unlockWrite(writeStamp);
        } else {
            stampedLock.unlockRead(stamp);
            //尝试获取写读
            stamp = stampedLock.tryWriteLock();
            if (stamp != 0) {
                //fixme 业务操作
                stampedLock.unlockWrite(writeStamp);
            }
        }
    }
}    
```

欢迎指正文中错误
---

#  参考文章
-	[并发之Striped64（l累加器）](https://www.cnblogs.com/gosaint/p/9129867.html)

