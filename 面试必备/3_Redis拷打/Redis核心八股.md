



## 1. Redis为什么这么快 (能说一说你为啥使用 Redis 吗？你觉得 Redis 最核心的功能是什么？)
1. Redis把数据存储再内存中，提供快速的读写的速度，相比传统的磁盘数据库，内存访问速度快很多
2. Redis使用单线程事件驱动模型结合I/O多路复用，避免了多线程的上下文切换和竞争条件
3. Redis提供多种高校的数据结构，这些结构经过优化，能快速完成各种操作

### 1.1详细讲讲Redis使用单线程事件驱动模型结合I/O多路复用
Redis 不靠多线程来处理请求，而是用 一个主线程配合 I/O 多路复用（epoll/kqueue/select） 去处理所有网络连接和命令

在源码里，有一个server.c的文件，这个main函数里有这么几个关键函数
首先是initServer()，这个函数会创建aeEventLoop，内部封装了epoll/kqueue
并且会监听 TCP（和可选的 Unix 域）端口，得到seversocket的句柄，也就是FD。
在监听 socket 创建出来时，就立刻把该 FD 的 AE_READABLE 事件注册到事件循环，并绑定 acceptTcpHandler（或对应封装）。等到将来客户端这个FD时，这个FD会被Epoll标记可读，事件循环才会回调 acceptTcpHandler。
调用了acceptTcpHandler，获得client socket，并为它注册 可读事件。
当 client socket 有数据，事件循环调 readQueryFromClient，读数据 → 解析成命令 → 在主线程里直接执行 → 把结果写到回复缓冲区。

这里面有两个IO多路复用过程，
server socket 阶段
Redis 一开始把监听 socket（server socket）注册到 epoll。
当有客户端发起连接时，内核把这个监听 FD 标记为“可读”。
事件循环就触发 acceptTcpHandler，为这个监听 FD 用 accept() 拿到一个新的 client socket。

新的 client socket 也会被注册到 epoll，监听它的“可读”事件。
当客户端真正发命令时，这个 client socket 就会被标记为“可读”。
事件循环就触发 readQueryFromClient，Redis 从这个 socket 里读数据 → 解析命令 → 执行 → 把结果放进回复缓冲区。

有数据要回给客户端时，再注册“可写事件”，由 sendReplyToClient 把结果写回去。

## Redis有哪些常见数据结构以及应用场景
常见的数据结构有：字符串String、哈希表Hash、列表List、集合Set、有序集合Sorted Set。除了这些，还有 Bitmaps、HyperLogLog、GEO，甚至还可以自定义数据结构。

String 类型是 Redis 最基础的数据结构，其它几种数据结构都是在字符串类型的基础上创建的。并且所有数据结构的 **key** 也都是字符串类型。

String 类型只能存储单个数据，即一个 **key** 对应一个 **value**。同时它还是二进制安全的，意思就是 String 类型严格按照二进制数据进行存取。

正因如此，它可以存储任何数据，包括字符串：简单的字符串，复杂的 JSON，XML 字符串；数字：整数，浮点数；甚至是二进制：图片，音频，视频等。

String 类型的内部编码有三种：int，embstr，raw，分别在当前值为 8 字节长整形，小于等于 39 字节的字符串，大于 39 字节的字符串时使用。

Redis 会根据当前 String 的值类型和大小自己决定使用哪种内部编码实现。

使用场景：
（1）充当缓冲：缓存用户的基本信息
这是一个最容易被想到的应用场景：由于用户基本信息更改频率比较低，但是像用户昵称，头像这些基本信息会用的比较频繁，所以我们可以对他进行缓存，例如：可以将 MySQL 中每个用户的信息转换为 JSON 格式存储在 String 类型中。

（2）计数
String 类型天然可自增，利用 incr 命令可以实现快速计数功能，同时它还是原子性的操作。比如可以用来记录网站上视频的播放量。

（3）在一段时间内限制请求次数
比如为了防止用户恶意刷新网页，可以限制一个 IP 地址在一段时间内刷新网页的次数。其伪代码为：

（4）分布式共享 Session
通常每个服务器只会存储自己的 Session，考虑在负载均衡的情况下，分布式服务会将用户的访问均衡到不同的服务器上，此时用户的信息只存在一个服务器上显然是不合适的。
这时就可以考虑使用 Redis，将所有用户的 Session 信息进行集中管理，各个服务器在需要读取用户信息时，只需向 Redis 中查询即可。