# Getting Started

### ChannelHandler

* 每个事件都可以被分发给 ChannelHandler 类中的某个用户实现的方法

### ChannelInboundHandler

* 提供里处理事件的接口方法
* `ChannelInboundHandlerAdapter`是ChannelInboundHandler适配器实现类
* `channelRead(ChannelHandlerContext, Object)`每当接收到消息时被调用
* `exceptionCaught(ChannelHandlerContext ctx, Throwable cause)`当出现Throwable对象时被调用

### EventLoop

* Selector会为每个Channel分配一个EventLoop，用于处理事件
* 由一个线程驱动，处理一个Channel的所有I/O事件

### EventLoopGroup

* 处理I/O操作的多线程循环器
* 一个服务端的应用，因此会有2个 NioEventLoopGroup 会被使用
  * boss，用来接收进来的连接
  * worker，用来处理已经被接收的连接，一旦boss接收到连接，就会把连接信息注册到worker上

### ServerBootstrap

* 启动NIO的辅助类
* 配置EventLoopGroup
* 配置Channel
* 配置自定义ChannelHandler

### ChannelFuture

* 提供在操作完成时通知应用程序的方式，一种回调机制
* `ChannelFutureListener.operationComplete(ChannelFuture future)`回调的接口方法



## Channel

* 基本的I/O操作（bind、connection、read、write）

## EventLoop

* 处理连接中的生命周期中所发生的事件
* 一个 EventLoopGroup 包含一个或者多个 EventLoop
* 一个 EventLoop 在它的生命周期内只和一个 Thread 绑定
* 所有由 EventLoop 处理的 I/O 事件都将在它专有的 Thread 上被处理
* 一个 Channel 在它的生命周期内只注册于一个 EventLoop
* 一个 EventLoop 可能会被分配给一个或多个 Channel。