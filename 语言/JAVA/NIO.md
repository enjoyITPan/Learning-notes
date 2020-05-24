Java NIO是为了解决高并发请求提出的设计模型，是基于IO多路复用设计出来的。底层又依赖于操作系统的支持(select、poll、epoll)。

在了解NIO之前，先来回顾下Java BIO(阻塞IO)的实现。

```java
public static void main(String[] args) throws Exception{
    ServerSocket serverSocket =  new ServerSocket();
    serverSocket.bind(new InetSocketAddress(8080));
    Socket socket = serverSocket.accept();

    System.out.println("连接成功");
    BufferedReader br = new BufferedReader(new InputStreamReader(socket.getInputStream()));
    System.out.println("准备读取数据");
    String msg = br.readLine();
    System.out.println(msg);
}
```

服务监听8080端口，并且调用accept()接受连接，请求连接完成后调用socket.getInputStream()获取数据流，最后调用readLine()读取数据。

这里需要注意accept() 和 readLine() 的调用都是阻塞，也就是如果没有请求连接获取数据发送过来，线程就只能干等着，什么也不能干。导致系统假死。

针对上面的情况可以有几种方案：

1、每个请求单独开一个线程进行处理

这样的方案在qps很低的情况也是可行的。当qps 增多时就无能为力了。毕竟系统资源是有限的，java 中每个线程会占据512K~1M的内存空间，而且线程过多时，可能执行线程切换的时间甚至会大于线程执行的时间，这时候带来的表现往往是系统load偏高、CPU sy使用率特别高（超过20%以上)，导致系统几乎陷入不可用的状态。系统消耗在线程上下文切换的时间上会更多，系统更加不可用。

2、使用线程池进行处理

这个方案限制了线程数，但是必然会导致系统的可服务的qps变低。无法满足高并发的需求，因为线程大部分时间是阻塞的，不干活的，这种其实是一种浪费。



所以，当面对十万甚至百万级连接的时候，传统的BIO模型是无能为力的。随着移动端应用的兴起和各种网络游戏的盛行，百万级长连接日趋普遍，此时，必然需要一种更高效的I/O处理模型。

从上面的场景可以看出，传统的BIO编程存在几个背景：

1、系统资源是有限的，不能无限制开启线程（如果服务器、数据库等可以资源无限，那么每个请求开一个线程肯定是可以的）

2、获取IO连接和获取IO数据都是阻塞的，线程这个时候不干活

系统资源有限这个问题可以无限的搭建集群去解决，但是这都是真金白银，而且系统利用率贼低，老板要是知道你这样设计，会让你准备准备卷铺盖走人。所以我们只能从问题2去解决。

问题2分析：

1、获取IO连接阻塞

io连接的监听其实只需要一个线程就可以解决，当获取到连接请求后，将请求封装为socket 传递给业务线程，监听线程就可以继续监听了，所以性能消耗不大，不是阻塞点。

2、获取IO数据阻塞

传统BIO获取数据阻塞的原因是：不知道数据何时到来，调用read后就会陷入系统调用，系统调用会阻塞该调用，直到将数据从TCP 缓存区 -> 系统内核缓冲区 -> 进程缓存区才会返回，在这个期间线程是没有办法做别的事情。

这个时候可以想到，如果有一种机制：使用很小的代价监听每个连接的数据准备情况，当数据可以读的时候通知线程，线程再去读取数据，这样线程就可以有效的被利用起来了。更进一步：如果可以直接把数据已经拷贝到了进程缓冲区，直接通知线程去处理。

其中第一种场景就是IO多路复用：操作系统内核提供一种机制(select、poll、epoll)允许用户将fd(linux 设计中万事万物即文件，网络连接也不例外)注册到内核中，由内核来帮忙管理状态，当状态可读可写时再唤醒线程。

其中select、poll、epoll的区别在于：

select、poll 的fd是保存在用户态的，需要拷贝到内核态进行遍历，查看事情是否就绪。并且即使有就绪事情发生也不会单独拎出来该事件，只有修改事件状态，具体是哪个事情需要线程自己去遍历找到。

epoll就是事件fd在内核，并且事件状态变更时，只返回已经就绪的事件。底层采用红黑树管理事件，插入、删除等性能高。select采用数组、poll采用链表，性能都不高。

java NIO  就是基于io 多路复用进行设计的。

```java
public static void selectMethod() throws Exception{

    ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
    serverSocketChannel.configureBlocking(false);
    serverSocketChannel.socket().bind(new InetSocketAddress(8080));

    Selector selector = Selector.open();

    serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

    while (true){
        System.out.println("start select");
        selector.select(); //阻塞的,也有非阻塞的selectNow
        System.out.println("end select");
        Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
        while (iterator.hasNext()){
            SelectionKey key = iterator.next();
            if(key.isAcceptable()){
                SocketChannel channel = serverSocketChannel.accept();
                channel.configureBlocking(false);
                channel.register(selector, SelectionKey.OP_READ);
            }else{
                SocketChannel socketChannel = (SocketChannel) key.channel();
                ByteBuffer buffer = ByteBuffer.allocate(256);
                System.out.println("准备读取数据");
                int i = socketChannel.read(buffer);
                System.out.println("读到数据");
                if( i != -1){
                    String msg = new String(buffer.array()).trim();
                    System.out.println(msg);
                }else{
                    socketChannel.close();
                }
            }
            iterator.remove();
        }
    }

}
```

这里的例子只是展示了java NIO  的编程改变，所以处理数据部分没有单独开线程。

可以看出java nio 采用了io多路复用的事件通知机制。通过register注册自己感兴趣的事件，然后调用 selector.select() 等待事件就绪，该方法是阻塞的。有事件就绪时，会返回就绪事件集合，使用SelectionKey进行了封装，遍历SelectionKey集合可以获取感兴趣的事件。

```java
isAcceptable：连接事件就绪
isReadable：可读事件就绪
isWritable：可写事件就绪
```

这样当事件就绪了再调用read时就会直接读取数据，不在等待数据。这里需要注意读取数据时是阻塞的，需要等到数据从内核缓冲区-> 进程缓存区后才会返回。不过一般情况下很快，是"有意义"的消耗。



Java NIO  的核心优化就是在于减少了无效阻塞，减少了线程的摸鱼时间。

今天你摸了多久鱼呢？

![一组上班摸鱼表情包](/Users/bytedance/Desktop/一组上班摸鱼表情包.png)

