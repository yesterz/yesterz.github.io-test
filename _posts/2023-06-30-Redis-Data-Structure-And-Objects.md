# 简单动态字符串 sinple dynamic string, SDS
## SDS 的定义
```c
struct sdsdr {
    // 记录 buf 数组中已使用的字节的数量
    // 等于 SDS 所保存字符串的长度
    int len;

    // 记录 buf 数组中未使用字节的数量
    int free;

    // 字节数组，用于保存字符串
    char buf[];
}
```
## SDS 与 C 字符串的区别

1. 常数复杂度 O(1) 获取字符串长度
2. 杜绝缓冲区溢出的问题
3. 减少修改字符串长度时所需的内存重新分配的次数
   1. 空间预分配策略：连续增长 N 次字符串所需要的内存分配次数必定 N 次**降低为最多 N 次**
   2. 惰性空间释放策略：用于优化 SDS 的字符串缩短操作，把多出来的字节用 free 属性记录起来，等待将来使用
4. 二进制安全（binary-safe）可以保存文本或者二进制数据
5. 兼容部分 C 字符串函数
# 链表
## 链表和链表节点的实现
```c
typedef struct listNode {
    // 前置节点
    struct listNode *prev;
    // 后置节点
    struct listNode *next;
    // 节点的值
    void *value;
}listNode;
```
多个 `listNode` 可以通过 prev 和 next 指针组成双端链表。
`list` 结构和 `listNode` 结构组成的链表示意图如下
![image.png](https://cdn.nlark.com/yuque/0/2023/png/22241519/1688295927751-381ccdc3-0234-47ee-bfcf-131b2c7da72c.png#averageHue=%23f2f2f2&clientId=u15a65edc-38b9-4&from=paste&height=230&id=u2edfdb21&originHeight=230&originWidth=599&originalType=binary&ratio=1&rotation=0&showTitle=false&size=63689&status=done&style=none&taskId=u1af440c5-91bc-4f4b-b21e-df5fe6d30fe&title=&width=599)
## Redis 的链表实现的特性总结如下：

- **双端：**链表节点带 `prev` 和 `next` 指针，获取某个节点的前置节点和后置节点的复杂度都是 O(1)。
- **无环：**表头节点的 `prev` 指针和表尾节点的 `next` 指针都指向 `NULL`，对链表的访问以 `NULL` 为终点。
- **带表头指针和表尾指针：**通过 `list` 结构的 head 指针和 tail 指针，程序获取链表的表头节点和表尾节点的复杂度为 O(1)。
- **带链表长度计数器：**程序使用 list 结构的 len 属性来对 list 持有的链表节点进行行计数，程序获取链表中节点数量的复杂度为 O(1)。
- **多态：**链表节点使用 void* 指针来保存节点值，并且可以通过 list 结构的 dup、free、match 三个属性为节点值设置类型特定函数，所以链表可以用于保存各种不同类型的值。
# 字典 dictionary
字典，又称为符号表（symbol table）、关联数组（associative array）或映射（map），是一种用于保存键值对（key-value pair）的抽象数据结构。
## 字典的实现
Redis 的字典使用哈希表作为底层实现，一个哈希表里面可以有多个哈希表节点，而每个哈希表节点就保存了字典中的一个键值对。
### 哈希表
```c
typedef struct dictht {
    // 哈希表数组
    dictEntry **table;
    // 哈希表大小
    unsigned long size;
    // 哈希表大小掩码，用于计算索引值
    // 总是等于 size-1
    unsigned long sizemask;
    // 该哈希表已有节点的数量
    unsigned long used;
} dictht;
```
table 属性是一个数组，数组中的每一个元素都是一个指向 dict.h/dictEntry 结构的指针，每个 dictEntry 结构保存着一个键值对。
### 哈希表节点
```c
typedef struct dictEntry {
    // 键
    void *key;
    // 值
    union{
        void *val;
        uint64_t u64;
        int64_t s64;
    } v;
    // 指向下个哈希表节点，形成链表
    struct dictEntry *next;
} dictEntry;
```
`next`属性是指向另一个哈希表节点的指针，这个指针可以将多个哈希值相同的键值对连接在一起，来解决键冲突（collision）的问题。
### 字典的实现
```c
typedef struct dict {
    // 类型特定函数
    dictType *type;
    // 私有数据
    void *privdata;
    // 哈希表
    dictht ht[2];
    // rehash 索引
    // 当 rehash 不在进行时，值为 -1
    int rehashidx;
    /* rehashing not in progress'if rehashidx == -1*/
}
```
ht 属性是一个包含两个项的数组，数组中的每个项都是一个 dictht 哈希表，一般情况下，字典只使用 ht[0] 哈希表，ht[1] 哈希表只会在对 ht[0] 哈希表进行 rehash 时使用。
## 哈希算法
当要将一个新的键值对添加到字典里面时，程序需要

   1. 先根据键值对的键计算出哈希值和索引值
   2. 然后再根据索引值，将包含新键值对的哈希表节点放到哈希表数据的指定索引上面。

当字典被用作数据库的底层实现，或者哈希键的底层实现时，Redis 使用 [MurmurHash2 算法](http://code.google.com/p/smhasher)来计算键的哈希值
## 解决键冲突
当两个或两个以上数量的键被分配到了哈希表数组的同一个索引上面时，我们称键发生了冲突（collision）。
**使用链地址法（separate chaining）来解决**
每个哈希表节点都有一个 next 指针，多个哈希表节点可以用 next 指针构成一个单向链表，被分配到同一个索引上的多个节点可以用这个单向链表连接起来，就解决了键冲突的问题。
## 重新散列 rehash
让哈希表的负载因子（load factor）维持在一个合理的范围之内，当哈希表保存的键值对数量太多或者太少时，程序需要对哈希表的大小进行相应的扩展或者收缩。通过执行 `rehash` 重新散列操作来完成扩展或收缩。
Redis 对字典的哈希表执行 rehash 的步骤如下：

1. 为字典的 ht[1] 哈希表分配空间，这个哈希表的空间大小取决于要执行的操作，以及 ht[0] 当前包含的键值对数量（也就是 ht[0].used 属性的值）：
   1. 如果执行的是扩展操作，那么 ht[1] 的大小作为第一个大于等于 ht[0].used*2 的 2^n（2 的 n 次方幂）；
   2. 如果执行的是收缩操作，那么 ht[1] 的大小为第一个大于等于 ht[0].used 的 2^n。
2. 将保存在 ht[0] 中的所有键值对 rehash 到 ht[1] 上面：rehash 指的是重新计算键的哈希值和索引值，然后将键值对放置到 ht[1] 哈希表的指定位置上。
3. 当 ht[0] 包含的所有键值对都迁移到了 ht[1] 之后（ht[0] 变为空表），释放 ht[0]，将 ht[1] 设置为 ht[0]，并在 ht[1] 新创建一个空白哈希表，为下一次 rehash 做准备。

**条件：**

1. 服务器目前没有在执行 bgsave 命令或者 bgrewriteaof 命令，并且哈希表的负载因子大于等于 1
2. 服务器目前正在执行 bgsave 命令或者 bgrewriteaof 命令，并且哈希表的负载因子大于等于 5；
3. 其中哈希表的负载因子公式： load_factor = ht[0].used / ht[0].size

负载因子 = 哈希表已保存节点数量 / 哈希表大小
## 渐进式重新散列 rehash
rehash 动作并不是一次性、集中式地完成的，而是分多次、渐进式地完成的。
**原因：键值对存储数量过于庞大，如果一次性 rehash 到 ht[1] 的话，会导致服务器在一段时间内停止服务。**
**哈希表渐进式 rehash 的详细步骤：**

1. 为 ht[1] 分配空间，让字典同时持有 ht[0] 和 ht[1] 两个哈希表。
2. 在字典中维持一个索引计数器变量 rehashidx，并将它的值设置为0，表示 rehash 工作正式开始。
3. 在 rehash 进行期间，每次对字典执行添加、删除、查找或者更新操作时，程序除了执行指定的操作以外，还会顺带将 ht[0] 哈希表在 rehashidx 索引上的所有键值对 rehash 到 ht[1] ，当 rehash 工作完成之后，程序将 rehashidx 属性值加一。
4. 随着字典操作的不断进行，最终在某个时间点上，ht[0] 的所有键值对 rehash 到 ht[1] ，这时程序将 rehashidx 属性设置为 -1，表示 rehash 操作已完成。
# 跳跃表 skiplist
一种有序数据结构，它通过**在每个节点维持多个指向其他节点的指针**，从而达到快速访问节点的目的。
Redis 使用跳跃表作为有序集合键的底层实现之一，如果一个有序集合包含的元素数量比较多，又或者有序集合中的元素成员（member）是比较长的字符串时，Redis 就会使用跳跃表来作为有序集合键的底层实现。
## 跳跃表的实现
由 zskiplistNode 表示跳跃表节点，zskiplist 结构用于保存跳跃表节点的相关信息。
### 跳跃表节点结构
```c
typedef struct zskiplistNode {
    // 层
    struct zskiplistLevel {
        // 前进指针
        struct zskiplistNode *forward;
        // 跨度
        unsigned int span;
    } level[];
    // 后退指针
    struct zskiplistNode *backward;
    // 分值
    double score;
    // 成员对象
    robj *obj;
} zskiplistNode;
```

1. level
   1. level 数组中包含多个元素，每个元素都包含一个指向其他节点的指针，程序可以通过这些层来加快访问其他节点的速度。一般来说，层的数量越多，访问其他节点的速度就越快。
   2. 层高都是 1-32 之间的随机数
2. forward
3. backward
4. span
   1. level[i].span 用于记录两个节点之间的距离
   2. 两个节点之间的跨度越大，相距得越远。
   3. 指向 NULL 的所有前进指针的跨度都为 0，因为它们没有连向任何节点。
5. score & obj
   1. sore 是一个 double 类型的浮点数，跳跃表中的所有节点都按照分值从小到大来排序
   2. 节点的成员对象 obj 是一个指针，它指向一个字符串对象，而字符串对象保存着一个 SDS 值。
### 跳跃表结构
```c
typedef struct zskiplist {
    // 表头节点和表尾节点
    strcut zskiplistNode *header, *tail;
    // 表中节点的数量
    unsigned long length;
    // 表中层数最大的节点的层数
    int level;
} zskiplist;
```
# 整数集合 intset
## 整数集合的实现
```c
typedef struct intset {
    // 编码方式
    uint32_t encoding;
    // 集合包含的元素数量
    uint32_t length;
    // 保存元素的数组
    int8_t contents[];
} intset;
```
## 升级 upgrade
把一个新元素添加到整数集合里面，并且新元素的类型比整数集合现有所有的元素的类型都要长时，就需要先升级才能添加。
**三步走：**

1. 根据新元素的类型，扩展整数集合底层数组的空间大小，并为新空间分配空间。
2. 将底层数组现有的所在元素都转换成于新元素相同的类型，并将类型转换后的元素放置到正确的位上，而且在放置元素的过程中，需要继续维持底层数组的有序性质不变。
3. 将新元素添加到底层数组里面。

**升级的好处：**提高灵活性 & 节约内存
## 降级 不支持
# 压缩列表 ziplist
压缩列表时列表键和哈希键的底层实现之一。当一个列表键只包含少量列表项，并且每个列表项要么就是小整数值，要么就是长度比较短的字符串，那么 Redis 就会使用压缩列表来做列表键的底层实现。
## 压缩列表的构成
`zlbytes`、`zltail`、`zllen`、`entry1`、...、`entryN`、`zlend`
ziplist 是为了节约内存而开发的，是由一系列特殊编码的连续内存块组成的顺序型（sequential）数据结构。一个压缩列表可以包含任意多个节点（entry）、每个节点可以保存一个字节数组或者一个整数值。
## 压缩列表节点的构成
`previout_entry_length`、`encoding`、`content`
每个压缩列表节点可以保存一个字节数组或者一个整数值

1. `previous_entry_length` 记录压缩列表中前一个节点的长度；
2. `encoding` 记录节点 content 属性所保存数据的类型以及长度；
3. `content` 负责保存节点的值
## 连锁更新 cascade update
添加新节点到压缩列表，或者从压缩列表中删除节点，可能会引发连锁更新操作，但这种操作出现的机率并不高。
连续多次空间扩展操作 
# 对象
Redis 基于上述介绍的数据结构创建了一个对象系统，这个系统包含字符串对象、列表对象、哈希对象、集合对象、有序集合对象，每种对象都用到了至少一种上述总结的数据结构。
## 对象的类型与编码
Redis 使用对象来表示数据库中的键和值，每次当我们在 Redis 的数据库中新创建一个键值对时，我们至少会创建两个对象，一个对象用作键值对的键（**键对象**），另一个对象用作键值对的值（**值对象**）
```c
typedef struct redisObject {
    // 类型
    unsigned type:4;
    // 编码
    unsigned encoding:4;
    // 指向底层实现数据结构的指针
    void *ptr;
    // ...
} robj;
```
### 类型
对应 Redis 数据库保存的键值对来说，键总是一个字符串对象，而值则可以是字符串对象、列表对象、哈希对象、集合对象或者有序集合对象的其中的一种。
**行话：**字符串键就是指的是数据库键对应值为字符串对象

| 对象 | 对象 type 属性的值 | type 命令的输出 |
| --- | --- | --- |
| 字符串对象 | REDIS_STRING | "string" |
| 列表对象 | REDIS_LIST | "list" |
| 哈希对象 | REDIS_HASH | "hash" |
| 集合对象 | REDIS_SET | "set" |
| 有序集合对象 | REDIS_ZSET | "zset" |

### 编码和底层实现
对象的`ptr`指针指向对象的底层实现数据结构，而这些数据结构由对象的 encoding 属性决定。
encoding 属性记录了对象所使用的编码，也就是说这个对象使用了什么数据结构作为对象的底层实现。
Redis 可以根据不同的使用场景来为一个对象设置不同的编码，从而优化对象在某一场景下的效率。
## 字符串对象
字符串对象的编码可以是 int、raw 或者 embstr。
**编码的转换？**
对于 int 编码的对象来说，执行了命令后让这个对象存储的不在是整数值而是一个字符串值，对象编码 `int -> raw`
另外，embstr 编码的字符串对象没有修改 API，实际上 embstr 编码的字符串对象是只读的。所有如果要修改 embstr 编码的对象，总是会编码 `embstr -> raw`
## 列表对象
列表对象的编码可以是 ziplist 或者 linkedlist
ziplist 编码的连别对象使用压缩列表作为底层实现，每个压缩列表节点（entry）保存了一个列表元素。
linedlist 编码的列表对象使用双端链表作为底层实现，每个双端链表节点（node）都保存了一个字符串对象，而每个字符串对象都保存了一个列表元素。
**编码转换？**`ziplist -> linkedlist`

1. 列表对象保存的所有字符串元素长度都小于 64 字节；
2. 列表对象保存的元素数量小于 512 个；

不能满足这两个条件的列表对象需要使用 linkedlist 编码。配置文件可修改这些数值。
## 哈希对象
哈希对象的编码可以是 ziplist 或者 hashtable
ziplist 编码：每当由新的键值对加入，程序会先保存键的ziplistNode推入到压缩列表表尾，然后再将保存了值的ziplistNode推入到压缩列表表尾。

1. 保存了同一键值对的两个节点总是紧挨在一起，保存键的节点在前，保存值的节点在后；
2. 先添加到哈希对象中的键值对会被放在压缩列表的表头方向，而后来添加到哈希对象中的键值对会被放在压缩列表的表尾方向。

hashtable 编码：使用字典作为底层实现，哈希对象中的每个键值对都使用一个字典键值对来保存.

1. 字典的每一个键都是一个字符串对象，对象中保存了键值对的键；
2. 字典中的每个值都是一个字符串对象，对象中保存了键值对的值。

**编码转换？**`ziplist -> hashtable`

1. 哈希对象保存的所有字符串元素长度都小于 64 字节；
2. 哈希对象保存的元素数量小于 512 个；

不能满足这两个条件的哈希对象需要使用 hashtable 编码。配置文件可修改这些数值。
## 集合对象
集合对象可以是 intset 或者 hashtable
intset 编码：所有元素都保存在整数集合里面
hashtable 编码：字典作为底层实现，字典的每个键都是一个字符串对象，每个字符串对象包含了一个集合元素，而字典的值则全部被设置为 NULL。
**编码转换？**`intset -> hashtable`

1. 集合对象保存的所有元素都是整数值；
2. 集合对象保存的元素数量小于 512 个；

不能满足这两个条件的集合对象需要使用 hashtable 编码。配置文件可修改这些数值。
## 有序集合对象
有序集合的编码可以是 ziplist 或者 skiplist
ziplist 编码：用压缩列表作为底层实现，每个集合元素使用两个紧挨在一起的压缩列表节点来保存，第一个节点保存元素的成员（member），而第二个元素则保存元素的分值（score）。压缩列表内的集合元素按分值从小到大进行排序，分值较小的元素被放置在靠近表头的方向，而分值较大的放置在靠近表尾的方向。
**编码转换？**`ziplist -> skiplist`

1. 有序集合保存的元素小于 128 个；
2. 有序集合保存的元素成员长度小于 64 字节；

不满足则`ziplist -> skiplist`
## 内存回收
Redis 在自己的对象系统中构建了一个引用计数技术实现的内存回收机制。
```c
typedef struct redisObject {
    // ...
    // 引用计数
    int refcount;
    // ...
} robj;
```
## 对象共享
`refcount`还有对象共享的作用。
Redis 会共享值为 0 到 9999 的字符串对象。
## 对象的空转时长
```c
typedef struct redisObject {
    // ...
    unsigned lru:22;
    // ...
} robj;
```
`lru`属性记录了对象最后一次被命令程序访问的时间，这个时间可以用于计算对象的空转时间。
