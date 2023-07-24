# ✅TCP沾包与拆包问题
## 问题产生的原因

1. 应用程序`write`写入的字节大小大于套接口发送缓冲区大小；
2. 进行MSS大小的TCP分段；
3. 以以太网帧的`playload`大于MTU进行IP分片。
## TCP沾包/拆包问题原因图示
![TCP沾包/拆包问题原因图示](https://cdn.nlark.com/yuque/0/2023/png/22241519/1689952407162-773b25b0-ce7c-40db-b12d-6ed912939b60.png#averageHue=%23f1f1f1&clientId=u8d8a00b2-2a3c-4&from=paste&height=407&id=u7ef2079a&originHeight=407&originWidth=686&originalType=binary&ratio=1&rotation=0&showTitle=true&size=113717&status=done&style=none&taskId=u70b945b6-119c-4f62-8022-ec3fedc7075&title=TCP%E6%B2%BE%E5%8C%85%2F%E6%8B%86%E5%8C%85%E9%97%AE%E9%A2%98%E5%8E%9F%E5%9B%A0%E5%9B%BE%E7%A4%BA&width=686 "TCP沾包/拆包问题原因图示")
## 沾包问题的解决策略
底层的TCP无法理解上层的业务数据，底层是无法保证数据包不被拆分和重组的。
所以说只能通过上层的应用协议栈设计来解决，根据业界的主流协议的解决方案，可用归纳如下。

1. **消息定长；**
   1. 每个报文大小为固定长度200字节
   2. 如果不够，空位补空格
2. **在包尾增加回车换行符进行分割；**
   1. 比如换行符`\n`
   2. 如果一个包被拆分了，则等待下一个包发送过来之后找到其中的`\n`，然后对其拆分的头部部分与前一个包的剩余部分进行合并即可。
3. **仿照TCP/IP将消息分为消息头和消息体；**
   1. 消息头中包含表示消息总长度（或者消息体长度）的字段
   2. 通常设计思路为消息头的第一个字段使用`int32`来表示消息的总长度
   3. 只有读到足够长度的消息之后才算读到了一个完整的消息。
4. **更复杂的应用层协议。**

**上面这4点其实就是对包的格式进行约束下，方便处理。**
## Netty自己的解决方式
针对TCP的粘包、拆包问题，Netty有自己的解决方式。Netty通过预先指定的数据流编解码器，按照预先约定好的规则进行数据的解析，即可解决对应的粘包、拆包问题。具体到代码层面，主要有以下几种解码器：

1. 按照换行符切割报文：`LineBasedFrameDecoder`
2. 按照自定义分隔符符号切割报文：`DelimiterBasedFrameDecoder`
3. 按照固定长度切割报文：`FixedLenghtFrameDecoder`
4. 基于数据包长度切割报文：`LengthFieldBasedFrameDecoder`
## LineBasedFrameDecoder原理分析
是一个以换行符为结束标志的解码器。
遍历`ByteBuf`中的可读字节，判断看是否有`\n`或者`\r\n`，如果有，就以此位置为结束位置。
从可读索引到结束位置区间的字节就组成一行。
如果连续读到最大长度后仍然没有发现换行符，抛出异常，并忽略掉之前读到的异常码流。
支持携带结束符或者不携带结束符两种编码方式，同时支持配置单行的最大长度。

# ✅编解码技术/Java 序列化
评判一个编解码框架的优劣，考虑因素：

1. 是否支持跨语言，支持的语言种类是否丰富；
2. 编码后的码流大小；
3. 编解码的性能；
4. 类库是否小巧，API使用是否方便；
5. 使用者需要手工开发的工作量和难度。
## 业界主流的编解码框架
Protobuf 和 Thrift
## Google 的 Protobuf 介绍
全称 Google Protocol Buffers
谷歌开源，数据结构以`.proto`文件进行描述，通过代码生成工具可以生成对应数据结构的POJO对象和Protobuf相关的方法和属性。
优点：

- 文本化的数据结构描述语言，可以实现语言和平台无关，特别适合异构间的集成；
- 通过标识字段的顺序，可以实现协议的前向兼容；
- 自动代码生成，不需要手工编写同样数据结构的Cpp和Java版本；
- 方便后续的管理和维护，相比于代码，结构化的文档更容易管理和维护。
## Facebook 的 Thrift 介绍
Thrift可以作为高性能的通信中间件使用，支持数据（对象）序列化和多种类型的RPC服务。
主要由5部分组成：

1. 语言系统以及IDL编译器：负责由用户给定的IDL文件生成相应语言的接口代码；
2. TProtocol：RPC的协议层，可以选择多种不同的对象序列化方式，如JSON和Binary；
3. TTransport：RPC传输层，同样可以选择不同的传输层实现，如socket、NIO、MemoryBuffer等；
4. TProcessor：作为协议层和用户提供的服务发现之间的纽带，负责调用服务发现的接口。
5. TServer：聚合TProtocol、TTransport、和TProcessor等对象。

我们重点关注的是编解码框架，与之对应的就是TProtocol。
由于Thrift的RPC服务调用和编解码框架绑定在一起，所以通常使用Thrift的时候会采取RPC框架的方式。
# ✅私有协议栈开发
使用私有协议的初衷

1. 跨界点的远程服务调用，除了链路层的物理连接外，还需要对请求和响应信息进行编解码。
2. 在请求和应答消息本身以外，也需要携带一些其他控制和管理类指令，例如链路建立的握手请求和响应消息、链路检测的心跳消息等。

上面两条功能组合到一起之后，就会形成私有协议。
## Netty协议栈功能设计
承载业务内部各模块之间的消息交互和服务调用，主要功能如下：

1. 基于Netty的NIO通信框架，提供高性能的异步通信能力；
2. 提供消息的编解码框架，可以实现POJO的序列化和反序列化；
3. 提供基于IP地址的白名单接入认证机制；
4. 链路的有效性校验机制；
5. 链路的断连重连机制。
## 通信模型
![Netty协议栈通信交互图](https://cdn.nlark.com/yuque/0/2023/png/22241519/1690121460588-06072e4e-6ae8-4dd1-aae6-f5c48c943701.png#averageHue=%23f0f0f0&clientId=u28947333-454d-4&from=paste&height=220&id=ueb0b85a5&originHeight=220&originWidth=297&originalType=binary&ratio=1&rotation=0&showTitle=true&size=44087&status=done&style=none&taskId=u81ddc43f-4477-43bf-a512-bbc3d38adfa&title=Netty%E5%8D%8F%E8%AE%AE%E6%A0%88%E9%80%9A%E4%BF%A1%E4%BA%A4%E4%BA%92%E5%9B%BE&width=297 "Netty协议栈通信交互图")
具体步骤如下：

1. Netty协议栈客户端发送握手请求消息，携带节点ID等有效身份认证信息；
2. Netty协议栈服务端对握手请求消息进行合法性校验，包括节点ID有效性校验、节点重复登录校验和IP地址合法性校验，校验通过后，返回登录成功的握手应答消息；
3. 链路建立成功之后，客户端发送业务消息；
4. 链路成功之后，服务端发送心跳消息；
5. 链路建立成功之后，客户端发送心跳消息；
6. 链路建立成功之后，服务端发送业务消息；
7. 服务端退出时，服务端关闭连接，客户端感知对方关闭连接后，被动关闭客户端连接。
## 链路的建立
在分布式组网环境中，一个节点可能既是服务端也是客户端。
如果A节点需要调用B节点的服务，但是A和B之间还没有建立物理链路，则由调用方主动发起连接，此时调用方为客户端，被调用方为服务端。
## 链路的关闭
由于采用长连接通信，在正常的业务运行期间，双方通过心跳和业务消息维持链路，任何一方都不需要主动关闭连接。
以下情况，客户端和服务端需要关闭连接：

1. 当对方宕机或者重启时，会主动关闭链路，另一方读取到操作系统的通知信号，得知对方REST链路，需要关闭连接，释放自身的句柄等资源。由于采用TCP全双工通信，通信双方都需要关闭连接，释放资源；
2. 消息读写过程中，发生了I/O异常，需要主动关闭连接；
3. 心跳消息读写过程中发生了I/O异常，需要主动关闭连接；
4. 心跳超时，需要主动关闭连接；
5. 发生编码异常等不可恢复错误时，需要主动关闭连接。
## 可靠性设计
非常恶劣的网络环境：网络超时、闪断、对方进程僵死或者处理缓慢，需要对可靠性进行统一规划和设计。
### 心跳机制
检测链路的互通性，一旦发现网络故障，立即关闭链路，主动重连。
具体的设计思路如下：

1. 当网络处于空闲状态持续时间达到T（_连续周期T没有读写消息_）时，客户端主动发送Ping心跳消息给服务端。
2. 如果在下一个周期T到来时客户端没有收到对方发送的Pong心跳应答消息或者读取到服务端发送的其他业务消息，则心跳失败计数器加1。
3. 每当客户端接收到服务的业务消息或者Pong应答消息时，将心跳失败计数器清零：连续N次没有接收到服务端的Pong消息或者业务消息，则关闭链路，间隔`INTERVAL`时间后发起重连操作。
4. 服务端网络空闲状态持续时间达到T后，服务器将心跳失败计数器加1：只要接收到客户端发送的Ping消息或者其他业务消息，计数器清零。
5. 服务端连续N次没有接收到客户端Ping消息或者其他业务消息，则关闭链路，释放资源，等待客户端重连。

Ping-Pong双向心跳机制，保证无论哪一方出现网络故障，都能被及时地检测出来。
防止误判（对方忙，短时间没及时回复），只有连续N次心跳都失败才判定链路以及损坏，需要关闭链路并重建链路。
### 重连机制
如果链路中断，等待INTERVAL时间后，由客户端发起重连操作，如果重连失败，**_间隔周期INTERVAL后_**再次发起重连，直到重连成功。
### 重复登录保护
当客户端握手成功之后，在链路处于正常状态下，不允许客户端重复登录，以防止客户端在异常状态下反复重连导致句柄资源被耗尽。
服务端接收到客户端的握手请求消息之后，首先对IP地址进行合法性检验，如果校验成功，在缓存的地址表中查看客户端是否已登录，如果已经登录，则拒绝重复登录，返回错误码-1，同时关闭TCP链路，并在服务端的日志中打印握手失败的原因。
### 消息缓存重发
无论客户端还是服务端，在链路中断后，恢复前，缓存在消息队列中待发送的消息不能丢失，等恢复后，重发这些消息，保证链路中断期间消息不丢失。
考虑内存溢出风险，消息缓存队列要设置上限，到达上限后，拒绝继续添加新的消息。
## 安全性设计
内部长连接采用基于IP地址的安全认证机制，服务端对握手消息的IP地址进行合法性校验：如果在白名单之内，则校验通过；否则，拒绝对方连接。
如果在公网中使用，要用更严格的安全认证机制，例如基于密钥和AES加密的用户名+密码认证机制，也可以采用SSL/TSL安全传输。
## 可扩展性设计
业务可以在消息头中自定义业务域字段。
通过Netty消息头中的可选附件attachment字段，业务可以方便地进行自定义扩展。
Netty协议栈架构需要具备一定的扩展能力，例如统一的消息拦截，接口日志，安全、加解密等可以方便的被添加和删除，不需要修改之前的逻辑代码。
# ✅服务端开发
Netty服务端创建时序图
![Netty服务端创建时序图](https://cdn.nlark.com/yuque/0/2023/png/22241519/1690123246563-36102887-1846-437f-b36e-7469748c62c3.png#averageHue=%23ebebeb&clientId=u28947333-454d-4&from=paste&height=360&id=uf16a4a69&originHeight=360&originWidth=650&originalType=binary&ratio=1&rotation=0&showTitle=true&size=115152&status=done&style=none&taskId=u6af9bbd7-72ce-4888-8c89-622fec6e1dd&title=Netty%E6%9C%8D%E5%8A%A1%E7%AB%AF%E5%88%9B%E5%BB%BA%E6%97%B6%E5%BA%8F%E5%9B%BE&width=650 "Netty服务端创建时序图")
对Netty服务端创建的关键步骤和原理进行介绍：

1. 创建`ServerBootstrap`实例。
2. 设置并绑定`Reactor`线程池。
3. 设置并绑定服务端`Channel`。
4. 链路建立的时候创建并初始化`ChannelPipeline`。
5. 初始化`ChannelPipeline`完成之后，添加并设置`ChannelHandler`。
6. 绑定并启动监听窗口。
7. `Selector`轮询。
8. 当轮询到准备就绪的`Channel`之后，就由`Reactor`线程`NioEventLoop`执行`ChannelPipeline`的对应方法，最终调度并执行`ChannelHandler`。
9. 执行Netty系统`ChannelHandler`和用户添加定制的`ChannelHandler`。`ChannelPipeline`根据网络事件的类型，调度并执行`ChannelHandler`。
# ✅客户端开发
![image.png](https://cdn.nlark.com/yuque/0/2023/png/22241519/1690123639979-f8c0b392-4a23-42d2-9b0e-0eb4f62c4109.png#averageHue=%23b7b7b7&clientId=u28947333-454d-4&from=paste&height=421&id=ud0a64ff6&originHeight=421&originWidth=655&originalType=binary&ratio=1&rotation=0&showTitle=false&size=212689&status=done&style=none&taskId=u16ce8954-2df0-4be9-99fd-3caef879caa&title=&width=655)

1. 用户线程创建`Bootstrap`实例，通过API设置创建客户端相关的参数，异步发起客户端连接。
2. 创建处理客户端连接、I/O读写的`Reactor`线程阻`NioEventLoopGroup`。**_可以通过构造函数指定I/O线程的个数，默认为CPU内核的2倍。_**
3. 通过`Bootstrap`的`ChannelFactory`和用户指定的`Channel`类型创建用于客户端连接的`NioSocketChannel`，它的功能类似于JDK NIO类库提供的`SocketChannel`。
4. 创建默认的Channel Handler Pipeline，用于调度和执行网络事件。
5. 异步发起TCP连接，判断连接是否成功。
   1. 如果成功，则直接将NioSocketChannel注册到多路复用器上，监听读写操作位，用于数据报读取和消息发送；
   2. 如果没有立即连接成功，则注册连接监听位到多路复用器，等待连接结果；
6. 注册对应的网络监听状态位到多路复用器；
7. 由多路复用器在I/O现场中轮询各`Channel`，处理连接结果；
8. 如果连接成功，设置`Future`结果，发送连接成功事件，触发`ChannelPipeline`执行；
9. 由`ChannelPipeline`调度执行系统和用户的ChannelHandler，执行业务逻辑。
# ✅Netty的线程模型

- Netty框架的主要线程就是I/O线程。
- 线程模型设计的好坏，决定了系统的吞吐量、并发性和安全性等架构质量属性。
- 提升框架的并发性能，很大程度上避免锁，局部实现了无锁化设计。
- 不同的NIO框架对于Reactor模式的实现存在差异，本质上还是遵循了Reactor的基础线程模型。
## Reactor单线程模型
Reactor单线程模型，说的是所有的I/O操作都在同一个NIO县城上面完成。
NIO线程的职责如下。

- 作为NIO服务端，接收客户端的TCP连接；
- 作为NIO客户端，向服务端发起TCP连接；
- 读取通信对端的请求或者应答消息；
- 向通信对端发送消息请求或者应答消息。

Reactor单线程模型如图：
![Reactor单线程模型](https://cdn.nlark.com/yuque/0/2023/png/22241519/1690112093384-8825fa60-3a42-4d5f-8a79-78500eee754d.png#averageHue=%23f0f0f0&clientId=u28947333-454d-4&from=paste&height=234&id=u4ef978a9&originHeight=234&originWidth=515&originalType=binary&ratio=1&rotation=0&showTitle=true&size=62444&status=done&style=none&taskId=ue68d3e9f-42b7-4f15-963c-92d7f7cc1ad&title=Reactor%E5%8D%95%E7%BA%BF%E7%A8%8B%E6%A8%A1%E5%9E%8B&width=515 "Reactor单线程模型")
对于高负载、大并发的应用场景不合适，原因如下：

1. 一个NIO线程同时处理成百上千的链路，性能无法支撑。
2. 当NIO线程负载过重之后，处理速度将变慢，这会让大量客户端连接超时，进一步导致重发，更加重了NIO线程的负载，导致大量消息积压和处理超时，称为系统的性能瓶颈。
3. 可靠性问题：一旦NIO线程意外跑飞，或者进入死循环，会导致整个系统通信模块不可用，不能接收和处理外部消息，造成节点故障。

为解决上述问题，演进出来Reactor多线程模型。
## Reactor多线程模型
Reactor多线程模型与单线程模型最大的区别：有一组NIO线程来处理I/O操作。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/22241519/1690112534030-c6bdd379-2993-495f-8e60-cf8eeca28f39.png#averageHue=%23f1f1f1&clientId=u28947333-454d-4&from=paste&height=264&id=uf4daa5d4&originHeight=264&originWidth=581&originalType=binary&ratio=1&rotation=0&showTitle=false&size=73559&status=done&style=none&taskId=uf8279385-9950-4ff1-a69b-a271518c6c6&title=&width=581)
Reactor多线程模型的特点：

1. **有专门一个NIO线程**，Acceptor线程用于监听服务端，接收客户端的TCP连接请求。
2. **网络I/O操作，读写等由一个NIO线程池负责**，线程池可以采用标准的JDK线程池实现，它包含一个任务队列和N个可用的线程，由这些NIO线程负责消息的读取、解码、编码和发送。
3. **一个NIO线程可以同时处理**_**N**_**条条链路**，但是一个链路只对应一个NIO线程，防止发生并发操作问题。

性能问题：
一个NIO线程负责监听和处理所有的客户端连接可能会存在性能问题。
单独一个Acceptor线程可能会存在性能不足的问题
所有有了下面的主从Reactor多线程模型。
## 主从Reactore多线程模型
特点：

- 服务端用于接收客户端连接的不再是一个单独的NIO线程，而是一个独立的NIO线程池。

Acceptor接收到客户端TCP连接请求并处理完成后（可能包含接入认证等），将新创建的SocketChannel注册到I/O线程池（sub reactor线程池）的某个I/O线程上，由它负责SocketChannel的读写和编解码工作。
Acceptor线程池仅仅用于客户端的登录、握手和安全认证，一旦链路建立成功，就将链路注册到后端subReactor线程池的I/O线程上，由I/O线程负责后续的I/O操作。
**主从Reactore多线程模型图示**
![主从Reactore多线程模型](https://cdn.nlark.com/yuque/0/2023/png/22241519/1690113135518-1ec31abe-07bc-41ae-b2d4-a1791277da79.png#averageHue=%23efefef&clientId=u28947333-454d-4&from=paste&height=363&id=uc9591c59&originHeight=363&originWidth=604&originalType=binary&ratio=1&rotation=0&showTitle=true&size=120155&status=done&style=none&taskId=uc228622e-03cd-41e7-aae5-492986e4c0a&title=%E4%B8%BB%E4%BB%8EReactore%E5%A4%9A%E7%BA%BF%E7%A8%8B%E6%A8%A1%E5%9E%8B&width=604 "主从Reactore多线程模型")
利用主从NIO线程模型，可用解决一个服务端监听线程无法有效处理所有客户端连接的性能不足问题。
## Netty线程模型
Netty的线程模型实际取决于用户的启动参数配置。
它同时支持Reactor单线程模型、多线程模型和主从Reactor多线程模型。
原理图
![image.png](https://cdn.nlark.com/yuque/0/2023/png/22241519/1690113297879-1c8b4b70-7900-40ff-8e8a-650cfceb0954.png#averageHue=%23cacaca&clientId=u28947333-454d-4&from=paste&height=298&id=u7d0675c6&originHeight=298&originWidth=807&originalType=binary&ratio=1&rotation=0&showTitle=false&size=140595&status=done&style=none&taskId=u630cb951-5cc4-482b-91fa-02174ad6b12&title=&width=807)
服务端启动时，创建了2个NioEventLoopGroup，实际上是2个独立的Reactor线程池。
1个用于接收客户端的TCP连接，1个用于处理I/O相关的读写操作，或者执行系统Task、定时任务Task等。

Netty用于接收客户端请求的线程池职责：

1. 接收客户端TCP连接，初始化Channel参数；
2. 将链路状态变更事件通知给ChannelPipeline。

Netty处理I/O操作的Reactor线程池职责：

1. 异步读取通信对端的数据报，发送读事件到ChannelPipeline；
2. 异步发送消息到通信对端，调用ChannelPipeline的消息发送接口；
3. 执行系统调用Task；
4. 执行定时任务Task，例如链路空闲状态检测定时任务。
## Neety无锁化的设计总结
在I/O线程内部进行串行操作，避免多线程竞争导致的性能下降问题。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/22241519/1690113707338-20d5029c-1d34-4cd1-b700-460542e70e58.png#averageHue=%23dcdcdc&clientId=u28947333-454d-4&from=paste&height=188&id=u252c0678&originHeight=188&originWidth=783&originalType=binary&ratio=1&rotation=0&showTitle=false&size=72045&status=done&style=none&taskId=u8da1121c-2aff-4362-9ee2-3f3ce89759b&title=&width=783)
Netty的NioEventLoop读取到消息之后，直接调用ChannelPipeline的`fireChannelRead(Object msg)`。
只要用户不主动切换线程，一直都是由NioEventLoop调用用户的Handler，期间不进行线程切换。
这就避免了多线程导致的锁竞争，从性能角度看最优。

1. **首先，**Netty基于Reactor线程模式实现并发请求处理，避免了线程阻塞与锁的竞争。
2. **其次，Netty实现了对象池，用来减少对象的创建和销毁，从而也能避免锁的竞争。**
3. **另外，Netty中还使用CAS和原子类来代替锁，来实现线程安全的操作。**比如，ChannelPipeline中的addLast()方法就是使用CAS来添加ChannelHandler的。
4. **而且，Netty中有许多组件都被设计为线程安全的。**例如，每个Channel都有一个唯一的EventLoop，用于处理所有事件。这样就会避免锁竞争和线程切换带来的开销。
## 最佳实践
Netty的多线程编程最佳实践如下。

1. 创建两个NioEventLoopGroup，用于逻辑隔离NIO Acceptor和NIO I/O线程。
2. 尽量不要在ChannelHandler中启动用户线程（解码后用于将POJO消息派发到后端业务线程的除外）。
3. 解码要放在NIO线程调用的解码Handler中进行，不要切换到用户线程中完成消息的解码。
4. 如果业务逻辑操作非常简单，没有复杂的业务逻辑计算，没有可能会导致线程被阻塞的磁盘操作、数据库操作、网络操作等，可以在NIO线程上完成业务逻辑编排，不需要切换到用户线程。
5. 如果业务逻辑处理复杂，不要在NIO线程上完成，建议将解码后的POJO消息封装成Task，派发到业务线程池中由业务线程执行，以保证NIO线程尽快被释放，处理其他的I/O操作。

推荐的线程数量计算公式有以下两种。

1. **公式1：线程数量=（线程总时间/瓶颈资源时间）x 瓶颈资源的线程并行数**
2. **公式2：QPS=1000/线程总时间x线程数**

用户场景不同，复杂系统很难计算出最优，只能根据测试数据和用户场景，结合公式给出一个相对合理的范围，然后对范围内的数据进行性能测试，选择相对最优值。
# ✅Netty架构剖析
## 逻辑架构
Netty采用了是典型的三层网络架构进行设计和开发
![Netty逻辑架构图](https://cdn.nlark.com/yuque/0/2023/png/22241519/1690116993848-6600dbb7-5251-43cb-b20f-3d6f856fcb57.png#averageHue=%23d6d6d6&clientId=u28947333-454d-4&from=paste&height=376&id=u19aaa675&originHeight=376&originWidth=481&originalType=binary&ratio=1&rotation=0&showTitle=true&size=122209&status=done&style=none&taskId=ue4a1b9b0-08aa-4cd2-8645-d2f0887b7ef&title=Netty%E9%80%BB%E8%BE%91%E6%9E%B6%E6%9E%84%E5%9B%BE&width=481 "Netty逻辑架构图")
分层设计充分实现了NIO框架各层之间的解耦。
对于业务开发者，只需要关心职责链的拦截和业务Handler的编排。
### Reactor通信调度层
由一系列辅助类完成

- Ractor线程NioEventLoop及其父类
- NioSocketChannel/NioServerSocketChannel及其父类
- BytBuffer以及由其衍生出来的各种Buffer
- Unsafe以及其衍生出的各种内部类等。

这一层主要是负责

1. 监听网络的读写和连接操作，负责将网络层的数据读取到内存缓冲区中，
2. 触发各种网络事件，例如连接创建、连接激活、读事件、写事件等，
3. 将各种网络事件触发到Pipeline中，由Pipeline管理的职责链来进行后续的处理。
### 职责链ChannelPipeline
负责事件在职责链中的有序传播，同时负责动态地编排职责链。
**职责链可以选择监听和处理自己关心的事件，它可以拦截处理和向后/向前传播事件。**
### 业务逻辑编排层（Service ChannelHandler）
业务逻辑编排层一般有两类：

1. _纯粹的业务逻辑编排_
2. _其他应用层插件，用于特定协议相关的会话和链路管理_。例如CMPP协议，用于管理和中国移动短信系统的对接。
## 关键架构质量属性
### 高性能
#### 影响因素：

- 软件因素：
   - 架构不合理导致的性能问题
   - 编码实现不合理导致的性能问题，例如锁的不恰当使用导致性能瓶颈。
- 硬件因素
   - 服务器配置太低
   - 带宽、磁盘的IOPS等限制导致的I/O操作性能差
   - 测试环境被共用导致被测试的软件产品受到影响。
#### Netty的架构设计是怎么实现的？

1. 采用异步非阻塞的I/O类库，基于Reactor模式实现
   1. 解决了传统同步阻塞I/O模式下一个服务器无法平滑地处理线性增长的客户端的问题。
2. TCP接收和发送缓冲区使用直接内存代替堆内存，避免了内存复制，提升了I/O读取和写入的性能
3. 支持通过内存池的方式循环利用ByteBuf，避免了频繁创建和销毁ByteBuf带来的性能损耗
4. 可配置的I/O线程数、TCP参数等
   1. 为不同的用户场景提供定制化的调优参数，满足不同的性能场景
5. 采用环形数组缓冲区实现无锁化并发编程，代替传统的线程安全容器或者锁。
6. 合理的使用线程安全容器、原子类，提升系统的并发处理能力。
7. **关键资源的处理使用单线程串行化的方式**，避免多线程并发访问带来的锁竞争和额外的CPU资源消耗问题。
8. 通过**引用计数器及时地申请释放不在被引用的对象**，细粒度的内存管理降低了GC的频率，减少了频繁GC带来的时延增大和CPU损耗。
### 可靠性
#### 链路有效性检测——心跳检测
为了支持心跳，Netty提供了如下两种链路空闲检测机制。

- **读空闲超时机制：**当连续周期T没有消息可读，触发超时Handler，用户可以基于读空闲超时发送心跳消息，进行链路检测：如果连续N个周期仍然没有读取到心跳消息，可以主动关闭链路。
- **写空闲超时机制：**当连续周期T没有消息要发送时，触发超时Handler，用户可以基于写空闲超时发送心跳消息，进行链路检测：如果连续N个周期仍然没有接收到对方的心跳消息，可以主动关闭链路。

Netty还提供了空闲状态检测事件通知机制，用户可以订阅读空闲超时事件、写空闲超时事件、读或者写超时事件，在接收到对应的空闲事件之后，灵活定制。
#### 内存保护机制
提供了多种内存保护机制：

1. 通过对象引用计数器对Netty的ByteBuf等内置对象进行细粒度的内存申请和释放，对非法的对象引用进行检测和保护。
2. 通过内存池来重用ByteBuf，节省内存。
3. 可设置的内容容量上限，包括ByteBuf解码保护、线程池线程数等。
#### 优雅停机
当系统退出时，JVM通过注册的Shutdown Hook拦截到退出信号量，然后执行退出操作，释放相关模块的资源占用，将缓冲区的消息处理完成或者清空，将待刷新的数据持久化到磁盘或者数据库中，等到资源回收和缓冲区消息处理完成之后，再退出。
### 可定制性

1. _责任链模式：ChannelPipeline基于责任链模式开发，便于业务逻辑的拦截、定制和扩展。_
2. _基于接口的开发：关键的类库都提供了接口或者抽象类，_如果Netty自身的实现无法满足用户的需求，可以由用户自定义实现相关接口。
3. _提供了大量工厂类_，通过重载这些工厂类可以按需创建出用户实现的对象。
4. _提供了大量的系统参数供用户按需设置，增强系统的场景定制性。_
### 可扩展性
业界存在大量的基于Netty框架开发的协议
基于Netty的HTTP协议、Dubbo协议、RocketMQ内部私有协议等等
# Java多线程编程在Netty中的应用
## Java内存模型与多线程编程
JVM规范定义了Java内存模型（Java Memory Model, JMM）来屏蔽掉各种操作系统、虚拟机实现厂商和硬件的内存访问差异，以确保Java程序在所有操作系统和平台上能够实现一次编写、到处运行的效果。
### 工作内存和主内存
Java内存模型规定：

1. 所有变量都存储在主内存中（JVM内存的一部分）
2. 每个线程有独立的工作内存，它保存了该线程使用的变量的主内存复制。
3. 线程对这些变量的操作都在自己的工作内存中进行，不能直接操作主内存和其他工作内存中存储的变量或者变量副本。
4. 线程间的变量访问需通过主内存来完成。

![Java内存访问模型](https://cdn.nlark.com/yuque/0/2023/png/22241519/1690157152563-36d18d2f-da9e-4917-bea2-5e6f39d87944.png#averageHue=%23c6c6c6&clientId=u23d75751-2427-4&from=paste&height=170&id=u0ef97ebe&originHeight=170&originWidth=576&originalType=binary&ratio=1&rotation=0&showTitle=true&size=62127&status=done&style=none&taskId=ubff036cd-2d80-47fd-80af-35e9f58f569&title=Java%E5%86%85%E5%AD%98%E8%AE%BF%E9%97%AE%E6%A8%A1%E5%9E%8B&width=576 "Java内存访问模型")
### Java内存交互协议
Java内存模型定义了8种操作来完成主内存和工作内存的变量访问。
`lock`、`unlock`、`read`、`load`、`use`、`assign`、`store`、`write`
### Java的线程
Java语言中，是通过单进程-多线程的模型进行多任务的并发处理。
线程是比进程更轻量级的调度执行单元，它可以把进程的资源分配和调度执行分开，各个线程可以共享内存、I/O等操作系统资源，但是又能够被操作系统发起的内核线程或者进程执行。各线程可以独立地启动、运行和停止，实现任务的解耦。
主流的操作系统目前实现线程的主要三种方式：

1. 内核线程（KLT）实现
2. 用户线程实现（UT）
3. 混合实现，将内核线程和用户线程混合在一起使用的方式。
## Netty的并发编程实战
### 对共享的可变数据进行正确的同步 synchronized


### 正确使用锁
`ForkJoinTask`中的一些多线程同步和协作方面的技巧，首先是当条件不满足时阻塞某个任务，直到条件满足后再继续执行。
```java
private int externalAwaitDone() {
    int s;
    ForkJoinPool cp = ForkJoinPool.common;
    if ((s = status)>=0) {
        if (cp != null) {
            if (this instanceof CountedCompleter)
                s = cp.externalHelpComplete((CountedCompleter<?>)this);
            else if (cp.tryExternalUnpush(this))
                s = doExec();
        }
        if (s >= 0 && (s = status) >= 0) {
            do {
                if (U.compareAndSwapInt(this, STATUS, s, s | SIGNAL)) {
                    synchronized (this) {
                        if (status >= 0) {
                            try {
                                wait();
                            } catch (InterruptedException ie) {
                                interrupted = true;
                            } // end try-catch block
                        }
                        else
                            notifyAll();
                    } // end synchronized block
                }
            } while ((s = status) >=0);
        }
    }
}
```
首先通过循环检测的方式对状态变量status进行判断，当它的状态大于等于0时，执行`wait()`，阻塞当前的调度线程，直到`status`小于0，唤醒所有被阻塞的线程，继续执行。

1. wait方法用来使线程等待某个条件，它必须在同步代码块内部被调用，这个同步代码块会锁定当前对象实例。
2. 始终使用wait循环来调用wait方法，永远不要在循环之外调用wait方法。这样做的原因是尽管并不满足被唤醒条件，但是由于其他线程调用notifyAll()方法会导致被阻塞线程意外唤醒，此时执行条件并不满足，它将破环被锁保护的约定关心，导致约定失效，引起意想不到的结果。
3. 唤醒线程，应该使用notify还是notifyAll?当不知道究竟该调用哪个方法时，保守的做法是调用notifyAll唤醒所有等待的线程。从优化的角度看，如果处于等待的所有线程都在等待同一个条件，而每次只有一个线程可以从这个条件中被唤醒，那么就应该选择调用notify。
### volatile的正确使用
当一个变量被volatile修饰后，他将具备以下两种特性：

1. 线程可见性：当一个线程修改了被volatile修饰的变量后，无论是否加锁，其他线程都可以立即看到最新的修改，而普通变量却做不到这点。
2. 禁止指令重排序优化，普通的变量仅仅保证在该方法的执行过程中所有依赖赋值结果的地方都能获取正确的结果，而不能保证变量赋值操作的顺序与代码的执行顺序一致。

**根据经验总结，volatile最适合使用的是一个线程写，其他线程读的场合，如果有多个线程并发写操作，仍然需要使用锁或者线程安全的容器或者原子变量来代替。**
### CAS指令和原子类
### 线程安全类的应用
### 读写锁的应用
### 不要依赖线程优先级
# netty为啥性能这么牛逼？
## RPC性能调用模型分析
### 传统RPC调用性能差的三宗罪

1. 网络传输方式，同步阻塞I/O
2. 序列化性能差
3. 线程模型，同步阻塞
### I/O通信性能三原则

1. 传输：用什么样的通道将数据发送给对方。可以选择BIO、NIO或者AIO，I/O模型在很大程度上决定了通信的性能；
2. 协议：采用什么样的通信协议，HTTP等公有协议或者内部私有协议。
3. 线程：数据报如何读取？读取之后的编解码在哪个线程进行，编解码后的消息如何派发，Reactor线程模型的不同，对性能的影响也非常大。
## Netty高性能之道

1. 异步非阻塞通信
   1. 利用多线程或者I/O多路复用技术进行处理
   2. Netty的I/O线程NioEventLoop聚合了多路复用器Selector
2. 高效的Reactor线程模型
   1. Reactor单线程模型
   2. Reactor多线程模型
   3. 主从Reactor多线程模型
3. 无锁化的串行设计
   1. 消息处理尽可能在同一个线程内完成，期间不进行线程切换，这就避免了多线程竞争和同步锁。
4. 高效的并发编程
   1. volatile的大量、正确使用；
   2. CAS和原子类的广泛使用；
   3. 线程安全容器的使用；
   4. 通过读写锁提升并发性能。
5. 高性能的序列化框架
   1. 序列化的码流大小（网络带宽的占用）
   2. 序列化&反序列化的性能（CPU资源占用）
   3. 是否支持跨语言（异构系统的对接和开发语言切换）
6. 零拷贝
   1. **Netty的接收和发送ByteBuffer采用DIRECT BUFFERS，使用堆外直接内存进行Socket读写，不需要进行字节缓冲区的二次拷贝。**
   2. CompositeByteBuf，它对外将多个ByteBuf封装成一个ByteBuf，对外提供统一封装后的ByteBuf接口。
7. 内存池
8. 灵活的TCP参数配置能力
# 可靠性
## Netty高可靠性设计

1. 网络通信类故障
   1. 客户端连接超时
      1. 在同步阻塞I/O模型中，连接操作是同步阻塞的，如果不设置超时时间，客户端I/O线程可能会被长时间阻塞，然后导致系统可用I/O线程数的减少。
      2. 业务层需要：大多数系统都会对业务流程执行时间有限制。客户端设置连接超时时间为了实现业务层的超时
   2. 通信对端强制关闭连接
   3. 链路关闭
   4. 定制I/O故障
2. 链路的有效性检测
   1. TCP层面的心跳检测，即TCP的Keep-Alive机制，它的作用域是整个TCP协议栈；
   2. 协议层的心跳检测，主要存在于长连接协议中。
   3. 应用层的心跳检测，主要由各业务产品通过约定方式定时给对方发送心跳消息实现。
3. Reactor线程的保护
   1. 异常处理要谨慎
      1. 某个消息的异常不应该导致整条链路不可用
      2. 某条链路不可用不可以应该导致其他链路不可用
      3. 某个进程不可以不应该导致其他集群节点不可用
   2. 规避NIO BUG
4. 内存保护
   1. 缓冲区的内存泄露保护
   2. 缓冲区溢出保护
5. 流量整形 traffic shaping
   1. 是一种主动调整流量输出速率的措施。
   2. 典型应用是基于下游网络节点的TP指标来控制本地流量的输出。
6. 优雅停机接口


