# Netty

## 一、Java IO的发展历程

### 	1.必须明白的几个重要的概念

#### 					1)、阻塞和非阻塞

​										阻塞：阻塞往往是需要等待缓冲区的数据准备之后才能做其他的事情。

​										非阻塞：当我们的线程访问我们缓冲区数据的时候，如果数据没有准备好，线程可以去做其他的事情。

#### 						2)、 同步和异步

​									同步：是应用程序要直接参与IO 读写的操作

​													同步处理IO流的时候，必须阻塞在某个方法上面等待我们的IO事件完成。

   												阻塞到IO 事件，阻塞到read 或则write。这个时候我们就完全不能做自己的事情。让读写方法加入到线
													程里面，然后阻塞线程来实现，对线程的性能开销比较大。

​									异步：所有的IO操作都是交给操作系统去处理。

​												所有的IO 读写都交给了操作系统。这个时候，我们可以去做其他的事情，并不需要去完成真正的IO 操作，当                   												操作完成IO 后，会给我们的应用程序一个通知。

### 2、BIO和NIO的区别

#### 1、BIO就是传统的IO流，是同步阻塞的

### 2、NIO是同步不阻塞的

BIO的话如果你进行写或者读数据的话，你必须将所有的数据都读完或者写完，才能继续干其他的事情。

Java NIO 的非阻塞模式，使一个线程从某通道发送请求读取数据，但是它仅能得到目前可用的数据，如果目前没有数据可用时，就什么都不会获取。而不是保持线程阻塞，所以直至数据变的可以读取之前，该线程可以继续做其他的事情。

在NIO中，会有一个selector会轮询所有的channel，不过返回的结果如何，线程都会做其他的事情。

### 3、面向流和缓冲区

#### 	3.1）BIO是面向流的，意思就是说每次只能从流中读一个字节或者多个字节，直到读取完所有的字节。

​		NIO是将数据缓存到缓存区的，需要时可以将缓存区的数据进行移动，增加了灵活性。

### 4、使用上的对比

   BIO的话，数据是一层一层进行读，比如

```java
Name:Tom
Age:18
Email: tom@qq.com
Phone:13888888888
```

读数据的方法：

```java
FileInputStream input = new FileInputStream("d://info.txt");
BufferedReader reader = new BufferedReader(new InputStreamReader(input));
String nameLine = reader.readLine();
String ageLine = reader.readLine();
String emailLine = reader.readLine();
String phoneLine = reader.readLine();
```

## 二、详解NIO的三大重要组件

### 1、buffer

#### 1、基本的api.

![image-20200609204517679](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200609204533.png)

观看源码我们能够看到，其组成是一个数组。在读数据的时候，先读到缓冲区，写数据的时候写到缓冲区的。下面是一个简单的使用。

```java
public static void main(String[] args) {  
        // 分配新的int缓冲区，参数为缓冲区容量
        // 新缓冲区的当前位置将为零，其界限(限制位置)将为其容量。它将具有一个底层实现数组，其数组偏移量将为零。  
        IntBuffer buffer = IntBuffer.allocate(8);
  
        for (int i = 0; i < buffer.capacity(); ++i) {  
            int j = 2 * (i + 1);  
            // 将给定整数写入此缓冲区的当前位置，当前位置递增  
            buffer.put(j);  
        }
        // 重设此缓冲区，将限制设置为当前位置，然后将当前位置设置为0  
        buffer.flip();
        // 查看在当前位置和限制位置之间是否有元素  
        while (buffer.hasRemaining()) {  
            // 读取此缓冲区当前位置的整数，然后当前位置递增  
            int j = buffer.get();  
            System.out.print(j + "  ");  
        }
	}  
```

在里面有几个特别重要的属性。

position：指定下一个将要被写入或者读取的元素索引，它的值由get()/put()方法自动更新，在新创建一个Buffer 对象
时，position 被初始化为0。
limit：指定还有多少数据需要取出(在从缓冲区写入通道时)，或者还有多少空间可以放入数据(在从通道读入缓冲区时)。
capacity：指定了可以存储在缓冲区中的最大数据容量，实际上，它指定了底层数组的大小，或者至少是指定了准许我
们使用的底层数组的容量。

从这几个属性，我们可以猜到 缓存区的数在移动。

```java
    static final Unsafe UNSAFE = Unsafe.getUnsafe();
    static final int SPLITERATOR_CHARACTERISTICS = 16464;
    private int mark = -1;
    private int position = 0;
    private int limit;
    private int capacity;
```

```java
 public static void main(String args[]) throws Exception {
        //这用用的是文件IO处理
        FileInputStream fin = new FileInputStream("E://test.txt");
        //创建文件的操作管道
        FileChannel fc = fin.getChannel();

        //分配一个10个大小缓冲区，说白了就是分配一个10个大小的byte数组
        ByteBuffer buffer = ByteBuffer.allocate(10);
        output("初始化", buffer);
        //先读一下
        fc.read(buffer);
        output("调用read()", buffer);

        //准备操作之前，先锁定操作范围
        buffer.flip();
        output("调用flip()", buffer);
        //判断有没有可读数据
        while (buffer.remaining() > 0) {
            byte b = buffer.get();
            System.out.print(b);
        }
        output("调用get()", buffer);
        //可以理解为解锁
        buffer.clear();
        output("调用clear()", buffer);
        //最后把管道关闭
        fin.close();
    }

    //把这个缓冲里面实时状态给答应出来
    public static void output(String step, ByteBuffer buffer) {
        System.out.println(step + " : ");
        //容量，数组大小
        System.out.print("capacity: " + buffer.capacity() + ", ");
        //当前操作数据所在的位置，也可以叫做游标
        System.out.print("position: " + buffer.position() + ", ");
        //锁定值，flip，数据操作范围索引只能在position - limit 之间
        System.out.println("limit: " + buffer.limit());
        System.out.println();
    }




//输出结果
capacity: 10, position: 0, limit: 10

调用read() : 
capacity: 10, position: 4, limit: 10

调用flip() : 
capacity: 10, position: 0, limit: 4

1051119945调用get() : 
capacity: 10, position: 4, limit: 4

调用clear() : 
capacity: 10, position: 0, limit: 10
```



### 2、channel

通道是一个对象，通过它可以读取和写入数据，当然了所有数据都通过Buffer 对象来处理。我们永远不会将字节直接
咕泡出品，写入通道中，相反是将数据写入包含一个或者多个字节的缓冲区。同样不会直接从通道中读取字节，而是将数据从通
道读入缓冲区，再从缓冲区获取这个字节。

### 3、selector

相当于一个轮询接口，会检测一下你的通道信息，有没有进行改变。

初始化构造方法，可以通过  selector.selectedKeys()获取到通道内的信息。

```java

    private Selector selector = null;    
public NIOChatServer(int port) throws IOException{

		this.port = port;

		ServerSocketChannel server = ServerSocketChannel.open();

		server.bind(new InetSocketAddress("127.0.0.1",this.port));
		server.configureBlocking(false);
		
		selector = Selector.open();

		server.register(selector, SelectionKey.OP_ACCEPT);

		System.out.println("服务已启动，监听端口是：" + this.port);
	}
```

进行轮序

```java
   public void listen() throws IOException{
       while(true) {
           int wait = selector.select();
           if(wait == 0) continue;
           Set<SelectionKey> keys = selector.selectedKeys();  //可以通过这个方法，知道可用通道的集合
           Iterator<SelectionKey> iterator = keys.iterator();
           while(iterator.hasNext()) {
         SelectionKey key = (SelectionKey) iterator.next();
         iterator.remove();
         process(key);
           }
       }

}
```

## 三、Netty

> Netty其实是对Nio的一个封装。

```java
//基础线程池   以默认的方法创建一个线程池
        EventLoopGroup bossGroup = new NioEventLoopGroup();
        //工作线程池
        EventLoopGroup workerGroup = new NioEventLoopGroup();

        try {
            ServerBootstrap b = new ServerBootstrap();
            b.group(bossGroup, workerGroup)
            		.channel(NioServerSocketChannel.class)

                    .childHandler(new ChannelInitializer<SocketChannel>() {

                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ChannelPipeline pipeline = ch.pipeline();
                            //自定义协议解码器
                            /** 入参有5个，分别解释如下
                             maxFrameLength：框架的最大长度。如果帧的长度大于此值，则将抛出TooLongFrameException。
                             lengthFieldOffset：长度字段的偏移量：即对应的长度字段在整个消息数据中得位置
                             lengthFieldLength：长度字段的长度。如：长度字段是int型表示，那么这个值就是4（long型就是8）
                             lengthAdjustment：要添加到长度字段值的补偿值
                             initialBytesToStrip：从解码帧中去除的第一个字节数
                             */
                            pipeline.addLast(new LengthFieldBasedFrameDecoder(Integer.MAX_VALUE, 0, 4, 0, 4));
                            //自定义协议编码器
                            pipeline.addLast(new LengthFieldPrepender(4));
                            //对象参数类型编码器
                            pipeline.addLast("encoder",new ObjectEncoder());
                            //对象参数类型解码器
                            pipeline.addLast("decoder",new ObjectDecoder(Integer.MAX_VALUE,ClassResolvers.cacheDisabled(null)));
                            pipeline.addLast(new RegistryHandler());
                        }
                    })
                    .option(ChannelOption.SO_BACKLOG, 128)
                    .childOption(ChannelOption.SO_KEEPALIVE, true);
            ChannelFuture future = b.bind(port).sync();
            System.out.println("GP RPC Registry start listen at " + port );
            future.channel().closeFuture().sync();
```

