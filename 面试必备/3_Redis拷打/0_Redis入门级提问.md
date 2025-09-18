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
