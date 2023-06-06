# 十、多线程

## 1.实现多线程

### 1.1多线程的实现方式1

**方式1**：继承Thread类

- 定义一个类MyThread继承Thread类
- 在MyThread类中重写run()方法
- 创建MyThread类的对象
- 启动线程

```java
public class MyThread extends Thread{
    @Override
    public void run() {
        for (int i = 0; i < 100; i++){
            System.out.println(i);
        }
    }
}
```

```java
public static void main(String[] args) {
    MyThread myThread1 = new MyThread();
    MyThread myThread2 = new MyThread();
    myThread1.start();
    myThread2.start();
}
```

两个小问题：

1.为什么要重写run()方法？

- 因为run()是用来封装被线程执行的代码

2.run()方法和start()方法的区别？

- run()：封装线程执行的代码，直接调用，相当于普通方法的调用
- start()：启动线程；然后由JVM调用此线程的run()方法

### 1.2设置和获取线程名称

Thread类中设置和获取线程名称的方法：

- void setName(String name)：将此线程的名称更改为name
- String getName()：返回此线程的名称

- 通过构造方法可以设置线程名称

如何获取当前main()方法所在的线程

- public static Thread currentThread()：返回当前正在执行的线程对象的引用

```java
// MyThread类
public class MyThread extends Thread{
    
    public MyThread() {
    }

    public MyThread(String name){
        super(name);
    }
    
    @Override
    public void run() {
        for (int i = 0; i < 100; i++){
            System.out.println(this.getName()+":"+i);
        }
    }
}
// main
public static void main(String[] args) {
        // Thread.currentThread()：返回当前正在执行的线程对象的引用
        Thread mainThread = Thread.currentThread();
        System.out.println("start:"+mainThread.getName());
        
        MyThread myThread1 = new MyThread("高铁");
        MyThread myThread2 = new MyThread();
        myThread2.setName("飞机");
        
        myThread1.start();
        myThread2.start();
    }
```

### 1.3线程调度

Thread类中设置和获取线程优先级的方法：

- public final int getPriority()：返回此线程的优先级
- public final void setPriority(int newPriority)：更改线程优先级

- 线程默认优先级是5，优先级范围是1~10
- 线程优先级高仅仅表示线程获取的CPU时间片的几率高

### 1.4线程控制

![image-20230506204852288](G:\笔记\JAVA\JAVAEE\assets\image-20230506204852288.png)



```java
public class MyThread extends Thread{
    public MyThread() {
    }

    public MyThread(String name){
        super(name);
    }
    @Override
    public void run() {
        for (int i = 0; i < 100; i++){
            System.out.println(this.getName()+":"+i);
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

public static void main(String[] args) throws InterruptedException {
    MyThread myThread1 = new MyThread("甘");
    MyThread myThread2 = new MyThread("文");
    MyThread myThread3 = new MyThread("崔");

    myThread1.start();
    myThread1.join();

    myThread2.setDaemon(true);
    myThread3.setDaemon(true);
    myThread2.start();
    myThread3.start();

    for (int i = 0; i <10000 ; i++)
        if(i%100 == 0)
            System.out.println(Thread.currentThread().getName()+":"+i);

}
```

### 1.5线程生命周期

![image-20230506211447557](G:\笔记\JAVA\JAVAEE\assets\image-20230506211447557.png)

### 1.6多线程的实现方式2

方式2：实现Runnable接口

- 定义一个类MyRunnable实现Runnable接口
- 在MyRunnable类中重写run()方法
- 创建MyRunnable类的对象
- 创建Thread类的对象，把MyRunnable对象作为构造方法的参数
- 启动线程

好处：

- 避免了JAVA单继承的局限性
- 适合多个相同程序的代码去处理同一个资源的情况，把线程和程序的代码、数据有效分离，较好的体现了面向对象的设计思想



## 2.线程同步

### 2.1案例：卖票

需求：电影院卖票，共有一百张票，三个窗口卖票。

思路：

- 定义一个类SellTicket实现Runnable接口，里面定义一个成员变量：private int ticket = 100;
- 在SellTicket类中重写run()方法实现卖票，步骤：
  - 判断票数大于0，就卖票，并告知是哪个窗口
  - 卖了票之后，总票数减1
  - 票没了，也可能有人来问，则一直死循环

- 定义一个测试类SellTicketDemo
  - 创建SellTicket类的对象
  - 创建三个Thread类对象，把SellTicket对象作为构造方法的参数，并给出窗口名称
  - 启动线程

- 注意多线程操作共享数据的安全问题
  - 把多条语句操作共享数据的代码锁起来，让任意时刻只能有一个线程执行
  - Java提供了同步代码块的方式来解决

### 2.2同步代码块

锁多条语句操作共享数据，可以使用同步代码块来实现

- 格式：

```java
synchronized(任意对象){
	多条语句操作共享数据的代码
}
```

- synchronized(任意对象)：给代码加锁，任意对象可以看成是一把锁

同步的好处与弊端：

- 好处：解决了多线程的数据安全问题
- 弊端：当线程很多时，因为每个线程都会去判断同步上的锁，这将导致资源浪费，降低程序运行效率

### 2.3案例实现（同步代码块）

```java
// 类
public class SellTicket implements Runnable{
    private int tickets = 100;
    private Object lock = new Object(); //一把锁
    
    @Override
    public void run() {
        while (true){
            synchronized (lock){
                // 有进程进入该代码块之后，就会把这一段代码锁起来
                if (tickets > 0){
                    // 买票时间
                    try {
                        Thread.sleep(10);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(
                        Thread.currentThread().getName() + "出售第" + (101 - tickets) + "张票");
                    tickets --;
                }
            }
            
        }
    }
}
// main
public static void main(String[] args) {
    SellTicket st = new SellTicket();

    Thread t1 = new Thread(st, "窗口1");
    Thread t2 = new Thread(st, "窗口2");
    Thread t3 = new Thread(st, "窗口3");

    t1.start();
    t2.start();
    t3.start();
}
```

### 2.4同步方法

同步方法：把synchronized关键字加到方法上

同步方法的锁对象：this

同步静态方法：把synchronized关键字加到静态方法上

格式：

- 修饰符 static synchronized 返回值类型 方法名(方法参数){}

同步静态方法的锁对象

- 类名.class

```java
public class SellTicket implements Runnable{
    private static int tickets = 100;
    private Object lock = new Object(); //一把锁
    private int x = 0;
    
    @Override
    public void run() {
        while (true){
            if (x %2 == 0){
                synchronized (SellTicket.class){
                    if (tickets > 0){
                        try {
                            Thread.sleep(10);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        System.out.println(
                            Thread.currentThread().getName() + "出售第" + (101 - tickets) + "张票");
                        tickets --;
                    }
                }
            } else {
                sellTicket();
            }
            
            
        }
    }

    private static synchronized void sellTicket() {
        if (tickets > 0){
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "出售第" + (101 - tickets) + "张票");
            tickets --;
        }
    }
}
```

### 2.5线程安全的类

![image-20230507170735266](G:\笔记\JAVA\JAVAEE\assets\image-20230507170735266.png)

### 2.6Lock锁

Lock实现提供了比使用synthronized方法和语句可以获得更广泛的锁定操作

Lock中提供了获得锁和释放锁的方法：

- void lock()：获得锁
- void unlock()：释放锁

Lock是接口不能直接实例化，这里采用它的实现类ReentrantLock来实例化

构造方法：

- ReentrantLock()：创建一个ReentrantLock的实例

```java
public class SellTicket implements Runnable{
    private static int tickets = 100;
    private Lock lock = new ReentrantLock();
    
    @Override
    public void run() {
        while (true){
            try {
                lock.lock();
                sellTicket();
            } finally {
                lock.unlock();
            }
        }
    }

    private void sellTicket() {
        if (tickets > 0){
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "出售第" + (101 - tickets) + "张票");
            tickets --;
        }
    }
}
```

## 3.生产者消费者模式

方法：

![image-20230507173044417](G:\笔记\JAVA\JAVAEE\assets\image-20230507173044417.png)

案例：

box类：

```java
public class Box {
    private int milk;
    private boolean state = false;

    public synchronized void setMilk(int milk) {
        //如果有牛奶，等待消费
        if (state){
            try {
                wait();
            } catch (InterruptedException e){
                e.printStackTrace();
            }
        }
        //如果没牛奶，生产牛奶
        this.milk = milk;
        System.out.println("生产者补充了第"+ milk + "瓶奶");
        
        //生成完毕后，修改状态
        state = true;
        //唤醒其他等待线程
        notifyAll();
    }
    
    public synchronized void getMilk() {
        //如果没有牛奶，等待生产
        if (!state){
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        //如果有牛奶，消费牛奶
        System.out.println("用户拿到第" + milk + "瓶奶");
        //消费完毕后，修改状态
        state = false;
        //唤醒其他等待线程
        notifyAll();
    }
}
```

Procedure

```java
public class Procedurer implements Runnable{
    private Box b;
    
    public Procedurer(Box b){
        this.b = b;
    }
    
    @Override
    public void run() {
        for (int i = 0; i <= 5; i++){
            b.setMilk(i);
        }
    }
}
```

Customer：

```java
public class Customer implements Runnable{
    private Box b;
    
    public Customer(Box b){
        this.b = b;
    }
    
    @Override
    public void run() {
        while (true){
            b.getMilk();
        }
    }
}
```

Demo：

```Java
public static void main(String[] args) {
    Box b = new Box();

    Procedurer p = new Procedurer(b);
    Customer c = new Customer(b);

    Thread t1 = new Thread(p);
    Thread t2 = new Thread(c);

    t1.start();
    t2.start();
}
```



# 十一、网络编程

## 1.网络编程概述

网络编程：

- 在网络通信协议下，实现网络互连的不同计算机程序间可以进行数据交换。

## 2.网络编程三要素

- IP地址
- 端口
- 协议

## 3.IP地址

IP地址：是网络中设备的唯一标识

IP地址分为两大类：

- IPv4：给每个连接在网络上的主机分配一个32bit的地址。按照TCP/IP规定，IP地址用二进制来表示，每个IP地址长32bit，也就是4个字节。
- IPv6：由于IP地址需求量愈来愈大，网络资源有限，使得IP地址分配紧张。为了扩大地址空间，通过IPv6重新定义地址空间，采用128位地址长度，每16个字节一组，分成8组十六进制数。

## 4.InetAddress类

​		为了方便我们对IP地址的获取和操作，Java提供了一个类InetAddress来使用

​	InetAddress：此类表示Internet协议地址

![image-20230512163119231](G:\笔记\JAVA\JAVAEE\assets\image-20230512163119231.png)

```java
public static void main(String[] args) throws UnknownHostException {
    InetAddress address = InetAddress.getByName("DESKTOP-9J1HSS0");
    String hostName = address.getHostName();
    String hostAddress = address.getHostAddress();

    System.out.println(hostName);
    System.out.println(hostAddress);
}
```

## 5.端口

端口：设备上应用程序的唯一标识

端口号：用两个字节表示的整数，取值范围是0~65535.其中，0~1023之间的端口号用于一些知名的网络服务和应用，普通的应用程序需要使用1024以上的端口号。如果端口号被另外一个服务或应用所占用，会导致当前程序启动失败。

## 6.协议

![image-20230512170424155](G:\笔记\JAVA\JAVAEE\assets\image-20230512170424155.png)

![image-20230512170603812](G:\笔记\JAVA\JAVAEE\assets\image-20230512170603812.png)

## 7.UDP通信原理

​		UDP协议是一种不可靠的网络协议，它在通信的两端各建立一个Socket对象，但是这两个Socket只是发送、接收数据的对象。因此对于基于UDP协议的通信双方而言，没有所谓的客户端和服务器的概念。

​		Java提供了DatagramSocket类作为基于UDP协议的Socket

### 7.1 UDP发送数据

发送数据的步骤：

- 创建发送端的Socket对象（DatagramSocket）
- 创建数据，把数据打包
- 调用DatagramSocket对象的方法发送数据
- 关闭发送端

```java
public static void main(String[] args) throws IOException {
    // DatagramSocket()构造数据报套接字并将其绑定到本地主机上的任何可用端口
    DatagramSocket datagramSocket = new DatagramSocket();

    //创建数据报包
    byte[] buf = "Hello World".getBytes();
    /**
     * DatagramPacket(byte[] buf, int length, InetAddress address, int port)
     * 构造一个数据包，发送长度为length的数据包到指定主机上的指定端口号
     */
    DatagramPacket p = new DatagramPacket(buf, buf.length, InetAddress.getByName("192.168.1.27"), 10086);
    
    // 调用DatagramSocket.send(DatagramPacket p)从此套接字发送数据报包
    datagramSocket.send(p);
    
    // 关闭发送端
    datagramSocket.close();
}
```

### 7.2UDP接收数据

接收数据的步骤：

- 创建接收端的Socket对象（DatagramSocket）
- 创建一个数据包，用于接收数据
- 调用DatagramSocket对象的方法接收数据
- 解析数据包，并把数据在控制台显示
- 关闭接收端

```java
public static void main(String[] args) throws IOException {
    // DatagramSocket(int port)构造数据报套接字并将其绑定到本地主机上的任何可用端口
    DatagramSocket datagramSocket = new DatagramSocket(10086);

    // DatagramPacket(byte[] buf, itn length) 创建数据报包,用于接收数据
    byte[] buf = new byte[1024];
    DatagramPacket datagramPacket = new DatagramPacket(buf, buf.length);

    // 调用DatagramSocket.receive(DatagramPacket datagramPacket)
    datagramSocket.receive(datagramPacket);
    
    // DatagramPacket.getData() 返回数据缓冲区；
    byte[] data = datagramPacket.getData();
    // 解析数据并显示在控制台
    int dataLength = datagramPacket.getLength();
    String dataString = new String(data,0,dataLength);
    System.out.println(dataString);
    
    //关闭接收端
    datagramSocket.close();
}
```

## 8.TCP通信原理

​		TCP通信协议是一种可靠的网络协议，它在通信的两端各建立一个Socket对象，从而在通信的两端形成网络虚拟链路，一旦建立了虚拟的网络链路，两端的程序就可以通过虚拟链路进行通信。

​		Java对基于TCP协议的网络提供了良好的封装，使用Socket对象来代表两端的通信端口，并通过Socket产生IO流来进行网络通信

​		Java为客户端提供了Socket类，为服务器提供了ServerSocket类

### 8.1TCP发送数据

发送数据的步骤：

- 创建客户端的Socket对象（Socket）
- 获取输出流，写数据
- 释放资源

```java
public static void main(String[] args) throws IOException {
    // 一、创建客户端的Socket对象（Socket）int port
    /**
     * 1.Socket(InetAddress address, int port) 创建流套接字并将其连接到指定IP地址的指定端口号
     * Socket s = new Socket(InetAddress.getByName("192.168.1.27"), 10086);
     * 2.Socket(String host, int port) 
     */
    Socket socket = new Socket("192.168.1.27", 10086);
    
    // 二、获取输出流，写数据
    /**
     * Socket.getOutputStream() 返回此套接字的输出流
     */
    OutputStream outputStream = socket.getOutputStream();
    outputStream.write("Hello TCP".getBytes());
    
    // 三、释放资源
    outputStream.close();
}
```

### 8.2TCP接收数据

接收数据的步骤：

- 创建服务器的Socket对象（ServerSocket）
- 获取输入流，读数据，并把数据显示在控制台上
- 释放资源

```java
public static void main(String[] args) throws IOException {
    // 一、创建服务器端的Socket对象（ServerSocket）
    /**
     * ServerSocket(int port) 创建绑定到端口的服务器套接字
     */
    ServerSocket serverSocket = new ServerSocket(10086);
    
    // 二、Socket ServerSocket.accept() 监听，连接到此套接字
    Socket socket = serverSocket.accept();

    // 三、获取输入流，读数据，并显示在控制台
    InputStream inputStream = socket.getInputStream();
    byte[] bys = new byte[1024];
    int readLen = inputStream.read(bys);
    String data = new String(bys, 0, readLen);
    System.out.println(data);

    // 三、释放资源
    // socket.close();  serverSocket.close()执行时，也会关闭socket
    serverSocket.close();
}
```



















