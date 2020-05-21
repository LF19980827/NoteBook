# 1、Java NIO简介

​	Java NIO (New IO)是从Java 1.4版本开始引入的一个新的IO API，可以替代标准的Java IO API。
​	NIO与原来的IO有同样的作用和目的，但是使用的方式完全不同，NIO支 持面向缓冲区的、基于通道的IO操作。NIO将以更加高效的方式进行文件的读写操作。

| IO     | NIO        |
| ------ | ---------- |
| 面向流 | 面向缓冲区 |
| 阻塞IO | 非阻塞IO   |
| 无     | 选择器     |



# 2、缓冲区(Buffer)和通道(Channel)

​	Java NIO系统的核心在于:通道(Channel)和缓冲区(Buffer)。 通道表示打开到IO设备的连接。若需要使用NIO系统，需要获取用于连接IO设备的通道以及用于容纳数据的缓冲区。然后操作缓冲区，对数据进行处理。

简而言之，**Channel（通道）负责传输,Buffer（缓冲区）负责存储**

## 2.1 缓冲区(Buffer)

### 1.缓冲区(Buffer)

​	在Java NIO中负责数据的存取。缓冲区就是数组。用于存储不同数据类型的数据

```java
//通过allocate获取缓冲区,分配一个指定大小的缓冲区
ByteBuffer buf = ByteBuffer.allocate(1024);
```

### 2.缓冲区的数据操作

- put()：存入数据到缓冲区
  - put(byteb)：将给定单个字节写入缓冲区的当前位置
  - put(byte[] src)：将src中的字节写入缓冲区的当前位置
  - put(int index, byte b)：将指定字节写入缓冲区的索引位置(不会移动position)
- get()：获取缓冲区中的数据
  - get()：读取单个字节
  - get(byte[] dst)：批量读取多个字节到dst中
  - get(int index)：读取指定索引位置的字节(不会移动position)

### 3.缓冲区的四个核心属性

- 容量(capacity)：（最大存储容量）表示Buffer最大数据容量，缓冲区容量不能为负，并且创建后不能更改

- 限制(limit)：（可以操作数据的大小）第一个不应该读取或写入的数据的索引，即位于limit后的数据不可读写。缓冲区的限制不能为负，并且不能大于其容量

- 位置(position)：（正在操作数据的位置）下一个要读取或写入的数据的索引。缓冲区的位置不能为负，并且不能大于其限制

- 标记(mark)与重置(reset)：标记是一个索引，通过Buffer中的mark()方法指定Buffer中一个特定的position,之后可以通过调用reset()方法恢复到这个position

  标记、位置、限制、容量遵守以下不变式: 

$$
0 <= mark <= position <= limit <= capacity
$$

### 4.Buffer的常用方法

| 方法                   | 描述                                                  |
| ---------------------- | ----------------------------------------------------- |
| Buffer clear ()        | 清空缓冲区并返回对缓冲区的引用                        |
| Buffer flip()          | 将缓冲区的界限设置为当前位置，并将当前位置充值为0     |
| int capacity()         | 返回Buffer的capacity 大小                             |
| boolean hasRemaining() | 判断缓冲区中是否还有元素                              |
| int limit ()           | 返回Buffer 的界限(limit)的位置                        |
| Buffer limit(int n)    | 将设置缓冲区界限为n,并返回一个具有新limit的缓冲区对象 |
| Buffer mark()          | 对缓冲区设置标记                                      |
| int position()         | 返回缓冲区的当前位置position                          |
| Buffer position(int n) | 将设置缓冲区的当前位置为n，并返回修改后的Buffer 对象  |
| int remaining()        | 返回position 和limit 之间的元素个数                   |
| Buffer reset()         | 将位置position 转到以前设置的mark 所在的位置          |
| Buffer rewind ()       | 将位置设为为0，取消设置 的mark                        |

### 5.直接缓冲区与非直接缓冲区

（1）非直接缓冲区：通过allocate()方法分配缓冲区,将缓冲区建立在JVM的内存中

![](G:\个人数据\笔记\NoteBook\NIO\img\2.1.png)



（2）直接缓冲区：通过allocateDirect()方法分配直接缓冲区,将缓冲区建立在物理内存中。可以提高效率

![](G:\个人数据\笔记\NoteBook\NIO\img\2.2.png)

# 3、文件通道(FileChannel)

​	通道(Channel) :用于源节点与目标节点的连接。在Java NIO中负责缓冲区中数据的传输。本身就是处理器，和DMA类似，不需要请求CPU

## 3.1 实现了 Channel接口的实现类

- FileChannel：用于读取、写入、映射和操作文件的通道。
- DatagramChannel：通过UDP 读写网络中的数据通道。
- SocketChannel：通过TCP读写网络中的数据。
- ServerSocketChannel：可以监听新进来的TCP连接，对每一个新进来的连接都会创建一个Socke tChannel。

## 3.2 获取通道的方法

- getChannel方法，支持这个方法的类有：
  - （本地IO）FileInputStream、FileOutputStream、RandomAccessFile
  - （网络IO）DatagramSocket、Socket、ServerSocket
- 在JDK1.7中的NI0.2针对各个通道提供了静态方法open()
- 在JDK 1.7中的NIO.2的Files工具类的newByteChanne1()

## 3.3 通道的数据传输

### 1.通道之间的数据传输（直接缓冲区）[重要]

```java
@Test
public void test() throws IOException{
	FileChannel inChannel = FileChannel.open(Paths.get("1.mkv"),StandardOpenOption.READ);
	FileChannel outChannel = FileChannel.open(Paths.get("2.mkv"),StandardOpenOption.WRITE,StandardOpenOption.READ,StandardOpenOption.CREATE);
	//inChannel.transferTo(0,inChannel.size(),outChannel);与下作用相同
	outChannel.transferFrom(inChannel,0,inChannel.size());
	
    inChannel.close();
	outChannel.close();
}
```

### 2.使用内存映射文件（直接缓冲区）

```java
@Test
public void test() throws IOException{
	FileChannel inChannel = FileChannel.open(Paths.get("1.mkv"),StandardOpenOption.READ);
	FileChannel outChannel = FileChannel.open(Paths.get("2.mkv"),StandardOpenOption.WRITE,StandardOpenOption.READ,StandardOpenOption.CREATE);
	
    //内存映射文件
    MappedByteBuffer inMappedBuf = inChannel.map(MapMode.READ_ONLY,0,inChannel.size());
	MappedByteBuffer outMappedBuf = outChannel.map(MapMode.READ_WRITE,0,inChannel.size());

    //直接对缓冲区进行数据的读写操作
    byte[] dst = new byte [inMappedBuf.limit()];
	inMappedBuf.get(dst);
	outMappedBuf.put(dst);


    inChannel.close();
	outChannel.close();
}
```

### 3.利用通道完成文件的复制（非缓冲区）

```java
@Test
public void test() {
	FileInputStream fis = nu1l;
	File0utputStream fos = nu1l;
	//获取通道
	FileChannel inChannel = nu1l;
	FileChannel outChannel = nu11;
    
	try {
		fis = new FileInputStream("1.jpg");
		fos = new FileOutputStream("2.jpg");
		inChannel = fis.getChannel();
		outChannel = fos.getChannel();
        
		//分配指定大小的缓冲区
		ByteBuffer buf = ByteBuffer.allocate(1024);
        
		//将通道中的数据存入缓冲区中
		while(inChannel.read(buf) != -1){
			buf.flip(); //切换读取数据的模式
			//将缓冲区中的数据写入通道中
			outChannel.write(buf);
			buf.clear(); //清空缓冲区
		}
	} catch (IOException e) {
        e.printStackTrace();
    }finally{
        if(outChannel != nu1l){
			try {
				outChannel.close();
			} catch (IOException e) {
				e. printStackTrace();
			}
        }     
		
         if(inChannel != nu1l){
			try {
				inChannel.close();
			} catch (IOException e) {
				e. printStackTrace();
			}
        }  
        
         if(fos != nu1l){
			try {
				fos.close();
			} catch (IOException e) {
				e. printStackTrace();
			}
        }  
        
         if(fis != nu1l){
			try {
				fis.close();
			} catch (IOException e) {
				e. printStackTrace();
			}
        }  
    }

}
```

## 3.4 分散(Scatter)和聚集(Gather)

- 分散读取(Scattering Reads)：将通道中的数据分散到多个缓冲区中
- 聚集写入(Gathering Writes)：将多个缓冲区中的数据聚集到通道中

## 3.5 编码与解码

```java
@Test
public void test() throws IOException{
	Charset cs1 = Charset.forName ("GBK");
	//获取编码器
	CharsetEncoder ce = cs1.newEncoder();
	//获取解码器
	CharsetDecoder cd = cs1.newDecoder();

    CharBuffer cBuf = CharBuffer.allocate(1024);
	cBuf.put("测试");
	cBuf.f1ip();

    //编码
	ByteBuffer bBuf = ce.encode(cBuf);
	for(int i=0;i<12;i++){
		System.out.println(bBuf.get());
    }
    
    //解码
	bBuf.flip();
	CharBuffer cBuf2 = cd.decode(bBuf);
	System.out.println(cBuf2.toString());

    
}
```

## 3.6 **FileChannel** 的常用方法

| 方法                           | 描述                                         |
| ------------------------------ | -------------------------------------------- |
| int read (ByteBuffer dst)      | 从Channel中读取数据到ByteBuffer              |
| long read (ByteBuffer[] dsts)  | 将Channel中的数据“分散”到ByteBuffer []       |
| int write (ByteBuffer src)     | 将ByteBuffer中的数据写入到Channel            |
| long write (ByteBuffer[] srcs) | 将ByteBuffer[] 中的数据“聚集”到Channel       |
| long position()                | 返回此通道的文件位置                         |
| FileChannel position(long p)   | 设置此通道的文件位置                         |
| long size()                    | 返回此通道的文件的当前大小                   |
| FileChannel truncate(long s)   | 将此通道的文件截取为给定大小                 |
| void force (boolean metaData)  | 强制将所有对此通道的文件更新写入到存储设备中 |



# 4、NIO的非阻塞式网络通信

使用NIO完成网络通信的三个核心：

- 通道（Channel）：负责连接（在java.nio.channels.Channel接口-->抽象实现类SelectableChannel的具体常用子类有：）
  - SocketChannel
  - ServerSocketChannel
  - DatagramChannel
  - Pipe.SinkChannel
  - Pipe.SourseChannel
- 缓冲区（Buffer）：负责数据的存取
- 选择器（Selector）：是SelectableChannel的多路复用器



## 4.1 阻塞与非阻塞

- 传统的 IO 流都是阻塞式的。也就是说，当一个线程调用 read() 或 write()  

  时，该线程被阻塞，直到有一些数据被读取或写入，该线程在此期间不 

  能执行其他任务。（传统的解决方式：多线程）

- Java NIO 是非阻塞模式的。当线程从某通道进行读写数据时，若没有数 

  据可用时，该线程可以进行其他任务。线程通常将非阻塞 IO 的空闲时 
  
  间用于在其他通道上执行 IO 操作，所以单独的线程可以管理多个输入 
  
  和输出通道。



## 4.2选择器(Selector)

​	选择器（Selector） 是 SelectableChannle 对象的多路复用器，Selector 可以同时监控多个 SelectableChannel 的 IO 状况，也就是说，利用 Selector可使一个单独的线程管理多个 Channel。Selector 是非阻塞 IO 的核心。

![](G:\个人数据\笔记\NoteBook\NIO\img\4.1.png)

​	当调用 register(Selector sel, int ops) 将通道注册选择器时，选择器对通道的监听事件，需要通过第二个参数 ops 指定。 可以监听的事件类型（可使用SelectionKey的四个常量表示），若注册时不止监听一个事件，则可以使用“位或”操作符连接。

- 读：SelectionKey.OP_ READ 

- 写：SelectionKey.OP_ WRITE

- 连接：SelectionKey.OP_ CONNECT

- 接收：SelectionKey.OP_ ACCEPT

  

   SelectionKey：表示 SelectableChannel 和 Selector 之间的注册关系。每次向选择器注册通道时就会选择一个事件(选择键)。选择键包含两个表示为整数值的操作集。操作集的每一位都表示该键的通道所支持的一类可选择操作。

| 方法                         | 描述                             |
| ---------------------------- | -------------------------------- |
| int interestOps ()           | 获取感兴趣事件集合               |
| int readyOps ()              | 获取通道已经准备就绪的操作的集合 |
| SelectableChannel channel () | 获取注册通道                     |
| Selector selector ()         | 返回选择器                       |
| boolean isReadable()         | 检测Channal中读事件是否就绪      |
| boolean isWritable()         | 检测Channal中写事件是否就绪      |
| boolean isConnectable()      | 检测Channel中连接是否就绪        |
| boolean isAcceptable()       | 检测Channel 中接收是否就绪       |



选择器Selector 的常用方法

| 方法                      | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| Set<SelectionKey> keys () | 所有的SelectionKey 集合。代表注册在该Selector上的Channel     |
| selectedKeys ()           | 被选择的SelectionKey 集合。返回此Se lector的已选择键集       |
| int select ()             | 监控所有注册的Channel,当它们中间有需要处理的IO操作时，该方法返回，并将对应得的SelectionKey 加入被选择的SelectionKey集合中，该方法返回这些Channel的数量。 |
| int select (long timeout) | 可以设置超时时长的select() 操作                              |
| int selectNow()           | 执行一个立即返回的select() 操作，该方法不会阻塞线程          |
| Selector wakeup()         | 使一个还未返回的select() 方法立即返回                        |
| void close ()             | 关闭该选择器                                                 |



## 4.3 非阻塞通信实例（简易聊天）

1.服务端

```java
	//服务端
    @Test
    public void server() throws IOException{
        //1.获取通道
        ServerSocketChannel ssChannel=ServerSocketChannel.open();

        //2.切换非阻塞式模式
        ssChannel.configureBlocking(false);

        //3.绑定连接
        ssChannel.bind(new InetSocketAddress(9898));

        //4.获取选择器
        Selector selector=Selector.open();

        //5.将通道注册到选择器上，并且指定“监听接收事件”
        ssChannel.register(selector,SelectionKey.OP_ACCEPT);

        //6.轮询式的获取选择器上已经“准备就绪”的事件
        while(selector.select()>0){

            //7.获取当前选择器中所有注册的“选择键（已就绪的监听事件）”
            Iterator<SelectionKey> it=selector.selectedKeys().iterator();

            while(it.hasNext()){
                //8.获取准备“就绪”的事件
                SelectionKey sk=it.next();

                //9.判断具体是什么时间准备就绪
                if(sk.isAcceptable()){
                    //10.若“接收就绪”，获取客户端连接
                    SocketChannel sChannel=ssChannel.accept();

                    //11.切换非阻塞模式
                    sChannel.configureBlocking(false);

                    //12.将该通道注册到选择器上
                    sChannel.register(selector, SelectionKey.OP_READ);
                }else if(sk.isReadable()){
                    //13.获取当前选择器上“读就绪”状态的通道
                    SocketChannel sChannel=(SocketChannel)sk.channel();
                    //14.读取数据
                    ByteBuffer buf=ByteBuffer.allocate(1024);
                    int len=0;
                    while((len=sChannel.read(buf))>0){
                        buf.flip();
                        System.out.println(new String(buf.array(),0,len));
                        buf.clear();
                    }
                }
                //15.取消选择键SelectionKey
                it.remove();
            }
        }
    }
```

2.客户端

```java
	 //客户端
    @Test
    public void client()throws IOException{
        //1.获取通道
        SocketChannel sChannel=SocketChannel.open(new InetSocketAddress("127.0.0.1", 9898));
        //2.切换非阻塞模式
        sChannel.configureBlocking(false);
        //3.分配指定大小的缓冲区
        ByteBuffer buf=ByteBuffer.allocate(1024);
        //4.发送数据给服务端
        Scanner scan=new Scanner(System.in);
        while(scan.hasNext()){
            String str=scan.next();
            buf.put((new Date().toString()+"\n"+str).getBytes());
            buf.flip();
            sChannel.write(buf);
            buf.clear();
        }
        //5.关闭通道
        sChannel.close();
    }
```



# 5、管道

​	Java NIO 管道是2个线程之间的单向数据连接。Pipe有一个source通道和一个sink通道。数据会被写到sink通道，从source通道读取。

![](G:\个人数据\笔记\NoteBook\NIO\img\5.1.png)