**同步与异步**

同步和异步关注的是消息通信机制 (synchronous communication/ asynchronous communication)。

所谓同步，就是在发出一个*调用*时，在没有得到结果之前，该*调用*就不返回。但是一旦调用返回，就得到返回值了。换句话说，就是由*调用者*主动等待这个*调用*的结果。

而异步则是相反，*调用*在发出之后，这个调用就直接返回了，所以没有返回结果。换句话说，当一个异步过程调用发出后，调用者不会立刻得到结果。而是在*调用*发出后，*被调用者*通过状态、通知来通知调用者，或通过回调函数处理这个调用。

举个通俗的例子：

你打电话问书店老板有没有《分布式系统》这本书，如果是同步通信机制，书店老板会说，你稍等，”我查一下"，然后开始查啊查，等查好了（可能是5秒，也可能是一天）告诉你结果（返回结果）。

而异步通信机制，书店老板直接告诉你我查一下啊，查好了打电话给你，然后直接挂电话了（不返回结果）。然后查好了，他会主动打电话给你。在这里老板通过“回电”这种方式来回调。

**阻塞与非阻塞**

阻塞和非阻塞关注的是程序在等待调用结果（消息，返回值）时的状态。

阻塞调用是指调用结果返回之前，当前线程会被挂起。调用线程只有在得到结果之后才会返回。

非阻塞调用指在不能立刻得到结果之前，该调用不会阻塞当前线程。

还是上面的例子：

你打电话问书店老板有没有《分布式系统》这本书，你如果是阻塞式调用，你会一直把自己“挂起”，直到得到这本书有没有的结果，如果是非阻塞式调用，你不管老板有没有告诉你，你自己先一边去玩了， 当然你也要偶尔过几分钟check一下老板有没有返回结果。

在这里阻塞与非阻塞与是否同步异步无关。跟老板通过什么方式回答你结果无关。

以下针对Java网络IO。

## BIO（同步阻塞IO）

服务端：

由一个独立的线程负责监听客户端的连接，它每次接收到客户端的连接请求后都会为该客户端创建一个新的线程，这个线程负责与对应的客户端进行数据收发。

客户端：

向服务端发起请求，如果没有响应则会等待或收到拒绝请求。

![img](https://pic3.zhimg.com/80/v2-8500b64977e726db44c74ba1a2b0b1d2_hd.jpg)

**实例代码**

服务端：

```
import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;

public class Server {
    public static void main(String[] args) throws IOException {
        ServerSocket server = new ServerSocket(10086);
        System.out.println("等待客户端连接");
        while (true) {
            Socket socket = server.accept();
            new Thread(new SocketThread(socket)).start();
        }
    }
}

```

服务端处理socket线程：

```
import java.io.*;
import java.net.Socket;

public class SocketThread implements Runnable {
    private Socket socket;

    public SocketThread(Socket socket) {
        this.socket = socket;
    }

    @Override
    public void run() {
        try {
            InputStream is = socket.getInputStream();
            InputStreamReader isr = new InputStreamReader(is);
            BufferedReader br = new BufferedReader(isr);
            String temp = null;
            while ((temp = br.readLine()) != null) {
                System.out.println("接收到端口号为" + socket.getPort() + "的客户端发来的数据：" + temp);
            }
            socket.shutdownInput();

            OutputStream os = socket.getOutputStream();
            PrintWriter pw = new PrintWriter(os);
            pw.write("客户端你好");
            pw.flush();

            pw.close();
            os.close();
            br.close();
            isr.close();
            is.close();
            socket.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

客户端：

```
import java.io.*;
import java.net.Socket;
import java.net.UnknownHostException;

public class Client {
    public static void main(String[] args) {
        for (int i = 0; i < 3; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        Socket client = new Socket("localhost",10086);
                        OutputStream os = client.getOutputStream();
                        PrintWriter pw = new PrintWriter(os);
                        pw.write("服务器你好");
                        pw.flush();
                        client.shutdownOutput();

                        InputStream is = client.getInputStream();
                        InputStreamReader isr = new InputStreamReader(is);
                        BufferedReader br = new BufferedReader(isr);
                        String temp = null;
                        while ((temp = br.readLine()) != null) {
                            System.out.println("接受到服务器发来的数据：" + temp);
                        }

                        br.close();
                        isr.close();
                        is.close();
                        pw.close();
                        os.close();
                        client.close();
                    } catch (UnknownHostException e) {
                        e.printStackTrace();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }).start();
        }
    }
}

```

服务端输出：

![img](https://pic1.zhimg.com/80/v2-dc71b50e65711c034080cf4b5aea6090_hd.jpg)

客户端输出：

![img](https://pic1.zhimg.com/80/v2-219bf00677887a28f74d52a49a5f2c70_hd.jpg)

从以上代码，很容易看出，BIO主要的问题在于每当有一个新的客户端请求接入时，服务端必须创建一个新的线程来处理这条链路，在需要满足高性能、高并发的场景是没法应用的（大量创建新的线程会严重影响服务器性能，甚至罢工）。

## NIO（同步非阻塞）

NIO我们一般认为是New I/O（也是官方的叫法），因为它是相对于老的I/O类库新增的，做了很大的改变。但民间跟多人称之为Non-block I/O，即非阻塞I/O，因为这样叫，更能体现它的特点。而下文中的NIO，不是指整个新的I/O库，而是非阻塞I/O。

NIO提供了与传统BIO模型中的Socket和ServerSocket相对应的SocketChannel和ServerSocketChannel两种不同的套接字通道实现。

新增的着两种通道都支持阻塞和非阻塞两种模式。阻塞模式使用就像传统中的支持一样，比较简单，但是性能和可靠性都不好；非阻塞模式正好与之相反。

对于低负载、低并发的应用程序，可以使用同步阻塞I/O来提升开发速率和更好的维护性；对于高负载、高并发的（网络）应用，应使用NIO的非阻塞模式来开发。

![img](https://pic1.zhimg.com/80/v2-75cb2f6e3c36f8c5b0a17f910950b68c_hd.jpg)

下面先介绍一些基础知识

**缓冲区 Buffer**

Buffer是一个对象，包含一些要写入或者读出的数据。

在NIO库中，所有数据都是用缓冲区处理的。在读取数据时，它是直接读到缓冲区中的；在写入数据时，也是写入到缓冲区中。任何时候访问NIO中的数据，都是通过缓冲区进行操作。

缓冲区实际上是一个数组，并提供了对数据结构化访问以及维护读写位置等信息。

具体的缓存区有这些：ByteBuffer、CharBuffer、 ShortBuffer、IntBuffer、LongBuffer、FloatBuffer、DoubleBuffer。他们实现了相同的接口：Buffer。

**通道 Channel**

我们对数据的读取和写入要通过Channel，它就像水管一样，是一个通道。通道不同于流的地方就是通道是双向的，可以用于读、写和同时读写操作。

底层的操作系统的通道一般都是全双工的，所以全双工的Channel比流能更好的映射底层操作系统的API。

Channel主要分两大类：

- SelectableChannel：用户网络读写
- FileChannel：用于文件操作

后面代码会涉及的ServerSocketChannel和SocketChannel都是SelectableChannel的子类。

**多路复用器 Selector**

Selector是Java NIO 编程的基础。

Selector提供选择已经就绪的任务的能力：Selector会不断轮询注册在其上的Channel，如果某个Channel上面发生读或者写事件，这个Channel就处于就绪状态，会被Selector轮询出来，然后通过SelectionKey可以获取就绪Channel的集合，进行后续的I/O操作。

一个Selector可以同时轮询多个Channel，因为JDK使用了epoll()代替传统的select实现，所以没有最大连接句柄1024/2048的限制。所以，只需要一个线程负责Selector的轮询，就可以接入成千上万的客户端。

**实例代码**

服务端：

```
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;

public class Server {
    public static void main(String[] args) throws IOException {
        // 打开通道
        ServerSocketChannel server = ServerSocketChannel.open();
        // 设为非阻塞
        server.configureBlocking(false);
        // 绑定端口
        server.bind(new InetSocketAddress(10086));
        // 获得一个selector
        Selector selector = Selector.open();
        // 将通道管理器和该通道绑定，并为该通道注册SelectionKey.OP_ACCEPT事件,注册该事件后，
        // 当该事件到达时，selector.select()会返回，如果该事件没到达selector.select()会一直阻塞。
        server.register(selector, SelectionKey.OP_ACCEPT);

        System.out.println("等待客户端连接");

        // 采用轮询的方式监听selector上是否有需要处理的事件，如果有，则进行处理
        while (true) {
            // 当注册的事件到达时，方法返回；否则,该方法会一直阻塞
            selector.select();
            // 获得selector中选中的项的迭代器，选中的项为注册的事件
            Iterator<SelectionKey> ite = selector.selectedKeys().iterator();
            while (ite.hasNext()) {
                SelectionKey key = (SelectionKey)ite.next();
                // 删除已选的key,以防重复处理
                ite.remove();

                if (key.isAcceptable()) { // 客户端请求连接事件
                    ServerSocketChannel serverSocketChannel = (ServerSocketChannel)key.channel();
                    // 获得和客户端连接的通道
                    SocketChannel channel = serverSocketChannel.accept();
                    // 设为非阻塞
                    channel.configureBlocking(false);
                    // 在这里可以给客户端发送信息哦
                    channel.write(ByteBuffer.wrap(new String("客户端你好")
                            .getBytes("utf-8")));
                    // 在和客户端连接成功之后，为了可以接收到客户端的信息，需要给通道设置读的权限。
                    channel.register(selector, SelectionKey.OP_READ);

                } else if (key.isReadable()) { // 获得了可读的事件
                    // 服务器可读取消息:得到事件发生的Socket通道
                    SocketChannel channel = (SocketChannel)key.channel();
                    // 创建读取的缓冲区
                    ByteBuffer buffer = ByteBuffer.allocate(512);
                    channel.read(buffer);
                    byte[] data = buffer.array();
                    String msg = new String(data).trim();
                    System.out.println("接收到客户端发来的数据：" + msg);
                }
            }
        }
    }
}

```

客户端：

```
import java.io.*;
import java.net.InetSocketAddress;
import java.net.Socket;
import java.net.UnknownHostException;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.SocketChannel;
import java.util.Iterator;

public class Client {
    public static void main(String[] args) {
        for (int i = 0; i < 3; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        // 获得一个Socket通道
                        SocketChannel channel = SocketChannel.open();
                        // 设为非阻塞
                        channel.configureBlocking(false);
                        // 获得一个selector
                        Selector selector = Selector.open();

                        // 客户端连接服务器,其实方法执行并没有实现连接，需要在listen（）方法中调
                        // 用channel.finishConnect();才能完成连接
                        channel.connect(new InetSocketAddress("localhost", 10086));
                        // 将通道管理器和该通道绑定，并为该通道注册SelectionKey.OP_CONNECT事件。
                        channel.register(selector, SelectionKey.OP_CONNECT);

                        // 采用轮询的方式监听selector上是否有需要处理的事件，如果有，则进行处理
                        while (true) {
                            // 选择一组可以进行I/O操作的事件，放在selector中,客户端的该方法不会阻塞，
                            // 这里和服务端的方法不一样，查看api注释可以知道，当至少一个通道被选中时，
                            // selector的wakeup方法被调用，方法返回，而对于客户端来说，通道一直是被选中的
                            selector.select();
                            // 获得selector中选中的项的迭代器
                            Iterator<SelectionKey> ite = selector.selectedKeys().iterator();
                            while (ite.hasNext()) {
                                SelectionKey key = (SelectionKey) ite.next();
                                // 删除已选的key,以防重复处理
                                ite.remove();
                                if (key.isConnectable()) { // 连接事件发生
                                    SocketChannel socketChannel = (SocketChannel) key.channel();
                                    // 如果正在连接，则完成连接
                                    if(socketChannel.isConnectionPending()){
                                        socketChannel.finishConnect();
                                    }
                                    // 设置成非阻塞
                                    socketChannel.configureBlocking(false);
                                    // 在这里可以给服务端发送信息哦
                                    socketChannel.write(ByteBuffer.wrap(new String("服务器你好").getBytes("utf-8")));
                                    //在 和服务端连接成功之后，为了可以接收到服务端的信息，需要给通道设置读的权限。
                                    socketChannel.register(selector, SelectionKey.OP_READ);
                                } else if (key.isReadable()) { // 获得了可读的事件
                                    // 服务器可读取消息:得到事件发生的Socket通道
                                    SocketChannel socketChannel = (SocketChannel) key.channel();
                                    // 创建读取的缓冲区
                                    ByteBuffer buffer = ByteBuffer.allocate(512);
                                    socketChannel.read(buffer);
                                    byte[] data = buffer.array();
                                    String msg = new String(data).trim();
                                    System.out.println("接受到服务器发来的数据：" + msg);
                                }
                            }
                        }
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }).start();
        }
    }
}

```

服务端和客户端输出和BIO一样

可以看到，创建NIO服务端的主要步骤如下：

1. 打开ServerSocketChannel，监听客户端连接
2. 绑定监听端口，设置连接为非阻塞模式
3. 创建Reactor线程，创建多路复用器并启动线程
4. 将ServerSocketChannel注册到Reactor线程中的Selector上，监听ACCEPT事件
5. Selector轮询准备就绪的key
6. Selector监听到新的客户端接入，处理新的接入请求，完成TCP三次握手，简历物理链路
7. 设置客户端链路为非阻塞模式
8. 将新接入的客户端连接注册到Reactor线程的Selector上，监听读操作，读取客户端发送的网络消息
9. 异步读取客户端消息到缓冲区
10. 对Buffer编解码，处理半包消息，将解码成功的消息封装成Task
11. 将应答消息编码为Buffer，调用SocketChannel的write将消息异步发送给客户端

## AIO（异步非阻塞IO）

采用“订阅-通知”模式：即应用程序向操作系统注册IO监听，然后继续做自己的事情。当操作系统发生IO事件，并且准备好数据后，在主动通知应用程序，触发相应的函数

![img](https://pic3.zhimg.com/80/v2-cc21094cd828763c037f0d8ce68767a6_hd.jpg)

注意：

- 在JAVA NIO框架中，我们说到了一个重要概念“selector”（选择器）。它负责代替应用查询中所有已注册的通道到操作系统中进行IO事件轮询、管理当前注册的通道集合，定位发生事件的通道等操操作；但是在JAVA AIO框架中，由于应用程序不是“轮询”方式，而是订阅-通知方式，所以不再需要“selector”（选择器）了，改由channel通道直接到操作系统注册监听。
- JAVA AIO框架中，只实现了两种网络IO通道“AsynchronousServerSocketChannel”（服务器监听通道）、“AsynchronousSocketChannel”（socket套接字通道）。但是无论哪种通道他们都有独立的fileDescriptor（文件标识符）、attachment（附件，附件可以使任意对象，类似“通道上下文”），并被独立的SocketChannelReadHandle类实例引用。



## IO流学习总结

### [一　Java IO，硬骨头也能变软](https://mp.weixin.qq.com/s?__biz=MzU4NDQ4MzU5OA==&mid=2247483981&idx=1&sn=6e5c682d76972c8d2cf271a85dcf09e2&chksm=fd98542ccaefdd3a70428e9549bc33e8165836855edaa748928d16c1ebde9648579d3acaac10#rd)

**（1） 按操作方式分类结构图：**

[![按操作方式分类结构图：](https://camo.githubusercontent.com/50f105c85f6b42d643d46e1ac7bb0f855b92cd9d/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f352f31362f313633363764346664316365316234363f773d37323026683d3130383026663d6a70656726733d3639353232)](https://camo.githubusercontent.com/50f105c85f6b42d643d46e1ac7bb0f855b92cd9d/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f352f31362f313633363764346664316365316234363f773d37323026683d3130383026663d6a70656726733d3639353232)

**（2）按操作对象分类结构图**

[![按操作对象分类结构图](https://camo.githubusercontent.com/8957efacdf1cd4eac15d844da8353a7f77a3c863/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f352f31362f313633363764363733623065323638643f773d37323026683d35333526663d6a70656726733d3436303831)](https://camo.githubusercontent.com/8957efacdf1cd4eac15d844da8353a7f77a3c863/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f352f31362f313633363764363733623065323638643f773d37323026683d35333526663d6a70656726733d3436303831)

### [二　java IO体系的学习总结](https://blog.csdn.net/nightcurtis/article/details/51324105)

1. **IO流的分类：**

   - 按照流的流向分，可以分为输入流和输出流；
   - 按照操作单元划分，可以划分为字节流和字符流；
   - 按照流的角色划分为节点流和处理流。

2. **流的原理浅析:**

   java Io流共涉及40多个类，这些类看上去很杂乱，但实际上很有规则，而且彼此之间存在非常紧密的联系， Java Io流的40多个类都是从如下4个抽象类基类中派生出来的。

   - **InputStream/Reader**: 所有的输入流的基类，前者是字节输入流，后者是字符输入流。
   - **OutputStream/Writer**: 所有输出流的基类，前者是字节输出流，后者是字符输出流。

3. **常用的io流的用法**

### [三　Java IO面试题](https://mp.weixin.qq.com/s?__biz=MzU4NDQ4MzU5OA==&mid=2247483985&idx=1&sn=38531c2cee7b87f125df7aef41637014&chksm=fd985430caefdd26b0506aa84fc26251877eccba24fac73169a4d6bd1eb5e3fbdf3c3b940261#rd)

## NIO与AIO学习总结

### [一 Java NIO 概览](https://mp.weixin.qq.com/s?__biz=MzU4NDQ4MzU5OA==&mid=2247483956&idx=1&sn=57692bc5b7c2c6dfb812489baadc29c9&chksm=fd985455caefdd4331d828d8e89b22f19b304aa87d6da73c5d8c66fcef16e4c0b448b1a6f791#rd)

1. **NIO简介**:

   Java NIO 是 java 1.4, 之后新出的一套IO接口NIO中的N可以理解为Non-blocking，不单纯是New。

2. **NIO的特性/NIO与IO区别:**

   - 1)IO是面向流的，NIO是面向缓冲区的；
   - 2)IO流是阻塞的，NIO流是不阻塞的;
   - 3)NIO有选择器，而IO没有。

3. **读数据和写数据方式:**

   - 从通道进行数据读取 ：创建一个缓冲区，然后请求通道读取数据。
   - 从通道进行数据写入 ：创建一个缓冲区，填充数据，并要求通道写入数据。

4. **NIO核心组件简单介绍**

   - **Channels**
   - **Buffers**
   - **Selectors**

### [二 Java NIO 之 Buffer(缓冲区)](https://mp.weixin.qq.com/s?__biz=MzU4NDQ4MzU5OA==&mid=2247483961&idx=1&sn=f67bef4c279e78043ff649b6b03fdcbc&chksm=fd985458caefdd4e3317ccbdb2d0a5a70a5024d3255eebf38183919ed9c25ade536017c0a6ba#rd)

1. **Buffer(缓冲区)介绍:**

   - Java NIO Buffers用于和NIO Channel交互。 我们从Channel中读取数据到buffers里，从Buffer把数据写入到Channels；
   - Buffer本质上就是一块内存区；
   - 一个Buffer有三个属性是必须掌握的，分别是：capacity容量、position位置、limit限制。

2. **Buffer的常见方法**

   - Buffer clear()
   - Buffer flip()
   - Buffer rewind()
   - Buffer position(int newPosition)

3. **Buffer的使用方式/方法介绍:**

   - 分配缓冲区（Allocating a Buffer）:

   ```
   ByteBuffer buf = ByteBuffer.allocate(28);//以ByteBuffer为例子
   ```

   - 写入数据到缓冲区（Writing Data to a Buffer）

   **写数据到Buffer有两种方法：**

   1.从Channel中写数据到Buffer

   ```
   int bytesRead = inChannel.read(buf); //read into buffer.
   ```

   2.通过put写数据：

   ```
   buf.put(127);
   ```

4. **Buffer常用方法测试**

   说实话，NIO编程真的难，通过后面这个测试例子，你可能才能勉强理解前面说的Buffer方法的作用。

### [三 Java NIO 之 Channel（通道）](https://mp.weixin.qq.com/s?__biz=MzU4NDQ4MzU5OA==&mid=2247483966&idx=1&sn=d5cf18c69f5f9ec2aff149270422731f&chksm=fd98545fcaefdd49296e2c78000ce5da277435b90ba3c03b92b7cf54c6ccc71d61d13efbce63#rd)

1. **Channel（通道）介绍**通常来说NIO中的所有IO都是从 Channel（通道） 开始的。NIO Channel通道和流的区别：
2. **FileChannel的使用**
3. **SocketChannel和ServerSocketChannel的使用**
4. **️DatagramChannel的使用**
5. **Scatter / Gather**Scatter: 从一个Channel读取的信息分散到N个缓冲区中(Buufer).Gather: 将N个Buffer里面内容按照顺序发送到一个Channel.
6. **通道之间的数据传输**在Java NIO中如果一个channel是FileChannel类型的，那么他可以直接把数据传输到另一个channel。transferFrom() :transferFrom方法把数据从通道源传输到FileChanneltransferTo() :transferTo方法把FileChannel数据传输到另一个channel

### [四 Java NIO之Selector（选择器）](https://mp.weixin.qq.com/s?__biz=MzU4NDQ4MzU5OA==&mid=2247483970&idx=1&sn=d5e2b133313b1d0f32872d54fbdf0aa7&chksm=fd985423caefdd354b587e57ce6cf5f5a7bec48b9ab7554f39a8d13af47660cae793956e0f46#rd)

1. **Selector（选择器）介绍**

   - Selector 一般称 为选择器 ，当然你也可以翻译为 多路复用器 。它是Java NIO核心组件中的一个，用于检查一个或多个NIO Channel（通道）的状态是否处于可读、可写。如此可以实现单线程管理多个channels,也就是可以管理多个网络链接。
   - 使用Selector的好处在于： 使用更少的线程来就可以来处理通道了， 相比使用多个线程，避免了线程上下文切换带来的开销。

2. **Selector（选择器）的使用方法介绍**

   - Selector的创建

   ```
   Selector selector = Selector.open();
   ```

   - 注册Channel到Selector(Channel必须是非阻塞的)

   ```
   channel.configureBlocking(false);
   SelectionKey key = channel.register(selector, Selectionkey.OP_READ);
   ```

   - SelectionKey介绍

     一个SelectionKey键表示了一个特定的通道对象和一个特定的选择器对象之间的注册关系。

   - 从Selector中选择channel(Selecting Channels via a Selector)

     选择器维护注册过的通道的集合，并且这种注册关系都被封装在SelectionKey当中.

   - 停止选择的方法

     wakeup()方法 和close()方法。

3. **模板代码**

   有了模板代码我们在编写程序时，大多数时间都是在模板代码中添加相应的业务代码。

4. **客户端与服务端简单交互实例**

### [五 Java NIO之拥抱Path和Files](https://mp.weixin.qq.com/s?__biz=MzU4NDQ4MzU5OA==&mid=2247483976&idx=1&sn=2296c05fc1b840a64679e2ad7794c96d&chksm=fd985429caefdd3f48e2ee6fdd7b0f6fc419df90b3de46832b484d6d1ca4e74e7837689c8146&token=537240785&lang=zh_CN#rd)

**一 文件I/O基石：Path：**

- 创建一个Path
- File和Path之间的转换，File和URI之间的转换
- 获取Path的相关信息
- 移除Path中的冗余项

**二 拥抱Files类：**

- Files.exists() 检测文件路径是否存在
- Files.createFile() 创建文件
- Files.createDirectories()和Files.createDirectory()创建文件夹
- Files.delete()方法 可以删除一个文件或目录
- Files.copy()方法可以吧一个文件从一个地址复制到另一个位置
- 获取文件属性
- 遍历一个文件夹
- Files.walkFileTree()遍历整个目录

### [六 NIO学习总结以及NIO新特性介绍](https://blog.csdn.net/a953713428/article/details/64907250)

- **内存映射：**

这个功能主要是为了提高大文件的读写速度而设计的。内存映射文件(memory-mappedfile)能让你创建和修改那些大到无法读入内存的文件。有了内存映射文件，你就可以认为文件已经全部读进了内存，然后把它当成一个非常大的数组来访问了。将文件的一段区域映射到内存中，比传统的文件处理速度要快很多。内存映射文件它虽然最终也是要从磁盘读取数据，但是它并不需要将数据读取到OS内核缓冲区，而是直接将进程的用户私有地址空间中的一部分区域与文件对象建立起映射关系，就好像直接从内存中读、写文件一样，速度当然快了。

### [七 Java NIO AsynchronousFileChannel异步文件通](http://wiki.jikexueyuan.com/project/java-nio-zh/java-nio-asynchronousfilechannel.html)

Java7中新增了AsynchronousFileChannel作为nio的一部分。AsynchronousFileChannel使得数据可以进行异步读写。

### [八 高并发Java（8）：NIO和AIO](http://www.importnew.com/21341.html)











