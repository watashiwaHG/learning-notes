# InnoDB内存相关的参数优化

## Buffer Pool参数优化

### 缓冲池内存大小配置

一个大的日志缓冲区允许大量的事务在提交之前不写日志到磁盘。因此，如果你有很多事务的更新，插入或删除操作，通过设置这个参数会大量的减少磁盘I/O的次数

建议：在专用数据库服务器上，可以将缓冲池大小设置为服务器物理内存的60% - 80%

- 查看缓冲池大小

  ```sql
  show variables like 'innodb_buffer_pool_size%';
  ```

- 在线调整InnoDB缓冲池大小

  innodb_buffer_pool_size可以动态设置，允许在不重新启动服务器的情况下调整缓冲池的大小

  ```sql
  SET GLOBAL innodb_buffer_pool_size = 268435456; -- 512
  
  show variables like 'innodb_buffer_pool_size%%';
  ```

- 监控在线调整缓冲池的进度

  ```sql
  SHOW STATUS WHERE Variable_name = 'InnoDB_buffer_pool_resize_status';
  ```

### InnoDB 缓存性能评估

当前配置的innodb_buffer_pool_size是否合适，可以通过分析InnoDB缓冲池的缓存命中率来验证

- 以下公式计算InnoDB buffer pool命中率：

  

  ```text
  命中率 = innodb_buffer_pool_read_requests / 
  (innodb_buffer_pool_read_requests + innodb_buffer_pool_reads) * 100
  
  参数1：innodb_buffer_pool_reads：表示InnoDB缓冲池无法满足的请求数。需要从磁盘中读取
  参数2：innodb_buffer_pool_read_requests：表示从内存中读取页的请求数
  ```

  ```sql
  show status like 'innodb_buffer_pool_read%';
  
  -- 此值低于90%，则可以考虑增加innodb_buffer_pool_size
  +---------------------------------------+------------+
  | Variable_name                         | Value      |
  +---------------------------------------+------------+
  | Innodb_buffer_pool_read_ahead_rnd     | 0          |
  | Innodb_buffer_pool_read_ahead         | 131140     |
  | Innodb_buffer_pool_read_ahead_evicted | 3975       |
  | Innodb_buffer_pool_read_requests      | 1282528295 |
  | Innodb_buffer_pool_reads              | 113109     |
  +---------------------------------------+------------+
  5 rows in set (0.04 sec)
  ```
## Page管理相关参数

**查看Page页的大小（默认16KB）**，```innodb_page_size```只能在初始化Mysql实例之前配置，不能在之后修改。如果没有指定值，则使用默认页面大小初始化实例

```sql
show variables like '%innodb_page_size%';
```

### Page页管理状态相关参数

```sql
show global status like '%innodb_buffer_pool_pages%';

+----------------------------------+---------+
| Variable_name                    | Value   |
+----------------------------------+---------+
| Innodb_buffer_pool_pages_data    | 31587   | -- 数据页 clean page和dirty page
| Innodb_buffer_pool_pages_dirty   | 0       | -- dirty page
| Innodb_buffer_pool_pages_flushed | 2744868 | -- 刷新脏页的请求数
| Innodb_buffer_pool_pages_free    | 1024    | -- free page
| Innodb_buffer_pool_pages_misc    | 157     | -- 非普通数据页（缓冲池中当前已经被用作管理用途或hash index而不能用作为普通数据页的数目）
| Innodb_buffer_pool_pages_total   | 32768   | -- 页的总数
+----------------------------------+---------+
6 rows in set (0.05 sec)
```

# InnoDB日志相关的参数优化

## 日志缓冲区相关参数配置

日志缓冲区的大小。一般默认值16MB是够用的，但如果事务之中含有blog/text等大字段，这个缓冲区会被很快填满会引起额外的IO负载。配置更大的日志缓冲区，可以有效地提高Mysql的效率

- innodb_log_buffer_size 缓冲区大小

  ```sql
  show variables like 'innodb_log_buffer_size';
  ```

- innodb_log_files_in_group 日志组文件个数

  日志组根据需要来创建。而日志组的成员则需要至少2个，实现循环写入并作为冗余策略

  ```sql
  show variables like 'innodb_log_files_in_group';
  ```

- innodb_log_file_size 日志文件大小

  参数innodb_log_file_size用于设定Mysql日志组中每个日志文件的大小（默认48MB）。此参数是一个全局的静态参数，不能动态修改

  参数innodb_log_file_size的最大值，二进制日志文件大小（innodb_log_file_size * innodb_log_files_in_group）不能超过512GB。所以单个日志文件的大小不能超过256G

  ```sql
  show variables like 'innodb_log_file_size';
  ```

## 日志文件参数优化

日志文件大小设置对性能的影响

- 设置过小

  1. 参数```innodb_log_file_size```设置太小，就会导致Mysql的日志文件（redo log）频繁切换，频繁的触发数据库的检查点（Checkpoint），导致刷新脏页到磁盘的次数增加。从而影响IO性能
  2. 处理大事务时，将所有的日志写满了，事务内容还没有写完，这样就会导致日志不能切换

- 设置过大

  参数```innodb_log_file_size```如果设置太大，虽然可以提升IO性能，但是当Mysql由于意外宕机时，二进制日志很大，那么恢复的时间必然很长。而且这个恢复时间往往不可控，受多方面因素影响

## 优化建议

如何设置合适的日志文件大小

- 根据实际生产场景的优化经验，一般是计算一段时间内生成的事务日志（redo log）的大小，而Mysql的日志文件的大小最少应该承载一个小时的业务日志量（官网文档中有说明）

想要估计一下InnoDB redo log的大小，需要抓取一段时间内Log SequenceNumber（日志顺序号）的数据，来计算一小时内产生的日志大小

> Log sequence number
>
> 自系统修改开始，就不断的生成redo日志。为了记录一共生成了多少日志，于是musql设计了全局变量log sequence number，简称lsn，但不是从0开始，是从8704字节开始

```sql
-- pager分页工具，只获取sequence的信息
pager grep sequence;

-- 查询状态，并倒计时一分钟
show engine innodb status\G select sleep(60);

-- 一分时间内所生成的数据量
show engine innodb status\G;

-- 关闭pager
nopager;
```

有了一分钟的日志量，据此推算一小时内的日志量

太大的缓冲池或非常不正常的业务负载可能会计算出非常大（或非常小）的日志大小。这也是公式不足之处，需要根据判断和经验。但这个计算方法是一个很好的参考标准

# InnoDB IO线程相关参数优化

# 写失效

InnoDB的页和操作系统的页大小不一致，InnoDB页大小一般为16KB。操作系统页大小为4KB，InnoDB的页写入到磁盘时，一个页需要分4次写

如果存储引擎正在写入页的数据到磁盘时发声了宕机，可能出现页只写了一部分的情况，比如只写了4K，就宕机了，这种情况叫做部分写失效（partial page write），可能会导致数据丢失

![](.\pic\mysql\写失效.jpg)

## 双写缓冲区 Doublewrite Buffer

为了解决写失效问题，InnoDB实现了double write buffer Files，它位于系统表空间，是一个存储区域

在BufferPool的page页刷新到磁盘真正的位置前，会先将数据存在Doublewrite缓冲区。这样在宕机重启时，如果出现数据页损坏，那么在应用redo log之前，需要通过该页的副本来还原该页，然后再进行redo log重做，double write实现了InnoDB引擎数据页的可靠性

默认情况下启用双写缓冲区，如果要禁用Doublewrite缓冲区，可以将```innodb_doublewrite```设置为0

```sql
show variables like '%innodb_doublewrite%';
```

数据双写流程

![](.\pic\mysql\双写流程.jpg)

- 共享表空间：连续磁盘空间，写入快

# 行溢出

## 什么是行溢出

MySQL中是以页为基本单位，进行磁盘与内存之间的数据交互的，我们知道一个页的大小是16KB，16KB = 16384字节，而一个varchar(m)类型列最多可以存储65532字节，一些大的数据类型比如TEXT可以存储更多

如果一个表中存在这样的大字段，那么一个页就无法存储一条完整的记录。这时就会发生行溢出，多出的数据就会存储在另外的溢出页中

总结：如果某些字段信息过长，无法存储在B树节点中，这时候会被单独分配空间，此时被称为溢出页，该字段被称为页外列

## Compact中的行溢出机制

InnoDB规定一页至少存储两条记录（B+树特点），如果页中只能存放下一条记录，InnoDB存储引擎会自动将行数据存放到溢出页中

当发生行溢出时，数据页只保存了前768字节的前缀数据，接着是20个字节的偏移量，指向行溢出页

![](.\pic\mysql\compact行溢出.jpg)

# JOIN优化

## 驱动表

- 多表关练查询时，第一个被处理的表就是驱动表，使用驱动表去关联其他表
- 驱动表的确定非常的关键，会直接影响多表关联的顺序，也决定后续关联查询的性能

驱动表的选择要遵循一个规律：

- 在对最终的结果集没有影响的前提下，优先选择结果集最小的那张表作为驱动表

## 三种JOIN算法

1. Simple Nested-Loop Join（简单的嵌套循环连接）
   - 简单来说嵌套循环连接算法就是一个双层for循环，通过循环外层表的行数据，逐个与内层表的所有行数据进行比较来获取结果

   - 这种算法是最简单的方案，性能也一般。对内循环没优化

   - 例如有这样一条sql：

     ```sql
     -- 连接用户表与订单表 连接条件是u.id = o.user_id
     select * from user t1 left join order t2 on t1.id = t2.user_id;
     -- user表为驱动表，order表为被驱动表
     ```

   - 转换成代码执行时的思路是这样的：

     ```java
     for (user表行 uRow : user表) {
     	for (Order表的行 oRow : order表) {
             if (uRow.id = oRow.user_id) {
                 return uRow;
             }
         }
     }
     ```

   - 匹配过程如下图：

     ![](.\pic\mysql\join连表.jpg)

   - SNL的特点

     - 简单粗暴容易理解，就是通过双重循环比较数据来获得结果
     - 查询效率非常慢，假设A表有N行，B表有M行。SNL的开销如下：
       - A表扫描1次
       - B表扫描M次
       - 一共有N个内循环，每个内循环要M次，一共有内循环N * M次

2. Index Nested-Loop Join（索引嵌套循环连接）

   - Index Nested-Loop Join其优化的思路：主要是为了减少内存表数据的匹配次数，最大的区别在于，用来进行join的字段已经在被驱动表中建立了索引

   - 从原来的```匹配次数 = 外层表行数 * 内层表行数```，变成了```匹配次数 = 外层表的行数 * 内层表索引的高度```，极大的提升了join的性能

   - 当order表的user_id为索引的时候执行过程会如下图：

     ![](.\pic\mysql\索引join连接.jpg)

   注意：使用Index Nested-Loop Join算法的前提是匹配的字段必须建立了索引

3. Block Nested-Loop Join（块嵌套循环连接）

   如果join的字段有索引，Mysql会使用INL算法。如果没有的话，Mysql会如何处理？

   因为不存在索引了，所以被驱动表需要进行扫描。这里Mysql并不会简单粗暴的应用SNL算法，而是加入了buffer缓冲区，降低了内循环的个数，也就是被驱动表的扫描次数

   ![](.\pic\mysql\缓冲join连接.jpg)

   - 在外层循环扫描user表中的所有记录。扫描的时候，会把需要进行join用到的列都缓存到buffer中。buffer中的数据有一个特点，里面的记录不需要一条一条地取出来和order表进行比较，而是整个buffer和order表进行批量比较
   - 如果我们把buffer空间开得很大，可以容纳下user表的所有记录，那么order表也只需要访问一次
   - Mysql默认buffer大小256K，如果n个join操作，会生成n - 1个join buffer

   ```sql
   show variables like '%join_buffer%';
   
   set session join_buffer_size = 262144;
   ```

4. JOIN优化总结

   1. 永远用小结果集驱动大结果集（其本质就是减少外层循环的数据数量）
   2. 为匹配的条件增加索引（减少内层表的循环匹配次数）
   3. 增大join buffer size的大小（一次缓存的数据越多，那么内层包的扫表次数就越少）
   4. 减少不必要的字段查询（字段越少，join buffer所缓存的数据就越多）

# 索引失效的情况

1. 查询条件包含or，会导致索引失效
2. 隐式类型转换，会导致索引失效，例如age字段类型是int，我们where age = '1'，这样就会触发隐式类型转换
3. like通配符会导致索引失效，注意：'ABC%'不会失效，会走range索引，‘%ABC’索引会失效
4. 联合索引，查询时的条件列不是联合索引中的第一个列，索引失效
5. 对索引字段进行函数运算
6. 对索引列运算（如+  - * /），索引失效
7. 索引字段上使用（!=或者<>，not in）时，会导致索引失效
8. 索引字段上使用is null，is not null，可能导致索引失效
9. 相join的两个表的字符编码不同，不能命中索引，会导致笛卡尔积的循环计算
10. mysql估计使用全表扫描要比使用索引快，则不使用索引

# 覆盖索引

覆盖索引是一种避免回表查询的优化策略：只需要在一棵索引树上就能获取sql所需的所有列数据，无需回表（通过聚集索引去定位行记录），速度更快

覆盖索引的定义与注意事项：

- 如果一个索引包含了所有需要查询的字段的值（不需要回表），这个索引就是覆盖索引
- Mysql只能使用B+树索引做覆盖索引（因为只有B+树能存储索引列值）
- 在explain的Extra列，如果出现```Using index```表示使用到了覆盖索引，所取得数据完全在索引中就能拿到

# Mysql中事务的特性

在关系型数据库管理系统中，一个逻辑工作单位要成为事务，必须满足这4个特性，即所谓的ACID：原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）和持久性（Durability）

## 原子性

事务作为一个整体被执行，包含在其中的对数据库的操作要么全部被执行，要么都不执行

InnoDB存储引擎提供了两种事务日志：redo log（重做日志）和undo log（回滚日志）。其中redo log用于保证事务持久性，undo log则是事务的原子性和隔离性实现的基础

![](.\pic\mysql\redo log和undo log.png)

每写一个事务，都会修改Buffer Pool，从而产生相应的Redo/Undo日志：

- 如果要回滚事务，那么就基于undo log来回滚就可以了，把之前对缓冲页做的修改都给回滚了就可以了
- 如果事务提交之后，redo log刷入磁盘，结果Mysql宕机了，是可以根据redo log恢复事务修改过的缓存数据的

实现原子性的关键，是当事务回滚时能够撤销所有已成功执行的sql语句

InnoDB实现回滚，靠的是undo log：当事务对数据库进行修改时，InnoDB会生成对应的undo log；如果事务执行失败或调用了rollback，导致事务需要回滚，便可以利用undo log中的信息将数据回滚到修改之前的样子

## 一致性

事务应确保数据库的状态从一个一致状态转变为另一个一致状态。一致状态的含义是数据库中的数据应满足完整性约束

- 约束一致性：创建表结构时所指定的外键、唯一索引等约束
- 数据一致性：是一个综合性的规定，因为他是由原子性、持久性、隔离性共同保证的结果，而不是单单依赖于某一种技术

## 隔离性

指的是一个事务的执行不能被其他事务干扰，即一个事务内部的操作及使用的数据对其他的并发事务是隔离的

不考虑隔离性会引发的问题：

- **脏读**：一个事务读取到了另一个事务修改但未提交的数据
- **不可重复读**：一个事务中多次读取同一行记录的结果不一致，后面读取的跟前面读取的结果不一致
- **幻读**：一个事务中多次按相同条件查询，结果不一致。后续查询的结果和前面查询结果不同，多了或少了几行记录

数据库事务的隔离级别有4个，由低到高依次为Read uncommitted、Read committed、Repeatable read、Serializable，这四个级别可以逐个解决脏读、不可重复读、幻读这几类问题

## 持久性

指的是一个事务一旦提交，他对数据库中数据的改变就应该是永久性的，后续的操作或故障不应该对其有任何影响，不会丢失

Mysql事务的持久性保证依赖的日志文件：redo log

- redo log也包括两部分：一是内存中的日志缓存（redo log buffer），该部分日志是易失性的；二是磁盘上的重做日志（redo log file），该部分日志是持久的，redo log是物理日志，记录的是数据库中物理页的情况
- 当数据发生修改时，InnoDB不仅会修改Buffer Pool中的数据，也会在redo log buffer记录这次操作，当事务提交时，会对redo log buffer进行刷盘，记录到redo log file中。如果Mysql宕机，重启时可以读取redo log file中的数据，对数据库进行恢复。这样就不需要每次提交事务都实时进行刷脏了

![](.\pic\mysql\原子性.jpg)

## ACID总结

- 事务的持久化是为了应对系统崩溃造成的数据丢失
- 只有保证了事务的一致性，才能保证执行结果的正确性
- 在非并发状态下，事务间天然保证隔离性，因此只需要保证事务的原子性即可保证一致性
- 在并发状态下，需要严格保证事务的原子性、隔离性

# 可重复读是怎么实现的

可重复读（repeatable read）定义：一个事务执行过程中看到的数据，总是跟这个事务在启动时看到的数据是一致的

## MVCC

- MVCC，多版本并发控制，用于实现**读已提交**和**可重复读**隔离级别
- MVCC的核心就是Undo log多版本链 + Read view，“MV”就是通过Undo log来保存数据的历史版本，实现多版本的管理“CC”是通过Read-view来实现管理，通过Read-view原则来决定数据是否显示。同时针对不同的隔离级别，Read-view的生成策略不同，也就实现了不同的隔离级别

## Undo log多版本链

每条数据都有两个隐藏字段：

- trx_id：事务id，记录最近一次更新这条数据的事务id
- roll_pointer：回滚指针，指向之前生成的undo log

![](.\pic\mysql\多版本链.jpg)

每一条数据都有多个版本，版本之间通过undo log链条进行连接通过这样的设计方式，可以保证每个事务提交的时候，一旦需要回滚操作，可以保证同一个事务只能读取到比当前版本更早提交的值，不能看到更晚提交的值

## ReadView

Read View是InnoDB在实现MVCC时用到的一致性读视图，即consistent read view，用于支持RC（Read Committed，读已提交）和RR（Repeatable Read，可重复读）隔离级别的实现

Read View简单理解就是对数据在某个时刻的状态拍成照片记录下来。那么之后获取某时刻的数据时就还是原来的照片上的数据，是不会变的

Read View中比较重要的字段有4个：

- m_ids：用来表示Mysql中哪些事务正在执行，但是没有提交
- min_trx_id：就是m_ids里最小的值
- max_trx_id：下一个要生成的事务id值，也就是最大事务id
- creator_trx_id：就是你这个事务的id

当一个事务第一次执行查询sql时，会生成一致性试图read-view（快照），查询时从undo log中最新的一条记录开始跟read-view做对比，如果不符合比较规则，就根据回滚指针回滚到上一条记录继续比较，直到得到符合比较条件的查询结果

### Read view判断记录某个版本是否可见的规则如下

![](.\pic\mysql\read-view的min_id和max_id.png)

1. 如果当前记录的事务id落在绿色部分（trx_id < min_id），表示这个版本是已提交的事务生成的，可读
2. 如果当前记录的事务id落在红色部分（trx_id > max_id），表示这个版本是由将来启动的事务生成的，不可读
3. 如果当前记录的事务id落在黄色部分（min_id <= trx_id <= max_id），则分为两种情况：
   1. 若当前记录的事务id在未提交事务的数组中，则此条记录不可读
   2. 若当前记录的事务id不在未提交事务的数组中，则此条记录可读

RC和RR隔离级别都是由MVCC实现，区别在于：

- RC隔离级别时，read-view是每次执行select语句时都生成一个
- RR隔离级别时，read-view是在第一次执行select语句时生成一个，同一事务中后面的所有select语句都复用这个read-view

# Repeatable Read解决了幻读问题吗？

可重复读：一个事务执行过程中看到的数据，总是跟这个事务在启动时看到的数据是一致的

不过理论上会出现幻读，简单的说幻读指的是当用户读取某一范围的数据行时，另一个事务又在该范围插入了新行，当用户在读取该范围的数据时会发现有新的幻影行

**注意在可重复读隔离级别下，普通的查询是快照读，是不会看到别的事务插入的数据的。因此幻读在“当前读”下才会出现（查询语句添加for update，表示当前读）**

在MVCC并发控制中，读操作可以分为两类：快照读（Snapshot Read）与当前读（Current Read）

- 快照读

  快照读是指读取数据时不是读取数据最新版本的数据，而是基于历史版本读取的一个快照信息，（mysql读取undo log历史版本），快照读可以使普通的select读取数据时不用对表数据进行加锁，从而解决了因为对数据库表的加锁而导致的两个如下问题

  1. 解决了因加锁导致的修改数据时无法对数据读取的问题
  2. 解决了因加锁导致读取数据时无法对数据进行修改的问题

- 当前读

  当前读是读取的数据库最新的数据，当前读和快照读不同，因为要读取最新的数据而且要保证事务的隔离性，所以当前读是需要对数据进行加锁的（```插入/更新/删除操作，属于当前读，需要加锁，select for update为当前读```）

## 例子

表结构

| id   | key  | value |
| ---- | ---- | ----- |
| 0    | 0    | 0     |
| 1    | 1    | 1     |

假设select * from where value = 1 for update，只在这一行加锁（注意这只是个假设），其他行不加锁，那么就会出现如下场景：

![](.\pic\mysql\幻读例子.png)

其中Q3读到value=1这一行的现象，就称之为幻读，**幻读指的是一个事务在前后两次查询同一个范围的时候，后一次查询看到了前一次查询没有看到的行**

先对“幻读”做出如下解释：

- 要讨论“可重复读”隔离级别的幻读现象，是要建立在“当前读”的情况下，而不是快照读，因为在可重复读隔离级别下，普通的查询是快照读，是不会看到别的事务插入的数据的

## Next-key Lock锁

产生幻读的原因是，行锁只能锁住行，但是新插入记录这个动作，要更新的是记录之间的“间隙”。因此，innoDB引擎为了解决“可重复读”隔离级别使用“当前读”而造成的幻读问题，就引出了next-key锁，就是记录锁和间隙锁的组合

- Record Lock锁：锁定单个行记录的锁。（记录锁，RC、RR隔离级别都支持）
- GapLock锁：间隙锁，锁定索引记录间隙（不包括记录本身），确保索引记录的间隙不变。（范围锁，RR隔离级别支持）
- Next-key Lock锁：记录锁和间隙锁组合，同时锁住数据，并且锁住数据前后范围（记录锁+范围锁，RR隔离级别支持）

![](.\pic\mysql\Next-key lock.jpg)

## 总结

- RR隔离级别下间隙锁才有效，RC隔离级别下没有间隙锁
- RR隔离级别下为了解决“幻读”问题，“快照读”依靠MVCC控制，“当前读”通过间隙锁解决
- 间隙锁和行锁合称next-key lock，每个next-key lock是前开后闭区间
- 间隙锁的引入，可能会导致同样语句锁住更大的范围，影响并发度