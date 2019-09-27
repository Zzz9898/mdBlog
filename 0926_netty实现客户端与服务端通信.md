```
title：netty实现客户端与服务端通信
tags：netty,client,server
grammar_zjwJava：true
```

- 引入依赖

  - ```
    <dependency>
        <groupId>io.netty</groupId>
        <artifactId>netty-all</artifactId>
        <version>4.1.33.Final</version>
    </dependency>
    ```

- 创建服务端

  - ```java
    public class TestServer {
        public static void main(String[] args) throws InterruptedException {
            EventLoopGroup worker = new NioEventLoopGroup();
            EventLoopGroup boss = new NioEventLoopGroup();
            try{
                ServerBootstrap bootstrap = new ServerBootstrap();
                bootstrap.group(boss,worker)
                        .channel(NioServerSocketChannel.class)
                        .localAddress("127.0.0.1",8000)
                        .childHandler(new ChannelInitializer<SocketChannel>() {
                            @Override
                            protected void initChannel(SocketChannel ch) throws Exception {
                                ch.pipeline().addLast("decoder", new StringDecoder(CharsetUtil.UTF_8));
                                ch.pipeline().addLast("encoder", new StringEncoder(CharsetUtil.UTF_8));
                                ch.pipeline().addLast(new TestServerHandler());
                            }
                        });
                ChannelFuture sync = bootstrap.bind("127.0.0.1", 8899).sync();
                sync.channel().closeFuture().sync();
            }finally {
                worker.shutdownGracefully();
                boss.shutdownGracefully();
            }
        }
    }
    ```

    EventLoopGroup是用来处理IO操作的多线程事件循环器，两个实例中worker负责处理客户端i/o事件、task任务、监听任务组，boss负责接收客户端连接线程。

    ServerBootstrap实例时启动NIO服务的辅助启动类，

    - .group();传入boss和worker，

    - .channel();传入NioServerSocketChannel.class指明为NIO，

    - .localAddress("127.0.0.1",8000)绑定服务器地址和端口，

    - .childHandler();添加一个ChannelInitializer<SocketChannel>初始化器，重写initChannel();方法，其中可以添加指定的Handler，由于这里是传递字符串消息，添加了字符串的编码和解码处理器和自定义的消息处理器。

      注：如果传递字符串消息，不添加字符串编解码器，那么消息是传不到客户端的。

    辅助启动类调用.bind();方法绑定地址和端口，最后调用sync.channel().closeFuture().sync();和关闭boss和worker。

  - 自定义服务端Handler

    - ```java
      public class TestServerHandler extends ChannelInboundHandlerAdapter {
      
          @Override
          public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
              System.out.println("客户端加入...");
          }
      
          @Override
          public void channelActive(ChannelHandlerContext ctx) throws Exception {
              System.out.println("激活客户端：" + ctx.channel().remoteAddress());
          }
      
          @Override
          public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
              System.out.println(msg);
              ctx.channel().writeAndFlush("服务端收到消息：" + msg);
          }
      
          @Override
          public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
              cause.printStackTrace();
              ctx.close();
          }
      }
      ```

    - 继承ChannelInboundHandlerAdapter，实现handlerAdded();channelActive();channelRead();exceptionCaught();四个方法，

      - handlerAdded();在有客户端连接加入的时候调用；
      - channelActive();在激活客户端时调用；
      - channelRead();在收到客户端消息时调用；
      - exceptionCaught();在出现异常时调用。

- 创建客户端

  - ```java
    public class TestClient {
        public static void main(String[] args) throws InterruptedException {
            EventLoopGroup worker = new NioEventLoopGroup();
            try{
                Bootstrap bootstrap = new Bootstrap();
                bootstrap.group(worker)
                        .channel(NioSocketChannel.class)
                        .handler(new ChannelInitializer<SocketChannel>() {
                            @Override
                            protected void initChannel(SocketChannel ch) throws Exception {
                                ch.pipeline().addLast("decoder", new StringDecoder(CharsetUtil.UTF_8));
                                ch.pipeline().addLast("encoder", new StringEncoder(CharsetUtil.UTF_8));
                                ch.pipeline().addLast(new TestClientHandler());
                            }
                        });
                ChannelFuture sync = bootstrap.connect("127.0.0.1", 8899).sync();
                sync.channel().writeAndFlush("连接...");
                sync.channel().closeFuture().sync();
            }finally {
                worker.shutdownGracefully();
            }
        }
    }
    ```

    这里只需要一个NioEventLoopGroup实例worker。

    创建辅助类Bootstrap实例。

    - .group();传入worker，

    - .channel();传入NioSocketChannel.class指明为NIO，注意这里的与服务端的不一致，

    - .childHandler();添加一个ChannelInitializer<SocketChannel>初始化器，重写initChannel();方法，其中可以添加指定的Handler，由于这里是传递字符串消息，添加了字符串的编码和解码处理器和自定义的消息处理器，与服务端类似。

      注：如果传递字符串消息，不添加字符串编解码器，那么消息是传不到服务端的。

      辅助启动类调用.connect();方法连接地址和端口，注意这里时连接不是bind();,然后调用sync.channel().writeAndFlush("连接...");方法，给服务端发送一条字符串消息，最后调用sync.channel().closeFuture().sync();和关闭worker。

  - 自定义客户端Handler

    - ```java
      public class TestClientHandler extends ChannelInboundHandlerAdapter {
          @Override
          public void channelActive(ChannelHandlerContext ctx) throws Exception {
              System.out.println("客户端激活...");
          }
      
          @Override
          public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
              System.out.println("收到服务端消息：" + msg);
          }
      
          @Override
          public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
              cause.printStackTrace();
              ctx.close();
          }
      }
      ```

    - 继承ChannelInboundHandlerAdapter，实现channelActive();channelRead();exceptionCaught();四个方法，

      - channelActive();在客户端被激活时调用；
      - channelRead();在收到服务端消息时调用；
      - exceptionCaught();在出现异常时调用。

- 启动

  - 先启动服务端，再启动客户端

  - 服务端打印

    - ```
      客户端加入...
      激活客户端：/127.0.0.1:53311
      ...
      连接...
      ```

  - 服务端打印

    - ```
      客户端激活...
      ...
      收到服务端消息：服务端收到消息：连接...
      ```

  如果你想客户端一直给服务端发消息，服务端收到消息再返回，那就可以修改客户端：

  ```java
  public class TestClient {
      public static void main(String[] args) throws InterruptedException {
          EventLoopGroup worker = new NioEventLoopGroup();
          try{
              Bootstrap bootstrap = new Bootstrap();
              bootstrap.group(worker)
                      .channel(NioSocketChannel.class)
                      .handler(new ChannelInitializer<SocketChannel>() {
                          @Override
                          protected void initChannel(SocketChannel ch) throws Exception {
                              ch.pipeline().addLast("decoder", new StringDecoder(CharsetUtil.UTF_8));
                              ch.pipeline().addLast("encoder", new StringEncoder(CharsetUtil.UTF_8));
                              ch.pipeline().addLast(new TestClientHandler());
                          }
                      });
              ChannelFuture sync = bootstrap.connect("127.0.0.1", 8899).sync();
  //            sync.channel().writeAndFlush("连接...");
              sendMsg(sync.channel());
              sync.channel().closeFuture().sync();
          }finally {
              worker.shutdownGracefully();
          }
      }
  
      private static void sendMsg(Channel channel) {
          Scanner scanner = new Scanner(System.in);
          String str = scanner.nextLine();
          while (!str.equals("q")){
              channel.writeAndFlush(str);
              str = scanner.nextLine();
          }
      }
  }
  ```

  这样就可以一直给服务端发消息了。