# 前言
数据序列化存储，或者数据通过网络传输时，会遇到不可避免将数据转成字节数组的场景。字节数组的读写不会太难，但又有点繁琐，为了避免重复造轮子，jdk推出了ByteBuffer来帮助我们操作字节数组；而netty是一款当前流行的java网络IO框架，它内部定义了一个ByteBuf来管理字节数组，和ByteBuffer大同小异

- ByteBuffer
- 零拷贝之MappedByteBuffer
- DirectByteBuffer堆外内存回收机制
- netty之ByteBuf

**关注公众号，一起交流，微信搜一搜: 潜行前行**
---
## Buffer结构
```java
public abstract class Buffer {
	//关系: mark <= position <= limit <= capacity
    private int mark = -1;
    private int position = 0;
    private int limit;
    private int capacity;
    long address;　// Used only by direct buffers，直接内存的地址
```
-	mark：调用mark()方法的话，mark值将存储当前position的值，等下次调用reset()方法时，会设定position的值为之前的标记值
-	position：是下一个要被读写的byte元素的下标索引
-	limit：是缓冲区中第一个不能读写的元素的数组下标索引，也可以认为是缓冲区中实际元素的数量
-	capacity：是缓冲区能够容纳元素的最大数量，这个值在缓冲区创建时被设定，而且不能够改变
## Buffer.API
```java
Buffer(int mark, int pos, int lim, int cap)
//Buffer创建时设置的最大数组容量值
public final int capacity()
//当前指针的位置
public final int position() 
//限制可读写大小
public final Buffer limit(int newLimit)
//标记当前position的位置
public final Buffer mark()
//配合mark使用，position成之前mark()标志的位置。先前没调用mark则报错
public final Buffer reset()
//写->读模式翻转，单向的
//position变成了初值位置0，而limit变成了写模式下position位置
public final Buffer flip()
//重置position指针位置为0，mark为-1；相对flip方法是limit不变
public final Buffer rewind() //复位
//和rewind一样，多出一步是limit会被设置成capacity
public final Buffer clear() 
//返回剩余未读字节数
public final int remaining()
```
## ByteBuffer结构
```java
public abstract class ByteBuffer extends Buffer 
			implements Comparable<ByteBuffer>{
    final byte[] hb;  //仅限堆内内存使用
    final int offset;
    boolean isReadOnly; 
```
## ByteBuffer.API
```java
//申请堆外内存
public static ByteBuffer allocateDirect(int capacity)
//申请堆内内存
public static ByteBuffer allocate(int capacity) 
//原始字节包装成ByteBuffer
public static ByteBuffer wrap(byte[] array, int offset, int length)
//原始字节包装成ByteBuffer
public static ByteBuffer wrap(byte[] array)
//创建共享此缓冲区内容的新字节缓冲区
public abstract ByteBuffer duplicate()
//分片，创建一个新的字节缓冲区
//新ByteBuffer的开始位置是此缓冲区的当前位置position
public abstract ByteBuffer slice()
//获取字节内容
public abstract byte get()
//从ByteBuffer偏移offset的位置，获取length长的字节数组，然后返回当前ByteBuffer对象
public ByteBuffer get(byte[] dst, int offset, int length)
//设置byte内存
public abstract ByteBuffer put(byte b);
//以offset为起始位置设置length长src的内容，并返回当前ByteBuffer对象
public ByteBuffer put(byte[] src, int offset, int length长)
//将没有读完的数据移到到缓冲区的初始位置，position设置为最后一没读字节数据的下个索引，limit重置为为capacity
//读->写模式，相当于flip的反向操作
public abstract ByteBuffer compact()
//是否是直接内存
public abstract boolean isDirect()
```

- ByteBuffer bf = ByteBuffer.allocate(10);`，创建大小为10的ByteBuffer对象
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2b4b6bb2fc5d4458aae0a3a0d1355458~tplv-k3u1fbpfcp-watermark.image)
- 写入数据
```java
    ByteBuffer buf ByteBuffer.allocate(10);
    buf.put("csc".getBytes());
```
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b927a62f9bf04e03b5ff15b7ea4dc6dd~tplv-k3u1fbpfcp-watermark.image)
- 调用flip转换缓冲区为读模式; `buf.flip();`
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/84e0bebef2e04f96855ae3395bc3b29d~tplv-k3u1fbpfcp-watermark.image)
- 读取缓冲区中到内容：get(); `System.out.println((char) buf.get());`
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b5fc80dff734463d8aaa8c442e3db03b~tplv-k3u1fbpfcp-watermark.image)
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/58c2908aa693479393857b82462fc946~tplv-k3u1fbpfcp-watermark.image)

## 零拷贝之MappedByteBuffer
-	共享内存映射文件，对应的ByteBuffer子操作类，MappedByteBuffer是基于mmap实现的。关于零拷贝的mmap的底层原理可以看看:[框架篇：小白也能秒懂的Linux零拷贝原理](https://juejin.cn/post/6887469050515947528)。MappedByteBuffer需要FileChannel调用本地map函数映射。C++代码可以查阅下[FileChannelImpl.c-Java_sun_nio_ch_FileChannelImpl_map0方法](https://github.com/unofficial-openjdk/openjdk/blob/jdk8u/jdk8u/jdk/src/solaris/native/sun/nio/ch/FileChannelImpl.c)
-	使用MappedByteBuffer和文件映射，其读写可以减少内存拷贝次数
```java
FileChannel readChannel = FileChannel.open(Paths.get("./cscw.txt"), StandardOpenOption.READ);
MappedByteBuffer data = readChannel.map(FileChannel.MapMode.READ_ONLY, 0, 1024 * 1024 * 40);
```

## DirectByteBuffer堆外内存回收机制Cleaner
- 下面我们看看直接内存的回收机制（java8）；DirectByteBuffer内部存在一个Cleaner对象，并且委托内部类Deallocator对象进行内存回收
```java
class DirectByteBuffer extends MappedByteBuffer implements DirectBuffer
{
	//构造函数
    DirectByteBuffer(int cap) { 
		....　//内存分配
        cleaner = Cleaner.create(this, new Deallocator(base, size, cap));
    	...
    }    
    private static class Deallocator implements Runnable{
    	...
    	public void run() {
            if (address == 0) {
                // Paranoia
                return;
            }
            unsafe.freeMemory(address);　//回收内存
            address = 0;
            Bits.unreserveMemory(size, capacity);
        }
}
```
- 细看下Cleaner，继承于PhantomReference，并且在`public void clean()`方法会调用Deallocator进行清除操作
```java
public class Cleaner extends PhantomReference<Object> {
    //如果DirectByteBuffer对象被回收，相应的Cleaner会被放入dummyQueue队列
    private static final ReferenceQueue<Object> dummyQueue = new ReferenceQueue();
    //构造函数
    public static Cleaner create(Object var0, Runnable var1) {
        return var1 == null ? null : add(new Cleaner(var0, var1));
    }
    private Cleaner(Object var1, Runnable var2) {
        super(var1, dummyQueue);
        this.thunk = var2;
    }
    private final Runnable thunk;
    public void clean() {
            if (remove(this)) {
                try {
                    this.thunk.run();
                } catch (final Throwable var2) {
                ....
```
- 在Reference内部存在一个守护线程，循环获取Reference，并判断是否Cleaner对象，如果是则调用其clean方法
```java
public abstract class Reference<T> 
    static {
        ThreadGroup tg = Thread.currentThread().getThreadGroup();
        for (ThreadGroup tgn = tg; tgn != null; g = tgn, tgn = tg.getParent());
        Thread handler = new ReferenceHandler(tg, "Reference Handler");
        ...
        handler.setDaemon(true);
        handler.start();
        ...
    }
	...
    //内部类调用 tryHandlePending
    private static class ReferenceHandler extends Thread {
        public void run() {
                    while (true) {
                        tryHandlePending(true);
                    }
                }
  	... 
    static boolean tryHandlePending(boolean waitForNotify) {
        Cleaner c;
        .... //从链表获取对象被回收的引用
        // 判断Reference是否Cleaner，如果是则调用其clean方法
        if (c != null) {
            c.clean(); //调用Cleaner的clean方法
            return true;
        }
        ReferenceQueue<? super Object> q = r.queue;
        if (q != ReferenceQueue.NULL) q.enqueue(r);
        return true;
```

## netty之ByteBuf
- ByteBuf原理
- Bytebuf通过两个位置指针来协助缓冲区的读写操作，分别是readIndex和writerIndex
```java
 *      +-------------------+------------------+------------------+
 *      | discardable bytes |  readable bytes  |  writable bytes  |
 *      |                   |     (CONTENT)    |                  |
 *      +-------------------+------------------+------------------+
 *      |                   |                  |                  |
 *      0 <= readerIndex　<=　writerIndex <= capacity
```
-	ByteBuf.API
```java
//获取ByteBuf分配器
public abstract ByteBufAllocator alloc()
//丢弃可读字节
public abstract ByteBuf discardReadBytes()
//返回读指针
public abstract int readerIndex()
//设置读指针
public abstract ByteBuf readerIndex(int readerIndex);
//标志当前读指针位置，配合resetReaderIndex使用
public abstract ByteBuf markReaderIndex()
public abstract ByteBuf resetReaderIndex()
//返回可读字节数
public abstract int readableBytes()
//返回写指针
public abstract int writerIndex()
//设置写指针
public abstract ByteBuf writerIndex(int writerIndex);
//标志当前写指针位置，配合resetWriterIndex使用
public abstract ByteBuf markWriterIndex()
public abstract ByteBuf resetWriterIndex()
//返回可写字节数
public abstract int writableBytes()
public abstract ByteBuf clear();
//设置读写指针
public abstract ByteBuf setIndex(int readerIndex, int writerIndex)
//指针跳过length
public abstract ByteBuf skipBytes(int length)
//以当前位置切分ByteBuf　todo
public abstract ByteBuf slice();
//切割起始位置为index，长度为length的ByteBuf todo
public abstract ByteBuf slice(int index, int length);
//Returns a copy of this buffer's readable bytes. //复制ByteBuf todo
public abstract ByteBuf copy()
//是否可读
public abstract boolean isReadable()
//是否可写
public abstract boolean isWritable()
//字节编码顺序
public abstract ByteOrder order()
//是否在直接内存申请的ByteBuf
public abstract boolean isDirect()
//转为jdk.NIO的ByteBuffer类
public abstract ByteBuffer nioBuffer()
```
-	使用示例
```java
public static void main(String[] args) {
    //分配大小为10的内存
    ByteBuf buf = Unpooled.buffer(10);
    //写入
    buf.writeBytes("csc".getBytes());
    //读取
    byte[] b =  new byte[3];
    buf.readBytes(b);
    System.out.println(new String(b));
    System.out.println(buf.writerIndex());
    System.out.println(buf.readerIndex());
}
－－－－result－－－－
csc
3
3
```
- ByteBuf初始化时，readIndex和writerIndex等于0，调用`writeXXX()方法`写入数据，writerIndex会增加(setXXX方法无作用)；调用`readXXX()方法`读取数据，则会使readIndex增加(getXXX方法无作用)，但不会超过writerIndex
- 在读取数据之后，0-readIndex之间的byte数据被视为discard，调用discardReadBytes()，释放这部分空间，作用类似于ByteBuffer的compact方法

---
#  参考文章
-	[java.nio.ByteBuffer用法小结](https://blog.csdn.net/mrliuzhao/article/details/89453082)
-	[Netty系列-一分钟了解ByteBuffer和ByteBuf结构](https://www.jianshu.com/p/3930150bf7f0)
-	[Netty之有效规避内存泄漏](https://www.jianshu.com/p/cec977b28079?from=timeline)
