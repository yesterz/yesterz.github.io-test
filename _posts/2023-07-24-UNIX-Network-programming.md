# OSI模型
## 理解两个词 ISO & OSI

1. **ISO** International Organizational for Standardization 国际标准化组织
2. **OSI** Open System Interconnection 计算机通信开放系统互连
## 图解：OSI模型和网际协议族中的各层
OSI模型就是描述一个网络中各个协议层的，中文是计算机通信开放系统互连（OSI）模型，是一个七层模型
![OSI模型和网际协议族中的各层](https://cdn.nlark.com/yuque/0/2023/png/22241519/1689684032849-f2d0ae00-5ffb-4960-86bb-a7af7e520c96.png#averageHue=%23f3f3f3&clientId=ub548144e-fd56-4&from=paste&height=451&id=ubbb409f8&originHeight=451&originWidth=797&originalType=binary&ratio=1&rotation=0&showTitle=true&size=85821&status=done&style=none&taskId=ubaab4875-5553-4d5e-8401-aeded4174d5&title=OSI%E6%A8%A1%E5%9E%8B%E5%92%8C%E7%BD%91%E9%99%85%E5%8D%8F%E8%AE%AE%E6%97%8F%E4%B8%AD%E7%9A%84%E5%90%84%E5%B1%82&width=797 "OSI模型和网际协议族中的各层")
网络层由IPv4和IPv6这两个协议处理。传输层有TCP和UDP协议。注意TCP和UDP中间用间隙，表明应用层的网络应用可以绕过传输层直接使用IPv4和IPv6，即使用原始套接字（row socket）
应用层是由OSI模型的顶上三层被合并成一层，称之为应用层。Web客户（浏览器）、Telnet客户、Web服务器、FTP服务器和其他我们在使用的网络应用所在的层。
所谓的套接字编程接口：从顶上三层（网际协议的应用层）进入传输层的接口。本书聚焦于怎么使用套接字编写使用TCP或UDP的网络应用程序。
> 图1-14中，XTI是啥？XTI（X/Open Transport Interface，X/Open传输接口）是一个面向连接的网络编程接口规范，由X/Open组织定义。它提供了一种独立于网络协议的编程接口，使应用程序能够在不同的操作系统和网络环境中进行网络通信。

## Q：为什么套接字提供的是从OSI模型的顶上三层进入传输层的接口？
Ans：

1.  顶上三层处理具体网络应用（FTP、Telnet、HTTP）的所有细节，对通信细节了解很少；相反，对应底下四层对具体网络应用了解不多，但可以处理所有通信细节。
   1. 通信细节：发送数据，等待确认，给无需到达的数据排序，计算并验证校验和等等
2. 顶上三层通常构成所谓的用户进程（user process），底下四层却通常作为操作系统内核的一部分提供。
   1. Unix与其他Unix-like系统都提供了分隔用户进程与内核的机制，就是所谓用户态和内核态。
   2. 用户态和内核态有安全性和隔离性，同时提供了高度可编程的接口。通过系统调用，应用程序可以请求内核执行特权操作，例如创建新的进程、读写文件、分配内存等。内核在收到系统调用请求后，会切换到内核态，并根据请求执行相应的操作。完成后，内核将结果返回给应用程序，并将控制权重新交还给应用程序，使其继续在用户态执行。【扩展】
# 传输层：TCP、UDP 和 SCTP
## 传输层概述
传输层包括TCP、UDP和SCTP
这些传输层协议都转而使用网络层协议IP：IPv4或者IPv6

- UDP：简单的，不可靠的用户数据报协议
- TCP：复杂的、可靠的，基于字节流的传输控制协议

Q：需要理解什么
Ans：1 这些传输层协议提供给应用进程的服务是啥？弄清这些协议处理什么？应用进程中又需要处理什么？重点理解TCP

- 调试工具：调试客户和服务器程序用`netstat`
## 总图：传输层协议一览表
协议族被称为“TCP/IP”，但是除了TCP、IP这俩主要协议外还要其他的，如图
> 协议族（Protocol family）是指一组相关的网络协议的集合

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22241519/1689686633870-bb13202a-07ab-4183-9a34-152bfd33d3a4.png#averageHue=%23f5f5f5&clientId=ub548144e-fd56-4&from=paste&height=610&id=ue8e8c912&originHeight=610&originWidth=908&originalType=binary&ratio=1&rotation=0&showTitle=false&size=150131&status=done&style=none&taskId=u9671a78c-65ec-44d2-8f35-00f0b1845f2&title=&width=908)
### 从最左边说起tcpdump

1. 命令 `**tcpdump**`
   1. 使用BSD分组过滤器（BSD packet filter, BPF），或者使用数据链路提供者接口（datalink provider interface, DLPI）直接与数据链路进行通信
   2. 图示右边9个应用下面的虚线标记为API，它通常是套接字或XTI（X/Open Transport Interface，X/Open传输接口），tcpdump应用则不使用socket或XTI
2. **命令**`**mrouted**`
3. **命令**`**ping**`
4. **命令**`**traceroute**`
   1. 使用两种套接字：1 IP套接字 2 ICMP套接字
   2. IP套接字用于访问IP，ICMP套接字用于访问ICMP
### 简单解释图中每个协议框

1. `IPv4` 网际版本协议（Internet Protocol version 4），IPv4给TCP、UDP、SCTP、ICMP和IGMP提供**分组递送服务，使用32位地址。**
> 分组递送服务（Packet Delivery Service）是一种网络服务，用于在计算机网络中将数据分组从源节点传送到目标节点。它是指网络层提供的基本传输服务，负责将数据包按照网络协议规定的路由路径进行传输和交付。 

2. `IPv6`网际版本协议（Internet Protocol version 6），IPv6给TCP、UDP、SCTP、ICMP和IGMPv6提供**分组递送服务，使用128位地址。**
3. `TCP`传输控制协议（Transmission Control Protocol）
   1. 面向连接、为用户进程提供可靠的全双工字节流。
   2. TCP套接字是一种流套接字（stream socket）。
   3. TCP关心确认、超时和重传之类的细节
4. `UDP` 用户数据包协议（User Datagram Protocol）
   1. 无连接协议
   2. UDP套接字是一种数据包套接字（datagram socket）
   3. UDP数据包不能保证最终到达它们的目的地
5. `SCTP` 流控制传输协议（Stream Control Transmission Protocol）
6. `ICMP` 网际控制消息协议（Internet Control Message Protocol）
7. `IGMP` 网际组管理协议（Internet Group Management Protocol）
8. `ARP` 地址解析协议（Address Resolution Protocol），作用是将IP地址解析成MAC地址，网络层的IP地址映射到数据链路层的MAC地址
9. `RARP` 反向地址解析协议（Reverse Address Resolution Protocol）
10. `ICMPv6` 网际控制消息协议版本6（Internet Control Message Protocol version 6）
   1. 6 综合了ICMPv4、IGMP和ARP的功能
11. `BPF` BSD分组过滤器（BSD packet filter），这个接口提供了对数据链路层的访问能力。
12. `DLPI`数据链路提供者接口（datalink provider interface），同样提供了对数据链路层的访问能力。

这些网际协议由一个或多个称为请求批注（Request for Comments, RFC）的文档定义，这些RFC就是它们的正式规范。
## 传输控制协议（TCP）特点介绍

1. TCP提供客户与服务器之间的连接（connection）。
2. **TCP客户与某个给定服务器建立一个连接，再跨该连接与那个服务器交换数据，然后终止这个连接。**
3. TCP是可靠性：TCP向一端发送数据时，要求对端返回确认。
4. TCP有动态估算客户和服务器之间的往返时间（round-trip time, RTT）的算法，所以它指导等待一个确认需要多少时间。
5. TCP会对所发送的数据进行排序：每个字节关联一个序列号，根据序列号判定数据重复，根据序列号来排序，排序后再传递给接收应用。
6. TCP提供流量控制（flow control）：TCP总是告知对端在任何时刻它一次能够从对端接收多少字节的数据，这称为通告窗口（advertised window）。在任何时刻，该窗口指出接收缓冲区中当前可用的空间量，从而确保发送端发送的数据不会让接收缓冲区溢出。
7. TCP连接时全双工的（full-duplex），这意味着在一个给定的连接上应用可以在任何时刻在进出两个方向上既发送数据又接收数据。
## TCP连接的建立和终止
完整的TCP连接：1 连接TCP建立 2 数据传送 3 释放TCP连接
> 物理层：二进制比特流传输；bit（比特流）；
> 数据链路层：：介质访问控制；frame（帧）；
> 网络层：确定地址和路由选择；packet（包），又叫做分组
> 传输层：端到端连接；也叫作数据包。但是谈论TCP等具体协议时有特殊叫法，TCP的数据单元叫数据段，segment（段），而UDP协议的数据单元称为数据报（datagram）
> 会话层、表示层、应用层：一般就称呼为消息（message)

### TCP连接建立：三次握手
建立一个TCP连接时会发生：

1. **Server **必须ready，服务器已经准备好接受外来的连接。
   1. 通常调用`socket`、`bind`和`listen`三个函数来完成
   2. 称之为 被动打开（passive open）
2. **Client** 通过调用`connect`发起主动打开（active open）
   1. 客户TCP发送一个SYN（同步）分节，它告诉服务器将在（待建立的）连接中发送的数据的初始序列号。
   2. 通常SYN分节不携带数据
3. Server 必须确认客户的SYN，同时自己也得发送一个SYN分节
   1. 这里发送的SYN分节有发送数据的初始序列号，这个数据就是服务器在这个连接中准备发送的数据。
   2. 服务器在单个分节中发送SYN和对客户SYN的ACK（确认）
4. Client 必须确认服务器的SYN

这些至少需要3个分组，所以叫TCP的三路握手（three-way handshake），如下图所示
![TCP的三路握手](https://cdn.nlark.com/yuque/0/2023/png/22241519/1689690614099-9fd9015c-ee1c-49d9-947f-7e6f1a1e9394.png#averageHue=%23f6f6f6&clientId=ub548144e-fd56-4&from=paste&height=281&id=uf179cf8a&originHeight=281&originWidth=663&originalType=binary&ratio=1&rotation=0&showTitle=true&size=55532&status=done&style=none&taskId=u695000a3-1caf-452b-a9c9-ae324d75ed8&title=TCP%E7%9A%84%E4%B8%89%E8%B7%AF%E6%8F%A1%E6%89%8B&width=663 "TCP的三路握手")
根据图，对于SYN序列号和ACK中确认号的说明：

1. 客户的初始序列号为J，服务器的初始序列号为K。
2. ACK中的确认号：指的是发送这个ACK的一端所期待的下一个序列号
   1. SYN占据1字节
   2. 所以每个SYN的ACK中的确认号是 SYN的初始序列号加1
3. 同样的，每一个FIN（表示结束）的ACK中的确认号就是FIN的序列号加1
### TCP选项

- MSS选项：发送SYN的TCP一端使用MSS告诉对端的最大分节大小（maximum segment size, MSS），就是指明连接中每个TCP分节传输可接受的最大数据量。
- 窗口规模选项：TCP连接任何一端能够通告对端的最大窗口大小是65535
- 时间戳选项：防止失而复现的分组可能造成的数据损坏。
### TCP连接终止：四次挥手
TCP建立一个连接需要3个分节，终止一个连接则需要4个分节

1. 某个应用进程首先调用`close`，该端的TCP于是乎发送一个FIN分节
   1. 进程调用了`close`，我们称之为该端执行主动关闭（active close）
   2. FIN分节（finish），表示数据发送完毕,FIN标志告知对端，我已经没有更多的数据要发送了，并且请求关闭连接。
2. 接收到这个FIN的对端执行被动关闭（passive close）
   1. 这个FIN由TCP确认？？？？
   2. 这个接收了也作为一个文件结束符（end-of-file）传递给接收端应用进程
3. 一端时间后，接收到这个文件结束符的应用进程将调用`close`关闭它的套接字
   1. 它的TCP同样要发送一个FIN
4. 接收这个最终FIN的原发送端TCP确认这个FIN
   1. 原发送端TCP，就是执行主动关闭的那一端

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22241519/1689693383444-ed3d33e0-ce67-48a6-982f-e6794eb61423.png#averageHue=%23f4f4f4&clientId=ub548144e-fd56-4&from=paste&height=350&id=u6f7d17d6&originHeight=350&originWidth=659&originalType=binary&ratio=1&rotation=0&showTitle=false&size=65901&status=done&style=none&taskId=u243ed0bd-a127-449a-a3b8-b18f4cd540f&title=&width=659)
**NOTES：**图片中是客户执行主动关闭的，其实无论客户还是服务器都可以执行主动关闭。
### TCP状态转换图
TCP为一个连接定义了11中状态。也叫TCP有限状态机【研究生课程的学习的内容？？】
> 有限状态机（英语：finite-state machine，缩写：FSM）又称有限状态自动机，简称状态机， 是表示有限个状态以及在这些状态之间的转移和动作等行为的数学模型。 有限状态机是一种用来进行对象行为建模的工具，其作用主要是描述对象在它的生命周期内所经历的状态序列， 以及如何响应来自外界的各种事件。 在计算机科学中，有限状态机被广泛用于建模应用行为、硬件电路系统设计、软件工程， 编译器、网络协议、和计算与语言的研究。比如非常有名的 TCP 协议状态机。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22241519/1689693559854-e2ccf762-2d4e-4a1d-b9ad-6f27d4df7ad8.png#averageHue=%23f4f4f4&clientId=ub548144e-fd56-4&from=paste&height=898&id=u6a34e4a4&originHeight=898&originWidth=745&originalType=binary&ratio=1&rotation=0&showTitle=false&size=229848&status=done&style=none&taskId=u056d6a03-62b3-4b46-b596-652a3ea2662&title=&width=745)
基于TCP状态转换图的说明：

1. 从`ESTABLISHED`状态引出的两个箭头处理连接的终止。两种情况
   1. active close 应用进程接收到1个FIN之前调用`close`（主动关闭），那就转换到`FIN_WAIT_1`状态
   2. passive close 应用进程在`ESTABLISHED`状态期间收到1个FIN（被动关闭），那就转换到`CLOSE_WAIT`状态
2. 图中**粗实线**表示通常的客户状态转换，用**粗虚线**表示通常的服务器状态转换。
   1. 同时打开（simultaneous open），发生在两端几乎同时发送SYN并且这两个SYN在网络中交错的情况下
   2. 同时关闭（simultaneous close），发生在两端几乎同时发送FIN的情况
   3. a & b 可能发生，但是及其罕见
3. **状态说明**
   1. **CLOSED：无连接是活动的或正在进行**
   2. **LISTEN：服务器在等待进入呼叫**
   3. **SYN_RECV：一个连接请求已经到达，等待确认**
   4. **SYN_SENT：应用已经开始，打开一个连接**
   5. **ESTABLISHED：正常数据传输状态**
   6. **FIN_WAIT1：应用说它已经完成**
   7. **FIN_WAIT2：另一边已同意释放**
   8. **TIMED_WAIT：等待所有分组死掉**
   9. **CLOSING：两边同时尝试关闭**
   10. **TIME_WAIT：另一边已初始化一个释放**
   11. **LAST_ACK：等待所有分组死掉**
### 图示一个完整的TCP连接所发生的实际分组交换情况
![TCP连接的分组交换](https://cdn.nlark.com/yuque/0/2023/png/22241519/1689694485497-15bc7fc2-748d-48ae-bc05-ddd7eeecf76c.png#averageHue=%23f2f2f2&clientId=ub548144e-fd56-4&from=paste&height=562&id=u4c57b77c&originHeight=562&originWidth=604&originalType=binary&ratio=1&rotation=0&showTitle=true&size=137600&status=done&style=none&taskId=ua138caa6-5629-4639-bc74-ec839b47914&title=TCP%E8%BF%9E%E6%8E%A5%E7%9A%84%E5%88%86%E7%BB%84%E4%BA%A4%E6%8D%A2&width=604 "TCP连接的分组交换")
## TIME_WAIT 状态
【`TIME_WAIT`状态是TCP中网络编程最不容易理解的部分】
等待
**执行主动关闭的一端**经历了这个状态。该端点停留在这个状态的持续时间是MSL的两倍，有时候也称之为2MSL。
MSL是任何IP数据报能够在因特网中存活的最长时间。这个时间是有限的，因为每个数据报含有一个称为跳限的8位字段（最大值255）。跳数有限制。
### 解释两个概念MSL&TTL

- MSL 最长分节生命周期（maximum segment lifetime, MSL）
- TTL Time to Live 跳限 hop limit   生存时间(_TTL_) _是_指数据包被设置为在被路由器丢弃之前存在于网络中的时间或“_跳_数”
### TIME_WAIT状态存在的理由

1. 可靠地实现TCP全双工连接的终止：**防止连接关闭时四次挥手中的最后一次ACK丢失**

TCP需要保证每一包数据都可靠的到达对端，包括正常连接状态下的业务数据报文，以及用于连接管理的握手、挥手报文，这其中在四次挥手中的最后一次ACK报文比较特殊，**TIME_WAIT状态就是为了应对最后一条ACK丢失的情况。**
	TCP保证可靠传输的前提是收发两端分别维护关于这条连接的状态信息（TCB控制块），当发生丢包时进行ARQ重传。如果连接释放了，就无法进行重传，也就无法保证发生丢包时的可靠传输。
	对于最后一条ACK，如果没有TIME_WAIT状态，主动关闭一方（客户端）就会在收到对端（服务器）的FIN并回复ACK后 直接从FIN_WAIT_2 进入 CLOSED状态，并释放连接，销毁TCB实例。此时如果最后一条ACK丢失，那么服务器重传的FIN将无人处理，最后导致服务器长时间的处于 LAST_ACK状态而无法正常关闭（服务器只能等到达到FIN的最大重传次数后关闭）。
至于将TIME_WAIT的时长设置为 2_MSL，是因为报文在链路中的最大生存时间为MSL（Maximum Segment Lifetime），超过这个时长后报文就会被丢弃。TIME_WAIT的时长则是：最后一次ACK传输到服务器的时间 + 服务器重传FIN 的时间，即为 2_MSL。

2. 允许老的重复分节在网络中消逝：**防止新连接收到旧链接的TCP报文**

 	TCP使用四元组区分一个连接（源端口、目的端口、源IP、目的IP），如果新、旧连接的IP与端口号完全一致，则内核协议栈无法区分这两条连接。
 	2*MSL 的时间足以保证两个方向上的数据都被丢弃，使得原来连接的数据包在网络中都自然消失，再出现的数据包一定是新连接上产生的。
## 端口号，TCP端口号与并发服务器
### 端口号 port number
客户与服务器通信用端口号（port number）来标识进程。端口号16位整数。

1. 众所周知的端口（well-known port）为0~1023。
   1. 比如80给Web服务器用，HTTP用80，HTTPS用443
   2. Unix系统的保留端口（reserved port），小于1024的都是，这些端口智能赋予特权用户进程的套接字。
2. 已登记的端口（registered port）为1024~49151
   1. 这些端口由IANA登记过
3. 动态的（dynamic）或者私用的（private）端口：49152~65535
   1. IANA不管这些，这些都是临时端口
### 套接字对 socket pair
定义：一个TCP连接的套接字对是一个定义该连接的两个端点的四元组：本地IP地址、本地TCP端口号、外地IP地址、外地TCP端口号。
套接字对唯一标识一个网络上的每个TCP连接。
**标识每个端点的两个值（IP地址和端口号）通常称为一个套接字。**
### TCP端口号与并发服务器
并发服务器中主服务器循环会派生一个子进程来处理每个新的连接。
**记号{*:21, *:*}指出服务器的套接字对。**

- 服务器在任意本地接口（第一个星号）的端口21上等待连接请求。
- 外地IP地址和外地端口都没有指定，用点号标识
- 称这些为监听套接字（listening socket）
- 上面的星号叫通配（wildcard）符

**处理两个客户连接的过程如下**
主机A

- 主机A的IP地址：206.168.112.219
- 客户1的连接套接字对：`{206.168.112.219:1500, 12.106.32.254:21}`
- 客户2的连接套接字对：`{206.168.112.219:1501, 12.106.32.254:21}`

主机B

- 这个主机是多宿主机，有两个IP地址，12.106.32.254和192.168.42.1
- 监听套接字是**{*:21, *:*}**

**过程说明：**

1. 主机A启动客户1，对主机B的IP地址12.106.32.254执行主动打开。
2. 主机B接收并接受客户1的连接时，它fork一个自身的副本，让子进程来处理该客户的情况。
   1. 已连接套接字（connected socket）使用和监听套接字相同的本地端口（21）
   2. 主机A的客户1与主机B连接一旦建立，已连接套接字的本地地址（12.106.32.254）随即填入
   3. 主机B fork 出来的子进程1的已连接套接字变为`{12.106.32.254:21, 206.168.112.219:1500}`
3. 然后假设主机A启动了另一个客户2，请求连接到同一个主机B上面
   1. 主机A的TCP则会为客户2的套接字分配一个未使用的临时端口，假设为1501
   2. 主机A的客户2的连接套接字对则会变为`{206.168.112.219:1501, 12.106.32.254:21}`
   3. 客户1与2的连接套接字对不同，端口号不一样
4. 主机B接收并接受客户2的连接，并fork出一个自身的副本子进程2来处理这个TCP连接
   1. 主机B fork 出来的子进程2的已连接套接字变为`{12.106.32.254:21, 206.168.112.219:1501}`
   2. 所有目的端口为21的其他TCP分节都被递送给拥有监听套接字的最初的那个父进程来处理
## 缓冲区大小及限制
### 影响IP数据报大小的一些限制
影响IP数据包大小的限制，进而会影响应用进程能够传送的数据

1. IPv4数据报的最大大小时65535字节，包括IPv4首部。
2. IPv6数据报的最大大小时65575字节，包括40字节的IPv6首部。
   1. IPv6有一个特大净荷（jumbo payload）选项，它把净荷长度字段扩展到32位
   2. a. 的操作需要MTU（maximum transmission unit, 最大传输单元）超过65535的数据链路提供支持
3. 许多网络有一个可由硬件规定的MTU。
   1. 以太网的MTU是1500字节
4. 在两个主机之间的路径中最小的MTU称为路径MTU（path MTU）
   1. 1500字节的以太网MTU是当今常见的路径MTU。
   2. 两个主机之间相反的两个方向上路径MTU可用不一致
   3. 因为在因特网上路由选择往往是不对称的。A到B的路径和B到A的路径不一样
5. 当一个IP数据报将从某个接口送出时，如果它的大小超过相应链路的MTU，IPv4和IPv6都将执行分片（fragmentation）
   1. 这些片段在到达最终目的地之前通常不会被重组（reassembling）
   2. IPv4主机对其产生的数据报执行分片，IPv4路由器则对其转发的数据报执行分片
   3. IPv6**只有主机**对其产生的数据报执行分片，IPv6路由器**不会**对其转发的数据报执行分片
6. IPv4首部的“不分片（don't fragment）”位（即DF位）若被设置，那么不管是发送数据报的主机还是转发数据报的路由器，都不允许对它们分片。
7. IPv4和IPv6都定义了最小重组缓冲区大小（minimum reassembly buffer size），它是IPv4或IPv6的任何实现都必须保证支持的最小数据报大小。
8. TCP有一个MSS（maximum segment size，最大分节大小），用于向对端TCP通告对端在每个分节中能发送的最大TCP数据量。
### TCP输出
某个应用进程写数据到一个TCP套接字中发生的步骤如下图
![应用进程写TCP套接字时涉及的步骤和缓冲区](https://cdn.nlark.com/yuque/0/2023/png/22241519/1689744371062-4685ba9c-2881-4827-8931-dd1c32a602ee.png#averageHue=%23f5f5f5&clientId=u5afcfbca-2f92-4&from=paste&height=603&id=ubeb12186&originHeight=603&originWidth=941&originalType=binary&ratio=1&rotation=0&showTitle=true&size=128860&status=done&style=none&taskId=u8e337c54-dbb5-4b02-93bc-fb120e70fe8&title=%E5%BA%94%E7%94%A8%E8%BF%9B%E7%A8%8B%E5%86%99TCP%E5%A5%97%E6%8E%A5%E5%AD%97%E6%97%B6%E6%B6%89%E5%8F%8A%E7%9A%84%E6%AD%A5%E9%AA%A4%E5%92%8C%E7%BC%93%E5%86%B2%E5%8C%BA&width=941 "应用进程写TCP套接字时涉及的步骤和缓冲区")
对步骤进行说明：

1. 当某个应用进程调用`write`时，内核从该应用进程的缓冲区中复制所有数据到所写套接字的发送缓冲区。
   1. 每一个TCP套接字有一个发送缓冲区
   2. SO_SNDBUF套接字选项来更改缓冲区大小
2. 内核将不从`write`系统调用返回，直到应用进程缓冲区中的所有数据都复制到套接字发送缓冲区。
   1. 如果套接字的发送缓冲区容不下该应用进程的所有数据，该应用进程将被投入睡眠。
      1. 可能是应用进程的缓冲区大于套接字的发送缓冲区
      2. 也可能是套接字的发送缓冲区已有其他数据
   2. 这里假设这个套接字是阻塞的（也可以设置非阻塞）
   3. 这里`write`调用成功返回只能说明能重新使用原来的应用进程缓冲区，不表示对端的TCP或应用进程已接收到数据。
3. 这一端的TCP提取套接字发送缓冲区中的数据并把它发送给对端TCP
4. 对端TCP必须确认收到的数据，伴随来自对端的ACK的不断送达，本端TCP至此才能从套接字发送缓冲区中丢弃已确认的数据。
   1. TCP必须为已发送的数据保留一个副本，直到它被对端确认为止。
5. 本端TCP以MSS大小的或更小的块把数据传递给IP，同时给每个数据块安上一个TCP首部以构成TCP分节
   1. 其中MSS可能是对端通告的值，或是536
6. IP给每个TCP分节安上一个IP首部以构成IP数据报，并按照其目的IP地址查找路由表项以确定外出接口，然后把数据报传递给相应的数据链路。
   1. 在IP数据报传递给数据链路之前，可能会对其分片
   2. MSS选项的作用一个就是会避免分片
   3. 较新的实现还用了路径MTU发现功能
7. 每个数据链路都有一个输出队列，如果该队列已满，那么新到的分组将被丢弃，并沿协议栈上返回一个错误
   1. 从数据链路到IP，再从IP到TCP。
   2. TCP看到这个错误后，就在后面某时刻重传对应的分节。
   3. 这时候应用进程并不知道这个暂时的情况。
# 基本的TCP连接过程函数
这里整理套接字API，从套接字地址结构开始。

- 这些结构可以在两个方向上传递：
   1. 从进程到内核
   2. 从内核到进程
- **地址转换函数**：在地址的文本表达和它们存放在套接字地址结构中的二进制值之间进行转换。
   - 地址转换函数有个问题：它们与所转换的地址类型协议相关，要考虑是IPv4还是IPv6地址。
   - 解决：有一组sock_开头的函数，会以协议无关的方式使用套接字地址结构。
- 编写一个完整的TCP客户/服务器程序所需的基本套接字函数`socket`、`connect`、`bind`、`listen`、`accept`、`fork & exec`、`close`
- 图示在一对TCP客户和服务器进程之间发生的一些典型事件的时间表。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22241519/1689753249962-6f58c0c8-b913-4021-9d9e-6990c55160b0.png#averageHue=%23f6f6f6&clientId=ufed1eac9-eea6-4&from=paste&height=786&id=u7f287ffb&originHeight=786&originWidth=702&originalType=binary&ratio=1&rotation=0&showTitle=false&size=113897&status=done&style=none&taskId=uf2aa4844-25d8-453f-b344-f961826e923&title=&width=702)

1. 服务器首先启动，稍后某时刻客户启动，它试图连接到服务器。
2. 假设客户给服务器发送一个请求，服务器处理该请求，并且给客户发回一个响应。
3. 这个过程一致持续下去，直到客户关闭连接的客户端，从而给服务器发送一个EOF（文件结束）通知为止。
4. 服务器接着也关闭连接的服务器端，然后结束运行或者等待新的客户连接。
## 套接字地址结构
大多数套接字函数都需要一个指向套接字地址结构的指针来做参数。
### IPv4套接字地址结构
也叫“网际套接字地址结构”，命名为`sockaddr_in`
网际（IPv4）套接字地址结构：`sockaddr_in`POSIX定义如下
```protobuf
struct in_addr {
  int_addr_t s_addr; /* 32-bit IPv4 address */
                     /* network byte ordered */
};
struct sockaddr_in {
  uint8_t sin_len;           /* length of structure (16) */
  sa_family_t sin_family;    /* AF_INET */
  in_port_t sin_port;        /* 16-bit TCP or UDP port number */
                             /* network byte ordered */
  struct in_addr sin_addr;   /* 32-bit IPv4 address */
                             /* network byte ordered */
  char sin_zero[8]; 				 /* unused */
}
```
几点说明：

1. IPv4地址和TCP或UDP端口号在套接字地址结构中总是以网络字节序来存储。
2. 套接字地址结构仅在给定主机上使用
   1. 某些字段用在不同主机之间通信
   2. 结构本身并不在主机之间传递
### 通用套接字地址结构
一个参数传递进任何套接字函数时，套接字地址结构总是以引用形式（指向该结构的指针）来传递。

- **问题：**套接字函数需要处理不同的支持任何协议族的套接字地址结构。
- **解决：**将传入的参数指向一个通用的套接字地址结构

所有这个通用套接字地址结构的唯一用途就是对指向特定协议的套接字地址结构的指针执行类型强制转换。
### IPv6套接字地址结构
书上说了几点注意

1. IPv6的地址族是`AF_INET6`，IPv4的地址族是`AF_INET`
2. 略
### 新的通用套接字地址结构
新的同样套接字地址结构：`struct sockaddr_storage`足以容纳系统所支持的任何套接字地址结构。
对比`sockaddr`有2点区别

1. 如果系统支持的任何套接字地址结构有对齐需要，那么`sockaddr_storage`**能满足最苛刻的对齐要求**。
2. `sockaddr_storage`足够大，能够容纳系统支持的任何套接字地址结构。
### 套接字地址结构的比较，一张图来解释
IPv4、IPv6、Unix域、数据链路和存储对比
![image.png](https://cdn.nlark.com/yuque/0/2023/png/22241519/1689752300475-2798bcba-eb6e-4ab1-876f-0bcce1ce630f.png#averageHue=%23e9e9e9&clientId=ufed1eac9-eea6-4&from=paste&height=603&id=u9eb2b9a1&originHeight=603&originWidth=809&originalType=binary&ratio=1&rotation=0&showTitle=false&size=252876&status=done&style=none&taskId=u742d2731-ac02-4536-bac6-733bec32fe3&title=&width=809)
## socket函数
要执行网络I/O，一个进程必须做的第一件事情就是调用`socket`函数，指定期望的通信协议类型（使用IPv4的TCP、使用IPv6的UDP、Unix域字节流协议等）
```
# include <sys/socket.h>
int socket(int family, int type, int protocol);
```

- 返回值：若成功则为非负描述符， 出错则为-1
- 参数family指明协议族，这个参数也叫协议域。

协议族family常数值：AF_INET（IPv4协议）、AF_INET6（IPv6协议）、AF_LOCA（Unix域套接字）L、AF_ROUTE（路由套接字）、AF_KEY（密钥套接字）	

- 参数type指明套接字类型

套接字类型type常数值：SOCK_STREAM（字节流套接字）、SOCK_DGRAM（数据报套接字）、SOCK_SEQPACKET（有序分组套接字）、SOCK_RAW（原始套接字）

- 参数protocol指明某个协议类型常值或者设置为0（置为0会选择给定family和type组合的系统默认值）

注意：也不是所有套接字family与type的组合都是有效的。
协议类型常值protocol：IPPROTO_CP（TCP传输协议）、IPPROTO_UDP（UDP传输协议）、IPPROTO_SCTP（SCTP传输协议）
`socket`函数在成功时返回一个小的非负整数，它与文件描述符类似，叫它套接字描述符（socket descriptor），简称`sockfd`。
看函数原型可知socket函数只制订了协议族和套接字类型，并没指定本地协议地址或远程协议地址。

对比`AF_XXX`和`PF_XXX`：AF_前缀表示地址族（Address Family）、PF_前缀表示协议族（Protocol Family）
## connect函数
TCP客户用`connect`函数来建立与TCP服务器端连接。
```
#include <sys/socket.h>
int connect(int sockfd, const struct sockaddr *servaddr, socklen_t addrlen);
```

- 返回值：成功0,出错-1
- sockfd：是`socket`函数返回的套接字描述符
- servaddr：指向套接字地址结构的指针
- addrlen：该结构的大小

客户在调用`connect`之前不用非得调用`bind`函数，如果需要的话内核会去确定源IP地址，并选择一个临时端口作为源端口。
如果是TCP套接字，调用`connect`函数将触发TCP的三路握手过程，而且尽在连接建立成功或出错的时候才返回，出错返回可能的几种情况：

1. 若TCP客户没有收到SYN分节的响应，则返回ETIMEDOUT错误。就是报文超时。
2. 若对客户的SYN的响应是RST（表示复位），则表明该服务器主机在我们指定的端口上没有进程在等待与之连接
   1. 服务器进程也许就没在运行
   2. 这是一种硬错误（hard error），客户一接收RST就马上返回ECONNREFUSED错误
   3. RST是TCP在发生错误时发送的一种TCP分节。产生的三个条件
      1. 目的地为某端口的SYN到达，然而这个端口上没有正在监听的服务器
      2. TCP想取消一个已有连接
      3. TCP接收到一个根本不存在的连接的分节
3. 若客户发出的SYN在中间的某个路由器上引发了一个“destination unreadable”（目的地不可达）ICMP错误
   1. 这是一种软错误（soft error），就是不会立刻返回错误，会按照一定时间间隔继续发送SYN。
   2. 如果在规定的时间还没收到响应就将ICMP错误，作为EHOSTUNREACH或ENETUNREACH错误返回给进程。
   3. 可能原因：
      1. 按照本地系统的转发表，根本没有到达远程系统的路径
      2. connect调用根本不等待就返回

根据TCP状态转换图，`connect`函数导致当前套接字从`CLOSED`状态转移到`SYN_SENT`状态

   1. 成功，再转移到`ESTABLISHED`状态
   2. 失败，则该套接字不再可用，必须关闭，这个套接字就不能再次调用connect函数来。
   3. 所以在每次`connect`失败后，都必须`close`当前的套接字描述符并重新调用`socket`
## bind函数
`bind`函数把本地协议地址**赋予一个套接字**。
协议地址：32位的IPv4地址（或128位的IPv6地址）与16位的TCP（或UDP端口号）组合
```
#include <sys/socket.h>
int bind(int sockdf, const struct sockaddr *myaddr, socklen_t addrlen);
```

- 返回值：成功0，出错-1
- myaddr：指向特定于协议的地址结构的指针
- addrlen：该地址结构的长度

对于TCP，调用`bind`函数可以指定一个端口号，或指定一个IP地址，也能都指定，或者都不指定。
常见返回的错误`EADDRINUSE`（“Address already in use”，地址已使用）
## listen函数
`listen`函数仅由TCP服务器调用，做2件事情：
当`socket`函数创建一个套接字时，假定为一个主动套接字（就是将会调用`connect`发起连接的客户套接字）

1. `listen`函数把一个未连接的套接字转换成一个被动套接字，指示内核要接受指向这个套接字的连接请求。
2. 调用`listen`函数会导致套接字从`CLOSED`状态转换到`LISTEN`状态。

`listen`函数的第二个参数是内核应该为相应套接字排队的最大连接个数。
```
#include <sys/socket.h>
int listen(int sockfd, int backlog);
```

- 返回值：成功0,出错-1

一般`listen`函数在调用`socket`和`bind`两个函数之后，并在调用`accept`之前调用。
对于`backlog`参数，内核为任何一个给定的监听套接字维护两个队列：

1. 未完成连接队列，某些SYN分节对应其中一项，这些套接字处于`SYN_RCVD`状态
   1. 某个客户以及发出并到达服务器，而服务器在等待完成相应的TCP三路握手过程。
2. 已完成连接队列，每个已完成TCP三路握手过程的客户对应其中的一项。这些套接字处于`ESTABLISHED`状态

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22241519/1689774170276-38ad8989-9ee8-405f-8eea-0c1f71d13a63.png#averageHue=%23f5f5f5&clientId=uda2ebabe-7b59-4&from=paste&height=420&id=u485d308b&originHeight=420&originWidth=649&originalType=binary&ratio=1&rotation=0&showTitle=false&size=74758&status=done&style=none&taskId=uba0f9eb8-9ec7-49f6-9269-5aa2eae94d0&title=&width=649)
每当在未完成连接队列中创建一项时，来自监听套接字的参数就复制到即将建立的连接中。
连接的创建过程完全自动，无需服务器进程插手。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/22241519/1689774192860-cdbcb17d-e97e-4281-a7d5-a4dab91bbc0f.png#averageHue=%23f4f4f4&clientId=uda2ebabe-7b59-4&from=paste&height=294&id=ubdb94ddb&originHeight=294&originWidth=615&originalType=binary&ratio=1&rotation=0&showTitle=false&size=63991&status=done&style=none&taskId=u9907c6b5-0739-4b8e-b97d-14d06766ca6&title=&width=615)

1. 当来自客户的SYN到达时，TCP在未完成连接队列中创建一个新项，然后响应以三路握手的第二个分节：服务器的SYN响应，其中捎带对客户SYN的ACK。
2. 这一项一直保留在未完成连接队列中，直到三路握手的第三个分节（客户对服务器SYN的ACK）到达或者该项超时为止。
3. 如果三路握手正常完成，该项从未完成队列移到已完成连接队列的队尾。
4. 当进程调用accept时，已完成连接队列中的队头项将返回给进程，或者如果该队列为空，那么进程将被投入睡眠，直到TCP在该队列放入一项才唤醒它。
## accept函数
accept函数由TCP服务器调用，用于从已完成队列队头返回下一个已完成连接。
如果已完成队列为空，那么进程将被投入睡眠（假定套接字为默认的阻塞方式）
```protobuf
#include <sys/socket.h>
int accept(int sockfd, struct sockaddr *cliaddr, socklen_t *addrlen);
```

- 返回值：成功为非负描述符，出错-1
- cliaddr：已连接的对端进程（客户）的协议地址
- addrlen：值-结果参数，调用前，将*addrlen所引用的整数值置为由cliaddr所指的套接字地址结构的长度，返回时，该整数值即为内核存放在该套接字地址结构内的确切字节数。

如果accept成功，其返回值时由内核自动生成的一个全新描述符，代表与返回客户的TCP连接。
本函数最多返回三个值

1. 可能新套接字描述符
2. 可能出错指示的整数
3. 客户进程的协议地址（*cliaddr）以及该地址的大小（*addrlen）

说明：

1. 一个服务器通常仅仅创建1个监听套接字，在服务器的生命期内一直存在。
2. 内核为每个服务器进程接受的客户连接创建一个已连接套接字（对于它的TCP三次握手已完成）。
3. 当服务器完成对某个给定客户的服务时，相应的已连接套接字就被关闭（关闭指调用`close`）。
## fork和exec函数
`fork`是Unix中派生新进程的唯一办法。
```protobuf
#include <unistd.h>
pid_t fork(void)
```

- 返回值：在子进程中为0，在父进程中为子进程ID，出错-1
   - 在调用进程（父进程）返回一次，返回值是新派生进程（子进程）的PID；
   - 在子进程又返回一次，返回值是0.
### Q：fork在子进程返回0而不是父进程的PID？
Ans：任何子进程只有一个父进程，子进程总是可以getppid获得父进程PID。相反，父进程可以只有许多子进程，而且无法获取各个子进程的PID。
如果父进程想要跟踪所有子进程的PID，则必须记录每次调用fork的返回值。
父进程中调用fork之前打开的所有描述符在fork返回之后由子进程分享。

1. 父进程调用accept之后调用fork。
2. 所接受的已连接套接字随后就在父进程与子进程之间共享。
3. 一般，子进程接着读写这个已连接套接字，父进程则关闭这个已连接套接字（关闭是指调用`close`）。
### fork两个典型用法

1. 一个进程创建一个自身的副本，每个副本都可以在另一个副本执行其他任务的同时处理各自的某个操作。
2. 一个进程想要执行另一个程序，就创建新进程的唯一办法是调用fork，该进程于是首先调用fork创建一个自身副本，然后其中一个副本（通常为子进程）调用exec把自身替换成新的程序。（像shell之类程序的典型用法）
### 介绍exec函数
存放在硬盘上的可执行程序文件能够被Unix执行的唯一办法：由现有进程调用6个exec函数中的一个。
exec把当前进程映像替换成新的程序文件，而且该新程序通常从main函数开始执行。进程ID不改变。
称调用exec的进程为调用进程（calling process），称新执行的程序为新程序（new program）
![image.png](https://cdn.nlark.com/yuque/0/2023/png/22241519/1689776601567-92140b12-1684-4b33-a42e-db78ffb9af14.png#averageHue=%23f3f3f3&clientId=uda2ebabe-7b59-4&from=paste&height=252&id=u8eac372a&originHeight=252&originWidth=856&originalType=binary&ratio=1&rotation=0&showTitle=false&size=64718&status=done&style=none&taskId=ud71ecd57-9be2-466e-90bd-027047ac1fc&title=&width=856)
一般只有execve是内核中的系统调用，其他是调用execve的库函数。
## close函数
通常的Unix `close`函数也用来关闭套接字，并终止TCP连接。
```protobuf
#include <unistd.h>
int close(int sockfd);
```

- 返回值：成功0，出错-1

close一个TCP套接字的默认行为是把该套接字标记成已关闭，然后立即返回到调用进程。可以用SO_LINGER套接字选项来改变这个默认行为。
### 描述符引用计数
并发服务器中父进程关闭已连接套接字只会让对应描述符的引用计数减1。
引用计数大于0，close调用并不引发TCP的四分组连接终止序列。
如果我们确实想关闭某个TCP连接，发送一个FIN，用shutdown函数，不用close。
如果父进程对每个由accept返回的已连接套接字都不调用close，那么TCP的四次挥手永远也不会发生，引用计数永远大于0，没有客户连接会被终止。
## 并发服务器
大多数TCP服务器是并发的，来一个客户连接就调用`fork`派生一个子进程。
### 并发服务器程序大致过程

1. 当一个连接建立时，accept返回，服务器接着调用fork，
   1. 连接被内核接受，新的套接字connfd（已连接套接字）被创建，由此跨连接读写数据。
   2. 并发服务器下一步调用fork，搞出一个子进程
   3. 此时listenfd和connfd这两个描述符都在父进程和子进程之间共享（被复制）
2. 然后由子进程服务客户（通过已连接套接字connfd），父进程则等待另一个连接（通过监听套接字listenfd）
   1. 这里父进程关闭已连接套接字，子进程关闭监听套接字，分别close。
   2. 引用计数减1
3. 此时新的客户由子进程服务，父进程就关闭已连接套接字（close）。
# I/O 复用：select和poll函数
TCP客户会同时处理两个输入：1标准输入（fgets） 2 TCP套接字
**出现的问题：**客户将阻塞在fgets期间，另一个客户数据到达，服务器忙不过来无法及时处理。
解决：I/O 多路复用，即同时监听N个客户，解决对多个I/O监听时，一个I/O阻塞影响其他I/O的问题。
I/O 复用（I/O multiplexing）：内核一旦发现进程指定的一个或多个I/O条件就绪，他就通知进程。
I/O复用的典型应用：

1. 客户处理多个描述符（交互式输入和网络套接字）
2. 一个客户同时处理多个套接字
3. 一个TCP服务器既要处理监听套接字，又要处理已连接套接字
4. 一个服务器既要处理TCP，又要处理UDP
5. 如果服务器要处理多个服务或者多个协议
## 五种I/O模型
**一个输入操作通常包括两个不同的阶段：**

1. 等待数据准备好 
   1. 等待数据从网络中到达
   2. 所有等待的分组到达时候，它被复制到内核中某个缓冲区
2. 从内核向进程复制数据
   1. 把数据从内核缓冲区复制到应用缓冲区

**五种I/O模型有**

1. blocking I/O
2. nonblocking I/O
3. I/O multiplexing
4. signal-driven I/O
5. asynchronous I/O
### 1 阻塞式I/O模型
默认情况，所有的套接字都是阻塞的。图示如下
![image.png](https://cdn.nlark.com/yuque/0/2023/png/22241519/1689779357438-84d3c65a-7160-4222-b597-4d646c057dad.png#averageHue=%23f7f7f7&clientId=uda2ebabe-7b59-4&from=paste&height=386&id=u80bca29e&originHeight=386&originWidth=678&originalType=binary&ratio=1&rotation=0&showTitle=false&size=64900&status=done&style=none&taskId=u1abee93f-4460-49ad-ac68-23af4d10a44&title=&width=678)

### 2 非阻塞式I/O模型
把套接字设置为非阻塞就是通知内核，所请求的I/O操作需要将进程投入睡眠时候才能完成，不要投入睡眠，而是返回一个错误。
如图，当一个进程像这样对一个非阻塞描述符循环调用recvfrom时，就叫轮询（polling）
![image.png](https://cdn.nlark.com/yuque/0/2023/png/22241519/1689779566129-f3a456b0-e9c0-4515-90ee-5152167798aa.png#averageHue=%23f2f2f2&clientId=uda2ebabe-7b59-4&from=paste&height=422&id=ubb7e2d26&originHeight=422&originWidth=757&originalType=binary&ratio=1&rotation=0&showTitle=false&size=111294&status=done&style=none&taskId=u281a9ed4-205b-4eb2-bfd0-728f22d8c5b&title=&width=757)
### 3 I/O复用模型
调用select或poll，阻塞在这两个系统调用某一个之上，而不是阻塞在真正的I/O系统调用上。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/22241519/1689779665235-7589f4ba-c9d0-4518-8409-270a75bac450.png#averageHue=%23f2f2f2&clientId=uda2ebabe-7b59-4&from=paste&height=416&id=u05c6084a&originHeight=416&originWidth=759&originalType=binary&ratio=1&rotation=0&showTitle=false&size=109642&status=done&style=none&taskId=u69fb4df4-e794-47fb-b868-756c1bab5b5&title=&width=759)
阻塞于select调用，等待数据报套接字可读。当返回可读时，就调用recvfrom把所读数据报复制到应用进程缓冲区。
select的优势在于可以等待多个描述符就绪。
### 4 信号驱动式I/O模型
用信号，让内核在描述符就绪时发送SIGIO信号通知。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/22241519/1689779828775-ef31656a-e365-4418-bb9c-4f86cc243d7c.png#averageHue=%23f7f7f7&clientId=uda2ebabe-7b59-4&from=paste&height=451&id=uf8a48e80&originHeight=451&originWidth=804&originalType=binary&ratio=1&rotation=0&showTitle=false&size=93108&status=done&style=none&taskId=u38d4c13b-670a-43c5-8b85-9b3a983f201&title=&width=804)
### 5 异步I/O模型
异步I/O模型是指内核会在I/O操作完成时通知你，等待I/O完成期间你不会被阻塞。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/22241519/1689780316848-9829e959-d6aa-462b-9fe7-e23aaf65b9c0.png#averageHue=%23f8f8f8&clientId=ufe7ed435-70df-4&from=paste&height=482&id=ua442647b&originHeight=482&originWidth=773&originalType=binary&ratio=1&rotation=0&showTitle=false&size=71826&status=done&style=none&taskId=u24f9f6a0-5691-4f1f-b0c9-ce01ae4ca4f&title=&width=773)
### 各种I/O模型对比

- 同步I/O操作（synchronous I/O opetaion）导致请求进程阻塞，直到I/O操作完成；
- 异步I/O操作（asynchronous I/O opetaion）不导致请求进程阻塞。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22241519/1689780447961-cad2a9a0-664e-4d94-bbe6-7efdecfed838.png#averageHue=%23f3f3f3&clientId=ufe7ed435-70df-4&from=paste&height=464&id=ua9677cf0&originHeight=464&originWidth=863&originalType=binary&ratio=1&rotation=0&showTitle=false&size=112557&status=done&style=none&taskId=u12b70622-4b75-4cb0-911d-14b165976d7&title=&width=863)
## select函数
`select`函数允许进程指示内核等待多个事件中的任何一个发生，并只在有一个或多个事件发生或经历一段指定的事件后才唤醒它。
**讲个例子：**
我们调用`select`，告知内核出现下面的几种情况的时候返回：

- 集合{1, 4, 5}中的任何描述符准备好读；
- 集合{2, 7}中的任何描述符准备好写；
- 集合{1, 4}中的任何描述符有异常条件等待处理；
- 已经历了10.2秒。

调用`select`告知内核对哪些描述符（读、写或异常条件）感兴趣以及等待多长时间。但不局限于套接字，任何描述符都可以用`select`来测试。
```protobuf
#include <sys/select.h>
#include <sys/time.h>
int select(int maxfdpl, fd_set *readset, fd_set *writeset, fd_set *exceptset,
					 const struct timeval *timeout);
```

- timeout：告知内核它等待的指定描述符中的任意一个就绪能花多长时间。有三种可能
   - 永远等待下去：只在有一个描述符准备好I/O的时候才返回（设置为空指针）
   - 等待一段固定时间：在有一个描述符准备好I/O时返回，但不超过设定的时间。
   - 根本不等等：**检查描述符后立即返回，这叫轮询（polling）(必须指向一个timeval结构，设定定时器值为0)**
- readset、writeset、exceptset：让内核测试读、写和异常条件的描述符。支持的异常条件只有俩
   - 某个套接字的带外数据的到达；
   - 某个已置为分组模式的伪终端存在可从其主端读取的控制状态信息。
### 描述符就绪条件
满足下列4个条件中的任何一个时，一个套接字准备好**读**

1. 该套接字接收缓冲区中的**数据字节数**大于等于套接字接收缓冲区低水位标记的当前大小。
2. 连接的读半部关闭（就是接收了FIN的TCP连接）。对这样子的套接字的读操作将不阻塞并返回0（就是返回EOF）
3. 套接字时一个监听套接字且已完成的连接数不是0.
4. 其上有一个套接字错误待处理。
   1. 这样的套接字的读操作将不阻塞并返回-1（就是返回一个错误）
   2. 同时把errno设置成确切的错误条件。
   3. 待处理错误（pending error）

满足下面4个条件中的任何一个时，一个套接字准备好写。

1. 该套接字发送缓冲区中的**可用空间字节数**大于等于套接字发送缓冲区低水位标记的当前大小，并且有下面俩条件的其中一个
   1. 该套接字已连接
   2. 该套接字不需要连接（UDP套接字）
2. 该连接的写半部关闭。
   1. 对这种套接字的写操作将产生SIGPIPE信号
3. 使用非阻塞式`connect`的套接字已建立连接，或者`connect`已经以失败告终。
4. 其上有一个套接字错误待处理。
   1. 这样的套接字的**写操作**将不阻塞并返回-1（就是返回一个错误）
   2. 同时把errno设置成确切的错误条件。

如果一个套接字存在带外数据或者还处于带外标记，那么它有异常条件待处理。
下图汇总了上述导致`select`返回某个套接字就绪的条件：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/22241519/1689866089469-b86ec7df-c884-47a1-a520-2bcf1342d700.png#averageHue=%23ebebeb&clientId=ucb60fe23-2424-4&from=paste&height=255&id=u3295e7a7&originHeight=255&originWidth=639&originalType=binary&ratio=1&rotation=0&showTitle=false&size=60769&status=done&style=none&taskId=u90892e30-58f0-48d7-975a-39284357188&title=&width=639)
### `select`的最大描述符数
大多数应用不会用到很多描述符，如果有那他也往往用`select`来复选描述符。
现在Unix版本允许每个进程使用事实上无限制的描述符（会受限于内存总量和管理性限制）
## poll函数
`poll`函数可用工作在任何描述符上。
`poll`函数提供的功能与`select`类似，不过在处理流设备时，它能提供额外的信息。
**一个进程或线程同时监视多个文件描述符的状态，以确定是否有可读、可写或异常等事件发生，而无需阻塞在单个I/O操作上。**
**在解释一下**
当调用`**poll()**`函数时，它会阻塞当前进程或线程，等待指定的文件描述符上发生感兴趣的事件。`**poll()**`函数使用**struct pollfd**数组来传递要监视的文件描述符和感兴趣的事件，并将实际发生的事件填充到同样的数组中的**revents**字段中。
poll的机制与select类似，与select在本质上没有多大差别，管理多个描述符也是进行轮询，根据描述符的状态进行处理，但是poll没有最大文件描述符数量的限制。
```protobuf
#include <poll.h>
int poll(struct pollfd *fdarray, unsigned long nfds, int timeout)
```

- 返回值：若有就绪描述符则为其数目，若超时为0，出错-1
- nfds：设置结构数组fdarray中的元素的个数
- timeout：指定`poll`函数返回前等待多长时间。
   - INFTIM：永远等待
   - 0：立即返回，不阻塞进程
   - >0：等待指定数目的毫秒数
- fdarray：指向一个结构数组第一个元素的指针，每个数组元素都是一个`pollfd`结构，用于指定测试某个给定描述符fd的条件。
```protobuf
struct pollfd {
  int fd;         /* descriptor to check */
  short events;   /* events of interest on fd */
  short revents;  /* events that occurred on fd */
}
```
测试条件由`events`成员指定，函数在相应的`revents`成员中返回该描述符的状态。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/22241519/1689867085773-7aaaf6ab-9fe1-4409-9af6-82ca823eeae1.png#averageHue=%23ededed&clientId=ucb60fe23-2424-4&from=paste&height=323&id=ue7a7f4bc&originHeight=323&originWidth=713&originalType=binary&ratio=1&rotation=0&showTitle=false&size=97988&status=done&style=none&taskId=u4ef13775-3fb0-443c-afb2-66701c609fb&title=&width=713)
**这个图的意思是**三个部分

1. 第一部分处理输入的四个常值
2. 第二部分处理输出的三个常值
3. 第三个部分处理错误的三个常值

其中第三部分的三个不能在`events`中设置，但是相应条件存在就需要在`revents`中返回。
`poll`识别三类数据：普通（normal）、优先级带（priority band）和高优先级（high priority），这些听不懂的术语都是基于流的实现。
什么情况下设置这些标志呢？

- 所有正规TCP数据和UDP数据都被认为是普通数据
- TCP读半部关闭时，也被认为是普通数据，随后读操作将返回0
- TCP连接存在错误也可以认为是普通数据，随后的读操作返回-1，错误码从errno中获得
- 监听套接字上有新的连接也可以认为是普通数据
