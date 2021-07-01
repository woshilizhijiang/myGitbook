## Netty

netty异步、事件驱动



### netty源码

#### NIO核心组件：

- Channel
- Buffer-ByteBuf
- Selector

#### Netty核心组件

- Bootstrap & ServerBootstrap
- **Channel**
- ChannelFuture
- **EventLoop** & EventLoopGroup
- ChannelHandler
- ChannelPipeline

##### 1. 细则类图1-Bootstrap和ServerBootstrap

继承抽象类AbstractBootstrap，都是启动器。

- Bootstrap和ServerBootstrap对于Netty，就相当于Spring boot是 Spring的启动器。脚手架。
- Bootstrap用于启动Netty TCP客户端，或者UDP
  - 通常使用 `#connet(...)` 方法连接到远程的主机和端口，作为一个 Netty TCP 客户端。
  - 也可以通过 `#bind(...)` 方法绑定本地的一个端口，作为 UDP 的一端。
  - 仅仅需要使用一个 EventLoopGroup 。
- ServerBootstrap启动服务端
  - 通常使用 `#bind(...)` 方法绑定本地的端口上，然后等待客户端的连接。
  - 使用**两个** EventLoopGroup 对象( 当然这个对象可以引用同一个对象 )：**第一个用于处理它本地 Socket 连接的 IO 事件处理**，而**第二个责负责处理远程客户端的 IO 事件处理**。

<details> <summary>类图查看</summary><img src="..\截图\netty_server_bootstrap.png"/></details>

##### 2. Channel

- 在 Channel 接口层，**采用 Facade 模式进行统一封装**，将网络 I/O 操作、网络 I/O 相关联的其他操作封装起来，统一对外提供。
- Channel 接口的定义尽量大而全，为 SocketChannel 和 ServerSocketChannel 提供统一的视图，由不同子类实现不同的功能，公共功能在抽象父类中实现，最大程度地实现功能和接口的重用。
- 具体实现采用聚合而非包含的方式，将相关的功能类聚合在 Channel 中，由 Channel 统一负责和调度，功能实现更加灵活。

##### 3. EventLoop && EventLoopGroup

Netty 基于**事件驱动模型**，使用不同的事件来通知我们状态的改变或者操作状态的改变。它定义了在整个连接的生命周期里当有事件发生的时候处理的核心抽象。

##### 4. ChannelFuture

Netty 为异步非阻塞，即所有的 I/O 操作都为异步的，因此，我们不能立刻得知消息是否已经被处理了。Netty 提供了 ChannelFuture 接口，通过该接口的 `#addListener(...)` 方法，注册一个 ChannelFutureListener，当操作执行成功或者失败时，监听就会自动触发返回结果。

##### 5. ChannelHandler

ChannelHandler ，连接通道处理器，我们使用 Netty 中**最常用**的组件。ChannelHandler 主要用来处理各种事件，这里的事件很广泛，比如可以是连接、数据接收、异常、数据转换等。

##### 6. ChannelPipeline

ChannelPipeline 为 ChannelHandler 的**链**，提供了一个容器并定义了用于沿着链传播入站和出站事件流的 API 。一个数据或者事件可能会被多个 Handler 处理，在这个过程中，数据或者事件经流 ChannelPipeline ，由 ChannelHandler 处理。在这个处理过程中，一个 ChannelHandler 接收数据后处理完成后交给下一个 ChannelHandler，或者什么都不做直接交给下一个 ChannelHandler。

### netty问题

#### 1.netty如何解决epoll空轮询问题

- **Selector BUG出现的原因**
  **若Selector的轮询结果为空，也没有wakeup或新消息处理，则发生空轮询，CPU使用率100%**，

- Netty的解决办法

  - 对Selector的select操作周期进行统计，每完成一次空的select操作进行一次计数
  - 若在某个周期内连续发生N次空轮询，则触发了epoll死循环bug。
  - 重建Selector，判断是否是其他线程发起的重建请求，若不是则将原SocketChannel从旧的Selector上去除注册，重新注册到新的Selector上，并将原来的Selector关闭。

- 源码部分

  ```java
  io.netty.channel.nio.NioEventLoop
   @Override
      protected void run() {
          for (;;) {
              try {
                  try {
                      switch (selectStrategy.calculateStrategy(selectNowSupplier, hasTasks())) {
                      case SelectStrategy.CONTINUE:
                          continue;
  
                      case SelectStrategy.BUSY_WAIT:
                          // fall-through to SELECT since the busy-wait is not supported with NIO
  
                      case SelectStrategy.SELECT:
                          select(wakenUp.getAndSet(false));
                          if (wakenUp.get()) {
                              selector.wakeup();
                          }
                          // fall through
                      default:
                      }
                  } catch (IOException e) {
                      // If we receive an IOException here its because the Selector is messed up. Let's rebuild
                      // the selector and retry. https://github.com/netty/netty/issues/8566
                      rebuildSelector0();
                      handleLoopException(e);
                      continue;
                  }
  
                  cancelledKeys = 0;
                  needsToSelectAgain = false;
                  final int ioRatio = this.ioRatio;
                  if (ioRatio == 100) {
                      try {
                          processSelectedKeys();
                      } finally {
                          // Ensure we always run tasks.
                          runAllTasks();
                      }
                  } else {
                      final long ioStartTime = System.nanoTime();
                      try {
                          processSelectedKeys();
                      } finally {
                          // Ensure we always run tasks.
                          final long ioTime = System.nanoTime() - ioStartTime;
                          runAllTasks(ioTime * (100 - ioRatio) / ioRatio);
                      }
                  }
              } catch (Throwable t) {
                  handleLoopException(t);
              }
              // Always handle shutdown even if the loop processing threw an exception.
              try {
                  if (isShuttingDown()) {
                      closeAll();
                      if (confirmShutdown()) {
                          return;
                      }
                  }
              } catch (Throwable t) {
                  handleLoopException(t);
              }
          }
      }
  ```

  















































