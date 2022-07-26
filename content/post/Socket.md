---
title: "JAVA - Socket"
date: 2022-07-26T22:10:11+08:00
categories:
    - Java
tags:
    - Web
    - Socket
draft: false
---

## TCP与UDP
&emsp;&emsp;java.net包提供了两种常见的网络协议支持：
* TCP：TCP是一种面向连接的、可靠的、基于字节流的传输层通信协议，可以保障两个应用程序间的可靠通信，连接会进行三次握手，断开也会进行四次挥手。
* UDP：UDP是一种无连接协议，不会建立可靠传输，也就是说传输过程中有可能会导致部分数据丢失，但是它比TCP传输更加简单高效，应用程序通常必须容许一些丢失、错误或重复的数据包。
  
![图片](https://s1.328888.xyz/2022/07/26/DIpCJ.png)

## Socket技术
&emsp;&emsp;通过Socket技术，我们就可以实现两台计算机之间的通信，Socket也被翻译为套接字，是操作系统底层提供的一项通信技术，它支持TCP和UDP两种协议。\
&emsp;&emsp;Java就对socket底层支持进行了一套完整的封装，我们可以通过Java来实现Socket通信。java.net.Socket 类代表一个套接字，并且 java.net.ServerSocket 类为服务器程序提供了一种来监听客户端，并与他们建立连接的机制。\
&emsp;&emsp;在建立TCP连接时，会有如下步骤：
* 服务端实例化一个ServerSocket对象，并调用accept()方法，该方法会一直等待到客户端连接到服务端所给定的端口。
* 客户端实例化一个Socket对象，指定服务端的名字与端口号来请求连接。
* 服务端上的accept()会返回新的socket引用，该socket连接到客户端的socket。
  
&emsp;&emsp;在连接建立以后，Socket通过I/O流进行通信，客户端的输入流连接到服务端的输出流，客户端的输出流连接到服务端的输入流。

## Socket服务端实例
```java
public static void main(String[] args) {
    try(ServerSocket server = new ServerSocket(8080))   //将服务端创建在端口8080
    {   
        System.out.println("正在等待客户端连接...");
        Socket socket = server.accept();
        System.out.println("客户端已连接，IP地址为："+socket.getInetAddress().getHostAddress());
    }catch (IOException e)
    {
        e.printStackTrace();
    }
}
```

## Socket客户端实例
```java
public static void main(String[] args) {
    try (Socket socket = new Socket("localhost", 8080))
    {
        System.out.println("已连接到服务端！");
    }catch (IOException e)
    {
        System.out.println("服务端连接失败！");
        e.printStackTrace();
    }
}
```

## 实例后的测试
&emsp;&emsp;先运行服务端：
```
正在等待客户端连接...
```
&emsp;&emsp;运行客户端进行连接：
```
已连接到服务端！
```
&emsp;&emsp;连接后的服务端：
```
正在等待客户端连接...
客户端已连接，IP地址为：127.0.0.1
```

## 利用Socket进行数据传输
&emsp;&emsp;利用Socket与I/O流进行数据传输：
```java
public static void main(String[] args) {
    try(ServerSocket server = new ServerSocket(8080))   //将服务端创建在端口8080上
    {    
        System.out.println("正在等待客户端连接...");
        Socket socket = server.accept();
        System.out.println("客户端已连接，IP地址为："+socket.getInetAddress().getHostAddress());
        BufferedReader reader = new BufferedReader(new InputStreamReader(socket.getInputStream()));  //通过
        System.out.print("接收到客户端数据：");
        System.out.println(reader.readLine());
      	socket.close();   //关闭socket
    }catch (IOException e)
    {
        e.printStackTrace();
    }
}
```

```java
public static void main(String[] args) {
    try (Socket socket = new Socket("localhost", 8080);)
    {
        Scanner scanner = new Scanner(System.in)
        System.out.println("已连接到服务端！");
        OutputStream stream = socket.getOutputStream();
        OutputStreamWriter writer = new OutputStreamWriter(stream);  //通过转换流来帮助快速写入内容
        System.out.println("请输入要发送给服务端的内容：");
        String text = scanner.nextLine();
        writer.write(text+'\n');   //因为是readLine()这里加个换行符
        writer.flush();
        System.out.println("数据已发送："+text);
    }catch (IOException e)
    {
        System.out.println("服务端连接失败！");
        e.printStackTrace();
    }finally
    {
        System.out.println("客户端断开连接！");
    }
}
```
\
&emsp;&emsp;服务端与客户端的单向流是可以关闭的，这里用到socket的`shutdownOutput()`与`shutdownInput()`方法：
```java
socket.shutdownOutput();  //关闭输出方向的流
socket.shutdownInput();  //关闭输入方向的流
```
&emsp;&emsp;如果我们不希望服务端等待太长时间，可以通过调用`setSoTimeout()`方法来设定I/O的超时时间：
```java
socket.setSoTimeout(3000);  //设置3秒的超时时间
```

&emsp;&emsp;如果超过设定时间都依然没有数据，会抛出异常：

```
java.net.SocketTimeoutException: Read timed out
	at java.net.SocketInputStream.socketRead0(Native Method)
	at java.net.SocketInputStream.socketRead(SocketInputStream.java:116)
	at java.net.SocketInputStream.read(SocketInputStream.java:171)
	at java.net.SocketInputStream.read(SocketInputStream.java:141)
	at sun.nio.cs.StreamDecoder.readBytes(StreamDecoder.java:284)
	at sun.nio.cs.StreamDecoder.implRead(StreamDecoder.java:326)
	at sun.nio.cs.StreamDecoder.read(StreamDecoder.java:178)
	at java.io.InputStreamReader.read(InputStreamReader.java:184)
	at java.io.BufferedReader.fill(BufferedReader.java:161)
	at java.io.BufferedReader.readLine(BufferedReader.java:324)
	at java.io.BufferedReader.readLine(BufferedReader.java:389)
	at com.test.Main.main(Main.java:41)
```

&emsp;&emsp;如果连接的双方发生意外而通知不到对方，导致一方还持有连接，这样就会占用资源，我们可以使用`setKeepAlive()`方法来防止此类情况发生：
```java
socket.setKeepAlive(true);
```
&emsp;&emsp;当客户端连接后，如果设置为 true，当对方没有发送任何数据过来，超过一个时间后会发送一个ack探测包发到对方，探测双方的TCP/IP连接是否有效。\
\
&emsp;&emsp;TCP在传输过程中，实际上会有一个缓冲区用于数据的发送和接收：\
![图片](https://s1.328888.xyz/2022/07/26/DVhaP.jpg)

&emsp;&emsp;此缓冲区大小为8192，可以手动调整其大小来优化传输效率：
```java
socket.setReceiveBufferSize(25565);   //TCP接收缓冲区
socket.setSendBufferSize(25565);    //TCP发送缓冲区
```