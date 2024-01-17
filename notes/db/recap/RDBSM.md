
# 数据库理解

1. 为什么需要数据库？
   - 很多应用用场景有共同的需求，有必要抽象出来，单独成为一个系统
2. 什么是一个好的数据管理系统？
   - 功能性
   - 易用性
   - 性能：处理数据的能力和容量
   - 没有一个系统在各个方面都能做到完美
     - 功能性强，会牺牲易用性
   - 数据库最重要的是数据模型

# 关系数据库概述

- 关系模型

  - 主要是为了数据管理的独立性（区别于网状db）

- 如何做到独立

  - 数据库暴露给应用简单的接口，只需要表达需要什么（声明式语言）

- 接口为关系演算—>SQL语言

  - 声明式语言，因此做到了app与dbsm的隔离性

- 表达能力

  - 实际上决定了功能性
  - 没有过程式语言强，可以达到一阶逻辑（fol）子集，比较强

- 关系代数表达式

  - 输入关系（集合），产生关系（选择，投影，连接，聚集cout，max，sum）

## 存储的正确性

CRUD操作保证原子性
- 要么成功要么失败，没有中间状态 -> 用日志实现
- 其他CRUD在它之前或者之后 -> 可线性化

### 日志

Undo日志

- 记录修改之前的值
- 用`start`和`commit`包裹起来
- **日志需要在数据之前到达磁盘（WAL）**
- 操作结束之前，所有的数据和日志必须到达磁盘（保证持久性），`commit`表示成功将所有数据写到磁盘
  - **操作结束之前必须将数据刷盘**
- 恢复时从新往后进行恢复数据

Redo日志

- 记录修改之后的值
- **操作结束之后，才能将数据写到磁盘**，也就是说，在操作结束之前，只有日志写入了磁盘（顺序写效率高）
  - 操作结束之前不能将数据刷盘(END之前)
- **数据到达磁盘后，需在日志中记录END**

Undo/Redo日志

- 克服了两者的缺点，**数据可以在任何时间到达磁盘**（操作结束前或之后）
- 但日志依然必须在数据之前到达磁盘
  - 操作结束之前日志必须到达磁盘

日志恢复

- 若有`commit`，则表示日志已经全部写完，用Redo（可以更新这次的值了）
- 若没有`commit`，则表示日志都还没写完，用Undo（必须将数据还原为未更新的值）

### 可线性化

如何实现并发控制？通过加锁实现

## 可用性

如何实现

- 冗余节点(Redundancy) + 数据复制(Replication) 
- 节点发生故障时，切换到(Failover)冗余节点 

单机热备：无法做到，若两个节点之间的通讯断了，都认为自己是主节点，数据无法同步

### Raft

1. 状态：节点有Candidate/Follower/Leader三种状态（最初都是follower）
2. Timeout
   1. election timeout：是follower等待转化为candidate的时间（150ms - 300ms）
   2. heartbeat timeout：每次leader发送AppendEntries的时间
3. 任期（term）
   - 每次都有一个任期（term）每个任期最多只有一个`leader`，有些任期可能因为选票瓜分导致没有`leader`。每一个服务器维护自己当前的任期，并在RPC时进行交换，若别人的任期大于自己的，则转为follower。
4. 选主（Election）
   - 集群通过心跳保持联系，若在随机等待的timeout中没有收到`Leader`的心跳，则自己变为`candidate`，向其他节点进行投票（不完整的log的follower一定不会变成leader）
   - 若收到大部分的选票，则投票成功，则自己变为Leader，这时候将自己的log复制到从节点中
   - 为了保证总是有`leader`被选出来，使用随机timeout的方式，若这个时间段中没有被选出来，则失败
5. Log replication 
   - `client`发送命令给`leader`，`leader`加入自己的log中，并将`AppendEntries`发送给`followers`
   - 之后返回给`leader`且进行commit，并让`follower`进行commit
   - 根据设置的参数，`WriteConcern`若为`majority`，则需要等到`followers`都commit之后才能返回给`client`
   - 若有log不相同，则找到最近相同的log，从这里用`AppendEntries`将其覆盖




# 数据库设计

对应用的理解，对数据的需求

1. 需求分析
2. 概念模型设计
   - ER图
3. 逻辑模型设计
   - 范式，函数依赖，多值依赖（见书）
4. 物理模型设计
   - Denoemalization
   - 什么时候用索引
   - 物化视图

## 关系模型

- 关系是笛卡尔积的子集，为二维表，每行对应一个元祖，每列对应一个域**（不能重复）**
- 候选码
  - 某一属性值能唯一的标识一个元祖，任意两个候选码不能相同
- 全码
  - 所有属性组是这个关系模型的候选码
- 主码
  - 在候选码中选定一个
- 主属性
  - 候选码的属性，其余为非主属性
- 外码
  - 参照对象不一定是不同的关系

连接：

- 等值连接
  - 自然连接：特殊的等值连接，会将公共属性只保留一个
  - 外连接：保留悬浮元祖
- 一般连接

除法：需要右边的公共属性都在左边某些元祖中都存在的那些列

关系代数（并/差/交/投影/连接）-> 关系演算 -> SQL

## 范式化

### 第一范式

- 只要满足关系的定义（笛卡尔积的子集），则满足第一范式

#### 函数依赖

若X的取值相等，则Y一定相等。记作 X->Y。

强函数依赖

- X->Y是一个完全函数依赖，如果Y中的属性在实际中从来不变或者很少变化，则为强函数依赖
- **Y的冗余并不会带来太多的更新异常**

弱函数依赖

- 若将关系R中的少数几个元祖去除之后，X->Y为一个函数依赖，则X->Y为一个弱函数依赖
- **若函数依赖涉及到的少数几个元祖可以单独作为一个关系**

#### 完全函数依赖

对于X，必须由其全部属性才能确定Y，则称为Y完全依赖于X。

#### 键

- 设R(U) 为属性集 U上的关系模式。 X是U的子集。  如果 X->U，并且不存在  X的子集 X’ 使得 X’ ->U， 那么 X为R(U) 的键/码（key）。
- 在R(U)中可能存在多个键，我们人为指定其中的一个键为主键（primary key）
- 包含在任意一个键中的属性，称为主属性（primary attribute）。

**代理键：**主码越小，则BTree的n越大（同样空间中能存储的值越多），则查询效率越高，因此使用代理键id作为主属性

### 第二范式

- 关系模式R满足第二范式，当且仅当，R满足第一范式，并且，R的每一个非主属性都完全依赖于R的每一个键。
- 若不是键，则不能决定其他属性。

### 第三范式

- 关系模式R满足第三范式，当且仅当，R中不存在这样的键X，属性组Y和非主属性Z，使得X->Y，Y->Z成立，且Y->X不成立。
- 即：**若X是键，Y不是键，但Y能决定Z，则不满足第三范式。**
- 满足第三范式的模式必满足第二范式。

### BC范式

- 关系模式 R满足 BC 范式，当且仅对任意一个属性集 A，如果存在不属于A一个属性 X， 使得 X函数依赖于A，那么所有R的属性都函数依赖于A。
- 任何满足BC范式的关系模式都满足第三范式。
- BCNF与[第三范式](https://zh.wikipedia.org/wiki/%E7%AC%AC%E4%B8%89%E8%8C%83%E5%BC%8F)的不同之处在于：
  - **第三范式中不允许[非主属性](https://zh.wikipedia.org/w/index.php?title=%E9%9D%9E%E4%B8%BB%E5%B1%9E%E6%80%A7&action=edit&redlink=1)被另一个非主属性决定，但第三范式允许主属性被非主属性决定；**
  - 而在BCNF中，任何属性（包括非主属性和主属性）都不能被非主属性所决定。

# 关系数据库内部实现

- 存储系统组织形式
  - 页
- 选择-索引 
  - btree
- 投影- 去重过程，
  - 外部排序（扫描3遍）
- 连接
  - sortmerge，hash，index
- 复杂度用io来衡量
- 查询计划通过流水线的方式执行

## 查询执行过程

**关系代数的表达式 -> 查询计划（解析）**

**查询计划 -> 最优查询计划（优化）**

**最优查询计划 -> 结果（查询执行）**

- 有多个查询计划都可以完成同一个操作
- 不同的数据会导致查询计划的效率不同，不能说哪一个查询计划一定最好

若要实现优化，首先需要考虑影响数据访问的性能差异

- 至少需要考虑内存到磁盘的差异
- 这里复杂度主要用I/O来衡量

### 映射去重

在SELECT DISTINCT操作中经常出现

1. 排序
   - 由于数据无法完全放入内存，因此考虑MergeSort
   - 边合并边去重
   - **至少需要将数据完整扫描三遍**
     - 首先在创建排序子表时需要读入每个块进入内存排序
     - 将各排序子表写入磁盘
     - 从子表中读入每个块进行归并排序（直接输出）
2. 散列
   - 使用M个缓冲区将其划分为大致相等的M-1个桶（最后一个用于装入数据库的buffer）
   - 将每一个桶和缓冲区联系起来，每次来一个块，将元组进行散列到不同的缓冲区中（缓冲区保存目前为止见过的每个元祖的一个副本）
   - 将每个桶都写入磁盘
   - 再读入每个桶的数据，用另外的哈希算法写入内存，保存每个元祖的副本，即可去重
   - 至少需要将数据完整扫描三遍I/O

### 连接

1. 嵌套循环（Nested loop）
   - 基本思想：拿一个元祖（S），扫描另一个集合（R）
   - 可以优化成拿尽可能多的元祖（外层循环S）一起扫描
   - $B(S) + \frac{B(S)B(R)}{M-1} = \frac{B(S)}{M-1}(M-1 + B(R))$
     - 外层循环S需要全部放到内存中一次($B(S)$)，加上有多少次内层循环
     - 内层循环需要$M-1+B(R)$，一共需要$\frac{B(S)}{M-1}$
     - 当两个数组都很大时，大致相当于$\frac{B(R)B(S)}{M}$
2. 合并排序
   - 对两个集合的元组分别使用**两阶段多路归并排序**
   - 再归并排好序的两个集合
   - 一般情况下磁盘I/O为：$3\times (B(R) + B(S))$
3. 散列连接
   - 用连接属性作为散列关键字，再对对应桶进行连接
   - $3\times (B(R) + B(S))$，需要每一对桶中至少有一个能全部放入缓冲区中
4. 利用索引
   - 当R表很小时，索引很好
   - 当S表很小时，则与嵌套循环类似（外层循环都可以直接放入内存）

## 查询执行

- 批处理
  - 依次执行每个操作，将一个操作在所有数据上执行完之后再执行下一个操作
  - **以表为传递**
- 流水线
  - 逐个数据上执行所有操作
  - **筛选出的元组一个个被传递（并行）**
- 火山模型
  - 每个数据库操作都使用共同的接口（`open(),next(),close()`）

### 查询优化

- 基于规则的优化
  - 聚簇索引/物化视图/hash join
- 基于代价的优化
  - 估算每个查询的I/O代价
  - 选择/join
- 查询改写
  - 删除多余的DISTINCT
  - 子查询效率较低，尽量使用join
  - 尽量避免使用存放中间结果的表（中间表没有办法优化）
  - 相关子查询改写

## 索引

B-Tree：多值查询可以加速（若为多值索引），可以进行单值的范围查询加速

Hash Table：多值查询可以加速，不能进行范围查询

聚簇索引：数据就存在索引的叶子结点，数据存储方式按照索引排列（会溢出，随着数据增加会变差）

# 事务处理

## 什么是事务

事务就是一段线性的程序

- acid性质
  - 原子性（Atomicity）
    - 一个事务要么没有开始，要么全部完成，使用日志实现
  - 一致性（Consistency）
    - 注意，**这里的数据正确指符合约束**（非零/主码）
  - 隔离性（Isolation）
    - 多个事务不会互相破坏，**指的是并发控制**
  - 持久性（Durability）
    - 事务一旦提交成功，对数据的修改不会丢失，使用日志实现

## 事务的实现

- 如何保证原子性/持久性（AD）：日志（undo，redo，undo+redo）
- 如何保证隔离性（I）：并发控制（加锁，时间戳，有效性验证）

### 可串行化调度

并发事务的正确性原则:

- 调度产生的结果与一次执行一个事务所产生的结果相同。
- 并不保证先后顺序

#### 冲突可串行化

-  **一个比可串行化更严格的条件** 
-  商用系统中的调度器采用 
-  可以通过移动非冲突的动作变成线性化调度

冲突 

- 对于调度中一对连续的动作，如果它们的顺序交换， 结果将改变。 

冲突的条件:

- 涉及同一个数据库元素
- 并且至少有一个是写操作的动作

冲突等价的调度: 

- 如果S1能通过一系列的非冲突交换变成S2（线性化调度），则S1 、S2 是冲突等价的调度。 

 冲突可串行化: 

- 一个调度是冲突可串行化的，如果它和某些串行调度是冲突等价的。
- 若一个调度是冲突可串行化，则一定是可串行化的调度
- 例子参考PPT

#### 如何实现冲突的可串行化？

1. 先全部加锁，再依次释放
2. **两阶段锁**
   - 在每个事务中，所有加锁请求先于解锁请求
   - 访问数据前申请锁
   - 事务结束后将锁一齐释放

#### 理解

- 冲突可串行化调度是可串行化调度的**充分条件**，不是必要条件。还有不满足冲突可串行化条件的可串行化调度。
- 两阶段锁协议是冲突可串行化的**充分条件，**不是必要条件。
- ![schedule](/images/schedule.png)
  - 在这里，Y的写顺序没有改变，而X的写顺序发生了改变，没有办法转换为串行调度，但不影响最终结果。
  - 例如，若两个事务对索引进行更新，属于冲突，但可以交换顺序，不满足冲突可串行化，但可以串行化。

## 线性化vs串行化

- 可串行化与时间无关
- 可线性化在可串行化的基础上加入时间限制，要求更高
- 可线性化优势
  - 能保证逻辑的正确性（先操作的一定先发生）
- 数据库很难保证可串行化（事务），操作能保证可线性化

## 事务的使用

begin comit 中间不能与用户交互

如何减小锁对性能的影响

- 尽量少加锁
- 多使用共享锁（读锁），少使用排他锁（写锁）
  - 共享锁：多个事务可对同一数据重复申请加读锁
  - 排他锁：一旦事务T对数据A加上了X锁，只允许T对其进行读写
- 使用更细粒度的锁
  - 表级锁 vs 行级锁
- 事务做到尽可能的简短，减少阻塞

# 调优及备份

## 数据库调优

触发点：数据库性能⽆法满足应用负载

- 响应时间明显延⻓，甚⾄不响应
- 事务回滚增加，导致⼤量任务失败
- 确认数据库为性能瓶颈

诊断点

- 关键查询是否被⾼高效执⾏?
  - 找到关键查询
    - 频繁被用户使用/执行过程消耗较多资源
  - 了解查询的执⾏⽅式
    - 查询计划
  - 排查询执⾏过程中的性能问题
- 系统组件是否有效地利用了硬件资源?
  - 缓冲区管理器/磁盘碎片/锁管理器/日志管理器
- 硬件资源是否足够？
  - 物理设计调整/查询改写/系统调参

## 备份

数据备份（backup）

- 预防错误或灾难
- 可以处理更多错误，例如误操作（已提交的事务需要撤回）
- 利用Replication机制可以实现更精确的恢复

Replication

- 实现高可用
- 在一定程度上也可以起到备份的作用
- 无法撤销已提交的事务（检查点之前到日志可能被删除）

备份时将全量数据进行复制，后面可以进行恢复。

**存档（Archiving）是将旧的数据移除数据库系统，转移到线下设备。**

# OLAP

- OLTP
  - 查询和更新，对现实世界状态数据的存储
  - 如状态信息/查询余额
- OLAP
  - 主要是查询，分析和发现知识
  - 历史流水

## 数据模型

Data Cube

- dimensions/facts

主要操作

- slicing/dicing
- Roll Up/Drill Down

## 数据仓库系统

提供data cube的各种操作

MOLAP

- Multidimensional OLAP
- 直接存一个cube
  - 一般会提前计算各种数据指标
  - 查询非常快速
- 空间浪费严重（稀疏性），没办法扩展到高维

ROLAP

- Relational OLAP
- 可以扩展到高维，空间存放合理，避免稀疏性问题
- 但性能不佳，需要格外指定索引
- SQL语句没办产生`subtotoal`
  - 提供`CUBE`操作

### 数据模式

由于很多数据有多个维度和属性，不能直接全部存在一个表中（冗余/宽表）

使用范式化技术，将表分为`dimension table` 和`fact table`

- dimension table
  - 一般不会有太多行，但会有很多列
  - 一般只会查找，不会修改
  - 存储现实世界的某些物体
- fact table
  - 一般非常多的行，列不会很多
  - 不断增加，但一般不会修改
  - 存储衡量某个事件的过程