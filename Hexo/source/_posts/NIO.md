---
title: NIO
typora-root-url: ../
date: 2021-05-14 15:18:45
tags:
- NIO
categories:
- [Java语法,进阶]
---

了解Java NIO和传统BIO的区别以及使用

<!--more-->

# 概述

NIO主要有三大核心部分：Channel(通道)，Buffer(缓冲区), Selector。传统IO基于字节流和字符流进行操作，而NIO基于Channel和Buffer(缓冲区)进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。Selector(选择区)用于监听多个通道的事件。下图可以完整体现通道和缓冲区的关系：

![image-20210514155321106](/images/image-20210514155321106.png)

对于**传统IO**有两个层次InputStream、Reader分别处理字节流和字符流，每次从流中读一个或多个字节（字符），直至读取所有字节，它们没有被缓存在任何地方。此外，它不能前后移动流中的数据。如果需要前后移动从流中读取的数据，需要先将它缓存到一个缓冲区。最重要的是，传统IO的各种流是**阻塞**的。这意味着，当一个线程调用read() 或 write()时，该线程被阻塞，直到有一些数据被读取，或数据完全写入。

而**NIO**的缓冲导向方法略有不同。数据读取到一个它稍后处理的**缓冲区**，需要时可在缓冲区中前后移动。这就增加了处理过程中的灵活性。NIO存在**非阻塞模式**，在此模式下的读，当一个线程从某通道发送请求读取数据，但是它仅能得到目前可用的数据，如果目前没有数据可用时，就什么都不会获取。而不是保持线程阻塞，线程通常将非阻塞IO的空闲时间用于在其它通道上执行IO操作，一个单独的线程现在可以**管理多个输入和输出通道**

# 一、Channel和Buffer

## Channel

实际上Channel和传统IO之间的流基本是一个层级，区别在于NIO中对Channel操作时加了Buffer一层抽象，先对Buffer操作，再将操作从Buffer传递到Channel。此外，还有一点，Channel是双向的，对一个Channel即可读又可以写

Java NIO 中常见 Channel实现如下：

- FileChannel，对文件进行IO操作
- DatagramChannel，对UDP进行IO操作
- SocketChannel，对TCP进行IO操作
- ServerSocketChannel，对TCP进行IO操作

前者可以称为**本地IO**，其中后三者就是**网络IO**，可以转化为**非阻塞模式**

**1.channel生成**

对于FileChannel，Java 针对支持通道的类提供了 getChannel() 方法，也有静态open方法

```java
FileInputStream/FileOutputStream
RandomAccessFile

// 例子
FileChannel fileChannel = new FileInputStream("src/nio.txt").getChannel();
```

对于网络IO，可以用工厂静态open方法获取，再进行一些参数设置

```java
ServerSocketChannel listenerChannel = ServerSocketChannel.open();
		// 与本地端口绑定
		listenerChannel.socket().bind(new InetSocketAddress(ListenPort));
		// 设置为非阻塞模式
		listenerChannel.configureBlocking(false);
```

**2.channel操作**

channel和buffer,主要**通过read和write向channel读取或者向channel写**

```java
 public abstract int read(ByteBuffer dst) throws IOException;

    public abstract long read(ByteBuffer[] dsts, int offset, int length)
        throws IOException;

    public final long read(ByteBuffer[] dsts) throws IOException {
        return read(dsts, 0, dsts.length);
    }

    public abstract int write(ByteBuffer src) throws IOException;

    public abstract long write(ByteBuffer[] srcs, int offset, int length)
        throws IOException;

    public final long write(ByteBuffer[] srcs) throws IOException {
        return write(srcs, 0, srcs.length);
    }
```

channel之间数据传输，可以通过`transferFrom()、transferTo()`进行操作

## Buffer

Buffer顾名思义：缓冲区，实际上是一个容器，一个连续数组。Channel提供从文件、网络读取数据的渠道，但是读写的数据都必须经过Buffer

对buffer进行操作首先要分配空间，在进行具体操作，这里用到**allocate()**

然后就是通常意义的**读写操作**

向Buffer中读写数据：

- 从Channel写到Buffer (channel.read(buf))
- 通过Buffer的put()方法 （buf.put(…)）

从Buffer中读取数据：

- 从Buffer读取到Channel (channel.write(buf))

- 使用get()方法从Buffer中读取数据 （buf.get()）

  

前面说到Buffer操作相对于流操作有更多的**灵活性**，灵活性主要由下体现：

buffer过几个变量来保存这个数据的当前位置状态：capacity, position, limit, mark

- capacity，缓冲区数组的总长度

- position，下一个要操作的数据元素的位置

- limit，缓冲区数组中不可操作的下一个元素的位置：limit<=capacity

- mark，用于记录当前position的前一个位置或者默认是-1

针对这些变量可以和一些方法相对应：

ByteBuffer.allocate(len)方法创建了一个len个byte的数组的缓冲区position的位置为0，capacity和limit默认都是数组长度。

当buf读入5字节数据时，position =  5，其他值不变

这时我们想要读取buf的数据，需要调用**buf.flip()**方法，令limit = position = 5 ，position设回0

可以说**postion-limit要么是可以写的空间范围，要么是可以读的空间范围**

**buf.clear()**：position将被设回0，limit设置成capacity，相当于是buffer清空

**compact()**：compact()方法将所有未读的数据拷贝到Buffer起始处。然后将position设到最后一个未读元素正后面。limit属性依然像clear()方法一样，设置成capacity。现在Buffer准备好写数据了，但是不会覆盖未读的数据。

**mark()**：可以标记Buffer中的一个特定的position，之后可以通过调用**reset()**方法恢复到这个position。**rewind()**：将position设回0，limit保持不变，所以你可以重读Buffer中的所有数据。

实际上上诉方法是和buffer的读写相关的，用flip是为了读，是向buffer写到读的切换，用clear或者compact，说明接下来需要向buffer写了

```java
public class NIOLearn {
    public static void main(String[] args) throws IOException {
        try (FileChannel fileChannel = new FileInputStream("src/nio.txt").getChannel()) {
            ByteBuffer buffer = ByteBuffer.allocate(2);
            int isread = fileChannel.read(buffer);
            while(isread != -1){
                // 写读切换
                buffer.flip();
                while(buffer.hasRemaining())
                {
                    System.out.print((char)buffer.get());
                }
                // 读写切换
                buffer.clear();
                isread = fileChannel.read(buffer);
            }

        }
    }
}
```

# 二、Selector

Selector（选择器）是Java NIO中能够检测一到多个NIO通道，并能够知晓通道是否为诸如读写事件做好准备的组件。**一个Selector可以管理多个通道**

使用Selector的过程也与通道息息相关

## **1.Selector和通道建立联系**

**`selectableChannel.register()`**



为了将Channel和Selector配合使用，必须将Channel注册到Selector上，通过**selectableChannel.register()**方法来实现：

```java
// selector建立
Selector selector = Selector.open();

// 通道建立和绑定          
ssc= ServerSocketChannel.open();
ssc.socket().bind(new InetSocketAddress(PORT));
ssc.configureBlocking(false);

// 两者建立联系
ssc.register(selector, SelectionKey.OP_ACCEPT);
```

上诉有几个补充

1、`selectableChannel`要求该通道是非阻塞的，这意味着不能将FileChannel与Selector一起使用，因为FileChannel不能切换到非阻塞模式。而套接字通道都可以。

2、`register()`方法的第二个参数。这是一个“interest集合”，意思是在通过Selector监听Channel时对什么事件感兴趣。可以监听四种不同类型的事件：

```
1. Connect
2. Accept
3. Read
4. Write
```

通道触发了一个事件意思是该事件已经就绪。所以，某个channel成功连接到另一个服务器称为“连接就绪”。一个server socket channel准备好接收新进入的连接称为“接收就绪”。一个有数据可读的通道可以说是“读就绪”。等待写数据的通道可以说是“写就绪”。

这四种事件用`SelectionKey`的四个常量来表示：

```
1. SelectionKey.OP_CONNECT
2. SelectionKey.OP_ACCEPT
3. SelectionKey.OP_READ
4. SelectionKey.OP_WRITE
```

## **2.等待通道就绪**

**`selctor.select()`**



当注册通道后，我们需要调用注册器几个重载的select()方法。这些方法返回你所感兴趣的事件已经准备就绪的那些通道。

下面是select()方法：

- int select()，**阻塞**到至少有一个通道在你注册的事件上就绪了。
- int select(long timeout)，select()一样，但是最长只阻塞timeout毫秒
- int selectNow()，不会阻塞

而select方法的返回值是一个int，表示有多少通道已经就绪。显然为0代表没有通道准备就绪



## **3.对就绪的通道进行操作**

**`SelectionKey`**



Java NIO 中对就绪的通道进行操作主要涉及到SelectionKey对象的操作

实际上在前面当向Selector注册Channel时，register()方法会返回一个SelectionKey对象

此外调用`Set selectedKeys = selector.selectedKeys();`也可以返回已选择键集，通过这个已选择键集对已就绪的通道进行操作，因为一个SelectionKey对象可以包含以下很多信息：

- Channel

- Selector

- 附加的对象（可选），比如通道对应的buffer

- interest集合

- ready集合

  

下列是具体操作：

- 获取已就绪的通道：`Channel  channel  = selectionKey.channel();`。
- 判断通道什么事件就绪：

```java
selectionKey.isAcceptable();
selectionKey.isConnectable();
selectionKey.isReadable();
selectionKey.isWritable();
```

- 向SelectionKey附着更多信息：`selectionKey.attach(obj);`

- 获取附着信息：`Object attachedObj = selectionKey.attachment();`

### 

下列是一个对于就绪通道操作的伪代码

```java
while(true){
                if(selector.select(TIMEOUT) == 0){
                    continue;
                }
                Iterator<SelectionKey> iter = selector.selectedKeys().iterator();
                while(iter.hasNext()){
                    SelectionKey key = iter.next();
                    if(key.isAcceptable()){
                        handleAccept(key);
                    }
                    if(key.isReadable()){
                        handleRead(key);
                    }
                    if(key.isWritable() && key.isValid()){
                        handleWrite(key);
                    }
                    if(key.isConnectable()){
                        System.out.println("isConnectable = true");
                    }
                    iter.remove();
                }
```

注意每次迭代末尾的keyIterator.remove()调用。Selector不会自己从已选择键集中移除SelectionKey实例。必须在处理完通道时自己移除。下次该通道变成就绪时，Selector会再次将其放入已选择键集中



# 三、简单demo

利用上诉知识，写一个小demo

```java
package io;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.*;
import java.util.Iterator;

public class SimpleNIOServer {
    private static final int BUFFER_SIZE = 1024;
    private static final int PORT = 8888;
    private static final int TIMEOUT = 1000;

    public static void handleAccept(SelectionKey key) throws IOException {
        ServerSocketChannel socketChannel = (ServerSocketChannel) key.channel();
        SocketChannel accept = socketChannel.accept();
        accept.configureBlocking(false);
        accept.register(key.selector(),SelectionKey.OP_READ, ByteBuffer.allocate(BUFFER_SIZE));
        System.out.println("连接完毕");
    }

    public static void handleRead(SelectionKey key) throws IOException {
        SocketChannel channel = (SocketChannel) key.channel();
        ByteBuffer buffer = (ByteBuffer) key.attachment();
        int read = channel.read(buffer);
        while(read != -1){
            buffer.flip();
            while(buffer.hasRemaining()){
                System.out.print((char)buffer.get());
        }
            buffer.clear();
            read  = channel.read(buffer);
        }
        System.out.println("读取完毕");
        channel.close();

    }
    public static void handleWrite(SelectionKey key) throws IOException {
        SocketChannel channel = (SocketChannel) key.channel();
        ByteBuffer buffer = (ByteBuffer) key.attachment();
        buffer.flip();
        while(buffer.hasRemaining())
            channel.write(buffer);
        buffer.compact();
        System.out.println("写完毕");
    }


    public static void server() throws IOException {
        Selector selector = Selector.open();
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        serverSocketChannel.socket().bind(new InetSocketAddress(PORT));
        serverSocketChannel.configureBlocking(false);
        serverSocketChannel.register(selector,SelectionKey.OP_ACCEPT);
        while (true){
            if(selector.select(TIMEOUT) == 0)
                continue;
            Iterator<SelectionKey> iterable = selector.selectedKeys().iterator();
            while(iterable.hasNext()){
                SelectionKey key= iterable.next();
                if(key.isAcceptable())
                    handleAccept(key);
                else if(key.isReadable())
                    handleRead(key);
                else if (key.isWritable())
                    handleWrite(key);
                else if(key.isConnectable())
                    System.out.println("连接中");
                iterable.remove();
            }

        }



    }
    public static void main(String[] args) throws IOException {
        server();
    }

}
```

