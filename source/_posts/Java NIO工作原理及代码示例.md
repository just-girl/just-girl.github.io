---
title: Java NIO工作原理及代码示例
tags: ["Java"]
categories: Java
date: 2019-03-14
grammar_cjkRuby: true
copyright: ture
---

简介：本文主要介绍了JAVA NIO中的Buffer, Channel, Selector的工作原理以及使用它们的若干注意事项，最后是利用它们实现服务器和客户端通信的代码实例。

<!--more-->

## ByteBuffer

### 直接缓冲区和非直接缓冲区

下面是创建`ByteBuffer`对象的几种方式

```java
static ByteBuffer allocate(int capacity)
static ByteBuffer allocateDirect(int capacity)
static ByteBuffer wrap(byte[] array)
static ByteBuffer wrap(byte[] array, int offset, int length)    
```

allocate方式创建的ByteBuffer对象我们称之为非直接缓冲区，这个ByteBuffer对象(和对象包含的缓冲数组)都位于JVM的堆区。wrap方式和allocate方式创建的ByteBuffer没有本质区别，都创建的是非直接缓冲区。

allocateDirect方法创建的ByteBuffer我们称之为直接缓冲区，此时ByteBuffer对象本身在堆区，而缓冲数组位于非堆区， ByteBuffer对象内部存储了这个非堆缓冲数组的地址。在非堆区的缓冲数组可以通过JNI（内部还是系统调用）方式进行IO操作，JNI不受gc影响，机器码执行速度也比较快，同时还避免了JVM堆区与操作系统内核缓冲区的数据拷贝，所以IO速度比非直接缓冲区快。然而allocateDirect方式创建ByteBuffer对象花费的时间和回收该对象花费的时间比较多，所以这个方法适用于创建那些需要重复使用的缓冲区对象。

### 重要属性和方法

ByteBuffer对象三个重要属性 position, limit和capacity。其中capacity表示了缓冲区的总容量，始终保持不变，初始时候position 等于 0 , limit 等于 capacity

#### put：向缓冲区放入数据

```java
abstract ByteBuffer put(byte b)
ByteBuffer put(byte[] src)
ByteBuffer put(byte[] src, int offset, int length)
```

调用put方法前，limit应该等于capacity，如果不等于，几乎可以肯定我们对缓冲区的操作有误。在put方法中0到position-1的区域表示有效数据,position到limit之间区域表示空闲区域。put方法会从position的当前位置放入数据，每放入一个数据position增加1，当position等于limit（即空闲区域使用完）时还继续放入数据就会抛出BufferUnderflowException异常

#### get：从缓冲区读取数据

```java
abstract byte get()
ByteBuffer get(byte[] dst)
ByteBuffer get(byte[] dst, int offset, int length)
```

在get方法中， 0到position-1的区域表示已读数据,position到limit之间的区域表示未读取的数据。每读取一个数据position增加1，当position等于limit时继续读取数据就会抛出BufferUnderflowException异常。

#### flip ：将写模式转换成读模式

```java
public final Buffer flip() {
    limit = position;
    position = 0;
    mark = -1;
    return this;
}
```

#### clear：清空缓冲区，将读模式转换写模式

```java
public final Buffer clear() {
    position = 0;
    limit = capacity;
    mark = -1;
    return this;
}
```

#### compact：保留未读取的数据，将读模式转换写模式

```java
public ByteBuffer compact() {
 
    int pos = position();
    int lim = limit();
    assert (pos <= lim);
    int rem = (pos <= lim ? lim - pos : 0);
 
    unsafe.copyMemory(ix(pos), ix(0), (long)rem << 0);
    position(rem);
    limit(capacity());
    discardMark();
    return this;
 
}
```

#### mark：保存当前position的位置到mark变量

```java
public final Buffer mark() {
    mark = position;
    return this;
}
```

#### rest：将position置为mark变量中的值

```java
public final Buffer reset() {
    int m = mark;
    if (m < 0)
        throw new InvalidMarkException();
    position = m;
    return this;
}
```

mark方法和rest方法联合使用可实现从指定位置的重读。

#### rewind：从头开始重读

```java
public final Buffer rewind() {
    position = 0;
    mark = -1;
    return this;
}
```

ByteBuffer对象使用时又很多需要注意的地方，自认为这个API设计的不是很友好。比如一定不能连续两次调用flip和compact方法,flip方法调用以后不能再调用put方法，等等。要避免这些错误，只能在使用ByteBuffer前弄清楚当前缓冲区中0到position-1以及position到limit中数据表示的含义，这才是避免bug的根本办法。

从上面的介绍中我们可以看出，ByteBuffer对象既可以读，也可以写。除非我们能保证在读操作一次性使用完ByteBuffer对象中的所有数据，并且保证写入ByteBuffer对象向中的内容全部写入完成，否则同时用于读写的ByteBuffer对象会造成数据的混乱和错误。**一般来说，我们都会创建两个ByteBuffer对象向，一个用于接收数据，另一个用于发送数据。**

#### 其它方法

ByteBuffer是面向字节的，为方便基本数据类型的读取，ByteBuffer中还提供getInt，putInt，getFloat，putFloat等方法，这些方法方便我们在缓冲区存取单个基本数据类型。如果需要从基本数据类型数组中写入到ByteBuffer中，或者从ByteBuffer中读取到基本数据类型的数组中，那么我们可以通过已创建好的ByteBuffer对象的asXxxBuffer方法创建基本数据类型的Buffer。

```java
abstract CharBuffer asCharBuffer()
abstract DoubleBuffer asDoubleBuffer()
abstract FloatBuffer asFloatBuffer()
abstract IntBuffer asIntBuffer()
abstract LongBuffer asLongBuffer()
```

假设有如下代码

```java
IntBuffer intBufferObj = byteBufferObj.asIntBuffer();
```

此时intBufferObj和byteBufferObj对象共享底层的数组。但是比较坑爹的是两个buffer的position，limit是独立的，这样极易产生bug，需要引起我们注意。

### ByteBuffer的编码和解码

数据传输中我们使用的是ByteBuffer对象作为缓冲区，如果在通道两端我们通信的内容是文本数据，这就涉及到ByteBuffer与CharBuffer的转换。我们可以使用Charset类实现这个转换的功能。

#### 解码示例

```java
ByteBuffer byteBuffer = ByteBuffer.allocate(128);
byteBuffer.put(new byte[]{-26, -120, -111, -25, -120, -79, -28, -67, -96});
byteBuffer.flip();
 
/*对获取utf8的编解码器*/
Charset utf8 = Charset.forName("UTF-8");
CharBuffer charBuffer = utf8.decode(byteBuffer);/*对bytebuffer中的内容解码*/
 
/*array()返回的就是内部的数组引用，编码以后的有效长度是0~limit*/
char[] charArr = Arrays.copyOf(charBuffer.array(), charBuffer.limit());
System.out.println(charArr); /*运行结果：我爱你*/
```

#### 编码示例

```java
CharBuffer charBuffer = CharBuffer.allocate(128);
charBuffer.append("我爱你");
charBuffer.flip();
 
/*对获取utf8的编解码器*/
Charset utf8 = Charset.forName("UTF-8");
ByteBuffer byteBuffer = utf8.encode(charBuffer); /*对charbuffer中的内容解码*/
 
/*array()返回的就是内部的数组引用，编码以后的有效长度是0~limit*/
byte[] bytes = Arrays.copyOf(byteBuffer.array(), byteBuffer.limit());
System.out.println(Arrays.toString(bytes));
/*运行结果：[-26, -120, -111, -25, -120, -79, -28, -67, -96] */
```

我们还可以通过代码中的utf8编解码器分别获取编码器对象和解码器对象

```java
CharsetEncoder utf8Encoder = utf8.newEncoder();
CharsetDecoder utf8Decoder = utf8.newDecoder();
```

然后通过下面编码器和解码器提供的方法进行编解码，其中一些方法可以使ByteBuffer和CharBuffer对象循环使用，不必每次都产生一个新的对象。

#### 解码器方法

```java
/* Convenience method that decodes the remaining content of a single input byte buffer into a newly-allocated character buffer.*/
CharBuffer decode(ByteBuffer in)

/* Decodes as many bytes as possible from the given input buffer, writing the results to the given output buffer.*/
CoderResult decode(ByteBuffer in, CharBuffer out, boolean endOfInput)
    
/*Decodes one or more bytes into one or more characters.*/
protected abstract CoderResult decodeLoop(ByteBuffer in, CharBuffer out)
```

#### 编码器方法

```java
/* Convenience method that encodes the remaining content of a single input character buffer into a newly-allocated byte buffer.*/
ByteBuffer encode(CharBuffer in)

/* Encodes as many characters as possible from the given input buffer, writing the results to the given output buffer.*/
CoderResult encode(CharBuffer in, ByteBuffer out, boolean endOfInput)

/* Encodes one or more characters into one or more bytes.*/
protected abstract CoderResult encodeLoop(CharBuffer in, ByteBuffer out)
```

注意 `encode` 和 `decode` 方法都会改变源 `buffer` 中的 `position` 的位置，这点也是容易产生 Bug 的方法。

## Channel

针对四种不同的应用场景，有四种不同类型的Channel对象。

|        类型         |          应用场景           |   是否阻塞   |
| :-----------------: | :-------------------------: | :----------: |
|     FileChannel     |            文件             |     阻塞     |
|   DatagramChannel   |           UDP协议           | 阻塞或非阻塞 |
|    SocketChannel    |           TCP协议           | 阻塞或非阻塞 |
| ServerSocketChannel | 用于TCP服务器端的监听和链接 | 阻塞或非阻塞 |

Channel 对象的创建都是通过调用内部的open静态方法实现的，此方法是线程安全的。不论哪种类型的 Channel对象，都有read（要理解为从通道中读取，写入缓冲区中）和 write（要理解为从缓冲区中读取数据，写入到通道中）方法，而且read和write方法都只针对ByteBuffer对象。

当我们要获取由通道传输过来的数据时，先调用 channel.read（byteBufferObj）方法，这个方法在内部调用了byteBufferObj 对象的put方法，将通道中的数据写入缓冲区中。当我们要获取由通道传输来的数据时，调用byteBufferObj.flip() ，然后调用 byteBufferObj 的 get 方法获取通道传过来的数据，最后调用 clear 或 compact 方法转换成写模式，为下次 channel.read 做准备。

当我们要向通道发送数据时，先调 channel.write（byteBufferObj）方法,这个方法内部调用了 byteBufferObj 的get方法获取数据，然后将数据写入通道中。当写入完成后调用 clear 或 compact 方法转换成写模式，为下次channel.write写入缓冲区取做准备。

### FileChannel

在文件通道中 read 和 write 方法都是阻塞的，对于 read 方法，除非遇到文件结束，否则会把缓冲区的剩余空间读满再返回。对于 write 方法，会一次性把缓冲区中的内容全部写入到文件中才会返回。

下面的代码展示了 FileChannel 的功能，首先向文本文件中写入 UTF-8 格式的中英文混合字符，然后再读取出来。读写过程中都涉及到编解码问题。

```java
package nioDemo;
 
import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
package nioDemo;
 
import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.CharBuffer;
import java.nio.channels.FileChannel;
import java.nio.charset.Charset;
import java.nio.file.Path;
import java.nio.file.Paths;
 
public class FileChannelDemo {
     
    public static void main(String[] args){ 
         
        /*创建文件，向文件中写入数据*/
        try {
             
            /*如果文件不存在，创建该文件,文件后缀是不是文本文件不重要*/
            File file = new File("E:/noi_utf8.data");
            if(!file.exists()){
                file.createNewFile();
            }
             
            /*根据文件输出流创建与这个文件相关的通道*/
            FileOutputStream fos = new FileOutputStream(file);
            FileChannel fc = fos.getChannel();
             
            /*创建ByteBuffer对象， position = 0, limit = 64*/
            ByteBuffer bb = ByteBuffer.allocate(64);
             
            /*向ByteBuffer中放入字符串UTF-8的字节, position = 17, limit = 64*/
            bb.put("Hello,World 123 \n".getBytes("UTF-8"));
             
            /*flip方法  position = 0, limit = 17*/
            bb.flip();
             
            /*write方法使得ByteBuffer的position到 limit中的元素写入通道中*/
            fc.write(bb);
             
            /*clear方法使得position = 0， limit = 64*/
            bb.clear();
             
            /*下面的代码同理*/
            bb.put("你好，世界 456".getBytes("UTF-8"));
            bb.flip();
             
            fc.write(bb);
            bb.clear();
             
            fos.close();
            fc.close();
             
        } catch (FileNotFoundException e) {
             
        } catch (IOException e) {
            System.out.println(e);
        }
         
         
        /*从刚才的文件中读取字符序列*/
        try {
             
            /*通过Path对象创建文件通道*/
            Path path = Paths.get("E:/noi_utf8.data");
            FileChannel fc = FileChannel.open(path);
             
            ByteBuffer bb = ByteBuffer.allocate((int) fc.size()+1);
             
            Charset utf8 = Charset.forName("UTF-8");
             
            /*阻塞模式，读取完成才能返回*/
            fc.read(bb);
             
            bb.flip();
            CharBuffer cb = utf8.decode(bb);
            System.out.print(cb.toString());
            bb.clear();
             
 
            fc.close();
             
        } catch (IOException e) {
            e.printStackTrace();
        }
         
    }
}
```

### ServerSocketChannel

服务器端用于创建TCP连接的通道，只能对 accept 事件感兴趣。 accept 方法会返回一个已和客户端连接好的SocketChannel 通道，它才服务器是真正传输数据的通道。

### SocketChannel

TCP客户端和TCP服务器端都用它来传输数据。

客户端必须调用 connect 方法去连接服务器。在非阻塞通模式中，该方法将当前通道加入到选择器的已注册集合中，然后通过异步方式进行创建TCP连接，然后该方法立刻返回。注意调用该方法后并不表示已经创建好了TCP连接，如果这个方法返回false，稍后必须调用 finishConnect 方法来完成客户端到服务器的TCP连接。在阻塞方式中， connect 方法会阻塞直到创建好了TCP连接。

 finishConnect 在非阻塞模式中仅仅是返回连接的状态。返回 true 时，表示连接创建好了。在阻塞模式下，直接调用方法 connect 即可完成连接，不需要使用 finishConnect 。

非阻塞模式下，读写操作要配合选择器一起使用。在阻塞模式下，创建好TCP连接后就可以直接对通道进行读写操作。

### DatagramChannel

connect 方法仅用于客户端到服务器端的连接，连接的作用仅仅是避免每次发送和接受数据时的安全检查，提高发送和接受数据的效率，而不是像TCP连接那样表示握手的意思。客户端通道只有调用了 connect 方法后，才能使用 read 和 write 方法读写数据。

客户端也可以不事先调用 connet 方法，而直接使用 receive 方法和 send 方法来实现数据的收发。

```java
abstract SocketAddress receive(ByteBuffer dst)
abstract int send(ByteBuffer src, SocketAddress target) 
```

### 服务器端DatagramChannel和SocketChannel的区别

对于服务器端 DatagramChannel（UDP）和 SocketChannel（TCP）有明显的区别，对于TCP连接，服务器端每创建一个连接就对应一个通道（不同的客户端ip：port 地址对应一个通道），而服务器端UDP的连接始终只有一个通道，所有客户端发送过来的报文都存放于同一个缓冲区中，这显然会降低服务器端的效率，好在  DatagramChannel 对象是线程安全的，可以用多个线程读写同一个 UDP 通道。

服务器端为什么只有一个通道呢？我猜想因为UDP是无状态的，不知道什么时客户端会发送数据，什么时候数据又发送完成，所以服务器端没有办法为每个客户端创建一个通道，就算服务器端根据客户端ip：port为每个客户端创建了通道，服务器端也不知道什么时候该释放这个通道，这就造成了资源的浪费。

## Selector

Selector 类表示选择器，通过这个类的对象可以选取已就绪的通道和这个通道感兴趣的事件。通过静态open方法创建。

### 注册

通道可以通过它的 register 方法，将通道注册到选择器上。

```java
/*Registers this channel with the given selector, returning a selection key.*/
SelectionKey register(Selector sel, int ops)
    
/* Registers this channel with the given selector, returning a selection key. */
abstract SelectionKey register(Selector sel, int ops, Object att)
```

这个该方法会返回一个 SeletctKey 对象，但在这里我们通常忽略这个返回值。SeletctionKey 对象内部包含了这个注册的通道和这个通道感兴趣的事件（ops参数），以及附带的对象（由att参数传递），这个附带的对象通常就是和这个通道相关的读写缓冲区。

### 通道的选择与取消

```java
/* Selects a set of keys whose corresponding channels are ready for I/O operations. */
abstract int select()
    
/* Selects a set of keys whose corresponding channels are ready for I/O operations.*/ abstract int select(long timeout)
    
/* Selects a set of keys whose corresponding channels are ready for I/O operations.*/
abstract int selectNow()
```

三个方法的返回值都表示就绪通道的数量。

select() 方法是个阻塞方法，有通道就绪才会返回。

select(long timeout) ，最多阻塞 timeout 毫秒，即使没有通道就绪也会返回，若超时返回，则当前线程中断标志位被设置。若阻塞时间内有通道就绪，就提前返回。

seletor.selectNow()，非阻塞方法。

一个 seletor 对象内部维护了三个集合。

1. 已注册集合：表示了所有已注册通道的SelectionKey对象。
2. 就绪集合：表示了所有已就绪通道的SelectionKey对象。
3. 取消集合：表示了所有需要取消注册关系的通道的SelectionKey对象。

SelectionKey 的 cancel 方法用于取消通道和选择器的注册关系，这个方法只是把表示当前通道的 SelectionKey 放入取消集合中，下次调用 select 方法时才会真正取消注册关系。

select 方法每次会从已注册的通道集合中删除所有已取消的通道的 SelectionKey，然后清空已取消的通道集合，最后从更新过的已注册通道集合中选出就绪的通道，放入已就绪的集合中。每次调用 select 方法，会向已就绪的集合中放入已就绪通道的 SelectionKey 对象，调用 selectedKeys 方法就会返回这个已就绪通道集合的引用。当我们处理完一个已就绪通道，该通道对应的SelectionKey对象仍然位于已就绪的集合中，这就要求我们处理一个已就绪的通道后就必须手动从已就绪的集合中删除它，否则下次调用 selectedKeys 时，已处理过的通道还存在于这个集合中，导致线程空转。这里也是极易产生Bug的。

### 通道的写方法注意事项

#### 1、写方法什么时候就绪？

写操作的就绪条件为 socket 底层写缓冲区有空闲空间，此时并不代表我们这时有（或者需要将）数据写入通道。而底层写缓冲区绝大部分时间都是有空闲空间的，所以当你注册写事件后，写操作基本一直是就绪的。这就导致只要有一个通道对写事件感兴趣，select 方法几乎总是立刻返回的，但是实际上我们可能没有数据可写的，所以使得调用 select 方法的线程总是空转。对于客户端发送一些数据，客户端返回一些数据的模型，我们可以在读事件完成后，再设置通道对写事件感兴趣，写操作完成后再取消该通道对写事件的兴趣，这样就可以避免上述问题。

#### 2、如何正确的发送数据

```java
while(writeBuffer.hasRemaining()){
    channel.write(writeBuffer);
}
```

上面发送数据的通常用的代码，当网络状况良好的情况下，这段代码能正常工作。 现在我们考虑一种极端情况，服务器端写事件就绪，我们向底层的写缓冲区写入一些数据后，服务器端到客户端的链路出现问题，服务器端没能把数据发送出去，此时底层的写缓冲区一直处于满的状态，假设 writeBuffer 中仍然还有没发送完的数据就会导致while 循环空转，浪费CPU资源，同时也妨碍这个selector管理的其它通道的读写。

为了解决个问题，我们应该使用下面的方法发送数据

```java
int len = 0;
while(writeBuffer.hasRemaining()){
    len = sc.write(writeBuffer);
    /*说明底层的socket写缓冲已满*/
    if(len == 0){
        break;
    }
}
```

## 代码示例

下面这个类，后面的代码都会用到，它只是两个缓冲区的包装

```java
package nioDemo;
 
import java.nio.ByteBuffer;
 
/*自定义Buffer类中包含读缓冲区和写缓冲区，用于注册通道时的附加对象*/
public class Buffers {
 
    ByteBuffer readBuffer;
    ByteBuffer writeBuffer;
     
    public Buffers(int readCapacity, int writeCapacity){
        readBuffer = ByteBuffer.allocate(readCapacity);
        writeBuffer = ByteBuffer.allocate(writeCapacity);
    }
     
    public ByteBuffer getReadBuffer(){
        return readBuffer;
    }
     
    public ByteBuffer gerWriteBuffer(){
        return writeBuffer;
    }
}
```

### TCP非阻塞示例

服务器端代码

```java
package nioDemo;
 
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.CharBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.nio.charset.Charset;
import java.util.Iterator;
import java.util.Random;
import java.util.Set;
 
 
/*服务器端，:接收客户端发送过来的数据并显示，
 *服务器把上接收到的数据加上"echo from service:"再发送回去*/
public class ServiceSocketChannelDemo {
     
    public static class TCPEchoServer implements Runnable{
         
        /*服务器地址*/
        private InetSocketAddress localAddress;
         
        public TCPEchoServer(int port) throws IOException{
            this.localAddress = new InetSocketAddress(port);
        }
         
         
        @Override
        public void run(){
             
            Charset utf8 = Charset.forName("UTF-8");
             
            ServerSocketChannel ssc = null;
            Selector selector = null;
             
            Random rnd = new Random();
             
            try {
                /*创建选择器*/
                selector = Selector.open();
                 
                /*创建服务器通道*/
                ssc = ServerSocketChannel.open();
                ssc.configureBlocking(false);
                 
                /*设置监听服务器的端口，设置最大连接缓冲数为100*/
                ssc.bind(localAddress, 100);
                 
                /*服务器通道只能对tcp链接事件感兴趣*/
                ssc.register(selector, SelectionKey.OP_ACCEPT);
                 
            } catch (IOException e1) {
                System.out.println("server start failed");
                return;
            } 
             
            System.out.println("server start with address : " + localAddress);
             
            /*服务器线程被中断后会退出*/
            try{
                 
                while(!Thread.currentThread().isInterrupted()){
                     
                    int n = selector.select();
                    if(n == 0){
                        continue;
                    }
 
                    Set<SelectionKey> keySet = selector.selectedKeys();
                    Iterator<SelectionKey> it = keySet.iterator();
                    SelectionKey key = null;
                     
                    while(it.hasNext()){
                             
                        key = it.next();
                        /*防止下次select方法返回已处理过的通道*/
                        it.remove();
                         
                        /*若发现异常，说明客户端连接出现问题,但服务器要保持正常*/
                        try{
                            /*ssc通道只能对链接事件感兴趣*/
                            if(key.isAcceptable()){
                                 
                                /*accept方法会返回一个普通通道，
                                     每个通道在内核中都对应一个socket缓冲区*/
                                SocketChannel sc = ssc.accept();
                                sc.configureBlocking(false);
                                 
                                /*向选择器注册这个通道和普通通道感兴趣的事件，同时提供这个新通道相关的缓冲区*/
                                int interestSet = SelectionKey.OP_READ;                             
                                sc.register(selector, interestSet, new Buffers(256, 256));
                                 
                                System.out.println("accept from " + sc.getRemoteAddress());
                            }
                             
                             
                            /*（普通）通道感兴趣读事件且有数据可读*/
                            if(key.isReadable()){
                                 
                                /*通过SelectionKey获取通道对应的缓冲区*/
                                Buffers  buffers = (Buffers)key.attachment();
                                ByteBuffer readBuffer = buffers.getReadBuffer();
                                ByteBuffer writeBuffer = buffers.gerWriteBuffer();
                                 
                                /*通过SelectionKey获取对应的通道*/
                                SocketChannel sc = (SocketChannel) key.channel();
                                 
                                /*从底层socket读缓冲区中读入数据*/
                                sc.read(readBuffer);
                                readBuffer.flip();
                                 
                                /*解码显示，客户端发送来的信息*/
                                CharBuffer cb = utf8.decode(readBuffer);
                                System.out.println(cb.array());
                     
                                readBuffer.rewind();
 
                                 
                                /*准备好向客户端发送的信息*/
                                /*先写入"echo:"，再写入收到的信息*/
                                writeBuffer.put("echo from service:".getBytes("UTF-8"));
                                writeBuffer.put(readBuffer);
                                 
                                readBuffer.clear();
                                 
                                /*设置通道写事件*/
                                key.interestOps(key.interestOps() | SelectionKey.OP_WRITE);
                                                                 
                            }
                             
                            /*通道感兴趣写事件且底层缓冲区有空闲*/
                            if(key.isWritable()){
                                 
                                Buffers  buffers = (Buffers)key.attachment();
                                 
                                ByteBuffer writeBuffer = buffers.gerWriteBuffer();
                                writeBuffer.flip();
                                 
                                SocketChannel sc = (SocketChannel) key.channel();
                                 
                                int len = 0;
                                while(writeBuffer.hasRemaining()){
                                    len = sc.write(writeBuffer);
                                    /*说明底层的socket写缓冲已满*/
                                    if(len == 0){
                                        break;
                                    }
                                }
                                 
                                writeBuffer.compact();
                                 
                                /*说明数据全部写入到底层的socket写缓冲区*/
                                if(len != 0){
                                    /*取消通道的写事件*/
                                    key.interestOps(key.interestOps() & (~SelectionKey.OP_WRITE));
                                }
                                 
                            }
                        }catch(IOException e){
                            System.out.println("service encounter client error");
                            /*若客户端连接出现异常，从Seletcor中移除这个key*/
                            key.cancel();
                            key.channel().close();
                        }
 
                    }
                         
                    Thread.sleep(rnd.nextInt(500));
                }
                 
            }catch(InterruptedException e){
                System.out.println("serverThread is interrupted");
            } catch (IOException e1) {
                System.out.println("serverThread selecotr error");
            }finally{
                try{
                    selector.close();
                }catch(IOException e){
                    System.out.println("selector close failed");
                }finally{
                    System.out.println("server close");
                }
            }
 
        }
    }
     
    public static void main(String[] args) throws InterruptedException, IOException{
        Thread thread = new Thread(new TCPEchoServer(8080));
        thread.start();
        Thread.sleep(100000);
        /*结束服务器线程*/
        thread.interrupt();
    }
     
}
```

客户端程序

```java
package nioDemo;
 
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.CharBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.SocketChannel;
import java.nio.charset.Charset;
import java.util.Iterator;
import java.util.Random;
import java.util.Set;
 
/*客户端:客户端每隔1~2秒自动向服务器发送数据，接收服务器接收到数据并显示*/
public class ClientSocketChannelDemo {
     
    public static class TCPEchoClient implements Runnable{
         
        /*客户端线程名*/
        private String name;
        private Random rnd = new Random();
         
        /*服务器的ip地址+端口port*/
        private InetSocketAddress remoteAddress;
         
        public TCPEchoClient(String name, InetSocketAddress remoteAddress){
            this.name = name;
            this.remoteAddress = remoteAddress;
        }
         
        @Override
        public void run(){
             
            /*创建解码器*/
            Charset utf8 = Charset.forName("UTF-8");
             
            Selector selector;
             
            try {
                 
                /*创建TCP通道*/
                SocketChannel sc = SocketChannel.open();
                 
                /*设置通道为非阻塞*/
                sc.configureBlocking(false);
                 
                /*创建选择器*/
                selector = Selector.open();
                 
                /*注册感兴趣事件*/
                int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;
                 
                /*向选择器注册通道*/
                sc.register(selector, interestSet, new Buffers(256, 256));
                 
                /*向服务器发起连接,一个通道代表一条tcp链接*/
                sc.connect(remoteAddress);
                 
                /*等待三次握手完成*/
                while(!sc.finishConnect()){
                    ;
                }
 
                System.out.println(name + " " + "finished connection");
                 
            } catch (IOException e) {
                System.out.println("client connect failed");
                return;
            }
             
            /*与服务器断开或线程被中断则结束线程*/
            try{
 
                int i = 1;
                while(!Thread.currentThread().isInterrupted()){
                     
                    /*阻塞等待*/
                    selector.select();
                     
                    /*Set中的每个key代表一个通道*/
                    Set<SelectionKey> keySet = selector.selectedKeys();
                    Iterator<SelectionKey> it = keySet.iterator();
                     
                    /*遍历每个已就绪的通道，处理这个通道已就绪的事件*/
                    while(it.hasNext()){
                         
                        SelectionKey key = it.next();
                        /*防止下次select方法返回已处理过的通道*/
                        it.remove();
                         
                        /*通过SelectionKey获取对应的通道*/
                        Buffers  buffers = (Buffers)key.attachment();
                        ByteBuffer readBuffer = buffers.getReadBuffer();
                        ByteBuffer writeBuffer = buffers.gerWriteBuffer();
                         
                        /*通过SelectionKey获取通道对应的缓冲区*/
                        SocketChannel sc = (SocketChannel) key.channel();
                         
                        /*表示底层socket的读缓冲区有数据可读*/
                        if(key.isReadable()){
                            /*从socket的读缓冲区读取到程序定义的缓冲区中*/
                            sc.read(readBuffer);
                            readBuffer.flip();
                            /*字节到utf8解码*/
                            CharBuffer cb = utf8.decode(readBuffer);
                            /*显示接收到由服务器发送的信息*/
                            System.out.println(cb.array());
                            readBuffer.clear();
                        }
                         
                        /*socket的写缓冲区可写*/
                        if(key.isWritable()){
                            writeBuffer.put((name + "  " + i).getBytes("UTF-8"));
                            writeBuffer.flip();
                            /*将程序定义的缓冲区中的内容写入到socket的写缓冲区中*/
                            sc.write(writeBuffer);
                            writeBuffer.clear();
                            i++;
                        }
                    }
                     
                    Thread.sleep(1000 + rnd.nextInt(1000));
                }
             
            }catch(InterruptedException e){
                System.out.println(name + " is interrupted");
            }catch(IOException e){
                System.out.println(name + " encounter a connect error");
            }finally{
                try {
                    selector.close();
                } catch (IOException e1) {
                    System.out.println(name + " close selector failed");
                }finally{
                    System.out.println(name + "  closed");
                }
            }
        }
         
    }
     
    public static void main(String[] args) throws InterruptedException{
         
        InetSocketAddress remoteAddress = new InetSocketAddress("192.168.1.100", 8080);
         
        Thread ta = new Thread(new TCPEchoClient("thread a", remoteAddress));
        Thread tb = new Thread(new TCPEchoClient("thread b", remoteAddress));
        Thread tc = new Thread(new TCPEchoClient("thread c", remoteAddress));
        Thread td = new Thread(new TCPEchoClient("thread d", remoteAddress));
         
        ta.start();
        tb.start();
        tc.start();
         
        Thread.sleep(5000);
 
        /*结束客户端a*/
        ta.interrupt();
         
        /*开始客户端d*/
        td.start();
    }
}
```

### UDP示例

客户端非阻塞模式，服务器端阻塞模式

服务器端代码（服务器端只有一个通道，对应一个读缓冲区，一个写缓冲区，所以使用非阻塞方式容易发生数据混乱）

```java
package nioDemo;
 
import java.io.IOException;
import java.net.InetSocketAddress;
import java.net.SocketAddress;
import java.nio.ByteBuffer;
import java.nio.CharBuffer;
import java.nio.channels.DatagramChannel;
import java.nio.charset.Charset;
 
public class ServiceDatagramChannelDemo {
     
    public static class UDPEchoService implements Runnable{
         
        private int port;
         
        public UDPEchoService(int port){
            this.port = port;
        }
         
        @Override
        public void run(){
             
            ByteBuffer readBuffer = ByteBuffer.allocate(256);
            ByteBuffer writeBuffer = ByteBuffer.allocate(256);
 
            DatagramChannel dc = null;
             
            try{
                 
                /*服务器端使用默认的阻塞IO的方式*/
                dc = DatagramChannel.open();
                dc.bind(new InetSocketAddress(port));
                 
                System.out.println("service start");
                while(!Thread.currentThread().isInterrupted()){
                     
                    try{
                         
                        /*先读取客户端发送的消息，直到读取到消息才会返回*/
                        /*只能调用receive方法，因为不知道哪个地址给服务器发信息，没法实现调用connect方法*/
                        /*dc是阻塞的，所以receive方法要等到接收到数据才返回*/
                        SocketAddress clientAddress = dc.receive(readBuffer);
                        readBuffer.flip();
                        CharBuffer charBuffer = Charset.defaultCharset().decode(readBuffer);
                        System.out.println(charBuffer.array());
                         
                        /*调用send方法向客户端发送的消息，
                         *dc是阻塞的,所以直到send方法把数据全部写入到socket缓冲区才返回*/
                        writeBuffer.put("echo : ".getBytes());
                        readBuffer.rewind();
                        writeBuffer.put(readBuffer);
                        writeBuffer.flip();
                        dc.send(writeBuffer, clientAddress);
                         
                        readBuffer.clear();
                        writeBuffer.clear();
                         
                    }catch(IOException e){
                        System.out.println("receive from or send to client failed");
                    }
                }
            }catch(IOException e){
                System.out.println("server error");
            }finally{
                try {
                    if(dc != null){
                        dc.close();
                    }
                } catch (IOException e) {
 
                }
            }
        }
    }
     
    public static void main(String[] args) throws IOException{
        new Thread(new UDPEchoService(8080)).start();
    }
}
```

客户端代码

```java
package nioDemo;
 
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.CharBuffer;
import java.nio.channels.DatagramChannel;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.charset.Charset;
import java.util.Iterator;
 
public class ClientDatagramChannelDemo {
     
    public static class UDPEchoClient implements Runnable{
         
        private String name;
        private InetSocketAddress serviceAddress;
         
        public UDPEchoClient(String name, InetSocketAddress serviceAddress){
            this.name = name;
            this.serviceAddress = serviceAddress;
        }
         
        @Override
        public void run(){
            DatagramChannel dc = null;
            try{
                 
                /*每个实际上可以创建多个通道连接同一个服务器地址，
                我们这里为了演示方便，只创建了一个通道*/
                dc = DatagramChannel.open();
                 
                /*客户端采用非阻塞模式*/
                dc.configureBlocking(false);
 
                /*这里的连接不是指TCP的握手连接，因为UDP协议本身不需要连接，
                 *这里连接的意思大概是提前向操作系统申请好本地端口号，以及高速操作系统要发送的目的
                 *连接后的UDP通道可以提高发送的效率，还可以调用read和write方法接收和发送数据
                 *未连接的UDP通道只能调用receive和send方法接收和发送数据*/
                dc.connect(serviceAddress);
                 
                Selector selector = Selector.open();
                int interest = SelectionKey.OP_READ | SelectionKey.OP_WRITE;
                dc.register(selector, interest, new Buffers(256, 256));
                 
                int i = 0;
                while(!Thread.currentThread().isInterrupted()){
                     
                    selector.select();
                     
                    Iterator<SelectionKey> it = selector.selectedKeys().iterator();
                    while(it.hasNext()){
                         
                        SelectionKey key = it.next();
                        it.remove();
                         
                        Buffers buffers = (Buffers)key.attachment();
                         
                        ByteBuffer readBuffer = buffers.getReadBuffer();
                        ByteBuffer writeBuffer = buffers.gerWriteBuffer();
                         
                        try{
                             
                            if(key.isReadable()){
                                dc.read(readBuffer);
                                readBuffer.flip();
                                CharBuffer charBuffer = Charset.defaultCharset().decode(readBuffer);
                                System.out.println(charBuffer.array());
                                readBuffer.clear();
                            }
                             
                            if(key.isWritable()){
                                writeBuffer.put((name + (i++)).getBytes());
                                writeBuffer.flip();
                                dc.write(writeBuffer);
                                writeBuffer.clear();
                                 
                                Thread.sleep(500);
                            }
                         
                        }catch(IOException e){
                            key.cancel();
                            key.channel().close();
                        }
                    }
                }
            }catch(InterruptedException e){
                System.out.println(name + "interrupted");
            } catch (IOException e) {
                System.out.println(name + "encounter connect error");
            } finally{
                try {
                    dc.close();
                } catch (IOException e) {
                    System.out.println(name + "encounter close error");
                }finally{
                    System.out.println(name + "closed");
                }
            }
        }
    }
     
    public static void main(String[] args){
         
        InetSocketAddress serviceAddress = new InetSocketAddress("192.168.1.100", 8080);
         
        UDPEchoClient clientA = new UDPEchoClient("thread a ", serviceAddress);
        UDPEchoClient clientB = new UDPEchoClient("thread b ", serviceAddress);
        UDPEchoClient clientC = new UDPEchoClient("thread c ", serviceAddress);
         
        new Thread(clientA).start();
        new Thread(clientB).start();
        new Thread(clientC).start();
         
    }
}
```

## 参考内容

[堆外内存之 DirectByteBuffer 详解](http://www.importnew.com/26334.html)

[SocketChannel---各种注意点](https://blog.csdn.net/billluffy/article/details/78036998)

[JDK 8 API 文档](https://docs.oracle.com/javase/8/docs/api/)



