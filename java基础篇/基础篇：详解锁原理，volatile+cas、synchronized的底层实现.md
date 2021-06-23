- 随着多进程多线程的出现，对共享资源(设备，数据等)的竞争往往会导致资源的使用表现为随机无序
- 例如：一个线程想在控制台输出"I am fine"，刚写到"I am"，就被另一线程抢占控制台输出"naughty"，导致结果是"I am naughty"；对于资源的被抢占使用，我们能怎么办呢？当然不是凉拌，可使用锁进行同步管理，使得资源在加锁期间，其他线程不可抢占使用

# 1 锁的分类
- 悲观锁
    - 悲观锁，每次去请求数据的时候，都认为数据会被抢占更新(悲观的想法)；所以每次操作数据时都要先加上锁，其他线程修改数据时就要等待获取锁。适用于写多读少的场景，synchronized就是一种悲观锁
- 乐观锁     
    - 在请求数据时，觉得无人抢占修改。等真正更新数据时，才判断此期间别人有没有修改过(预先读出一个版本号或者更新时间戳，更新时判断是否变化，没变则期间无人修改)；和悲观锁不同的是，期间数据允许其他线程修改
- 自旋锁
    - 一句话，魔力转转圈。当尝试给资源加锁却被其他线程先锁定时，不是阻塞等待而是循环再次加锁
    - 在锁常被短暂持有的场景下，线程阻塞挂起导致CPU上下文频繁切换，这可用自旋锁解决；但自旋期间它占用CPU空转，因此不适用长时间持有锁的场景

# 2 synchronized底层原理
- 代码使用synchronized加锁，在编译之后的字节码是怎样的呢
```java
public class Test {
    public static void main(String[] args){
        synchronized(Test.class){
            System.out.println("hello");
        }
    }
}
```
截取部分字节码，如下
```c
    4: monitorenter
    5: getstatic    #9    // Field java/lang/System.out:Ljava/io/PrintStream; 
    8: ldc           #15   // String hello
    10: invokevirtual #17  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
    13: aload_1
    14: monitorexit
```
字节码出现了4: monitorenter和14: monitorexit两个指令；字面理解就是监视进入，监视退出。可以理解为代码块执行前的加锁，和退出同步时的解锁
- 那monitorenter和monitorexit，又背着我们干了啥呢？
- 执行monitorenter指令时，线程会为锁对象关联一个ObjectMonitor对象
```c
objectMonitor.cpp
  ObjectMonitor() {
    _header       = NULL;
    _count        = 0;   \\用来记录获取该锁的线程数
    _waiters      = 0,
    _recursions   = 0;    \\锁的重入次数
    _object       = NULL;
    _owner        = NULL;  \\当前持有ObjectMonitor的线程
    _WaitSet      = NULL;  \\wait()方法调用后的线程等待队列
    _WaitSetLock  = 0 ;
    _Responsible  = NULL ;
    _succ         = NULL ;
    _cxq          = NULL ; \\阻塞等待队列
    FreeNext      = NULL ;
    _EntryList    = NULL ; \\synchronized 进来线程的排队队列
    _SpinFreq     = 0 ;
    _SpinClock    = 0 ;  \\自旋计算
    OwnerIsThread = 0 ;
  }
```
- 每个线程都有两个ObjectMonitor对象列表，分别为free和used列表，如果当前free列表为空，线程将向全局global list请求分配ObjectMonitor
- ObjectMonitor的owner、WaitSet、Cxq、EntryList这几个属性比较关键。WaitSet、Cxq、EntryList的队列元素是包装线程后的对象-ObjectWaiter；而获取owner的线程，既为获得锁的线程
- **monitorenter对应的执行方法**
```c
void ATTR ObjectMonitor::enter(TRAPS)  {
    ...
    //获取锁：cmpxchg_ptr原子操作，尝试将_owner替换为自己，并返回旧值
    cur = Atomic::cmpxchg_ptr (Self, &_owner, NULL) ;
    ...
    // 重复获取锁，次数加1，返回
    if (cur == Self) {
        _recursions ++ ;
        return ;
    }
    //首次获取锁情况处理
    if (Self->is_lock_owned ((address)cur)) {
        assert (_recursions == 0, "internal state error");
        _recursions = 1 ;
        _owner = Self ;
        OwnerIsThread = 1 ;
        return ;
    }
    ...
    //尝试自旋获取锁
    if (Knob_SpinEarly && TrySpin (Self) > 0) {
    ...
```
- **monitorexit对应的执行方**法`void ATTR ObjectMonitor::exit(TRAPS)...`代码太长，就不贴了。主要是recursions减1、count减少1或者如果线程不再持有owner(非重入加锁)则设置owner为null，退锁的持有状态，并唤醒Cxq队列的线程

**总结**
- 线程遇到synchronized同步时，先会进入EntryList队列中，然后尝试把owner变量设置为当前线程，同时monitor中的计数器count加1，即获得对象锁。否则通过**尝试自旋一定次数加锁**，失败则进入Cxq队列阻塞等待
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAyMC83LzE4LzE3MzYwMDgwMDAyY2Q5NmU?x-oss-process=image/format,png)
- 线程执行完毕将释放持有的owner，owner变量恢复为null，count自减1，以便其他线程进入获取锁
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAyMC83LzE4LzE3MzYwNDJkODJiOTJiMjk?x-oss-process=image/format,png)
- synchronized修饰方法原理也是类似的。只不过没用monitor指令，而是使用ACC_SYNCHRONIZED标识方法的同步
```c
    public synchronized void lock(){
        System.out.println("world");
    }
....
  public synchronized void lock();
    descriptor: ()V
    flags: (0x0029) ACC_PUBLIC, ACC_SYNCHRONIZED
    Code:
      stack=2, locals=0, args_size=0
         0: getstatic     #20                 // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #26                 // String world
         5: invokevirtual #28                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V

```
- synchronized是可重入，非公平锁，因为entryList的线程会先自旋尝试加锁，而不是加入cxq排队等待，不公平

# 3 Object的wait和notify方法原理
- wait，notify必须是持有当前对象锁Monitor的线程才能调用 (对象锁代指ObjectMonitor/Monitor，锁对象代指Object)
- 上面有说到，当在sychronized中锁对象Object调用wait时会加入waitSet队列，WaitSet的元素对象就是ObjectWaiter
```java
class ObjectWaiter : public StackObj {
 public:
  enum TStates { TS_UNDEF, TS_READY, TS_RUN, TS_WAIT, TS_ENTER, TS_CXQ } ;
  enum Sorted  { PREPEND, APPEND, SORTED } ;
  ObjectWaiter * volatile _next;
  ObjectWaiter * volatile _prev;
  Thread*       _thread;
  ParkEvent *   _event;
  volatile int  _notified ;
  volatile TStates TState ;
  Sorted        _Sorted ;           // List placement disposition
  bool          _active ;           // Contention monitoring is enabled
 public:
  ObjectWaiter(Thread* thread);
  void wait_reenter_begin(ObjectMonitor *mon);
  void wait_reenter_end(ObjectMonitor *mon);
};
```
**调用对象锁的wait()方法时，线程会被封装成ObjectWaiter，最后使用park方法挂起**
```c
//objectMonitor.cpp
void ObjectMonitor::wait(jlong millis, bool interruptible, TRAPS){
    ...
    //线程封装成 ObjectWaiter对象
    ObjectWaiter node(Self);
    node.TState = ObjectWaiter::TS_WAIT ;
    ...
    //一系列判断操作，当线程确实加入WaitSet时，则使用park方法挂起
    if (node._notified == 0) {
        if (millis <= 0) {
            Self->_ParkEvent->park () ;
        } else {
            ret = Self->_ParkEvent->park (millis) ;
        }
    }

```
**而当对象锁使用notify()时**
-  如果waitSet为空，则直接返回
-  waitSet不为空从waitSet获取一个ObjectWaiter，然后根据不同的Policy加入到EntryList或通过`Atomic::cmpxchg_ptr`指令自旋操作加入**cxq队列**或者直接unpark唤醒
```c
void ObjectMonitor::notify(TRAPS){
    CHECK_OWNER();
    //waitSet为空，则直接返回
    if (_WaitSet == NULL) {
        TEVENT (Empty-Notify) ;
        return ;
    }
    ...
    //通过DequeueWaiter获取_WaitSet列表中的第一个ObjectWaiter
    Thread::SpinAcquire (&_WaitSetLock, "WaitSet - notify") ;
    ObjectWaiter * iterator = DequeueWaiter() ;
    if (iterator != NULL) {
    ....
    if (Policy == 2) {      // prepend to cxq
         // prepend to cxq
         if (List == NULL) {
             iterator->_next = iterator->_prev = NULL ;
             _EntryList = iterator ;
         } else {
            iterator->TState = ObjectWaiter::TS_CXQ ;
            for (;;) {
                ObjectWaiter * Front = _cxq ;
                iterator->_next = Front ;
                if (Atomic::cmpxchg_ptr (iterator, &_cxq, Front) == Front) {
                    break ;
                }
            }
         }
     }
```
- Object的notifyAll方法则对应`voidObjectMonitor::notifyAll(TRAPS)`，流程和notify类似。不过会通过for循环取出WaitSet的ObjectWaiter节点，再依次唤醒所有线程

# 4 jvm对synchronized的优化
- 先介绍下32位JVM下JAVA对象头的结构
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAyMC83LzE4LzE3MzYxYTFjMDU5ZGFiYTE?x-oss-process=image/format,png)
- 偏向锁
    - 未加锁的时候，锁标志为01，包含哈希值、年龄分代和偏向锁标志位(0)
    - 施加偏向锁时，哈希值和一部分无用内存会转化为锁主人的线程信息，以及加锁时的时间戳epoch，此时锁标志位没变，偏向锁标志改为1
    - 加锁时先判断当前线程id是否与MarkWord的线程id是否一致，一致则执行同步代码；不一致则检查偏向标志是否偏向，未偏向则使用CAS加锁；**未偏向CAS加锁失败**和**存在偏向锁**会导致偏向锁膨胀为轻量级锁，或者重新偏向
    - 偏向锁只有遇到其他线程竞争偏向锁时，持有偏向锁的线程才会释放锁，线程不会主动去释放偏向锁

- 轻量级锁
    - 当发生多个线程竞争时，偏向锁会变为轻量级锁，锁标志位为00
    - 获得锁的线程会先将偏向锁撤销(在安全点)，并在栈桢中创建锁记录LockRecord，对象的MarkWord被复制到刚创建的LockRecord，然后CAS尝试将记录LockRecord的owner指向锁对象，再将锁对象的MarkWord指向锁，加锁成功
    - 如果CAS加锁失败，线程会**自旋一定次数加锁**，再失败则升级为重量级锁
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAyMC83LzE5LzE3MzY1Y2UwNGZkY2RkOTE?x-oss-process=image/format,png)

- 重量级锁
    - 重量级锁就是上面介绍到synchronized使用监视器Monitor实现的锁机制
    - 竞争线程激烈，锁则继续膨胀，变为重量级锁，也是互斥锁，锁标志位为10，MarkWord其余内容被替换为一个指向对象锁Monitor的指针
- 自旋锁
    - 减少不必要的CPU上下文切换；在轻量级锁升级为重量级锁时，就使用了自旋加锁的方式
- 锁粗化
    - 多次加锁操作在JVM内部也是种消耗，如果多个加锁可以合并为一个锁，就可减少不必要的开销
```java
Test.class
//编译器会考虑将两次加锁合并
public void test(){
    synchronized(this){
        System.out.println("hello");   
    }
    synchronized(this){
        System.out.println("world");   
    }
}
```
- 锁消除
    - 删除不必要的加锁操作，如果变量是独属一个线程的栈变量，加不加锁都是安全的，编译器会尝试消除锁
    - 开启锁消除需要在JVM参数上设置`-server -XX:+DoEscapeAnalysis -XX:+EliminateLocks`

```java
//StringBuffer的append操作会加上synchronized，
//但是变量buf不加锁也安全的，编译器会把锁消除
public void test() {
    StringBuffer buf = new StringBuffer();
    buf.append("hello").append("world");
}
```
- 其他锁优化方法
    - 分段锁，分段锁也并非一种实际的锁，而是一种思想；ConcurrentHashMap是学习分段锁的最好实践。主要是将大对象拆成小对象，然后对大对象的加锁操作变成对小对象加锁，增加了并行度

# 5 CAS的底层原理
- 在`volatile int i = 0; i++`中，volatile类型的读写是原子同步的，但是i++却不能保证同步性，我们该怎么呢？
- 可以使用synchronized加锁；还有就是用CAS(比较并交换)，使用乐观锁的思想同步，先判断共享变量是否改变，没有则更新。下面看看不同步版本的CAS
```java
int expectedValue = 1;
public boolean compareAndSet(int newValue) {
    if(expectedValue == 1){
        expectedValue = newValue;
        return ture;
    }
    return false;
}
```
在jdk是有提供同步版的CAS解决方案，其中使用了UnSafe.java的底层方法
```java
//UnSafe.java
    @HotSpotIntrinsicCandidate
    public final native boolean compareAndSetInt(Object o, long offset, int expected, int x) ..
    @HotSpotIntrinsicCandidate
    public final native int compareAndExchangeInt(Object o, long offset, int expected, int x)...
```
我们再来看看本地方法，Unsafe.cpp中的compareAndSwapInt
```c
//unsafe.cpp
UNSAFE_ENTRY(jboolean, Unsafe_CompareAndSwapInt(JNIEnv *env, jobject unsafe, jobject obj, jlong offset, jint e, jint x))
  UnsafeWrapper("Unsafe_CompareAndSwapInt");
  oop p = JNIHandles::resolve(obj);
  jint* addr = (jint *) index_oop_from_field_offset_long(p, offset);
  return (jint)(Atomic::cmpxchg(x, addr, e)) == e;
UNSAFE_END
```
在Linux的x86，Atomic::cmpxchg方法的实现如下
```c
/**
    1 __asm__表示汇编的开始；
    2 volatile表示禁止编译器优化；//禁止指令重排
    3 LOCK_IF_MP是个内联函数，
      根据当前系统是否为多核处理器，
      决定是否为cmpxchg指令添加lock前缀 //内存屏障
*/
inline jint Atomic::cmpxchg (jint exchange_value, volatile jint* dest, jint compare_value) {
  int mp = os::is_MP();
  __asm__ volatile (LOCK_IF_MP(%4) "cmpxchgl %1,(%3)"
                    : "=a" (exchange_value)
                    : "r" (exchange_value), "a" (compare_value), "r" (dest), "r" (mp)
                    : "cc", "memory");
  return exchange_value;
}
```
到这一步，可以总结到：**jdk提供的CAS机制，在汇编层级，会禁止变量两侧的指令优化，然后使用cmpxchg指令比较并更新变量值(原子性)，如果是多核则使用lock锁定(缓存锁、MESI)**

# 6 CAS同步操作的问题
- ABA问题
    - 线程X准备将变量的值从A改为B，然而这期间线程Y将变量的值从A改为C，然后再改为A；最后线程X检测变量值是A，并置换为B。但实际上，A已经不再是原来的A了
    - 解决方法，是把变量定为唯一类型。值可以加上版本号，或者时间戳。如加上版本号，线程Y的修改变为A1->B2->A3，此时线程X再更新则可以判断出A1不等于A3 
- 只能保证一个共享变量的原子操作
    - 只保证一个共享变量的原子操作，对多个共享变量同步时，循环CAS是无法保证操作的原子

# 7 基于volatile + CAS 实现同步锁的原理
- CAS只能同步一个变量的修改，我们又应该如何用它来锁住代码块呢？
- 先说说实现锁的要素
    - 1 同步代码块同一时刻只能有一个线程能执行
    - 2 加锁操作要happens-before同步代码块里的操作，而代码块里的操作要happens-before解锁操作
    - 3 同步代码块结束后相对其他线程其修改的变量是可见的 (内存可见性)
- 要素1：可以利用CAS的原子性来实现，任意时刻只有一个线程能成功操作变量
    - 先设想CAS操作的共享变量是一个关联代码块的同步状态变量，同步开始之前先CAS更新**状态变量**为加锁状态，同步结束之后，再CAS**状态变量**为无锁状态
    - 如果期间有第二个线程来加锁，则会发现状态变量为加锁状态，则放弃执行同步代码块
- 要素2：使用volatile修饰状态变量，禁止指令重排
    - volatile保证同步代码里的操作happens-before解锁操作，而加锁操作happens-before代码块里的操作
- 要素3：还是用volatile，volatile变量写指令前后会插入内存屏障
    - volatile修饰的状态变量被CAS为无锁状态前，同步代码块的脏数据就会被更新，被各个线程可见
```java
//伪代码
volatile state = 0 ;   // 0-无锁 1-加锁；volatile禁止指令重排，加入内存屏障
...
if(cas(state, 0 , 1)){ // 1 加锁成功，只有一个线程能成功加锁
    ...                // 2 同步代码块
    cas(state, 1, 0);  // 3 解锁时2的操作具有可见性
}

```
# 8 LockSupport了解一下
- LockSupport是基于Unsafe类，由JDK提供的线程操作工具类，主要作用就是**挂起线程，唤醒线程**。Unsafe.park，unpark操作时，会调用当前线程的变量parker代理执行。Parker代码
```c
JavaThread* thread=JavaThread::thread_from_jni_environment(env);
...
thread->parker()->park(isAbsolute != 0, time);
```
```c
class PlatformParker : public CHeapObj {
  protected:
    //互斥变量类型
    pthread_mutex_t _mutex [1] ; 
   //条件变量类型
    pthread_cond_t  _cond  [1] ;
    ...
}

class Parker : public os::PlatformParker {  
private:  
  volatile int _counter ;  
  ...  
public:  
  void park(bool isAbsolute, jlong time);  
  void unpark();  
  ...  
}
```
- 在Linux系统下，用的POSIX线程库pthread中的mutex(互斥量)，condition来实现线程的挂起、唤醒
- 注意点：当park时，counter变量被设置为0，当unpark时，这个变量被设置为1
- unpark和park执行顺序不同时，counter和cond的状态变化如下
    - 先park后unpark; park：counter值不变，但会设置一个cond; unpark：counter先加1，检查cond存在，counter减为0
    - 先unpark后park；park：counter变为1，但不设置cond；unpark：counter减为0(线程不会因为park挂起)
    - 先多次unpark；counter也只设置为为1 

# 9 LockSupport.park和Object.wait区别
- 两种方式都有具有挂起的线程的能力
- 线程在Object.wait之后必须等到Object.notify才能唤醒
- LockSupport可以先unpark线程，等线程执行LockSupport.park是不会挂起的，可以继续执行
- 需要注意的是就算线程多次unpark；也只能让线程第一次park是不会挂起


# 10 AbstractQueuedSynchronizer(AQS)
- AQS其实就是基于volatile+cas实现的锁模板；如果需要线程阻塞等待，唤醒机制，则使用LockSupport挂起、唤醒线程
```java
//AbstractQueuedSynchronizer.java
public class AbstractQueuedSynchronizer{
    //线程节点
    static final class Node {
        ...
        volatile Node prev;
        volatile Node next;
        volatile Thread thread;
        ...
    }    
    ....
    //head 等待队列头尾节点
    private transient volatile Node head;
    private transient volatile Node tail;
    // The synchronization state. 同步状态
    private volatile int state;  
    ...
    //提供CAS操作，状态具体的修改由子类实现
    protected final boolean compareAndSetState(int expect, int update) {
        return STATE.compareAndSet(this, expect, update);
    }
}
```
- AQS内部维护一个同步队列，元素就是包装了线程的Node
- 同步队列中首节点是获取到锁的节点，它在释放锁的时会唤醒后继节点，后继节点获取到锁的时候，会把自己设为首节点
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAyMC83LzE5LzE3MzY1NmFiNzQ5NDgwY2Y?x-oss-process=image/format,png)
```java
public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
}
```
- 线程会先尝试获取锁，失败则封装成Node，CAS加入同步队列的尾部。在加入同步队列的尾部时，会判断前驱节点是否是head结点，并尝试加锁(可能前驱节点刚好释放锁)，否则线程进入阻塞等待

**在AQS还存一个ConditionObject的内部类，它的使用机制和Object.wait、notify类似**
```java
//AbstractQueuedSynchronizer.java
public class ConditionObject implements Condition, java.io.Serializable {
    //条件队列;Node 复用了AQS中定义的Node
    private transient Node firstWaiter;
    private transient Node lastWaiter;
    ...
```
- 每个Condition对象内部包含一个Node元素的FIFO条件队列
- 当一个线程调用Condition.await()方法，那么该线程将会释放锁、构造Node加入条件队列并进入等待状态
```java
//类似Object.wait
public final void await() throws InterruptedException{
    ...
    Node node = addConditionWaiter(); //构造Node,加入条件队列
    int savedState = fullyRelease(node);
    int interruptMode = 0;
    while (!isOnSyncQueue(node)) {
        //挂起线程
        LockSupport.park(this);
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
    }
    //notify唤醒线程后，加入同步队列继续竞争锁
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
```
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAyMC83LzE5LzE3MzY1ODllOWFmYzVhNTQ?x-oss-process=image/format,png)
- 调用Condition.signal时，获取条件队列的首节点，将其移动到同步队列并且利用LockSupport唤醒节点中的线程。随后继续执行wait挂起前的状态，调用acquireQueued(node, savedState)竞争同步状态
```java
    //类似Object.notify
    private void doSignal(Node first) {
        do {
            if ( (firstWaiter = first.nextWaiter) == null)
                lastWaiter = null;
            first.nextWaiter = null;
        } while (!transferForSignal(first) &&
                 (first = firstWaiter) != null);
    }
```
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAyMC83LzE5LzE3MzY1OGYzZTRmMDQwNmQ?x-oss-process=image/format,png)
- **volatile+cas机制保证了代码的同步性和可见性，而AQS封装了线程阻塞等待挂起，解锁唤醒其他线程的逻辑**。AQS子类只需根据状态变量，判断是否可获取锁，是否释放锁成功即可
- 继承AQS需要选性重写以下几个接口
```java
protected boolean tryAcquire(int arg);//尝试独占性加锁
protected boolean tryRelease(int arg);//对应tryAcquire释放锁
protected int tryAcquireShared(int arg);//尝试共享性加锁
protected boolean tryReleaseShared(int arg);//对应tryAcquireShared释放锁
protected boolean isHeldExclusively();//该线程是否正在独占资源，只有用到condition才需要取实现它
```

# 11 ReentrantLock的原理
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAyMC83LzE5LzE3MzY1NTUwMGYwOTI1Yjc?x-oss-process=image/format,png)
- ReentrantLock实现了Lock接口，并使用内部类Sync(Sync继承AbstractQueuedSynchronizer)来实现同步操作
- ReentrantLock内部类Sync
```java
abstract static class Sync extends AbstractQueuedSynchronizer{
    .... 
    final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                //直接CAS状态加锁，非公平操作
                if (compareAndSetState(0, acquires)) { 
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
    ...
    //重写了tryRelease
    protected final boolean tryRelease(int releases) {
        c = state - releases; //改变同步状态
        ...
        //修改volatile 修饰的状态变量
        setState(c); 
        return free;
    }
}
```
- Sync的子类NonfairSync和FairSync都重写了tryAcquire方法
- 其中NonfairSync的tryAcquire调用父类的nonfairTryAcquire方法, FairSync则自己重写tryAcquire的逻辑。其中调用hasQueuedPredecessors()判断是否有排队Node，存在则返回false（false会导致当前线程排队等待锁）
```java
    static final class NonfairSync extends Sync {
        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
    }
    ....
    static final class FairSync extends Sync {
        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (!hasQueuedPredecessors() &&   
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
    ....    
```
# 12 AQS排他锁的实例demo
```java
public class TwinsLock implements Lock {

    private final Sync sync = new Sync(2);
    @Override
    public void lockInterruptibly() throws InterruptedException {  throw new RuntimeException(""); }
    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {throw new RuntimeException("");}
    @Override
    public Condition newCondition() {  return sync.newCondition(); }
    @Override
    public void lock() {  sync.acquireShared(1); }
    @Override
    public void unlock() {  sync.releaseShared(1); } }
    @Override
    public boolean tryLock() { return sync.tryAcquireShared(1) > -1;  }
}
```
再来看看Sync的代码
```java
class Sync extends AbstractQueuedSynchronizer {
        Sync(int count) {
            if (count <= 0) {
                throw new IllegalArgumentException("count must large than zero");
            }
            setState(count);
        }
        @Override
        public int tryAcquireShared(int reduceCount) {
            for (; ; ) {
                int current = getState(); 
                int newCount = current - reduceCount;
                if (newCount < 0 || compareAndSetState(current, newCount)) {
                    return newCount;
                }
            }
        }
        @Override
        public boolean tryReleaseShared(int returnCount) {
            for (; ; ) {
                int current = getState();
                int newCount = current + returnCount;
                if (compareAndSetState(current, newCount)) {
                    return true;
                }
            }
        }
        public Condition newCondition() {
            return new AbstractQueuedSynchronizer.ConditionObject();
        }
    }
```
# 13 使用锁，能防止线程死循环吗
- 答案是不一定的；对于单个资源来说是可以做的；但是多个资源会存在死锁的情况，例如线程A持有资源X，等待资源Y，而线程B持有资源Y，等待资源X
- 有了锁，可以对资源加状态控制，但是我们还需要防止死锁的产生，打破产生死锁的四个条件之一就行
- 1 资源不可重复被两个及以上的使用者占用
- 2 使用者持有资源并等待其他资源
- 3 资源不可被抢占
- 4 多个使用者形成等待对方资源的循环圈

# 14 ThreadLocal是否可保证资源的同步
- 当使用ThreadLocal声明变量时，ThreadLocal为每个使用该变量的线程提供独立的变量副本，每一个线程都可以独立地改变自己的副本，而不会影响其它线程所对应的副本
- 从上面的概念可知，ThreadLocal其实并不能保证变量的同步性，只是给每一个线程分配一个变量副本

# 关注公众号，大家一起交流
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAyMC83LzE5LzE3MzY3Yjk0ZGEwZTlmZDI?x-oss-process=image/format,png)
#  参考文章
- [objectMonitor.cpp](https://github.com/openjdk-mirror/jdk7u-hotspot/blob/50bdefc3afe944ca74c3093e7448d6b889cd20d1/src/share/vm/runtime/objectMonitor.cpp)
- [Moniter的实现原理](https://www.hollischuang.com/archives/2030)
- [JVM源码分析之Object.wait/notify实现](https://www.jianshu.com/p/f4454164c017)
- [Java对象头与锁](https://www.cnblogs.com/ZoHy/p/11313155.html)
- [LockSupport中park与unpark基本使用与原理](https://blog.csdn.net/e891377/article/details/104551335/)
- [Java并发之Condition](https://www.cnblogs.com/gemine/p/9039012.html )
