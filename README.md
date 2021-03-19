# 数据库

## MySQL

### 数据库优化

**优化顺序**

sql语句优化 -> 索引优化 -> 加缓存 -> 读写分离 -> 分区 -> 分布式数据库（垂直切分）-> 水平切分 

**设计表时要注意**

-  表字段避免null值出现，null值很难查询优化且占用额外的索引空间，推荐默认数字0代替null。
-  尽量使用INT而非BIGINT，如果非负则加上UNSIGNED（这样数值容量会扩大一倍），当然能使用TINYINT、SMALLINT、MEDIUM_INT更好。
-  使用枚举或整数代替字符串类型
-  尽量使用TIMESTAMP而非DATETIME
-  单表不要有太多字段，建议在20以内
-  用整型来存IP

**索引**

-  索引并不是越多越好，要根据查询有针对性的创建，考虑在WHERE和ORDER BY命令上涉及的列建立索引，可根据EXPLAIN来查看是否用了索引还是全表扫描
-  应尽量避免在WHERE子句中对字段进行NULL值判断，否则将导致引擎放弃使用索引而进行全表扫描
-  值分布很稀少的字段不适合建索引，例如"性别"这种只有两三个值的字段
-  字符字段只建前缀索引
-  字符字段最好不要做主键
-  不用外键，由程序保证约束
-  尽量不用UNIQUE，由程序保证约束
-  使用多列索引时主意顺序和查询条件保持一致，同时删除不必要的单列索引

**简言之就是使用合适的数据类型，选择合适的索引**

1.选择合适的数据类型

- （1）使用可存下数据的最小的数据类型，整型 < date,time < char,varchar < blob
- （2）使用简单的数据类型，整型比字符处理开销更小，因为字符串的比较更复杂。如，int类型存储时间类型，bigint类型转ip函数
- （3）使用合理的字段属性长度，固定长度的表会更快。使用enum、char而不是varchar
- （4）尽可能使用not null定义字段
- （5）尽量少用text，非用不可最好分表

 2.选择合适的索引列

- （1）查询频繁的列，在where，group by，order by，on从句中出现的列
- （2）where条件中<，<=，=，>，>=，between，in，以及like 字符串+通配符（%）出现的列
- （3）长度小的列，索引字段越小越好，因为数据库的存储单位是页，一页中能存下的数据越多越好
- （4）离散度大（不同的值多）的列，放在联合索引前面。查看离散度，通过统计不同的列值来实现，count越大，离散程度越高：

**sql的编写需要注意优化**

-  使用limit对查询结果的记录进行限定
-  避免select *，将需要查找的字段列出来
-  使用连接（join）来代替子查询
-  拆分大的delete或insert语句
-  可通过开启慢查询日志来找出较慢的SQL
-  不做列运算：SELECT id WHERE age + 1 = 10，任何对列的操作都将导致表扫描，它包括数据库教程函数、计算表达式等等，查询时要尽可能将操作移至等号右边
-  sql语句尽可能简单：一条sql只能在一个cpu运算；大语句拆小语句，减少锁时间；一条大sql可以堵死整个库
-  OR改写成IN：OR的效率是n级别，IN的效率是log(n)级别，in的个数建议控制在200以内
-  不用函数和触发器，在应用程序实现
-  避免%xxx式查询
-  少用JOIN
-  使用同类型进行比较，比如用'123'和'123'比，123和123比
-  尽量避免在WHERE子句中使用!=或<>操作符，否则将引擎放弃使用索引而进行全表扫描
-  对于连续数值，使用BETWEEN不用IN：SELECT id FROM t WHERE num BETWEEN 1 AND 5
-  列表数据不要拿全表，要使用LIMIT来分页，每页数量也不要太大

**MyISAM和InnoDB的区别**

1. InnoDB支持事务，MyISAM不支持，对于InnoDB每一条SQL语言都默认封装成事务，自动提交，这样会影响速度，所以最好把多条SQL语言放在begin和commit之间，组成一个事务； 
2. InnoDB支持外键，而MyISAM不支持。对一个包含外键的InnoDB表转为MYISAM会失败； 
3. InnoDB不保存表的具体行数，执行select count(*) from table时需要全表扫描。而MyISAM用一个变量保存了整个表的行数，执行上述语句时只需要读出该变量即可，速度很快； 
4. Innodb不支持全文索引，而MyISAM支持全文索引，查询效率上MyISAM要高； 
5. 锁机制不同: InnoDB 为行级锁，myisam 为表级锁。 

**索引查找、索引扫描**

全表扫描：读取表中所有的行
索引扫描：类似全表扫描
索引查找：定位到索引指向的局部位置

- 隐式转换容易从索引查找变成索引扫描

- where子句中的谓词不是联合索引的第一列对于联合索引最左边一列存有统计信息，其他列sqlserver不存统计信息

- where 子句里串联会导致索引失效 where A+B = ... (索引为A，B联合索引)

- =,>,<,>=,<=,between,以及部分like(like'%XXX') 
- 两个相关联的表的格式不一致

**事务的ACID**

1. 原子性：整个事务中的所有操作，要么全部完成，要么全部不完成 

   （由redo/undo log日志保证，它记录了需要回滚的日志信息，事务回滚时撤销已经执行成功的sql）

2. 一致性：事务必须始终保持系统处于一致的状态，不管在任何给定的时间[**并发事务有多少

   （一般由代码层面来保证）

3. 隔离性：同一时间仅有一个请求用于同一数据（MVCC来保证）

4. 持久性：在事务完成以后，该事务对数据库所作的更改便持久的保存在数据库之中，并不会被回滚

   （由内存+redo log来保证，mysql修改数据同时在内存和redo log记录这次操作，事务提交的时候通过redo log刷盘，宕机的时候可以从redo log恢复）

**事务的隔离级别**

1. Read Uncommitted（未提交读） ：事务中的修改，即使没有提交，其他事务也可以看得到，会导致“脏读”、“幻读”和“不可重复读取;
2. READ COMMITTED （提交读）：大多数主流数据库的默认事务等级，保证了一个事务不会读到另一个并行事务已修改但未提交的数据，避免了“脏读”，但不能避免“幻读”和“不可重复读取”。该级别适用于大多数系统。
3. REPEATABLE READ（重复读） ：保证了一个事务不会修改已经由另一个事务读取但未提交（回滚）的数据。避免了“脏读取”和“不可重复读取”的情况，但不能避免“幻读”，但是带来了更多的性能损失。
4. Serializable （串行化）：最严格的级别，事务串行执行，资源消耗最大；

脏读：读取了另一个事务未提交的数据

不可重复读：读取一次，在第二次读之前更新了这个数据导致两次数据不同。

幻读：读取一次，在读第二次之前insert或者是delete，导致读取的记录数不同

​		   例如：第一个事务查询一个User表id=100发现不存在该数据行，这时第二个事务又进来了，新增了一条id=100的数据行并且提交了事务。这时第一个事务新增一条id=100的数据行会报主键冲突，第一个事务再select一下，发现id=100数据行已经存在，这就是幻读。

**锁的类型**

事务的ACID是通过InnoDB日志和锁来保证。事务的隔离性是通过数据库锁的机制实现的，持久性通过redo log（重做日志）来实现，原子性和一致性通过Undo log（回撤日志）来实现。Undo Log的原理很简单，为了满足事务的原子性，在操作任何数据之前，首先将数据备份到一个地方（这个存储数据备份的地方称为Undo Log）。然后进行数据的修改。如果出现了错误或者用户执行了roll back语句，系统可以利用Undo Log中的备份将数据恢复到事务开始之前的状态。 和Undo Log相反，Redo Log记录的是新数据的备份。在事务提交前，只要将RedoLog持久化即可，不需要将数据持久化。当系统崩溃时，虽然数据没有持久化，但是RedoLog已经持久化。系统可以根据Redo Log的内容，将所有数据恢复到最新的状态。

只有增、删、改才会加上排它锁，而只是查询并不会加锁，只能通过在select语句后显式加lock in share mode或者for update来加共享锁或者排它锁。

**什么是全表扫描**

全表扫描 (又称顺序扫描) 就是在数据库中进行逐行扫描，顺序读取表中的每一行记录，然后检查各个列是否符合查询条件。这种扫描是已知最慢的，因为需要进行大量的磁盘 I/O，而且从磁盘到内存的传输开销也很大。

**为什么数据库的性能瓶颈一般出现在IO上面**

IO永远是数据库最容易瓶颈的地方，大部分数据库操作中超过90%的时间都是 IO 操作所占用的，减少 IO 次数是 SQL 优化中需要第一优先考虑。

innodb_buffer池相关的，以及跟读数据块最相关的核心函数buf_page_get_gen函数以及其调用的相关子函数

buf_page_get_gen函数的作用是从Buffer bool里面读数据页

### 主从复制

1. master开启bin-log功能，日志文件用于记录数据库的读写增删
2. 需要开启3个线程，master IO线程，slave开启 IO线程 SQL线程
3. Slave 通过IO线程连接master，并且请求某个bin-log，position之后的内容。
4. MASTER服务器收到slave IO线程发来的日志请求信息，io线程去将bin-log内容，position返回给slave IO线程。
5. slave服务器收到bin-log日志内容，将bin-log日志内容写入relay-log中继日志，创建一个master.info的文件，该文件记录了master ip 用户名 密码 master bin-log名称，bin-log position。
6. slave端开启SQL线程，实时监控relay-log日志内容是否有更新，解析文件中的SQL语句，在slave数据库中去执行。

### MVCC

实现MVCC时用到了一致性视图，用于支持读提交和可重复读的实现。在实现可重复读的隔离级别，只需要在事务开始的时候创建一致性视图，也叫做快照，之后的查询里都共用这个一致性视图，后续的事务对数据的更改是对当前事务是不可见的，这样就实现了可重复读。而读提交，每一个语句执行前都会重新计算出一个新的视图，这个也是可重复读和读提交在MVCC实现层面上的区别。

**两个事务执行写操作，又怎么保证并发**

假如事务1和事务2都要执行update操作，事务1先update数据行的时候，先回获取行锁，锁定数据，当事务2要进行update操作的时候，也会取获取该数据行的行锁，但是已经被事务1占有，事务2只能wait。

若是事务1长时间没有释放锁，事务2就会出现超时异常

## Redis

数据类型：

Redis支持五种数据类型：string（字符串），hash（哈希），list（列表），set（集合）及zset(sorted set：有序集合)。

### 持久化

- Redis DataBase(简称RDB)
  - 执行机制：快照，直接将databases中的key-value的二进制形式存储在了rdb文件中
  - 优点：性能较高（因为是快照，且执行频率比aof低，而且rdb文件中直接存储的是key-values的二进制形式，对于恢复数据也快）
  - 使用单独子进程来进行持久化，主进程不会进行任何IO操作，保证了redis的高性能
  - 缺点：在save配置条件之间若发生宕机，此间的数据会丢失
  - RDB是间隔一段时间进行持久化，如果持久化之间redis发生故障，会发生数据丢失。所以这种方式更适合数据要求不严谨的时候
- Append-only file (简称AOF)
  - 执行机制：将对数据的每一条修改命令追加到aof文件
  - 优点：数据不容易丢失
  - 可以保持更高的数据完整性，如果设置追加file的时间是1s，如果redis发生故障，最多会丢失1s的数据；且如果日志写入不完整支持redis-check-aof来进行日志修复；AOF文件没被rewrite之前（文件过大时会对命令进行合并重写），可以删除其中的某些命令（比如误操作的flushall
  - 缺点：性能较低（每一条修改操作都要追加到aof文件，执行频率较RDB要高，而且aof文件中存储的是命令，对于恢复数据来讲需要逐行执行命令，所以恢复慢）
  - AOF文件比RDB文件大，且恢复速度慢。

### 哨兵模式

### 集群

## 文档型数据库

- MongoDB与EleasticSearch相同点：

1、都是以json格式管理数据的nosql数据库。
2、都支持CRUD操作。
3、都支持聚合和全文检索。
4、都支持分片和复制。
5、都支持阉割版的join操作。
6、都支持处理超大规模数据。
7、目前都不支持事务或者叫支持阉割版的事务。

- 不同点：

1、开发语言不同：ES的Java语言(restful)，Mongo是C++语言(driver)，从开发角度来看，ES对Java更方便
2、分片方式：ES是hash，Mongo是range和hash
3、分布式：ES的主副分片自动组合和配置，Mongo需要手动配置集群“路由+服务配置+sharding”
4、索引：ES自建倒排索引，检索力度强，Mongo手动创建索引（B树），不支持倒排索引；es所有字段自动索引，mongodb的字段需要手动索引。
5、检索字段：ES全文检索，可用的检索插件较多，Mongo对索引字段个数有限制，全文检索效率低乃至不采用

- 倒排索引

倒排索引源于实际应用中需要**根据属性的值**来查找记录。这种索引表中的每一项都包括**一个属性值和具有该属性值的各记录的地址**。由于不是由记录来确定属性值，而是由属性值来确定记录的位置，因而称为倒排索引(inverted index) [参考](https://www.cnblogs.com/fly1988happy/archive/2012/04/01/2429000.html)

# 通信

## TCP

**为什么连接的时候是三次握手，关闭的时候却是四次握手？**

因为当Server端收到Client端的SYN连接请求报文后，可以直接发送SYN+ACK报文。其中ACK报文是用来应答的，SYN报文是用来同步的。但是关闭连接时，当Server端收到FIN报文时，很可能并不会立即关闭SOCKET，所以只能先回复一个ACK报文，告诉Client端，"你发的FIN报文我收到了"。只有等到我Server端所有的报文都发送完了，我才能发送FIN报文，因此不能一起发送。故需要四步握手。

**TCP建立连接的过程采用三次握手，已知第三次握手报文的发送序列号为1000，确认序列号为2000，请问第二次握手报文的发送序列号和确认序列号分别为？**
参考上面TCP连接建立的图。
客户端：发送X
服务端：发送Y， 确认X+1
客户端：发送X+1（1000），确认Y+1（2000）
可以反推第二次为1999,确认1000

**如何保证TCP连接的可靠性**

1. 校验和：发送的数据包的二进制相加然后取反，目的是检测数据在传输过程中的任何变化。如果收到段的检验和有差错，TCP将丢弃这个报文段和不确认收到此报文段。 
2. 确认应答+序列号：TCP给发送的每一个包进行编号，接收方对数据包进行排序，把有序数据传送给应用层。 
3. 超时重传：当TCP发出一个段后，它启动一个定时器，等待目的端确认收到这个报文段。如果不能及时收到一个确认，将重发这个报文段。 
4. 流量控制：TCP连接的每一方都有固定大小的缓冲空间，TCP的接收端只允许发送端发送接收端缓冲区能接纳的数据。当接收方来不及处理发送方的数据，能提示发送方降低发送的速率，防止包丢失。TCP使用的流量控制协议是可变大小的滑动窗口协议。  
5. 拥塞控制：当网络拥塞时，减少数据的发送。       

## RabbitMQ

# 数据结构

## 链表

单链表的特点

- 链表增删元素的时间复杂度为O(1),查找一个元素的时间复杂度为 O(n);
- 单链表不用像数组那样预先分配存储空间的大小，避免了空间浪费
- 单链表不能进行回溯操作，如：只知道链表的头节点的时候无法快读快速链表的倒数第几个节点的值。

## B树、B+树

**平衡二叉树**

是基于二分法的策略提高数据的查找速度的二叉树的数据结构，用这个树形结构的数据减少无关数据的检索，大大的提升了数据检索的速度（AVL：高度平衡二叉树）。（特点：

（1）非叶子节点最多拥有两个子节点；

（2）非叶子节值大于左边子节点、小于右边子节点；

（3）树的左右两边的层级数相差不会大于1;

（4）没有值相等重复的节点;

**B树**

- 根节点至少有两个子节点（所以B树不一定是二叉树）
- 每个节点有M-1个key，并且以升序排列
- 位于M-1和M key的子节点的值位于M-1 和M key对应的Value之间
- 其它节点至少有M/2个子节点

B树相对于平衡二叉树的不同是，每个节点包含的关键字增多了，特别是在B树应用到数据库中的时候，数据库充分利用了磁盘块的原理（磁盘数据存储是采用块的形式存储的，每个块的大小一般为4K，每次IO进行数据读取时，同一个磁盘块的数据可以一次性读取出来）把节点大小限制和充分使用在磁盘快大小范围；把树的节点关键字增多后树的层级比原来的二叉树少了，减少数据查找的次数和复杂度;

**B+树**

B+树是B树的一个升级版，相对于B树来说B+树更充分的利用了节点的空间，让查询速度更加稳定，其速度完全接近于二分法查找。为什么说B+树查找的效率要比B树更高、更稳定；我们先看看两者的区别

（1）B+跟B树不同B+树的非叶子节点不保存关键字记录的指针，这样使得B+树每个节点所能保存的关键字大大增加；

（2）B+树叶子节点保存了父节点的所有关键字和关键字记录的指针，每个叶子节点的关键字从小到大链接；

（3）B+树的根节点关键字数量和其子节点个数相等;

（4）B+的非叶子节点只进行数据索引，不会存实际的关键字记录的指针，所有数据地址必须要到叶子节点才能获取到，所以每次数据查询的次数都一样；

特点：

在B树的基础上每个节点存储的关键字数更多，树的层级更少所以查询数据更快，所有指关键字指针都存在叶子节点，所以每次查找的次数都相同所以查询速度更稳定;

## 红黑树

R-B Tree，全称是Red-Black Tree，又称为“红黑树”，它一种特殊的二叉查找树。红黑树的每个节点上都有存储位表示节点的颜色，可以是红(Red)或黑(Black)。

红黑树的特性:

（1）每个节点或者是黑色，或者是红色。
（2）根节点是黑色。
（3）每个叶子节点（NIL）是黑色。 [注意：这里叶子节点，是指为空(NIL或NULL)的叶子节点！]
（4）如果一个节点是红色的，则它的子节点必须是黑色的。
（5）从一个节点到该节点的子孙节点的所有路径上包含相同数目的黑节点。

# JVM

## 内存结构

<p>
<image src="./documents/img/3.jpg"></image>    
</p>

## 编译过程

JAVA源码编译由三个过程组成：

源码编译机制。类加载机制。类执行机制

**代码编译**

由JAVA源码编译器来完成。主要是将源码编译成字节码文件（class文件）。字节码文件格式主要分为两部分：常量池和方法字节码。

**类加载机制**

- 加载：根据一个类的全限定名(如cn.edu.hdu.test.HelloWorld.class)来读取此类的二进制字节流到JVM内部
- 验证：验证阶段主要包括四个检验过程：文件格式验证、元数据验证、字节码验证和符号引用验证
- 准备：为类中的所有静态变量分配内存空间，并为其设置一个初始值
- 解析：将常量池中所有的符号引用转为直接引用
- 初始化：则是根据程序员自己写的逻辑去初始化类变量和其他资源

**自定义类加载器**

要创建用户自己的类加载器，只需要继承java.lang.ClassLoader类，然后覆盖它的findClass(String name)方法即可，即指明如何获取类的字节码流。

如果要符合双亲委派规范，则重写findClass方法（用户自定义类加载逻辑）；要破坏的话，重写loadClass方法(双亲委派的具体逻辑实现)

https://www.cnblogs.com/aspirant/p/7200523.html

## 装箱与拆箱

有了基本类型之后为什么还要有包装器类型呢？核心：让基本类型具备对象的特征,实现更多的功能.

装箱过程是通过调用包装器的valueOf方法实现的，而拆箱过程是通过调用包装器的 xxxValue方法实现的。（xxx代表对应的基本数据类型）

## 双亲委派机制

概念：

如果一个类加载器收到了类加载请求，它并不会自己先去加载，而是把这个请求委托给父类的加载器去执行，如果父类加载器还存在其父类加载器，则进一步向上委托，依次递归，请求最终将到达顶层的启动类加载器，如果父类加载器可以完成类加载任务，就成功返回，倘若父类加载器无法完成此加载任务，子加载器才会尝试自己去加载

双亲委派模型有效解决了以下问题：

- 每一个类都只会被加载一次，避免了重复加载
- 每一个类都会被尽可能的加载（从引导类加载器往下，每个加载器都可能会根据优先次序尝试加载它）
- 有效避免了某些恶意类的加载（比如自定义了Java.lang.Object类，一般而言在双亲委派模型下会加载系统的Object类而不是自定义的Object类）

> 问：可以不可以自己写个String类
>
> 答案：不可以，因为 根据类加载的双亲委派机制，会去加载父类，父类发现冲突了String就不再加载了;

## GC算法

**标记-清除收集算法**

首先标记出所有需要回收的对象，在标记完成后统一回收所有被标记的对象

不足：

1. 标记和清除效率都不高
2. 标记清除后悔产生大量不连续的内存碎片

**复制算法**

将内存划分为大小相同的两块，每次只使用其中一块。当这一块内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用的内存空间一次清理掉

不足：

1. 对象存活率较高时要进行较多的复制操作，效率将会变低
2. 如果不想浪费50%的空间，需要有额外的空间进行分配担保，以应对被是使用的内粗中所有对象都100%存活的极端情况

**标记-整理算法**

标记过程与 “标记-清除” 算法一致，但后续步骤部署直接对可回收对象进行清理，而是让所有存活对象都向一端移动，然后直接清理掉端边界以外内存

**分代收集算法**

没有什么新的思想，加入了“代零”的概念，进行分代回收

## JVM调优



# 部署相关

## Docker

## k8s

## Nginx

# 语言相关

## Java

### 集合


- List集合是有序集合，集合中的元素可以重复，访问集合中的元素可以根据元素的索引来访问，查找元素效率高，插入删除效率低。
- Set集合是无序集合，集合中的元素不可以重复，检索效率低下，删除和插入效率高，访问集合中的元素只能根据元素本身来访问
- Map集合中保存Key-value对形式的元素，访问时只能根据每项元素的key来访问其value。

**ArrayList与Vector的区别**

1. 两者都是基于索引，内部结构是数组

2. 元素存取有序并都允许为null

3. 都支持fail-fast机制

4. Vector是同步的，不会过载，而ArrayList不是，但ArrayList效率比Vector高，如果在迭代中对集合做修改可以使用CopyOnWriteArrayList

5. 初始容量都为10，但ArrayList默认增长为原来的50%，而Vector默认增长为原来的一倍，并且可以设置
   ArrayList更通用，可以使用Collections工具类获取同步列表和只读列表

   适用场景分析： 
   1、Vector是线程同步的，所以它也是线程安全的，而ArrayList是线程异步的，是不安全的。如果不考虑到线程的安全因素，一般用ArrayList效率比较高。 
   2、如果集合中的元素的数目大于目前集合数组的长度时，在集合中使用数据量比较大的数据，用Vector有一定的优势。

**ArrayList与LinkedList的区别**

1. ArrayList基于数组，LinkedList基于链表
2. ArrayList查找快，LinkedList插入删除快
3. 随机查找频繁用ArrayList,插入删除频繁用LinkedList

**ArrayList,Vector,HashMap,Hashtable扩容机制？**

1. arraylist,初始容量10，(oldCapacity * 3)/2 + 1
2. vector,初始容量10，oldCapacity * 2
3. hashmap,初始容量16，达到阀值扩容，为原来的两倍
4. hashtable，初始容量11，达到阀值扩容，oldCapacity * 2 + 1

**HashMap实现原理**

- hashmap是数组和链表的结合体，数组每个元素存的是链表的头结点
- 往hashmap里面放键值对的时候先得到key的hashcode，然后重新计算hashcode，（让1分布均匀因为如果分布不均匀，低位全是0，则后来计算数组下标的时候会冲突），然后与length-1按位与，计算数组出数组下标
- 如果该下标对应的链表为空，则直接把键值对作为链表头结点，如果不为空，则遍历链表看是否有key值相同的，有就把value替换，没有就把该对象最为链表的第一个节点，原有的节点最为他的后续节点
- 初始容量16，达到阀值扩容，阀值等于最大容量*负载因子，扩容每次2倍，总是2的n次方

**HashMap HashTable区别？，Hashmap key可以是任何类型吗？**

- 区别：
  1. HashTable的方法是同步的，HashMap方法未经同步，所以在多线程场合要手动同步HashMap这个区别和Vector和ArrayList一样
  2. Hashtable不允许null值（key和value都不可以），HashMap允许null值（key和value都可以）
  3. 哈希值的使用不同，Hashtable直接使用对象的HashCode。二HashMap重新计算Hash值，而且用于代替求模。
  4. HashTable中Hash数字默认大小是11，增加的方式是old*2+1。HashMap中Hash数组的默认大小是16，而且一定是2的指数。
- 需要同时重写该类的hashCode()方法和它的equals()方法。

**并发集合—ConcurrentHashMap**

ConcurrentHashMap是线程安全的，用来替代HashTable。

ConcurrentHashMap使用分段锁技术，将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问，能够实现真正的并发访问。

**并发集合—CopyOnWriteArrayList和CopyOnWriteArraySet**

CopyOnWrite容器即写时复制的容器。通俗的理解是当我们往一个容器添加元素的时候，不直接往当前容器添加，而是先将当前容器进行Copy，复制出一个新的容器，然后新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器。这样做的好处是我们可以对CopyOnWrite容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素。所以CopyOnWrite容器也是一种读写分离的思想，读和写不同的容器。

### 多线程


**同步、异步、阻塞、非阻塞**

阻塞非阻塞：描述的是一个线程/进程的状态 它等在那里了 他就是阻塞了；

同步异步： 描述的是一个消息机制或者说一个消息系统 是关于组件之间如何传递和处理消息的

阻塞可以是异步的（a阻塞了但a和b是异步的）；非阻塞也可以是同步的（a非阻塞但a和b是同步的）。

**实现多线程方式**

- 实现多线程方式
  - 继承Thread类，重写run函数
  - 实现Runnable接口
  - 实现Callable接口
- 三种方式区别
  - 实现Runnable接口可以避免java单继承特性带来的局限；增强程序的健壮性，代码能够被多个线程共享，代码与数据是独立的；适合多个相同程序代码的线程区处理同一资源的情况
  - 继承Thread和实现Runnable接口启动线程都是使用start方法，然后JVM虚拟机将此线程放倒就绪队列中，如果有处理机可用，则执行run方法
  - 实现Callable接口要实现call方法，并且线程执行完毕后会有返回值，其他两种都是重新run，没有返回值

**Lock和synchronized的选择**

1. Lock是一个接口，而synchronized是Java中的关键字，synchronized是内置的语言实现；
2. synchronized在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象发生；而Lock在发生异常时，如果没有主动通过unLock()去释放锁，则很可能造成死锁现象，因此使用Lock时需要在finally块中释放锁；
3. Lock可以让等待锁的线程响应中断，而synchronized却不行，使用synchronized时，等待的线程会一直等待下去，不能够响应中断；
4. 通过Lock可以知道有没有成功获取锁，而synchronized却无法办到。
5. Lock可以提高多个线程进行读操作的效率。

**Volatile**

解释：被volatile修饰的变量对所有线程可见，它是放在共享内存中的

- 具有可见性：确保释放锁之前对共享数据做出的更改对于随后获得该锁的另一个线程是可见的
- 没有原子性：只有一个线程能够执行一段代码，这段代码通过一个monitor object保护。从而防止多个线程在更新共享状态时相互冲突

使用场景：

1. 希望用轻量级的同步提高性能
2. 对变量的写操作不依赖于当前值
3. 该变量没有包含在具有其他变量的不变式中

**wait,notify,notifyAll**

- wait:导致当前线程等待，这个方法会释放锁，所以需要在同步代码块中调用(否则会发生
  IllegalMonitorStateException的异常
- notify:随机选择一个等待中的线程将其唤醒；notify()调用后，并不是马上就释放对象锁
  的，而是在相应的synchronized(){}语句块执行结束，自动释放锁后，JVM会在wait()对象
  锁的线程中随机选取一线程，赋予其对象锁，唤醒线程，继续执行。
- notifyAll:将所有等待的线程唤醒

**join,sleep,yield**

- join:等待调用该方法的线程执行完毕后再往下继续执行(该方法也要捕获异常)
- sleep:使调用该方法的线程暂停执行一段时间，让其他线程有机会继续执行，但它并不释
  放对象锁。也就是如果有Synchronized同步块，其他线程仍然不同访问共享数据(注意该
  方法要捕获异常)
- yeild:与sleep()类似，只是不能由用户指定暂停多长时间，并且yield()方法只能让同优先
  级的线程有执行的机会

**可重入锁**

可重入的主语是已经获得该锁的线程，可重入指的就是可以再次进入，因此，意思就是已经获得该锁的线程可以再次进入被该锁锁定的代码块。内部通过计数器实现。

1. 可重入锁（递归锁）：可以再次进入方法A，就是说在释放锁前此线程可以再次进入方法A（方法A递归）。
2. 不可重入锁（自旋锁）：不可以再次进入方法A，也就是说获得锁进入方法A是此线程在释放锁钱唯一的一次进入方法A

CAS算法 即compare and swap（比较与交换），是一种有名的无锁算法。当且仅当 V 的值等于 A时，CAS通过原子方式用新值B来更新V的值，否则不会执行任何操作

**并发编程特性**

1. 原子性：即一个或者多个操作作为一个整体，要么全部执行，要么都不执行，并且操作在执行过程中不会被线程调度机制打断
2. 可见性：当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值
3. 有序性：即程序执行的顺序按照代码的先后顺序执行

**ThreadLocal**

ThreadLocal类提供了如下方法：

```java
public T get() { }
public void set(T value) { }
public void remove() { }
protected T initialValue() { }
```

- ThreadLocal的作用是数据隔离，在每一个线程中创建一个新的数据对象，每一个线程使用的是不一样的
- ThreadLocal内部的ThreadLocalMap键为弱引用，会有内存泄漏的风险。(使用完ThreadLocal之后，记得调用remove方法)
- 适用于无状态，副本变量独立后不影响业务逻辑的高并发场景。

在著名的框架Hiberante中，数据库连接的代码：

```java
private static final ThreadLocal threadSession = new ThreadLocal();  

public static Session getSession() throws InfrastructureException {  
    Session s = (Session) threadSession.get();  
    try {  
        if (s == null) {  
            s = getSessionFactory().openSession();  
            threadSession.set(s);  
        }  
    } catch (HibernateException ex) {  
        throw new InfrastructureException(ex);  
    }  
    return s;  
}  
```

**线程池**

构造函数：

```java
public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,BlockingQueue<Runnable> workQueue);
```

- corePoolSize：核心池的大小
- maximumPoolSize：线程池最大线程数，它表示在线程池中最多能创建多少个线程
- keepAliveTime：表示线程没有任务执行时最多保持多久时间会终止。
- unit：参数keepAliveTime的时间单位
- workQueue：一个阻塞队列，用来存储等待执行的任务

在ThreadPoolExecutor类中有几个非常重要的方法：

```
`execute()``submit()``shutdown()``shutdownNow()`
```

线程池的关闭　　

- shutdown()：不会立即终止线程池，而是要等所有任务缓存队列中的任务都执行完后才终止，但再也不会接受新的任务
- shutdownNow()：立即终止线程池，并尝试打断正在执行的任务，并且清空任务缓存队列，返回尚未执行的任务

**CountDownLatch、CyclicBarrier和Semaphore**

1. CountDownLatch：利用它可以实现类似计数器的功能。比如有一个任务A，它要等待其他4个任务执行完毕之后才能执行，此时就可以利用CountDownLatch来实现这种功能了。

   ```java
   //调用await()方法的线程会被挂起，它会等待直到count值为0才继续执行
   public void await() throws InterruptedException { };   
   //和await()类似，只不过等待一定的时间后count值还没变为0的话就会继续执行
   public boolean await(long timeout, TimeUnit unit) throws InterruptedException { }; 
   public void countDown() { };  //将count值减1
   ```

2. CyclicBarrier：所有等待线程都被释放以后，CyclicBarrier可以被重用

   当所有线程线程写入操作完毕之后，所有线程就继续进行后续的操作了。如果说想在所有线程写入操作完之后，进行额外的其他操作可以为CyclicBarrier提供Runnable参数

3. Semaphore：翻译成字面意思为 信号量，Semaphore可以控同时访问的线程个数，通过 acquire() 获取一个许可，如果没有就等待，而 release() 释放一个许可。

**Syschronized关键字 Sychronized代码块区别？static synchroniezd?**

- synchronized代码块比synchronized方法要灵活。因为也许一个方法中只有一部分代码只需要同步，如果此时对整个方法用synchronized进行同步，会影响程序执行效率。而使用synchronized代码块就可以避免这个问题，synchronized代码块可以实现只对需要同步的地方进行同步。
- 如果一个线程执行一个对象的非static synchronized方法，另外一个线程需要执行这个对象所属类的static synchronized方法，此时不会发生互斥现象，因为访问static synchronized方法占用的是类锁，而访问非static synchronized方法占用的是对象锁，所以不存在互斥现象

**Callable、Future、FutureTask**



### NIO

参考：http://wiki.jikexueyuan.com/project/java-nio-zh/java-nio-tutorial.html

Java NIO基本组件如下：


- 通道和缓冲区(*Channels and Buffers*)：在标准I/O API中，使用字符流和字节流。 在NIO中，使用通道和缓冲区。数据总是从缓冲区写入通道，并从通道读取到缓冲区。
- 选择器(*Selectors*)：Java NIO提供了“选择器”的概念。这是一个可以用于监视多个通道的对象，如数据到达，连接打开等。因此，单线程可以监视多个通道中的数据。
- 非阻塞I/O(*Non-blocking I/O*)：Java NIO提供非阻塞I/O的功能。这里应用程序立即返回任何可用的数据，应用程序应该具有池化机制，以查明是否有更多数据准备就绪。

### 反射

获取类对象有三种方法：

- 通过forName() -> 示例：Class.forName(“PeopleImpl”)
- 通过getClass() -> 示例：new PeopleImpl().getClass()
- .class直接获取 -> 示例：PeopleImpl.class

常用方法：

- getName()：获取类完整方法；
- getSuperclass()：获取类的父类；
- newInstance()：创建实例对象；
- getFields()：获取当前类和父类的public修饰的所有属性；
- getDeclaredFields()：获取当前类（不包含父类）的声明的所有属性；
- getMethod()：获取当前类和父类的public修饰的所有方法；
- getDeclaredMethods()：获取当前类（不包含父类）的声明的所有方法；

**动态代理**

动态代理是一种方便运行时动态构建代理、动态处理代理方法调用的机制，很多场景都是利用类似机制做到的，比如用来包装 RPC 调用、面向切面的编程（AOP）

动态代理技术的常见实现方式有两种：

1. 基于接口的 JDK 动态代理

   JDK Proxy 是通过实现 InvocationHandler 接口来实现的，代码如下：

   ```java
   // JDK 代理类
   class AnimalProxy implements InvocationHandler {
       private Object target; // 代理对象
       public Object getInstance(Object target) {
           this.target = target;
           // 取得代理对象
           return Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(), this);
       }
       @Override
       public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
           System.out.println("调用前");
           Object result = method.invoke(target, args); // 方法调用
           System.out.println("调用后");
           return result;
       }
   }
   public static void main(String[] args) {
       // JDK 动态代理调用
       AnimalProxy proxy = new AnimalProxy();
       Animal dogProxy = (Animal) proxy.getInstance(new Dog());
       dogProxy.eat();
   }
   </pre>
   ```

   注意：JDK Proxy 只能代理实现接口的类（即使是extends继承类也是不可以代理的）。

2. 基于继承的 CGLib 动态代理

   Cglib 是针对类来实现代理的，他的原理是对指定的目标类生成一个子类，并覆盖其中方法实现增强，但因为采用的是继承，所以不能对 final 修饰的类进行代理。Cglib 可以通过 Maven 直接进行版本引用

JDK Proxy 的优势：

- 最小化依赖关系，减少依赖意味着简化开发和维护，JDK 本身的支持，更加可靠；
- 平滑进行 JDK 版本升级，而字节码类库通常需要进行更新以保证在新版上能够使用；

Cglib 框架的优势：

- 可调用普通类，不需要实现接口；
- 高性能；

### 异常处理

Exception和RuntimeException区别

1.RuntimeException是Exception的一个子类，因此，通常说的区别，即Exception和继承Exception的RuntimeException的区别

​     RuntimeException:运行时异常，可以理解为必须运行才能发现的异常，因此运行之前可以不catch，抛异常时，则交由上级(JVM)处理，bug中断程序

​     非RuntimeException:必须有try...catch处理

2.从方法的设计者角度来说

​    RuntimeException：方法使用者无法处理的异常

​    非RuntimeException：方法使用者能处理的异常，如读取文件，使用者完全可以处理文件不处理的情况

3.从2的角度出发，可以看看异常都有哪些

   RuntimeException：NullPointerException、NumberFormatException、ArrayIndexOutOfBoundsException等转换、越界、计算类型异常

 非RuntimeException：SQLException、IOException

Error：

一般留给JDK内部自己使用，比如内存溢出OutOfMemoryError，这类严重的问题，应用进程什么都做不了，只能终止。用户抓住此类Error，一般无法处理，尽快终止往往是最安全的方式

《Effictive Java》：对于可以恢复的情况使用检查异常，对于编程中的错误使用运行异常

## C#

# Spring

## IOC与AOP

**ioc是什么，有什么用？**

依赖倒置原则：
a.高层模块不应该依赖于底层模块，二者都应该依赖于抽象。
b.抽象不应该依赖于细节，细节应该依赖于抽象。

概念：

资源不由使用资源的双方管理，而由不使用资源的第三方管理，这可以带来很多好处。第一，资源集中管理，实现资源的可配置和易管理。第二，降低了使用资源双方的依赖程度。

**bean作用域有哪些，说一下各种使用场景？**

单例（Singleton）：在整个应用中，只创建bean的一个实例；
原型（Prototype）：每次注入或者通过Spring上下文获取的时候，都会创建一个新的bean实例；
会话（Session）：在Web应用中，为每个会话创建一个bean实例；
请求（Request）：在Web应用中，为每次请求创建一个bean实例；

**aop是什么，有哪些实现方式？**

```text
面向切面编程，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术
```

- Aspect（切面）： Aspect 声明类似于 Java 中的类声明，在 Aspect 中会包含着一些 Pointcut 以及相应的 Advice。
- Joint point（连接点）：表示在程序中明确定义的点，典型的包括方法调用，对类成员的访问以及异常处理程序块的执行等等，它自身还可以嵌套其它 joint point。
- Pointcut（切点）：表示一组 joint point，这些 joint point 或是通过逻辑关系组合起来，或是通过通配、正则表达式等方式集中起来，它定义了相应的 Advice 将要发生的地方。
- Advice（增强）：Advice 定义了在 Pointcut 里面定义的程序点具体要做的操作，它通过 before、after 和 around 来区别是在每个 joint point 之前、之后还是代替执行的代码。
- Target（目标对象）：织入 Advice 的目标对象。
- Weaving（织入）：将 Aspect 和其他对象连接起来, 并创建 Adviced object 的过程

实现方式：

1. 为什么不直接都使用JDK动态代理：JDK动态代理只能代理接口类，所以很多人设计架构的时候会使用XxxService, XxxServiceImpl的形式设计，一是让接口和实现分离，二是也有助于代理。
2. 为什么不都使用Cgilb代理：因为JDK动态代理不依赖其他包，Cglib需要导入ASM包，对于简单的有接口的代理使用JDK动态代理可以少导入一个包。

**拦截器是什么，什么场景使用？**

Servlet中的过滤器Filter是实现了javax.servlet.Filter接口的服务器端程序，主要的用途是过滤字符编码、做一些业务逻辑判断等。其工作原理是，只要你在web.xml文件配置好要拦截的客户端请求，它都会帮你拦截到请求，此时你就可以对请求或响应(Request、Response)统一设置编码，简化操作；同时还可进行逻辑判断，如用户是否已经登陆、有没有权限访问该页面等等工作

**aop里面的cglib原理是什么？**

参考前面反射

**aop切方法的方法的时候，哪些方法是切不了的？为什么？**



## 注解

**@PathVariable **

当使用@RequestMapping URI template 样式映射时， 即 someUrl/{paramId}, 这时的paramId可通过 @Pathvariable注解绑定它传过来的值到方法的参数上。

**@RequestHeader、@CookieValue**

可以把Request请求header部分的值绑定到方法的参数上

**@RequestBody**

作用： 

​      i) 该注解用于读取Request请求的body部分数据，使用系统默认配置的HttpMessageConverter进行解析，然后把相应的数据绑定到要返回的对象上；

​      ii) 再把HttpMessageConverter返回的对象数据绑定到 controller中方法的参数上。

**@ResponseBody**

作用： 

​      该注解用于将Controller的方法返回的对象，通过适当的HttpMessageConverter转换为指定格式后，写入到Response对象的body数据区。

使用时机：

​      返回的数据不是html标签的页面，而是其他某种格式的数据时（如json、xml等）使用；

Statement:(用于执行不带参数的简单 SQL 语句)

PreparedStatement:(用于执行带或不带 IN 参数的预编译 SQL 语句)

CallableStatement :(用于执行对数据库已存储过程的调用)

## MyBatis

 **Mabatis中#{}和${}的区别**

${} 则只是简单的字符串替换；#{} 在预处理时，会把参数部分用一个占位符 ? 代替

#{} 的参数替换是发生在 DBMS 中，而 ${} 则发生在动态解析过程中

优先使用 #{}。因为 ${} 会导致 sql 注入的问题

**通常一个Xml映射文件，都会写一个Dao接口与之对应，请问，这个Dao接口的工作原理是什么？Dao接口里的方法，参数不同时，方法能重载吗？**

Dao接口里的方法，是不能重载的，因为是全限名+方法名的保存和寻找策略。

Dao接口的工作原理是JDK动态代理，Mybatis运行时会使用JDK动态代理为Dao接口生成代理proxy对象，代理对象proxy会拦截接口方法，转而执行MappedStatement所代表的sql，然后将sql执行结果返回。

# 其他

## 分布式

### SOA和微服务

SOA（Service Oriented Architecture）“面向服务的架构”:他是一种设计方法，其中包含多个服务， 服务之间通过相互依赖最终提供一系列的功能。一个服务 通常以独立的形式存在于操作系统进程中。各个服务之间 通过网络调用。

微服务架构:其实和 SOA 架构类似，微服务是在 SOA 上做的升华，微服务架构强调的一个重点是“业务需要彻底的组件化和服务化”，原有的单个业务系统会拆分为多个可以独立开发、设计、运行的小应用。这些小应用之间通过服务完成交互和集成。

### 分布式事务一致性