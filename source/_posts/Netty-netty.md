---
title: Netty
date: 2022-11-20 22:11:32.74
updated: 2022-12-23 22:07:26.892
url: /archives/netty
categories: 
tags: 
---

## NIO

### 三大组件

#### 1. Channel & Buffer

channel是读写数据的**双向通道**，连接要读取的文件和内存，可以从channel把数据读入buffer，也可以将buffer的数据写入channel，而stream是单向的，channel比stream更加底层。

简而言之，通道负责传输，缓冲区负责存储

常见的channel有：
- FileChannel
- DatagramChannel
- SocketChannel
- ServerSocketChannel

常见的buffer有：
- ByteBuffer
	- MappedByteBuffer
	- DirectByteBuffer
	- HeapByteBuffer
- ShortBuffer
- IntBuffer(Long/Float/Double/Char)

#### 2. Selector
selector的作用就是配合一个线程来管理多个channel，获取这些channel上发生的事件。这些channel工作在非阻塞模式下，不会让线程吊死在一个channel上。适合连接数多，但流量低的场景。

![selector](/upload/2022/11/selector.png)

调用selector的select()会阻塞直到channel发生了读写就绪事件

### ByteBuffer
#### 使用方式
1. 向buffer写入数据
2. 调用flip()切换读模式
	-  flip会使得buffer中的limit变为position，position变为0
3. 从buffer读取数据
	-  调用clear()方法时position=0，limit变为capacity
	-  调用compact()方法时，会将缓冲区中的未读数据压缩到缓冲区前面
4. 调用clear()或compact()切换至写模式

#### 属性与方法
- capacity：缓冲区的容量。通过构造函数赋予，一旦设置，无法更改
- limit：缓冲区的界限。位于limit 后的数据不可读写。缓冲区的限制不能为负，并且不能大于其容量
- position：下一个读写位置的索引（类似PC）。缓冲区的位置不能为负，并且不能大于limit
- mark：记录当前position的值。position被改变后，可以通过调用reset() 方法恢复到mark的位置

### 粘包与半包
网络上有多条数据发送给服务端，数据之间使用 \n 进行分隔，但由于某种原因这些数据在接收时被进行了重新组合。

发送方在发送数据时，并不是一条一条地发送数据，而是将数据整合在一起，当数据达到一定的数量后再一起发送。这就会导致多条信息被放在一个缓冲区中被一起发送出去，出现粘包现象。

接收方的**缓冲区的大小是有限**的，当接收方的缓冲区满了以后，就需要将信息截断，等缓冲区空了以后再继续放入数据。这就会发生一段完整的数据最后被截断的现象，也就是半包现象。



**根本原因**：TCP是流式协议，消息无边界

**解决方法**：通过get(index)方法遍历ByteBuffer，遇到分隔符时进行处理。

### 网络编程
#### 阻塞
- 阻塞模式下，相关方法都会导致线程暂停
	- ServerSocketChannel.accept 会在没有连接建立时让线程暂停
	- SocketChannel.read 会在通道中没有数据可读时让线程暂停
	- 阻塞的表现其实就是线程暂停了，暂停期间不会占用 cpu，但线程相当于闲置
- 单线程下，阻塞方法之间相互影响，几乎不能正常工作，需要多线程支持
- 但多线程下，有新的问题：
	-  一个线程 1024k，如果连接数过多，必然导致 OOM，并且线程太多，反而会因为频繁上下文切换导致性能降低
	-  可以采用线程池技术来减少线程数和线程上下文切换，但治标不治本，如果有很多连接建立，但长时间 inactive，会阻塞线程池中所有线程，因此不适合长连接，只适合短连接

#### 非阻塞
- 非阻塞模式下，相关方法都不会让线程暂停
	- 在ServerSocketChannel.accept没有连接建立时，会返回null，继续运行
	- SocketChannel.read在没有数据可读时，会返回0，但线程不阻塞，可以去执行其他方法
	- 写数据时，线程只是等待数据写入Channel即可，无需等待Channel把数据发送出去
- 但是，即使没有连接建立，线程仍然不断轮询，造成cpu空转
- 数据复制过程中，线程实际还是阻塞的

#### 多路复用
单线程可以配合 Selector 完成对多个 Channel 可读写事件的监控，这称之为多路复用

- 多路复用仅针对网络 IO，普通文件 IO 无法利用多路复用
- 如果不用 Selector 的非阻塞模式，线程大部分时间都在做无用功，而 Selector 能够保证
	- 有可连接事件时才去连接
	- 有可读事件才去读取
	- 有可写事件才去写入，限于网络传输能力，Channel 未必时时可写，一旦 Channel 可写，会触发 Selector 的可写事件

### NIO与BIO区别
#### Stream与Channel
- stream 不会自动缓冲数据，channel 会利用系统提供的发送缓冲区、接收缓冲区（更为底层）
- stream 仅支持阻塞 API，channel 同时支持阻塞、非阻塞 API，网络 channel 可配合 selector 实现多路复用
- 二者均为全双工，即读写可以同时进行，虽然Stream是单向流动的，但是它也是全双工的

#### IO模型
1. 阻塞IO：用户线程进行read操作时，需要等待操作系统执行实际的read操作，此期间用户线程是被阻塞的，无法执行其他操作
2. 非阻塞IO：用户线程在一个循环中（浪费cpu资源）一直调用read方法，若内核空间中还没有数据可读，立即返回
3. 多路复用：当没有事件时，调用select方法会被阻塞住，一旦有一个或多个事件发生后，就会处理对应的事件，从而实现多路复用
4. 异步IO：线程1调用方法后立即返回，不会被阻塞也不需要立即获取结果，当方法的运行结果出来以后，由线程2将结果返回给线程1

## Netty
Netty 是一个异步的、基于事件驱动的网络应用框架，用于快速开发可维护、高性能的网络服务器和客户端

### 组件
#### EventLoop
事件循环对象，EventLoop 本质是一个单线程执行器（同时维护了一个 Selector），里面有 run 方法处理 Channel 上源源不断的 io 事件。

它的继承关系：
* 一条线是继承自 j.u.c.ScheduledExecutorService 因此包含了线程池中所有的方法
* 另一条线是继承自 netty 自己的 OrderedEventExecutor，
  * 提供了 boolean inEventLoop(Thread thread) 方法判断一个线程是否属于此 EventLoop
  * 提供了 parent 方法来看看自己属于哪个 EventLoopGroup



事件循环组，EventLoopGroup 是一组 EventLoop，Channel 一般会调用 EventLoopGroup 的 register 方法来绑定其中一个 EventLoop，后续这个 Channel 上的 io 事件都由此 EventLoop 来处理（保证了 io 事件处理时的线程安全）

* 继承自 netty 自己的 EventExecutorGroup
  * 实现了 Iterable 接口提供遍历 EventLoop 的能力
  * 另有 next 方法获取集合中下一个 EventLoop

#### Channel
channel 的主要作用：
* close() 可以用来关闭 channel
* closeFuture() 用来处理 channel 的关闭
  * sync 方法作用是同步等待 channel 关闭
  * 而 addListener 方法是异步等待 channel 关闭
* pipeline() 方法添加处理器
* write() 方法将数据写入
* writeAndFlush() 方法将数据写入并刷出

#### Future & Promise
在异步处理时，经常用到这两个接口，netty 的 Future 继承自 jdk 的 Future，而 Promise 又对 netty Future 进行了扩展

* jdk Future 只能同步等待任务结束（或成功、或失败）才能得到结果
* netty Future 可以同步等待任务结束得到结果，也可以异步方式得到结果，但都是要等任务结束
* netty Promise 不仅有 netty Future 的功能，而且脱离了任务独立存在，只作为两个线程间传递结果的容器

| 功能/名称    | jdk Future                     | netty Future                                                 | Promise      |
| ------------ | ------------------------------ | ------------------------------------------------------------ | ------------ |
| cancel       | 取消任务                       | -                                                            | -            |
| isCanceled   | 任务是否取消                   | -                                                            | -            |
| isDone       | 任务是否完成，不能区分成功失败 | -                                                            | -            |
| get          | 获取任务结果，阻塞等待         | -                                                            | -            |
| getNow       | -                              | 获取任务结果，非阻塞，还未产生结果时返回 null                | -            |
| await        | -                              | 等待任务结束，如果任务失败，不会抛异常，而是通过 isSuccess 判断 | -            |
| sync         | -                              | 等待任务结束，如果任务失败，抛出异常                         | -            |
| isSuccess    | -                              | 判断任务是否成功                                             | -            |
| cause        | -                              | 获取失败信息，非阻塞，如果没有失败，返回null                 | -            |
| addLinstener | -                              | 添加回调，异步接收结果                                       | -            |
| setSuccess   | -                              | -                                                            | 设置成功结果 |
| setFailure   | -                              | -                                                            | 设置失败结果 |


#### Handler & Pipeline
ChannelHandler 用来处理 Channel 上的各种事件，分为入站、出站两种。所有 ChannelHandler 被连成一串，就是 Pipeline

* 入站处理器通常是 ChannelInboundHandlerAdapter 的子类，主要用来读取客户端数据，写回结果
* 出站处理器通常是 ChannelOutboundHandlerAdapter 的子类，主要对写回结果进行加工

#### ByteBuf
是对字节数据的封装，可以创建池化基于堆的 ByteBuf

```java
ByteBuf buffer = ByteBufAllocator.DEFAULT.heapBuffer(10);
```

也可以创建池化基于直接内存的 ByteBuf

```java
ByteBuf buffer = ByteBufAllocator.DEFAULT.directBuffer(10);
```

* 直接内存创建和销毁的代价昂贵，但读写性能高（少一次内存复制），适合配合池化功能一起用
* 直接内存对 GC 压力小，这部分内存不受 JVM 垃圾回收的管理，注意主动释放

ByteBuf 由四部分组成

![ByteBuf ](/upload/2022/12/0010.png)

方法列表：

| 方法签名                                                     | 含义                   | 备注                                        |
| ------------------------------------------------------------ | ---------------------- | ------------------------------------------- |
| writeBoolean(boolean value)                                  | 写入 boolean 值        | 用一字节 01\|00 代表 true\|false            |
| writeByte(int value)                                         | 写入 byte 值           |                                             |
| writeShort(int value)                                        | 写入 short 值          |                                             |
| writeInt(int value)                                          | 写入 int 值            | Big Endian，即 0x250，写入后 00 00 02 50    |
| writeIntLE(int value)                                        | 写入 int 值            | Little Endian，即 0x250，写入后 50 02 00 00 |
| writeLong(long value)                                        | 写入 long 值           |                                             |
| writeChar(int value)                                         | 写入 char 值           |                                             |
| writeFloat(float value)                                      | 写入 float 值          |                                             |
| writeDouble(double value)                                    | 写入 double 值         |                                             |
| writeBytes(ByteBuf src)                                      | 写入 netty 的 ByteBuf  |                                             |
| writeBytes(byte[] src)                                       | 写入 byte[]            |                                             |
| writeBytes(ByteBuffer src)                                   | 写入 nio 的 ByteBuffer |                                             |
| int writeCharSequence(CharSequence sequence, Charset charset) | 写入字符串             |                                             |

容量不够（初始容量是 10）时会引发扩容：
* 如果写入后数据大小未超过 512，则选择下一个 16 的整数倍，例如写入后大小为 12 ，则扩容后 capacity 是 16
* 如果写入后数据大小超过 512，则选择下一个 2^n，例如写入后大小为 513，则扩容后 capacity 是 2^10=1024（2^9=512 已经不够了）
* 扩容不能超过 max capacity 会报错

读过的内容，就属于废弃部分了，再读只能读那些尚未读取的部分，可以使用标记重复读取。

由于 Netty 中有堆外内存的 ByteBuf 实现，堆外内存最好是手动来释放，而不是等 GC 垃圾回收。
* UnpooledHeapByteBuf 使用的是 JVM 内存，只需等 GC 回收内存即可
* UnpooledDirectByteBuf 使用的就是直接内存了，需要特殊的方法来回收内存
* PooledByteBuf 和它的子类使用了池化机制，需要更复杂的规则来回收内存


Netty 采用了引用计数法来控制回收内存，每个 ByteBuf 都实现了 ReferenceCounted 接口，基本规则是，**最后使用者负责 release**
* 每个 ByteBuf 对象的初始计数为 1
* 调用 release 方法计数减 1，如果计数为 0，ByteBuf 内存被回收
* 调用 retain 方法计数加 1，表示调用者没用完之前，其它 handler 即使调用了 release 也不会造成回收
* 当计数为 0 时，底层内存会被回收，这时即使 ByteBuf 对象还在，其各个方法均无法正常使用

### 粘包与半包

粘包
* 现象，发送 abc def，接收 abcdef
* 原因
  * 应用层：接收方 ByteBuf 设置太大（Netty 默认 1024）
  * 滑动窗口：假设发送方 256 bytes 表示一个完整报文，但由于接收方处理不及时且窗口大小足够大，这 256 bytes 字节就会缓冲在接收方的滑动窗口中，当滑动窗口中缓冲了多个报文就会粘包
  * Nagle 算法：会造成粘包

半包
* 现象，发送 abcdef，接收 abc def
* 原因
  * 应用层：接收方 ByteBuf 小于实际发送数据量
  * 滑动窗口：假设接收方的窗口只剩了 128 bytes，发送方的报文大小是 256 bytes，这时放不下了，只能先发送前 128 bytes，等待 ack 后才能发送剩余部分，这就造成了半包
  * MSS 限制：当发送的数据超过 MSS 限制后，会将数据切分发送，就会造成半包

解决方案
1. 短链接，发一个包建立一次连接，这样连接建立到连接断开之间就是消息的边界，缺点效率太低
2. 每一条消息采用固定长度，缺点浪费空间
3. 每一条消息采用分隔符，例如 \n，缺点需要转义
4. 每一条消息分为 head 和 body，head 中包含 body 的长度