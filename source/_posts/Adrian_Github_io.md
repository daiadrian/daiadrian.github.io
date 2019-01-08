---
title: welcome baby!
date: 2019-01-08 11:00:20
tags: [netty,nio]
---

````java
public class NettyTest {

    public static void server() {
        /*
         * Netty内部都是通过线程在处理各种数据
         * EventLoopGroup就是用来管理调度他们的，注册Channel，管理他们的生命周期。
         * NioEventLoopGroup 是用来处理I/O操作的多线程事件循环器，
         * Netty提供了许多不同的EventLoopGroup的实现用来处理不同传输协议。
         * 有2个NioEventLoopGroup会被使用。
         * 第一个经常被叫做‘boss’，用来接收进来的连接。
         * 第二个经常被叫做‘worker’，用来处理已经被接收的连接，
         * 一旦‘boss’接收到连接，就会把连接信息注册到‘worker’上。
         * 如何知道多少个线程已经被使用，如何映射到已经创建的Channels上都需要依赖于EventLoopGroup的实现，
         * 并且可以通过构造函数来配置他们的关系。
        */
        //因为bossGroup仅接收客户端连接，不做复杂的逻辑处理，为了尽可能减少资源的占用，取值越小越好
        /**
         * 与 bossGroup 相关联的 EventLoopGroup 将分配一个负责为传入连接请求创建Channel 的 EventLoop。
         * 一旦连接被接受，workerGroup 就会给它的 Channel 分配一个 EventLoop
         */
        NioEventLoopGroup bossGroup = new NioEventLoopGroup(1);
        NioEventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            /*
             * ServerBootstrap 是一个启动NIO服务的辅助启动类
             * 你可以在这个服务中直接使用Channel
             */
            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap.group(bossGroup, workerGroup)
                    /*
                     * ServerSocketChannel以NIO的selector为基础进行实现的
                     * 指定使用NioServerSocketChannel产生一个Channel用来接收连接
                     */
                    .channel(NioServerSocketChannel.class)
                    .option(ChannelOption.SO_BACKLOG, 100) // 连接数
                     /*
                      * option()是提供给NioServerSocketChannel用来接收进来的连接。
                      * childOption()是提供给由父管道ServerChannel接收到的连接
                      */
                    .option(ChannelOption.TCP_NODELAY, true) // 不延迟，消息立即发送
                    /*
                     * 是否启用心跳保活机制。在双方TCP套接字建立连接后（即都进入ESTABLISHED状态）
                     * 并且在两个小时左右上层没有任何数据传输的情况下，这套机制才会被激活
                     */
                    .childOption(ChannelOption.SO_KEEPALIVE, true) // 长连接
                    .handler(new LoggingHandler(LogLevel.INFO))
                     /*
                      * 这里的事件处理类经常会被用来处理一个最近的已经接收的Channel。
                      * ChannelInitializer是一个特殊的处理类，
                      * 他的目的是帮助使用者配置一个新的Channel。
                      * 也许你想通过增加一些处理类比如NettyServerHandler来配置一个新的Channel
                      * 或者其对应的ChannelPipeline来实现你的网络程序。
                      * 当你的程序变的复杂时，可能你会增加更多的处理类到pipline上，
                      * 然后提取这些匿名类到最顶层的类上。
                      */
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            ChannelPipeline pipeline = socketChannel.pipeline();
                            // 添加NettyServerHandler，用来处理Server端接收和处理消息的逻辑
                            // ChannelPipeline用于存放管理ChannelHandel
                            // ChannelHandler用于处理请求响应的业务逻辑相关代码
                            pipeline.addLast(new NettyServerHandler());
                        }
                    });
            /*
             * 绑定端口并启动去接收进来的连接
             * 对 sync()方法的调用将导致当前 Thread 阻塞，一直到绑定操作完成为止
             */
            ChannelFuture sync = serverBootstrap.bind(8081).sync();
            /*
             * 这里会一直等待，直到socket被关闭或者被唤醒
             *
             * sync()会同步等待连接操作结果
             * 用户线程将在此wait()，直到连接操作完成之后，线程被notify(),用户代码继续执行
             * closeFuture()当Channel关闭时返回一个ChannelFuture,用于链路检测
             */
            sync.channel().closeFuture().sync();

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }

    public static void client() {
        /*
         * 如果你只指定了一个EventLoopGroup，
         * 那他就会即作为一个‘boss’线程，
         * 也会作为一个‘workder’线程，
         * 尽管客户端不需要使用到‘boss’线程。
         */
        NioEventLoopGroup group = new NioEventLoopGroup();
        try {
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(group)
                    .channel(NioSocketChannel.class)
                    /**
                     * 不像在使用ServerBootstrap时需要用childOption()方法，
                     * 因为客户端的SocketChannel没有父channel的概念。
                     */
                    .option(ChannelOption.TCP_NODELAY, true) // 不延迟，消息立即发送
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            ChannelPipeline pipeline = socketChannel.pipeline();
                            pipeline.addLast(new NettyClientHandler());
                        }
                    });
            ChannelFuture sync = bootstrap.connect(InetAddress.getLocalHost(),8081).sync();
            if (sync.isSuccess()) {
                System.err.println("连接服务器成功");
            }
            sync.channel().closeFuture().sync();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            group.shutdownGracefully();
        }
    }

}

````

