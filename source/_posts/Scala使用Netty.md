---
title: Scala使用Netty
date: 2017-10-08 19:03:11
tags: scala
---



java能写的换做scala也可以写，而且scala在语法上面右比java简洁很多，所以尝试用scala写一个简单的netty demo

<!-- more -->

### 服务端

ServerHandler 处理消息
```scala
class ServerHandler extends ChannelInboundHandlerAdapter{

  /**
    * 有客户端建立连接后调用
    */
  override def channelActive(ctx: ChannelHandlerContext) :Unit ={
    println("客户端建立连接后调用")
  }

  /**
    * 接受客户端发送来的消息
    */
  override def channelRead(ctx: ChannelHandlerContext, msg: scala.Any): Unit = {
    println("channelRead invoked 接受客户端发送来的消息" + msg)

    val back = "connection success"
    println("返回消息" + back)
    ctx.writeAndFlush(back)
  }
}
```
NettyServer

```scala
class NettyServer {

  def bind(host: String, port: Int): Unit = {
    //配置服务端线程池组
    //用于服务器接收客户端连接
    val bossGroup = new NioEventLoopGroup
    //用户进行SocketChannel的网络读写
    val workerGroup = new NioEventLoopGroup

    try {
      //是Netty用户启动NIO服务端的辅助启动类，降低服务端的开发复杂度
      val bootStrap = new ServerBootstrap
      //将两个NIO线程组作为参数传入到ServerBootstrap
      bootStrap.group(bossGroup, workerGroup)
        //创建NioServerSocketChannel
        .channel(classOf[NioServerSocketChannel])
        //绑定I/O事件处理类
        .childHandler(new ChannelInitializer[SocketChannel] {
        override def initChannel(ch: SocketChannel): Unit = {
          //解码器
          ch.pipeline().addLast(new StringDecoder)
          ch.pipeline().addLast(new StringEncoder)
          ch.pipeline().addLast(new ServerHandler)
        }
      })
      //绑定端口，调用sync方法等待绑定操作完成
      val channelFuture = bootStrap.bind(host, port).sync()
      println("服务已开启")
      channelFuture.channel().closeFuture().sync()

    } finally {
      bossGroup.shutdownGracefully()
      workerGroup.shutdownGracefully()
    }
  }
}

object NettyServer {
  def main(args: Array[String]): Unit = {
    val host = "localhost"
    val port = 8083
    val server = new StringNettyServer
    println(s"IP$host,端口号$port")
    server.bind(host, port)
  }
}
```

### 客户端

ClientHandler

```scala
class ClientHandler extends ChannelInboundHandlerAdapter {
  /**
    * 有客户端建立连接后调用
    */
  override def channelActive(ctx: ChannelHandlerContext): Unit = {
    println("有客户端建立连接后调用")
    val content = "你好,我是客户端"
    ctx.writeAndFlush(content)
  }


  override def channelRead(ctx: ChannelHandlerContext, msg: scala.Any): Unit = {
    println("读取服务端的消息" + msg)
  }
}

```
NettyClient

```scala
class StringNettyClient {

  def connect(host: String, port: Int): Unit = {
    //创建客户端NIO线程组
    val eventGroup = new NioEventLoopGroup
    //创建客户端辅助启动类
    val bootStrap = new Bootstrap

    try {
      bootStrap.group(eventGroup)
        //创建NioSocketChannel
        .channel(classOf[NioSocketChannel])
        //绑定I/O事件处理类
        .handler(new ChannelInitializer[SocketChannel] {
        override def initChannel(ch: SocketChannel): Unit = {
          ch.pipeline().addLast(new StringEncoder)
          ch.pipeline().addLast(new StringDecoder)
          ch.pipeline().addLast(new ClientHandler)
        }
      })
      //发起异步连接操作
      val channelFuture = bootStrap.connect(host, port).sync()
      //等待服务关闭
      channelFuture.channel().closeFuture().sync()

    } finally {
      //优雅的退出，释放线程池资源
      eventGroup.shutdownGracefully()
    }
  }
}

object StringNettyClient {
  def main(args: Array[String]): Unit = {
    val host = "localhost"
    val port = 8083
    val client = new StringNettyClient
    client.connect(host, port)
  }
}
```

### 运行
先开启服务端
```
IPlocalhost,端口号8083
服务已开启
```
再开启客户端
```
有客户端建立连接后调用
```

服务端接收并反馈
```
有客户端建立连接后调用
接受客户端发送来的消息你好,我是客户端
返回消息connection success
```
客户端接收服务端结果
```
读取服务端的消息connection success
```

### 总结
上述通讯使用了`StringEncoder`编码器和`StringDecoder`解码器,所以调用`ctx.writeAndFlush()`方法的时候可以传入字符串，自动处理。
