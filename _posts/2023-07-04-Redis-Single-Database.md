---
title: "Redis 单机数据库的实现"
categories:
  - Redis
tags:
  - Redis
  - Java
toc: true
---
# 数据库
## 服务器中的数据库
```c
struct redisServer {
    // ...
    // 一个数组，保存着服务器中的所有数据库
    redisDb *db;
    // ...
    // 服务器的数据库数量，默认16个
    int dbnum;
    // ...
};
```
初始化服务器，会根据 dbnum 属性来决定创建多少个数据库
## 切换数据库
客户端切换0号数据库命令 `SELECT 0`
```c
typedef struct redisClient {
    // ...
    // 记录客户端当前正在使用的数据库
    redisDb *db;
    // ...
} redisClient;
```
redisClient.db 指针指向 redisServer.db 数组中的其中一个元素，就是客户端的目标数据库。
## 数据库键空间
```c
typedef struct redisDb {
    // ...
    // 数据库键空间，保存着数据库中的所有键值对
    dict *dict;
    // ...
} redisDb;
```
redisDb 结构的 dict 字典保存了数据库中的所有键值对，我们称之为键空间（key space）;

- 键空间的键也就是数据库的键，每个都是一个字符串对象。
- 键空间的值就是数据库的值，每个值可以是任意一种 Redis 对象。

**添加、删除、更新、取值等操作**

1. **添加新键：**实际就是将一个新键值对添加到键空间字典中（键是字符串对象，值是任意一种 Redis 对象） `SET`
2. 删除键：就是删除键空间的键值对对象 `DEL`
3. 更新键：`SET`
4. 取值：`GET`
5. 清空整个数据库：删除键空间所有的键值对`FLUSHDB`
6. 随机返回数据库中某个键：`RANDOMKEY`
## 设置键的生存时间或过期时间
`EXPIRE`或者`PEXPIRE`，客户端可以以秒或者毫秒精度为数据库中的某个键设置生存时间（Time To Live，TTL），经过TTL之后，服务器就会自动删除生存时间为0的键；
`EXPIREAT`或者`PEXPIREAT`以秒或毫秒给数据库的某个键设置过期时间（expire time），过期时间是一个 UNIX 时间戳，当键的过期时间来临时，服务器就会自动从数据库删除这个键。
`TTL`和`PTTL`检查键的剩余生存时间，返回距离被服务器自动删除还有多长时间。
`PERSIST`移除一个键的过期时间，在过期字典中查找给定的键，并解除键和值（过期时间）在过期字典中的关联。

- `过期键的判定：`
   1. 检查键是否存在于过期字典：如果存在，那么取得键的过期时间。
   2. 检查当前 UNIX 时间戳是否大于键的过期时间：如果是的话，那么键已经过期；否则未过期。
## 过期键删除策略

1. 定时删除：在设置键的过期时间的同时，创建一个定时器，让定时器在过期时间来临时，立即执行对键的删除操作。
2. 惰性删除：放任过期不管，但是每次从键空间获取键时，都检查取得的键是否过期，如果过期的话，就删除该键；如果没有过期，就返回该键。
3. 定期删除：每隔一段时间，程序就对数据库进行一次检查，删除里面的过期键。至于要删除多少过期键，以及要检查多少个数据库，则由算法决定。
## Redis 的过期键删除策略
Redis 使用的时惰性删除和定期删除两种策略。
## AOF、RDB 和复制功能对过期键的处理

1. 生成 ADB 文件：执行 `SAVE`或者`BGSAVE`创建一个新的 RDB 文件时 ，程序会对数据库中的键进行检查，已过期的键不会被保存到新创建的 RDB 文件中。
2. 载入 RDB 文件：
   1. 主服务器模式运行：载入 RDB 文件时，会检查保存的键，未过期的会被载入，过期的则忽略。
   2. 从服务器模式运行：载入 RDB文件时，不论过期都载入。
3. AOF 文件写入：AOF 持久化模式运行时，键过期但没被删除，不会影响 AOF 文件；但是过期被删除后，程序会向 AOF 文件追加一条 `DEL`命令，来显示地记录该键已被删除。
4. AOF 重写：执行 AOF 重写过程中，程序会检查的，过期的键不会保存到重写后的 AOF 文件中。
5. 复制：由主服务器来控制从服务器统一地删除过期键，保证主从服务器数据的一致性。
## 数据库通知
当 Redis 命令对数据库进行修改之后，服务器会根据配置向客户端发送数据库通知。

1. 键空间通知（key-space notification） 某个键执行了什么命令
2. 键事件通知（key-event notification） 某个命令被什么键执行了
# RDB 持久化
RDB 持久化功能所生成的 RDB 文件是一个经过压缩的二进制文件，通过该文件可以还原生成 RDB 文件时的数据库状态。
## RDB 文件的创建与载入
`SAVE`和`BGSAVE`可以用于生成 RDB 文件。
`SAVE`会阻塞 Redis 服务器进程，直到 RDB 文件创建完毕为止，在阻塞期间，服务器不能处理任何命令请求；
`BGSAVE`会派生出一个子进程，由子进程负责创建 RDB 文件，服务器进程（父进程）继续处理命令请求。
## 自动间隔性保存
配置 save 选项的保存条件，自动执行 `BGSAVE`命令。
默认的是 `save 900 1`、`save 300 10`、`save 60 10000`。
`struct saveparam *saveparams;`数组，每个元素都保存一个 save 选项设置的保存条件；
`long long dirty;`计数器记录距离上次成功执行 save 或 bgsave 命令之后，服务器对数据库状态（服务器中的所有的数据库）进行了多少次修改；
`time_t lastsave;`UNIX 时间戳，记录了服务器上一次成功执行 save 或 bgsave 的事件。
Redis 服务器周期性操作函数 serverCron 默认每隔 100 ms就会执行一次，检查 save 选项所设置的保存条件是否满足，满足则执行 bgsave；
## RDB 文件结构
各个部分如下
`REDIS`、`db_version`、`databases`、`EOF`、`check_sum`

1. `REDIS`5字节保存“REDIS”五个字符，在载入文件时，快速检查是否为 RDB 文件；
2. `db_version`4字节，记录 RDB 文件的版本号；
3. `databases`包含0或任意多个数据库，以及各个数据库中的键值对数据；
   1. 组成：`SELECTDB`、`db_number`、`key_value_pairs`
   2. `SELECTDB`1字节标志接下来读入的时一个数据库号码
   3. `db_number`1/2/5字节，调用 select 命令，切换数据库
   4. `key_value_pairs`保存了所有的键值对
4. `EOF`1字节，这个常量标志RDB文件正文内容介绍，读入程序到这，所有的键值对都载入完毕；
5. `check_sum`8字节无符号整数，保存一个校验和，对前面四部分的内容计算出来的，检查出错或者损坏情况。
# AOF 持久化
AOF 持久化 Append Only File 通过保存 Redis 服务器所执行的写命令来记录数据库状态的。

1. AOF 功能分命令追加（append）、文件写入、文件同步（sync）三个步骤
   1. 命令追加：服务器执行完一个写命令之后，会以协议格式将被执行的写命令追加到服务器状态的 `aof_buf`缓冲区的末尾；
   2. 写入与同步：不同 appendfsync 值会删除不同的持久化行为
      1. always 将 aof_buf 缓冲区中的所有内容写入并同步到 AOF 文件
      2. everysec 将 aof_buf 缓冲区中的所有内容写入到 AOF 文件，如果上次同步 AOF 文件事件距离现在超过1s，那么再对 AOF 文件进行同步，并且这个同步操作时由一个线程专门负责执行的。
      3. no 将 aof_buf 缓冲区中的所有内容写入到 AOF 文件，但不对 AOF 文件进行同步，何时同步由操作系统来决定。
2. AOF 文件的载入与数据还原

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22241519/1688396305274-f6011afc-ab8d-4b7e-af56-eed3ddef59be.png#averageHue=%23eeeeee&clientId=u87ed6a2f-8d93-4&from=paste&height=338&id=u722fb5f3&originHeight=338&originWidth=399&originalType=binary&ratio=1&rotation=0&showTitle=false&size=69214&status=done&style=none&taskId=u4f3ab78d-aafc-49ed-a07a-51284d28c84&title=&width=399)

3. AOF 重写：解决 AOF 文件体积膨胀的文件，Redis 提供了 AOF 文件重写功能；是通过读取数据库中的键值对来实现的，程序无需对现有 AOF 文件进行任何读入、分析或者写入操作。
4. 在执行 `BGREWRITEAOF`命令时，Redis 服务器会维护一个 AOF 重写缓冲区，该缓冲区会在子进程创建新 AOF 文件期间，记录服务器执行的所有写命令。当子进程完成创建新 AOF 文件的工作之后，服务器会将重写缓冲区中的所有内容追加到新 AOF 文件的末尾，使得新旧两个 AOF 文件所保存的数据库状态一致。最好，服务器用新的 AOF 文件替换旧的 AOF 文件，以此来完成 AOF 文件重写操作。
# 事件
Redis 服务器时一个事件驱动程序，服务器需要处理以下两类事件：

1. **文件事件（file event）**：Redis 服务器通过套接字与客户端（或者其他 Redis 服务器）进行连接，而文件事件就是服务器对套接字操作的抽象。服务器与客户端（或者其他服务器）的通信会产生相应的文件事件，而服务器则通过监听并处理这些事件来完成一系列网络通信操作。
2. **时间事件（time event）**：Redis 服务器中的一些操作（比如 serverCron 函数）需要在给定的事件点执行，而时间事件就是服务器对这类定时操作的抽象。
## 文件事件
文件事件处理器（file event handler）：Redis 基于 Reactor 模式开发了自己的网络事件处理器；

1. 使用 I/O 多路复用（multiplexing）程序来同时监听多个套接字，并根据套接字目前执行的任务来为套接字关联不同的事件处理器。
2. 当被监听的套接字准备好执行连接应答（accept）、读取（read）、写入（write）、关闭（close）等操作时，与操作相对应的文件事件就会产生，这时文件事件处理器就会调用套接字之间关联好的事件处理器来处理这些事件。

文件事件处理器构成
![image.png](https://cdn.nlark.com/yuque/0/2023/png/22241519/1688397593558-b9dec125-5733-456b-a202-82d0a70a8bb6.png#averageHue=%23eeeeee&clientId=u87ed6a2f-8d93-4&from=paste&height=310&id=u5337ddb4&originHeight=310&originWidth=417&originalType=binary&ratio=1&rotation=0&showTitle=false&size=77373&status=done&style=none&taskId=u9799d40f-dd78-49d8-bcf0-2d7c2e9c5dc&title=&width=417)
I/O 多路复用程序的实现：都是通过包装常见的 select、epoll、evport 和 kqueue 这些 I/O 多路复用函数库来实现的。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/22241519/1688397723209-b81e231b-34af-4a21-9ad6-b2cae1ea5339.png#averageHue=%23ebebeb&clientId=u87ed6a2f-8d93-4&from=paste&height=137&id=u2ce8461b&originHeight=137&originWidth=381&originalType=binary&ratio=1&rotation=0&showTitle=false&size=33980&status=done&style=none&taskId=u776b42ef-256f-4fc4-adaa-eaca4468ee9&title=&width=381)
问？[套接字是什么玩意](https://zh.wikipedia.org/wiki/%E7%B6%B2%E8%B7%AF%E6%8F%92%E5%BA%A7)

1. 连接应答处理器：对连接服务器的各个客户端进行应答
2. 命令请求处理器：接收客户端传来的命令请求
3. 命令回复处理器：向客户端返回命令的执行结果
4. 复制处理器：主服务器和从服务器进行复制
## 时间事件
**Redis 的时间事件分为以下两类：**

1. 定时事件：让一段程序在指定的时间之后执行一次，当前时间的30秒后执行一次
2. 周期性事件：让一段程序每隔指定时间执行一次，每隔30ms执行一次

**时间事件的组成：**

1. `id` 服务器为时间事件创建的全局唯一ID（标识号）
2. `when`毫秒精度的 UNIX 时间戳，记录时间事件的到达（arrive）时间
3. `timeProc`时间事件处理器，一个函数。

**实现：**Redis 服务器将所有时间事件都放在一个**无序链表**中，每当时间事件执行器运行时，就遍历整个链表，查找所有已到达的时间事件，并调用相应的事件处理器。
时间事件应用实例：`serverCron` 函数
## 事件的调度与执行
服务器中同时存在文件事件和时间事件，事件调度执行由 `ae.c/aeProcessEvents` 函数负责，事件处理角度下的服务器运行流程：
`ae.c/aeProcessEvents `简单描述：

1. 拿到到达时间离当前时间最接近的时间事件 `time_event`
2. 计算出 `time_event` 距离到达还有多少毫秒 `remaind_ms`
3. 如果 `time_event` 已经到达了，那么 `remaind_ms`的值可能为负数，设定为 0
4. 根据`remaind_ms`的值，创建 `timeval` 结构
5. 堵塞并等待文件事件产生，最大阻塞事件 由 `timeval`结构决定，如果`remaind_ms`的值为0，那么这里的`aeApiPoll`执行完立马返回，不阻塞
6. 然后处理所有已产生的文件事件`processFileEvents()`
7. 然后处理所有已到达的时间事件`processTimeEvents()`

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22241519/1688476093131-43d4ca6e-c4bc-4e43-85b5-93a459d10981.png#averageHue=%23efefef&clientId=ue7670c87-cfbc-4&from=paste&height=301&id=u55abf397&originHeight=301&originWidth=416&originalType=binary&ratio=1&rotation=0&showTitle=false&size=60684&status=done&style=none&taskId=ua32fa89b-4662-4fe7-a8d9-2925c162eea&title=&width=416)
文件事件和时间事件是合作关系，服务器会轮流处理这两种事件，并且处理事件的过程中不会进行抢占。
时间事件的实际处理时间通常会比设定的到达时间晚一些。
# 客户端
Redis 服务器是典型的**一对多**服务器程序：一个服务器可以与多个客户端建立网络连接，每个客户端可以向服务器发送命令请求，而服务器则接收并处理客户端发送的命令请求，并向客户端返回命令回复。
使用** I/O 多路复用技术**实现的文件事件处理器，**单线程单进程**的方式处理命令请求，并与多个客户端进行网络通信。
```c
struct redisServer {
    // ...
    // 一个链表，保存了所有客户端状态
    list *clients;
    // ...
}
```
Redis 服务器状态结构的 clients 属性是一个**链表**，保存了所有与服务器链接的客户端的状态结构。
## 客户端属性
客户端状态包含的属性可以分两类：一类是比较通用的属性，另一类是和特定功能相关的属性
```c
typedef struct redisClient {
    // ...
    int fd;
    robj *name;
    // 记录客户端的角色
    int flags;
    // 输入缓冲区，最大不超过1GB，否则关闭该Client
    // Client 发送到Server的命令放在这里
    sds querybuf;
    // 分析出来的命令参数和个数
    robj **argv;
    int argc;
    // 命令的实现函数
    struct redisCommand *cmd;
    // 输出缓冲区，固定的输出缓冲区
    char buf[REDIS_REPLY_CHUNK_BYTES];
    int bufpos;
    // 可变大小的输出缓冲区
    list *reply;
    // 身份验证，0未通过，1通过
    int authenticated;
    // 时间相关属性
    time_t ctime;
    time_t lastinteraction;
    time_t obuf_soft_limit_reached_time;
    // ...
} redisClient;
```
## 客户端的创建与关闭

1. **创建普通客户端：**Client 通过网络与服务器连接，在客户端使用`connect`函数连接服务器，服务器调用连接事件处理器并为客户端创建相应的客户端状态，然后把这个新的客户端状态添加到**服务器状态结构**`**clients**`**链表的末尾。**
2. **关闭普通客户端：**。。。
3. **lua 脚本的伪客户端：**`redisClient *lua_client;`伪客户端在服务器运行的整个生命周期一直存在，直到服务器关闭，才关闭。
4. **AOF 文件的伪客户端：**服务器在载入 AOF 文件时，会创建一个伪客户端，载入完成就关闭。
# 服务器
Redis 服务器作用：

1. 负责与多个客户端建立网络连接
2. 处理命令请求
3. 在数据库中保存执行命令所产生的数据，并通过资源管理来维持服务器自身运转
## 命令请求的执行过程

1. 客户端将命令请求发送给服务器；
   1. 用户键入一个命令请求到客户端
   2. 客户端将命令请求转换成协议格式，发送到服务器
2. 服务器读取命令请求，并分析出命令参数；
   1. 因为客户端写入，客户端与服务端的连接套接字变得可读
   2. 读取套接字中协议格式的命令请求，并将其保存到客户端状态的输入缓冲区
   3. 对输入缓冲区的命令请求进行分析，提取其中的命令参数和参数个数，分别保存到客户端状态的 `argv` 和 `argc` 属性中
   4. 调用命令执行器，执行客户端指定的命令
3. 命令执行器根据参数查找命令的实现函数，然后执行实现函数并得出命令回复；
   1. 命令执行器：根据客户端状态的 `argv[0]` 参数，在命令表中查找参数所指定的命令，并保存到客户端状态的`cmd`属性中；命令表是一个字典，键是命令名字，值是一个个 `redisCommand` 结构，`cmd` 指向的就是 `redisCommand` 结构
   2. 命令执行器：执行预备操作验证，检查等等
   3. 命令执行器：调用命令的实现函数 `client->cmd->proc(client);`
   4. 命令执行器：执行后续工作，写日志，ROF 持久化，复制等
4. 服务器将命令回复返回给客户端。
   1. 命令实现函数将命令回复保存到客户端输出缓冲区
   2. 为客户端的套接字关联命令回复处理器
   3. 在客户端套接字可写时，服务器执行命令回复处理器，将客户端缓冲区中的命令回复（协议格式）发送给客户端。
   4. 协议格式`+OK\r\n`
## serverCron 函数
这个函数默认每隔100ms执行一次，管理服务器资源，保持服务器自身运转良好。
一些函数执行的操作和有关属性

1. 更新服务器时间缓存
2. 更新LRU时钟
3. 更新服务器每秒执行命令次数：抽样计算的方式，估算并记录服务器在最近一秒钟处理的命令请求数量
4. 更新服务器内存峰值记录 `stat_peak_memory`
5. 处理`SIGTERM`信号
6. 管理客户端资源：两个检查1连接是否超时2输入缓冲区是否过大
7. 管理数据库资源：删除过期键，收缩字典等待
8. 执行被延迟的`BGREWRITEAOF`
9. 检查持久化操作的运行状态
10. 将 AOF 缓冲区的内容写入 AOF 文件
11. 关闭异步客户端
12. 增加 `cronloops`计数器的值
## 初始化服务器
服务器从启动到能够处理客户端命令请求还要以下步骤

1. 初始化服务器状态
2. 载入服务器配置
3. 初始化服务器数据结构
4. 还原数据库状态
   1. 有 AOF 持久化功能，则先用 AOF 文件还原
   2. 否，用 RDB 文件还原数据库状态
5. 执行事件循环
