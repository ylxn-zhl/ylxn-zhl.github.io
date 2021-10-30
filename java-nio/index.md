# Java NIO


# NIO

## 1，java NIO概述

- 替代java io的一个操作
- 面向缓冲区也可以基于通道操作
- 更高效的进行文件的读写操作

### 1.1 阻塞 IO

读或者写数据的时候，会阻塞直到数据能够正常的读或者写入。在传统的方法中，服务器为客户端建立一个线程，这种模式带来了如果线程增多，大量线程会操成服务器的压力。为了解决这种问题，采用线程池的思想，并设置线程池的大小，但是超出线程数的上限之后的线程就访问不了。

### 1.2 非阻塞 IO（NIO）

非阻塞IO指的是IO事件本身不阻塞，是获取IO事件的select()方法是需要阻塞等待的，区别是阻塞IO会阻塞IO操作上，而非阻塞IO阻塞在事件获取上，没有事件就没有IO，select() 阻塞的时候还没有发生，何谈IO阻塞，本质是延迟IO操作，真正发生IO的时候才执行，	而不是发生的时候在阻塞，用Selector负责去监听多个通道，注册感兴趣的特定IO事件，之后系统进行通知。

![image-20211024140913585](/Users/cenghaile/Documents/zhl/static/image-20211024140913585.png)

当有读或写等任何注册的事件发生时，可以从 Selector 中获得相应的 SelectionKey，同时从 SelectionKey 中可以找到发生的事件和该事件所发生的具体的 SelectableChannel，以获得客户端发送过来的数据。



| IO     | NIO        |
| ------ | ---------- |
| 面向流 | 面向缓冲区 |
| 阻塞IO | 非阻塞IO   |
| 无     | 选择器     |

### 1.3 NIO概念

Java NIO 由以下几个核心部分组成，还有其他组件（pipe、filelock）

-  **Channel**（双向的，既可以用来进行读操作，又可以用来进行写操作）
  - 主要有如下：FileChannel（IO）、DatagramChannel（UDP ）、SocketChannel （TCP中Server ）和 ServerSocketChannel（TCP中Client）。
- **Buffer**
  - 主要有：ByteBuffer, CharBuffer, DoubleBuffer, FloatBuffer,IntBuffer, LongBuffer, ShortBuffer
- **Selector**（处理多个 Channel）

## 2 Channel（通道）

- 可以进行读取和写入，或者进行读取操作，全双工。
- 操作的数据源可以是多种，比如文件，网络和socket。
- Channel 用于在字节缓冲区和位于通道另一侧的实体（通常是一个文件或套接字）之间有效地传输数据
- 从通道读取数据到缓冲区，从缓冲区写入数据到通道

Java NIO类似于流，但又有所不同。

- 既可以从通道中读取数据，又可以写数据到通道。但流的读写通常是单向的。
- 通道可以异步地读写。
- 通道中的数据总是要先读到一个 Buffer，或者总是要从一个 Buffer 中写入

主要是接口实现，不同操作系统不同接口实现，通过代码也可以看到其代码为接口

```java 
public interface Channel extends Closeable {
    /**
     * Tells whether or not this channel is open.
     *
     * @return <tt>true</tt> if, and only if, this channel is open
     */
    public boolean isOpen();
    /**
     * Closes this channel.
     *
     * <p> After a channel is closed, any further attempt to invoke I/O
     * operations upon it will cause a {@link ClosedChannelException} to be
     * thrown.
     *
     * <p> If this channel is already closed then invoking this method has no
     * effect.
     *
     * <p> This method may be invoked at any time.  If some other thread has
     * already invoked it, however, then another invocation will block until
     * the first invocation is complete, after which it will return without
     * effect. </p>
     *
     * @throws  IOException  If an I/O error occurs
     */
    public void close() throws IOException;
}
```

实现接口主要有以下几个常用类：

- FileChannel 从文件中读写数据。
- DatagramChannel 能通过 UDP 读写网络中的数据。
- SocketChannel 能通过 TCP 读写网络中的数据。
- ServerSocketChannel 可以监听新进来的 TCP 连接，像 Web 服务器那样。对每一个新进来的连接都会创建一个 SocketChannel。

### 2.1 FileChannel 

主要是文件IO，最常用的一个类。

**以下是FileChannel常用的方法**

|             方法              |                     描述                     |
| :---------------------------: | :------------------------------------------: |
|   int read(ByteBuffer dst)    |       从Channel中读取数据到 ByteBuffer       |
| long read(ByteBuffer[] dsts)  |    将Channel中的数据“分散”到 ByteBuffer[]    |
|   int write(ByteBuffer src)   |      将ByteBuffer中的数据写入到Channel       |
| long write(ByteBuffer[] srcs) |    将ByteBuffer[]中的数据“聚集”到Channel     |
|        long position()        |             返回此通道的文件位置             |
| FileChannel position(long p） |             设置此通道的文件位置             |
|          long size()          |          返回此通道的文件的当前大小          |
| FileChannel truncate(long s)  |         将此通道的文件截取为给定大小         |
| void force(boolean metaData)  | 强制将所有对此通道的文件更新写入到存储设备中 |

**Buffer 通常的操作**

1. 将数据写入缓冲区
2. 调用 buffer.flip() 反转读写模式
3. 从缓冲区读取数据
4. 调用 buffer.clear() 或 buffer.compact() 清除缓冲区内容

**部分步骤代码展示：**

- 先打开文件，无法直接打开一个FileChannel，需要通过使用一个 InputStream、OutputStream 或RandomAccessFile 来获取一个 FileChannel。

  ```java 
  //创建FileChannel
  RandomAccessFile aFile = new RandomAccessFile("b://1.txt","rw");
  FileChannel channel = aFile.getChannel();
  ```

- 创建Buffer

  ```java 
  ByteBuffer buf = ByteBuffer.allocate(1024);
  ```

- 从 FileChannel 读取数据read()方法返回的 int 值表示了有多少字节被读到了 Buffer 中。如果返回-1，表示到了文件末尾.

  ```java 
   int bytesRead = channel.read(buf);
  ```

- FileChannel.write()方法向 FileChannel 写数据，该方法的参数是一个 Buffer。在 while 循环中调用的。因为无法保证 write()方法一次能向 FileChannel 写入多少字节，因此需要重复调用 write()方法，直到 Buffer 中已经没有尚未写入通道的字节

**读数据主要的代码思路步骤是**：

1. 创建一个FileChannel
2. 创建一个数据缓冲区
3. 读取数据到缓冲区中
4. 判断数据是否有，如果有，则取出，判断的依据是获取到的数据是否为-1，取出的数据要先反转读写操作，之后如果数据缓冲区还有，则取出，最后清除数据缓冲区后在判断是否缓冲区还有数据。关闭FileChannel.

**完整代码展示**

```java 
public class FileChannelDemo1 {
    //FileChannel读取数据到buffer中
    public static void main(String[] args) throws Exception {
        //创建FileChannel
        RandomAccessFile aFile = new RandomAccessFile("b://1.txt","rw");
        FileChannel channel = aFile.getChannel();

        //创建Buffer
        ByteBuffer buf = ByteBuffer.allocate(1024);

        //读取数据到buffer中
        int bytesRead = channel.read(buf);
        while(bytesRead != -1) {
            System.out.println("读取了："+bytesRead);
            buf.flip();
            while(buf.hasRemaining()) {
                System.out.println((char)buf.get());
            }
            buf.clear();
            bytesRead = channel.read(buf);
        }
        aFile.close();
        System.out.println("结束了");
    }
}
```

**写数据主要代码思路是：**

1. 创建一个FileChannel
2. 创建一个数据缓冲区
3. 创建要写入的数据对象，以及清空以下缓冲区（防止出错）
4. 要写入的数据写入到缓冲区中
5. 缓冲区读写反转
6. 判断缓冲区是否有数据，将数据一个一个写入到FileChannel
7. 关闭FileChannel

**完整代码展示：**

```java
//FileChanne写操作
public class FileChannelDemo2 {

    public static void main(String[] args) throws Exception {
        // 打开FileChannel
        RandomAccessFile aFile = new RandomAccessFile("b://1.txt","rw");
        FileChannel channel = aFile.getChannel();

        //创建buffer对象
        ByteBuffer buffer = ByteBuffer.allocate(1024);

        String newData = "manongyanjiuseng";
        buffer.clear();

        //写入内容
        buffer.put(newData.getBytes());

        buffer.flip();

        //FileChannel完成最终实现
        while (buffer.hasRemaining()) {
            channel.write(buffer);
        }

        //关闭
        channel.close();
    }
}
```

### 2.2 其他常用方法

```java
以下举例的方法有：
position 
size 
truncate
force
transferTo 和 transferFrom
```

- position方法

  需要在 FileChannel 的某个特定位置进行数据的读/写操作。可以通过调用position()方法获取 FileChannel 的当前位置。也可以通过调用 position(long pos)方法设置 FileChannel 的当前位置

  注意这样设置会造成两个后果：
  位置如果设置在文件结束符之后，读取数据的文件结束标志返回-1，而且写入数据的时候前面会有间隙，导致文件空洞

  ```java
  long pos = channel.position();
  channel.position(pos +123);
  ```

- size方法

  返回该实例所关联文件的大小

- truncate方法

  截取一个文件。截取文件时，文件将中指定长度，后面的部分将被删除而且截取的数据长度是以字节截取

- force方法

  尚未写入磁盘的数据强制写到磁盘上

- **transferTo 和 transferFrom 方法**

  进行通道之间的传输
  注意一个To与From的区别，一个主动一个被动

以下是transferFrom 的完整代码：

```java 
//通道之间数据传输
public class FileChannelDemo3 {

    //transferFrom()
    public static void main(String[] args) throws Exception {
        // 创建两个fileChannel
        RandomAccessFile aFile = new RandomAccessFile("b://1.txt","rw");
        FileChannel fromChannel = aFile.getChannel();

        RandomAccessFile bFile = new RandomAccessFile("b://2.txt","rw");
        FileChannel toChannel = bFile.getChannel();

        //fromChannel 传输到 toChannel
        long position = 0;
        long size = fromChannel.size();
        toChannel.transferFrom(fromChannel,position,size);

        aFile.close();
        bFile.close();
        System.out.println("over!");
    }
}
```

以下是transferTo的完整代码：

```java 
//通道之间数据传输
public class FileChannelDemo4 {

    //transferTo()
    public static void main(String[] args) throws Exception {
        // 创建两个fileChannel
        RandomAccessFile aFile = new RandomAccessFile("b://1.txt","rw");
        FileChannel fromChannel = aFile.getChannel();

        RandomAccessFile bFile = new RandomAccessFile("b://2.txt","rw");
        FileChannel toChannel = bFile.getChannel();

        //fromChannel 传输到 toChannel
        long position = 0;
        long size = fromChannel.size();
        fromChannel.transferTo(0,size,toChannel);

        aFile.close();
        bFile.close();
        System.out.println("over!");
    }
}
```

## 2.3 Socket通道

所有的 socket 通道类(DatagramChannel、SocketChannel 和 ServerSocketChannel)都继承了位于 java.nio.channels.spi 包中的 AbstractSelectableChannel。这意味着我们可以用一个 Selector 对象来执行socket 通道的就绪选择

- 非阻塞模式中，有更多的伸缩性和灵活性。使用socket，可以用一个线程或者多个线程就可以管理成百上千个socket，管理功能强大，而且没有性能损失，可以通过 Selector监听多个通道
- DatagramChannel 和 SocketChannel 实现定义读和写功能的接口而ServerSocketChannel 不实现。ServerSocketChannel 负责监听传入的连接和创建新的 SocketChannel 对象，它本身从不传输数据。究其代码是因为它实现的接口比较少
- socket通道可以被重复使用，socket不可以被重复使用
- 设置socket的通道模式为阻塞还是非阻塞，可以通过AbstractSelectableChannel.java 中实现的 configureBlocking()方法。传递参数值为 true 则设为阻塞模式，参数值为 false 值设为非阻塞模式

### 2.3.1 ServerSocketChannel

- 本身不传入数据，主要是为了监听，可以在非阻塞模式下运行
- 没有bind()绑定方法，通过对等的socket来绑定端口并进行监听
  - ServerSocketChannel的对象.socket().bind();
- ServerSocketChannel的对象.socket().bind();

其它 Socket 的 accept()方法会阻塞返回一个 Socket 对象。如果ServerSocketChannel 以非阻塞模式被调用，当没有传入连接在等待时，ServerSocketChannel.accept( )会立即返回 null。正是这种检查连接而不阻塞的能力实现了可伸缩性并降低了复杂性。可选择性也因此得到实现。我们可以使用一个选择器实例来注册 ServerSocketChannel 对象以实现新连接到达时自动通知的功能。

具体的代码思路步骤为：

- 打开ServerSocketChannel
- 绑定端口号
- 设置非阻塞模式`configureBlocking(false);`
- 监听连接，通过accept
- 将其buffer归为0指针`rewind()`，并且写入buffer
- 关闭 ServerSocketChannel

具体通过代码演示
如下代码

```java
public class ServerSocketChannelDemo {

    public static void main(String[] args) throws Exception {
        //端口号
        int port = 8888;

        //buffer
        ByteBuffer buffer = ByteBuffer.wrap("hello atguigu".getBytes());

        //ServerSocketChannel
        ServerSocketChannel ssc = ServerSocketChannel.open();
        //绑定
        ssc.socket().bind(new InetSocketAddress(port));

        //设置非阻塞模式
        ssc.configureBlocking(false);

        //监听有新链接传入
        while(true) {
            System.out.println("Waiting for connections");
            SocketChannel sc = ssc.accept();
            if(sc == null) { //没有链接传入
                System.out.println("null");
                Thread.sleep(2000);
            } else {
                System.out.println("Incoming connection from: " + sc.socket().getRemoteSocketAddress());
                buffer.rewind(); //指针0
                sc.write(buffer);
                sc.close();
            }
        }
    }
}
```

通过浏览器点击127.0.0.1:8888，就会出现监听的提示

### 2.3.2 SocketChannel

- 用于连接到TCP的套接字
- 实现多路复用
- 面向缓冲区

```bash
主要的特征有：
（1）对于已经存在的 socket 不能创建 SocketChannel
（2）SocketChannel 中提供的 open 接口创建的 Channel 并没有进行网络级联，需要使用 connect 接口连接到指定地址
（3）未进行连接的 SocketChannle 执行 I/O 操作时，会抛出
NotYetConnectedException
（4）SocketChannel 支持两种 I/O 模式：阻塞式和非阻塞式
（5）SocketChannel 支持异步关闭。如果 SocketChannel 在一个线程上 read 阻塞，另一个线程对该 SocketChannel 调用 shutdownInput，则读阻塞的线程将返回-1 表示没有读取任何数据；如果 SocketChannel 在一个线程上 write 阻塞，另一个线程对该SocketChannel 调用 shutdownWrite，则写阻塞的线程将抛出AsynchronousCloseException
（6）
SocketChannel 支持设定参数
SO_SNDBUF 套接字发送缓冲区大小
SO_RCVBUF 套接字接收缓冲区大小
SO_KEEPALIVE 保活连接
O_REUSEADDR 复用地址
SO_LINGER 有数据传输时延缓关闭 Channel (只有在非阻塞模式下有用)
TCP_NODELAY 禁用 Nagle 算法
```


