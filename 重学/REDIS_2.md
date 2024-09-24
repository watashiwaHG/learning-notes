# 网络I/O与进程阻塞

## 网络IO

数据到达（接收数据）：网卡会把接收到的数据写入内存中（DMA），网卡向CPU发出一个中断信号，CPU就知道数据到了，所以可以读取数据

cpu在接收到中断信号后，执行中断处理程序：

1. 将数据写入socket的接收缓冲区（内核空间到用户空间）
2. 将进程放入工作队列中

## 进程阻塞

进程在等待某个事件（数据到达）发生之前的等待状态

```python
s = socket(ip, port)
	bind()
    listen()
int c = accept(s) // client连接
data = recv(c) // 接收client发送的数据
```

recv就是阻塞方法，执行到要等待数据到达

![](.\pic\redis\IO阻塞.jpg)

# I/O多路复用

> 正常情况下，如果有多个socket，是需要按顺序处理，但如果其中一个很慢卡住了，那么后面的socket都会被卡住，所以IO多路复用的作用就是一次性将所有需要处理的socket都发送给内核，让内核去监听这些socket，如果有哪一个socket已就绪可操作，那么就返回，这样就避免了已就绪的socket因为顺序的原因被等待的socket卡住无法得到处理

如果每个socket都用一个线程的来处理，会导致线程数量过多，哪怕是线程池也需要考虑多线程的问题

Redis利用I/O多路复用来实现网络通信

1. I/O多路复用建立在多路事件分离函数select，poll，epoll之上
2. 将需要进行IO操作的socket添加到select中
3. 然后阻塞等待select函数调用返回
4. 当数据到达时，select被激活，select函数返回
5. 用户线程发起read请求，读取数据并继续执行

![](.\pic\redis\IO多路复用简易.jpg)

## Reactor设计模式

IO多路复用模型使用Reactor设计模式实现

![](.\pic\redis\Reactor设计模式.png)

- Handle（事件载体）

  handle在linux中一般称为文件描述符（fd），而在window称为句柄，两者的含义一样。handle是事件的发源地。比如一个网络socket、磁盘文件等。而发生在handle上的事件可以有connection、ready for read、ready for write等

- Reactor

  反应器，也叫事件分发器，负责事件的注册、删除与转发event handler

- Synchronous Event Demultiplexer

  同步事件分离器，本质上是系统调用。比如linux中的select、poll、epoll等。比如，select 方法会一直阻塞直到handle上有事件发生时才会返回

- Event Handler

  事件处理器，其会定义一些回调方法或者称为钩子函数，当handle上有事件发生时，回调方法便会执行，一种事件处理机制

- Concrete Event Handler：具体的事件处理器，实现了Event Handler。在回调方法中会实现具体的业务逻辑

### 处理流程

1. 当应用向Reactor注册Concrete Event Handle时，应用会标识出该事件处理器希望Reactor在某种类型的事件发生时向其通知事件与handle关练
2. Reactor要求注册在其上面的Concrete Event Handler传递内部关联的handle，该handle会向操作系统标识
3. 当所有的Concrete Event Handler都注册到Reactor上后，应用会调用handle_evenets方法来启动Reactor的事件循环，这时Reactor会将每个Concrete Event Handler关练的handle合并，并使用Synchronous Event Demultiplexer来等待这些handle上事件的发生
4. 当与某个事件源对应的handle变为ready时，Synchronous Event Demultiplexer便会通知Reactor
5. Reactor会触发事件处理器的回调方法。当事件发生时，Reactor会将被一个“key”（表示一个激活的handle）定位和分发给特定的Event Handler的回调方法
6. Reactor调用特定的Concrete Event Handler的回调方法来响应其关联的handle上发生的事件

![](.\pic\redis\Reactor工作原理简易图.jpg)

时序图：

![](.\pic\redis\Reactor时序图.jpg)

## select

select模式是I/O多路复用模式的一种早期实现。也是支持操作系统最多的模式（windows）

### 工作模式

![](.\pic\redis\select工作模式.jpg)

### 工作流程

1. 应用进程在调用select之前告诉select应用进程需要监控哪些fd可读、可写、异常事件，这些分别都存在一个fd_set数组中
2. 然后应用进程调用select的时候把fd_set传给内核（这里也就产生了一次fd_set在用户空间到内核空间的复制）
3. 内核收到fd_set后对fd_set进行遍历，然后一个个去扫描对应fd是否满足可读写事件
4. 如果发现了有对应的fd有读写事件后，内核会把fd_set里没有事件状态的fd句柄清除，然后把事件的fd返回给应用进程（这里又会把fd_set从内核空间复制到用户空间）
5. 最后应用进程收到了select返回的活跃事件类型的fd句柄后，再向对应的fd发起数据读取或者写入数据操作

![](.\pic\redis\select工作流程.jpg)

![](.\pic\redis\select函数例子.jpg)

- 传rset给select函数告诉需要监听的fd，rset是一个bitmap，索引表示fd的编号，1表示需要监听的fd
- 第一个参数表示监听bitmap中索引范围在max + 1以内的fd，max为之前fd编号中最大的编号，后面的参数代表监听fd的读，写，异常和超时时间
- 执行select后会阻塞，将用户态的rset复制一份到内核态中，内核对rset中的fd进行遍历，如果有一个或多个fd有数据，将rset中的对应的位置的值置位并返回（貌似是把其他fd置为0了，从内核态拷贝回用户态）
- 用户遍历rset找到被置位的fd，读取数据

### 缺点

- 虽然比起用户自己一个一个调用内核方法找fd是否有数据，效率高了
- 单个进程能够监视的文件描述符的数量存在最大限制（1024）
- 内核/用户空间内存拷贝问题，select需要复制大量的句柄数据结构
- select返回的是含有整个句柄的数组，应用程序需要遍历整个数组才能发现哪些句柄发生了事件O(n)
- 添加等待队列后就会阻塞，每次调用都有这两个步骤
- rset不可复用，每次要重置一遍

> select后还有个poll，他是select的改良，没有了监视文件描述符数量的最大限制

## poll

![](.\pic\redis\poll例子.jpg)

- 定义了一种数据结构pollfd，fd为fd编号，events表示监听的事件，多个事件用或（写 或 读），revents表示哪个事件可以被处理了
- poll函数传入pollfd数组，fd的个数，超时时间
- 执行后阻塞，将pollfd数组复制到内核态内存中，内核遍历哪些fd对应的事件有数据了，有一个或多个有数据了就将相应的pollfd的revents置位
- 遍历pollfd数组判断哪个pollfd的revents被置位了，做相应的处理，并且要将revents重置

> 比起select来说少了1024的限制，pollfd可以复用，但是要将revents重置

## epoll

对select的优化

![](.\pic\redis\epoll例子.jpg)

- 调用epoll_create创建epfd，参数没意义，但是传0会报错，返回的是epoll实例的fd编号
- epoll_ctl函数向epfd中添加epoll_event，第二个参数为操作类型，第三个参数为fd编号，第四个参数为epoll_event，epoll_event中含有fd编号和需要监听的事件类型events，添加的epoll_event会放到红黑树上
- epoll_wait是真正的获取可操作的fd，第一个参数是epoll实例的fd编号，events使用户分配好的数组，内核会把event复制到events（用户态和内核态会共享？没有复制的步骤），第三个参数是本次可以返回的最大时间数目，第四个参数等待时间
- 调用epoll_wait如果双向链表不为空，则直接返回就绪的fd，否则会被放到阻塞队列中阻塞，有数据的fd会调用回调函数放到双向链表中，赋值到events的前面，返回值为几个fd有数据了，所以只需要遍历有限个events前面的epoll_event就能进行操作

![](.\pic\redis\epoll图.png)

# Redis主从复制原理

从总体上来说，Redis主从复制的策略就是：当主从服务器刚建立连接的时候，进行全量同步；全量复制结束后，进行增量复制。当然，如果有需要，slave在任何时候都可以发起全量同步

## 全量同步

![](.\pic\redis\主从复制.png)

1. slave服务器连接到master服务器，便开始进行数据同步，发送psync命令（Redis2.8之前是sync命令）

2. master服务器收到psync命令之后，开始执行bgsave命令生成RDB快照文件并使用缓存区记录此后执行的所有写命令

   - 如果master收到了多个slave并发连接请求，他只会进行一次持久化，而不是每个连接都执行一次，然后再把这一份持久化的数据发送给多个并发连接的slave
   - 如果RDB复制时间超过60秒（repl-timeout），那么slave服务器就会认为复制失败，可以适当调节大这个参数

3. master服务器bgsave执行完之后，就会向所有slave服务器发送快照文件，并在发送期间继续在缓冲区内记录被执行的写命令

   > client-output-buffer-limit slave 256MB 64MB 60,如果在复制期间，内存缓存区持续消耗超过64MB，或者一次性超过256MB，那么停止复制。复制失败

4. slave服务器收到RDB快照文件后，会将接收到的数据写入磁盘，然后清空所有旧数据，在本地磁盘载入收到的快照到内存中，同时基于旧的数据版本对外提供服务

5. master服务器发送完RDB快照文件后，便开始向slave服务器发送缓冲区中的写命令

6. slave服务器完成对快照的载入，开始接收命令请求，并执行来自主服务器缓冲区的写命令

7. 如果slave node开启了AOF，那么会立即执行BGREWIRTEAOF，重写AOF

## 增量复制

Redis的增量复制是指在初始化的全量复制并开始正常工作之后，master服务器将发生的写操作同步到slave服务器的过程，增量复制的过程主要是master服务器每执行一个写命令就会向slave服务器发送相同的写命令，slave服务器接收并执行收到的写命令

# Redis主从不一致的问题

1. redis的确默认是弱一致性，异步的同步主从
2. 锁不能用主从（单实例/分片集群/redlock）==>redisson
3. 在配置中提供了必须有多少个client连接能同步，你可以配置同步因子，趋向于强一致性
4. wait 2 0 等待全部同步才结束？redis挂了会出问题 小心
5. 34点就有点违背redis的初衷了

# Redis集群方案

奇数个节点，至少三主三从

虚拟槽分区

redis-cli

集群功能的限制：

- 无法支持批量命令mset、mget在多个节点，只支持具有相同slot的key

- 事务不支持多个key分布在不同的节点

- key作为数据分区的最小粒度，因此不能将一个大的键值对象如hash、list等映射到不同的节点
- 不支持多数据库空间，单机下的Redis支持16个数据库，集群模式下只能使用一个数据库空间，即db0
- 复制结构只支持一层，从节点只能复制主节点，不支持嵌套树状复制结构

节点数量过多，影响带宽，不建议超过1000

# Redis集群方案什么情况下会导致整个集群不可用

为了保证集群完整性，默认情况下当集群16384个槽任何一个没有指派到节点时整个集群不可用。执行任何键命令返回（error）CLUSTERDOWN Hash slot not served错误。这是对集群完整性的一种保护措施，保证所有的槽都指派给在线的节点。当时当持有槽的主节点下线时，从故障发现到自动完成转移期间整个集群是不可用状态，对于大多数业务无法容忍这种情况，因此可以将参数cluster-require-full-coverage配置为no，当主节点故障时只影响他负责槽的相关命令执行，不会影响其他主节点的可用性

但是从集群的故障转移的原理来说，集群会出现不可用，当：

1. 当访问一个Master和Slave节点都挂了的时候，cluster-require-full-coverage=yes，会报槽无法获取
2. 集群主库半数宕机（根据failover原理，fail掉一个主需要一半以上主都投票通过才可以）

另外，当集群Master节点个数小于3个的时候，或者集群可用节点个数为偶数的时候，基于fail的这种选举机制的自动主从切换过程可能会不能正常工作，一个是标记fail的过程，一个是选举新的master的过程，都有可能 异常

# Redis哈希槽的概念

虚拟哈希槽

槽数量16384

# Redis集群会有写操作丢失吗？为什么？

Redis集群弱一致性

主从同步是一个异步操作，当主机宕机或者一些情况时，导致从节点没能同步写操作

# Redis常见性能问题和解决方案有哪些

1. 持久化 性能问题

   全量复制、部分复制（一台机器）

   主从 主不要做持久化，从节点做持久化

2. 数据比较重要，AOF持久化 slave开启AOF，策略每秒同步一次

3. 主从复制 流畅，同一局域网

4. 尽量避免 主库 压力很大的时候，增加从库，压力更大

5. 主从复制 不要采用网状结构 使用线性结构减少主库的性能损耗