# MySQL

# MySQL 中事务的实现

在关系型数据库中，事务的重要性不言而喻，只要对数据库稍有了解的人都知道事务具有 ACID 四个基本属性，而我们不知道的可能就是数据库是如何实现这四个属性的；在这篇文章中，我们将对事务的实现进行分析，尝试理解数据库是如何实现事务的，当然我们也会在文章中简单对 MySQL 中对 ACID 的实现进行简单的介绍。

![Transaction-Basics](https://img.draveness.me/2017-08-20-Transaction-Basics.jpg-1000width)

事务其实就是**并发控制的基本单位**；相信我们都知道，事务是一个序列操作，其中的操作要么都执行，要么都不执行，它是一个不可分割的工作单位；数据库事务的 ACID 四大特性是事务的基础，了解了 ACID 是如何实现的，我们也就清楚了事务的实现，接下来我们将依次介绍数据库是如何实现这四个特性的。

## 原子性

在学习事务时，经常有人会告诉你，事务就是一系列的操作，要么全部都执行，要都不执行，这其实就是对事务原子性的刻画；虽然事务具有原子性，但是原子性并不是只与事务有关系，它的身影在很多地方都会出现。

![Atomic-Operation](https://img.draveness.me/2017-08-20-Atomic-Operation.jpg-1000width)

由于操作并不具有原子性，并且可以再分为多个操作，当这些操作出现错误或抛出异常时，整个操作就可能不会继续执行下去，而已经进行的操作造成的副作用就可能造成数据更新的丢失或者错误。

事务其实和一个操作没有什么太大的区别，它是一系列的数据库操作（可以理解为 SQL）的集合，如果事务不具备原子性，那么就没办法保证同一个事务中的所有操作都被执行或者未被执行了，整个数据库系统就既不可用也不可信。

### 回滚日志

想要保证事务的原子性，就需要在异常发生时，对已经执行的操作进行**回滚**，而在 MySQL 中，恢复机制是通过*回滚日志*（undo log）实现的，所有事务进行的修改都会先记录到这个回滚日志中，然后在对数据库中的对应行进行写入。

![Transaction-Undo-Log](https://img.draveness.me/2017-08-20-Transaction-Undo-Log.jpg-1000width)

这个过程其实非常好理解，为了能够在发生错误时撤销之前的全部操作，肯定是需要将之前的操作都记录下来的，这样在发生错误时才可以回滚。

回滚日志除了能够在发生错误或者用户执行 `ROLLBACK` 时提供回滚相关的信息，它还能够在整个系统发生崩溃、数据库进程直接被杀死后，当用户再次启动数据库进程时，还能够立刻通过查询回滚日志将之前未完成的事务进行回滚，这也就需要回滚日志必须先于数据持久化到磁盘上，是我们需要先写日志后写数据库的主要原因。

回滚日志并不能将数据库物理地恢复到执行语句或者事务之前的样子；它是逻辑日志，当回滚日志被使用时，它只会按照日志**逻辑地**将数据库中的修改撤销掉看，可以**理解**为，我们在事务中使用的每一条 `INSERT` 都对应了一条 `DELETE`，每一条 `UPDATE` 也都对应一条相反的 `UPDATE` 语句。

![Logical-Undo-Log](https://img.draveness.me/2017-08-20-Logical-Undo-Log.jpg-1000width)

在这里，我们并不会介绍回滚日志的格式以及它是如何被管理的，本文重点关注在它到底是一个什么样的东西，究竟解决了、如何解决了什么样的问题，如果想要了解具体实现细节的读者，相信网络上关于回滚日志的文章一定不少。

### 事务的状态

因为事务具有原子性，所以从远处看的话，事务就是密不可分的一个整体，事务的状态也只有三种：Active、Commited 和 Failed，事务要不就在执行中，要不然就是成功或者失败的状态：

![Atomitc-Transaction-State](https://img.draveness.me/2017-08-20-Atomitc-Transaction-State.jpg-1000width)

但是如果放大来看，我们会发现事务不再是原子的，其中包括了很多中间状态，比如部分提交，事务的状态图也变得越来越复杂。

![Nonatomitc-Transaction-State](https://img.draveness.me/2017-08-20-Nonatomitc-Transaction-State.jpg-1000width)

> 事务的状态图以及状态的描述取自 [Database System Concepts](https://www.amazon.com/Database-System-Concepts-Computer-Science/dp/0073523321) 一书中第 14 章的内容。

- Active：事务的初始状态，表示事务正在执行；
- Partially Commited：在最后一条语句执行之后；
- Failed：发现事务无法正常执行之后；
- Aborted：事务被回滚并且数据库恢复到了事务进行之前的状态之后；
- Commited：成功执行整个事务；

虽然在发生错误时，整个数据库的状态可以恢复，但是如果我们在事务中执行了诸如：向标准输出打印日志、向外界发出邮件、没有通过数据库修改了磁盘上的内容甚至在事务执行期间发生了转账汇款，那么这些操作作为可见的外部输出都是没有办法回滚的；这些问题都是由应用开发者解决和负责的，在绝大多数情况下，我们都需要在整个事务提交后，再触发类似的无法回滚的操作。

![Shutdown-After-Commited](https://img.draveness.me/2017-08-20-Shutdown-After-Commited.jpg-1000width)

以订票为例，哪怕我们在整个事务结束之后，才向第三方发起请求，由于向第三方请求并获取结果是一个需要较长时间的操作，如果在事务刚刚提交时，数据库或者服务器发生了崩溃，那么我们就非常有可能丢失发起请求这一过程，这就造成了非常严重的问题；而这一点就不是数据库所能保证的，开发者需要在适当的时候查看请求是否被发起、结果是成功还是失败。

### 并行事务的原子性

到目前为止，所有的事务都只是串行执行的，一直都没有考虑过并行执行的问题；然而在实际工作中，并行执行的事务才是常态，然而并行任务下，却可能出现非常复杂的问题：

![Nonrecoverable-Schedule](https://img.draveness.me/2017-08-20-Nonrecoverable-Schedule.jpg-1000width)

当 Transaction1 在执行的过程中对 `id = 1` 的用户进行了读写，但是没有将修改的内容进行提交或者回滚，在这时 Transaction2 对同样的数据进行了读操作并提交了事务；也就是说 Transaction2 是依赖于 Transaction1 的，当 Transaction1 由于一些错误需要回滚时，因为要保证事务的原子性，需要对 Transaction2 进行回滚，但是由于我们已经提交了 Transaction2，所以我们已经没有办法进行回滚操作，在这种问题下我们就发生了问题，[Database System Concepts](https://www.amazon.com/Database-System-Concepts-Computer-Science/dp/0073523321) 一书中将这种现象称为*不可恢复安排*（Nonrecoverable Schedule），那什么情况下是可以恢复的呢？

> A recoverable schedule is one where, for each pair of transactions Ti and Tj such that Tj reads a data item previously written by Ti , the commit operation of Ti appears before the commit operation of Tj .

简单理解一下，如果 Transaction2 依赖于事务 Transaction1，那么事务 Transaction1 必须在 Transaction2 提交之前完成提交的操作：

![Recoverable-Schedule](https://img.draveness.me/2017-08-20-Recoverable-Schedule.jpg-1000width)

然而这样还不算完，当事务的数量逐渐增多时，整个恢复流程也会变得越来越复杂，如果我们想要从事务发生的错误中恢复，也不是一件那么容易的事情。

![Cascading-Rollback](https://img.draveness.me/2017-08-20-Cascading-Rollback.jpg-1000width)

在上图所示的一次事件中，Transaction2 依赖于 Transaction1，而 Transaction3 又依赖于 Transaction1，当 Transaction1 由于执行出现问题发生回滚时，为了保证事务的原子性，就会将 Transaction2 和 Transaction3 中的工作全部回滚，这种情况也叫做*级联回滚*（Cascading Rollback），级联回滚的发生会导致大量的工作需要撤回，是我们难以接受的，不过如果想要达到**绝对的**原子性，这件事情又是不得不去处理的，我们会在文章的后面具体介绍如何处理并行事务的原子性。

## 持久性

既然是数据库，那么一定对数据的持久存储有着非常强烈的需求，如果数据被写入到数据库中，那么数据一定能够被安全存储在磁盘上；而事务的持久性就体现在，一旦事务被提交，那么数据一定会被写入到数据库中并持久存储起来。

![Compensating-Transaction](https://img.draveness.me/2017-08-20-Compensating-Transaction.jpg-1000width)

当事务已经被提交之后，就无法再次回滚了，唯一能够撤回已经提交的事务的方式就是创建一个相反的事务对原操作进行『补偿』，这也是事务持久性的体现之一。

### 重做日志

与原子性一样，事务的持久性也是通过日志来实现的，MySQL 使用重做日志（redo log）实现事务的持久性，重做日志由两部分组成，一是内存中的重做日志缓冲区，因为重做日志缓冲区在内存中，所以它是易失的，另一个就是在磁盘上的重做日志文件，它是持久的。

![Redo-Logging](https://img.draveness.me/2019-02-21-Redo-Logging.jpg)

当我们在一个事务中尝试对数据进行修改时，它会先将数据从磁盘读入内存，并更新内存中缓存的数据，然后生成一条重做日志并写入重做日志缓存，当事务真正提交时，MySQL 会将重做日志缓存中的内容刷新到重做日志文件，再将内存中的数据更新到磁盘上，图中的第 4、5 步就是在事务提交时执行的。

在 InnoDB 中，重做日志都是以 512 字节的块的形式进行存储的，同时因为块的大小与磁盘扇区大小相同，所以重做日志的写入可以保证原子性，不会由于机器断电导致重做日志仅写入一半并留下脏数据。

除了所有对数据库的修改会产生重做日志，因为回滚日志也是需要持久存储的，它们也会创建对应的重做日志，在发生错误后，数据库重启时会从重做日志中找出未被更新到数据库磁盘中的日志重新执行以满足事务的持久性。

### 回滚日志和重做日志

到现在为止我们了解了 MySQL 中的两种日志，回滚日志（undo log）和重做日志（redo log）；在数据库系统中，事务的原子性和持久性是由事务日志（transaction log）保证的，在实现时也就是上面提到的两种日志，前者用于对事务的影响进行撤销，后者在错误处理时对已经提交的事务进行重做，它们能保证两点：

1. 发生错误或者需要回滚的事务能够成功回滚（原子性）；
2. 在事务提交后，数据没来得及写会磁盘就宕机时，在下次重新启动后能够成功恢复数据（持久性）；

在数据库中，这两种日志经常都是一起工作的，我们**可以**将它们整体看做一条事务日志，其中包含了事务的 ID、修改的行元素以及修改前后的值。

![Transaction-Log](https://img.draveness.me/2017-08-20-Transaction-Log.jpg-1000width)

一条事务日志同时包含了修改前后的值，能够非常简单的进行回滚和重做两种操作，在这里我们也不会对重做和回滚日志展开进行介绍，可能会在之后的文章谈一谈数据库系统的恢复机制时提到两种日志的使用。

## 隔离性

其实作者在之前的文章 [『浅入浅出』MySQL 和 InnoDB](https://draveness.me/mysql-innodb) 就已经介绍过数据库事务的隔离性，不过为了保证文章的独立性和完整性，我们还会对事务的隔离性进行介绍，介绍的内容可能稍微有所不同。

事务的隔离性是数据库处理数据的几大基础之一，如果没有数据库的事务之间没有隔离性，就会发生在 [并行事务的原子性](https://draveness.me/mysql-transaction#%E5%B9%B6%E8%A1%8C%E4%BA%8B%E5%8A%A1%E7%9A%84%E5%8E%9F%E5%AD%90%E6%80%A7) 一节中提到的级联回滚等问题，造成性能上的巨大损失。如果所有的事务的执行顺序都是线性的，那么对于事务的管理容易得多，但是允许事务的并行执行却能能够提升吞吐量和资源利用率，并且可以减少每个事务的等待时间。

![Reasons-for-Allowing-Concurrency](https://img.draveness.me/2017-08-20-Reasons-for-Allowing-Concurrency.jpg-1000width)

当多个事务同时并发执行时，事务的隔离性可能就会被违反，虽然单个事务的执行可能没有任何错误，但是从总体来看就会造成数据库的一致性出现问题，而串行虽然能够允许开发者忽略并行造成的影响，能够很好地维护数据库的一致性，但是却会影响事务执行的性能。

### 事务的隔离级别

所以说数据库的隔离性和一致性其实是一个需要开发者去权衡的问题，为数据库提供什么样的隔离性层级也就决定了数据库的性能以及可以达到什么样的一致性；在 SQL 标准中定义了四种数据库的事务的隔离级别：`READ UNCOMMITED`、`READ COMMITED`、`REPEATABLE READ` 和 `SERIALIZABLE`；每个事务的隔离级别其实都比上一级多解决了一个问题：

- `RAED UNCOMMITED`：使用查询语句不会加锁，可能会读到未提交的行（Dirty Read）；
- `READ COMMITED`：只对记录加记录锁，而不会在记录之间加间隙锁，所以允许新的记录插入到被锁定记录的附近，所以再多次使用查询语句时，可能得到不同的结果（Non-Repeatable Read）；
- `REPEATABLE READ`：多次读取同一范围的数据会返回第一次查询的快照，不会返回不同的数据行，但是可能发生幻读（Phantom Read）；
- `SERIALIZABLE`：InnoDB 隐式地将全部的查询语句加上共享锁，解决了幻读的问题；

以上的所有的事务隔离级别都不允许脏写入（Dirty Write），也就是当前事务更新了另一个事务已经更新但是还未提交的数据，大部分的数据库中都使用了 READ COMMITED 作为默认的事务隔离级别，但是 MySQL 使用了 REPEATABLE READ 作为默认配置；从 RAED UNCOMMITED 到 SERIALIZABLE，随着事务隔离级别变得越来越严格，数据库对于并发执行事务的性能也逐渐下降。

![Isolation-Performance](https://img.draveness.me/2017-08-20-Isolation-Performance.jpg-1000width)

对于数据库的使用者，从理论上说，并不需要知道事务的隔离级别是如何实现的，我们只需要知道这个隔离级别解决了什么样的问题，但是不同数据库对于不同隔离级别的是实现细节在很多时候都会让我们遇到意料之外的坑。

如果读者不了解脏读、不可重复读和幻读究竟是什么，可以阅读之前的文章 [『浅入浅出』MySQL 和 InnoDB](https://draveness.me/mysql-innodb)，在这里我们仅放一张图来展示各个隔离层级对这几个问题的解决情况。

![Transaction-Isolation-Matrix](https://img.draveness.me/2017-08-20-Transaction-Isolation-Matrix.jpg-1000width)

### 隔离级别的实现

数据库对于隔离级别的实现就是使用**并发控制机制**对在同一时间执行的事务进行控制，限制不同的事务对于同一资源的访问和更新，而最重要也最常见的并发控制机制，在这里我们将简单介绍三种最重要的并发控制器机制的工作原理。

#### 锁

锁是一种最为常见的并发控制机制，在一个事务中，我们并不会将整个数据库都加锁，而是只会锁住那些需要访问的数据项， MySQL 和常见数据库中的锁都分为两种，共享锁（Shared）和互斥锁（Exclusive），前者也叫读锁，后者叫写锁。

![Shared-Exclusive-Lock](https://img.draveness.me/2017-08-20-Shared-Exclusive-Lock.jpg-1000width)

读锁保证了读操作可以并发执行，相互不会影响，而写锁保证了在更新数据库数据时不会有其他的事务访问或者更改同一条记录造成不可预知的问题。

#### 时间戳

除了锁，另一种实现事务的隔离性的方式就是通过时间戳，使用这种方式实现事务的数据库，例如 PostgreSQL 会为每一条记录保留两个字段；*读时间戳*中包括了所有访问该记录的事务中的最大时间戳，而记录行的*写时间戳*中保存了将记录改到当前值的事务的时间戳。

![Timestamps-Record](https://img.draveness.me/2017-08-20-Timestamps-Record.jpg-1000width)

使用时间戳实现事务的隔离性时，往往都会使用乐观锁，先对数据进行修改，在写回时再去判断当前值，也就是时间戳是否改变过，如果没有改变过，就写入，否则，生成一个新的时间戳并再次更新数据，乐观锁其实并不是真正的锁机制，它只是一种思想，在这里并不会对它进行展开介绍。

#### 多版本和快照隔离

通过维护多个版本的数据，数据库可以允许事务在数据被其他事务更新时对旧版本的数据进行读取，很多数据库都对这一机制进行了实现；因为所有的读操作不再需要等待写锁的释放，所以能够显著地提升读的性能，MySQL 和 PostgreSQL 都对这一机制进行自己的实现，也就是 MVCC，虽然各自实现的方式有所不同，MySQL 就通过文章中提到的回滚日志实现了 MVCC，保证事务并行执行时能够不等待互斥锁的释放直接获取数据。

### 隔离性与原子性

在这里就需要简单提一下在在原子性一节中遇到的级联回滚等问题了，如果一个事务对数据进行了写入，这时就会获取一个互斥锁，其他的事务就想要获得改行数据的读锁就必须等待写锁的释放，自然就不会发生级联回滚等问题了。

![Shared-Lock-and-Atomicity](https://img.draveness.me/2017-08-20-Shared-Lock-and-Atomicity.jpg-1000width)

不过在大多数的数据库，比如 MySQL 中都使用了 MVCC 等特性，也就是正常的读方法是不需要获取锁的，在想要对读取的数据进行更新时需要使用 `SELECT ... FOR UPDATE` 尝试获取对应行的互斥锁，以保证不同事务可以正常工作。

## 一致性

作者认为数据库的一致性是一个非常让人迷惑的概念，原因是数据库领域其实包含两个一致性，一个是 ACID 中的一致性、另一个是 CAP 定义中的一致性。

![ACID-And-CAP](https://img.draveness.me/2017-08-20-ACID-And-CAP.jpg-1000width)

这两个数据库的一致性说的**完全不是**一个事情，很多很多人都对这两者的概念有非常深的误解，当我们在讨论数据库的一致性时，一定要清楚上下文的语义是什么，尽量明确的问出我们要讨论的到底是 ACID 中的一致性还是 CAP 中的一致性。

### ACID

数据库对于 ACID 中的一致性的定义是这样的：如果一个事务原子地在一个一致地数据库中独立运行，那么在它执行之后，数据库的状态一定是一致的。对于这个概念，它的第一层意思就是对于数据完整性的约束，包括主键约束、引用约束以及一些约束检查等等，在事务的执行的前后以及过程中不会违背对数据完整性的约束，所有对数据库写入的操作都应该是合法的，并不能产生不合法的数据状态。

> A transaction must preserve database consistency - if a transaction is run atomically in isolation starting from a consistent database, the database must again be consistent at the end of the transaction.

我们可以将事务理解成一个函数，它接受一个外界的 SQL 输入和一个一致的数据库，它一定会返回一个一致的数据库。

![Transaction-Consistency](https://img.draveness.me/2017-08-20-Transaction-Consistency.jpg-1000width)

而第二层意思其实是指逻辑上的对于开发者的要求，我们要在代码中写出正确的事务逻辑，比如银行转账，事务中的逻辑不可能只扣钱或者只加钱，这是应用层面上对于数据库一致性的要求。

> Ensuring consistency for an individual transaction is the responsibility of the application programmer who codes the transaction. - [Database System Concepts](https://www.amazon.com/Database-System-Concepts-Computer-Science/dp/0073523321)

数据库 ACID 中的一致性对事务的要求不止包含对数据完整性以及合法性的检查，还包含应用层面逻辑的正确。

CAP 定理中的数据一致性，其实是说分布式系统中的各个节点中对于同一数据的拷贝有着相同的值；而 ACID 中的一致性是指数据库的规则，如果 schema 中规定了一个值必须是唯一的，那么一致的系统必须确保在所有的操作中，该值都是唯一的，由此来看 CAP 和 ACID 对于一致性的定义有着根本性的区别。

## 总结

事务的 ACID 四大基本特性是保证数据库能够运行的基石，但是完全保证数据库的 ACID，尤其是隔离性会对性能有比较大影响，在实际的使用中我们也会根据业务的需求对隔离性进行调整，除了隔离性，数据库的原子性和持久性相信都是比较好理解的特性，前者保证数据库的事务要么全部执行、要么全部不执行，后者保证了对数据库的写入都是持久存储的、非易失的，而一致性不仅是数据库对本身数据的完整性的要求，同时也对开发者提出了要求 - 写出逻辑正确并且合理的事务。

# MongoDB 和 WiredTiger

MongoDB 是目前主流的 NoSQL 数据库之一，与关系型数据库和其它的 NoSQL 不同，MongoDB 使用了面向文档的数据存储方式，将数据以类似 JSON 的方式存储在磁盘上，因为项目上的一些历史遗留问题，作者在最近的工作中也不得不经常与 MongoDB 打交道，这也是这篇文章出现的原因。

![logo](https://img.draveness.me/2017-09-06-logo.png-1000width)

虽然在之前也对 MongoDB 有所了解，但是真正在项目中大规模使用还是第一次，使用过程中也暴露了大量的问题，不过在这里，我们主要对 MongoDB 中的一些重要概念的原理进行介绍，也会与 MySQL 这种传统的关系型数据库做一个对比，让读者自行判断它们之间的优势和劣势。

## 概述

MongoDB 虽然也是数据库，但是它与传统的 RDBMS 相比有着巨大的不同，很多开发者都认为或者被灌输了一种思想，MongoDB 这种无 Scheme 的数据库相比 RDBMS 有着巨大的性能提升，这个判断其实是一种误解；因为数据库的性能不止与数据库本身的设计有关系，还与开发者对表结构和索引的设计、存储引擎的选择和业务有着巨大的关系，如果认为**仅进行了数据库的替换就能得到数量级的性能提升**，那还是太年轻了。

![its-not-always-simple-banner](https://img.draveness.me/2017-09-06-its-not-always-simple-banner.jpg-1000width)

### 架构

现有流行的数据库其实都有着非常相似的架构，MongoDB 其实就与 MySQL 中的架构相差不多，底层都使用了『可插拔』的存储引擎以满足用户的不同需要。

![MongoDB-Architecture](https://img.draveness.me/2017-09-06-MongoDB-Architecture.jpg-1000width)

用户可以根据表中的数据特征选择不同的存储引擎，它们可以在同一个 MongoDB 的实例中使用；在最新版本的 MongoDB 中使用了 WiredTiger 作为默认的存储引擎，WiredTiger 提供了不同粒度的并发控制和压缩机制，能够为不同种类的应用提供了最好的性能和存储效率。

在不同的存储引擎上层的就是 MongoDB 的数据模型和查询语言了，与关系型数据库不同，由于 MongoDB 对数据的存储与 RDBMS 有较大的差异，所以它创建了一套不同的查询语言；虽然 MongoDB 查询语言非常强大，支持的功能也很多，同时也是可编程的，不过其中包含的内容非常繁杂、API 设计也不是非常优雅，所以还是需要一些学习成本的，对于长时间使用 MySQL 的开发者肯定会有些不习惯。

```
db.collection.updateMany(
   <filter>,
   <update>,
   {
     upsert: <boolean>,
     writeConcern: <document>,
     collation: <document>
   }
)

```

查询语言的复杂是因为 MongoDB 支持了很多的数据类型，同时每一条数据记录也就是文档有着非常复杂的结构，这点是从设计上就没有办法避免的，所以还需要使用 MongoDB 的开发者花一些时间去学习各种各样的 API。

### RDBMS 与 MongoDB

MongoDB 使用面向文档的的数据模型，导致很多概念都与 RDBMS 有一些差别，虽然从总体上来看两者都有相对应的概念，不过概念之间细微的差别其实也会影响我们对 MongoDB 的理解：

![Translating-Between-RDBMS-and-MongoDB](https://img.draveness.me/2017-09-06-Translating-Between-RDBMS-and-MongoDB.jpg-1000width)

传统的 RDBMS 其实使用 `Table` 的格式将数据逻辑地存储在一张二维的表中，其中不包括任何复杂的数据结构，但是由于 MongoDB 支持嵌入文档、数组和哈希等多种复杂数据结构的使用，所以它最终将所有的数据以 [BSON](http://bsonspec.org/) 的数据格式存储起来。

RDBMS 和 MongoDB 中的概念都有着相互对应的关系，数据库、表、行和索引的概念在两中数据库中都非常相似，唯独最后的 `JOIN` 和 `Embedded Document` 或者 `Reference` 有着巨大的差别。这一点差别其实也影响了在使用 MongoDB 时对集合（Collection）Schema 的设计，如果我们在 MongoDB 中遵循了与 RDBMS 中相同的思想对 Collection 进行设计，那么就不可避免的使用很多的 “JOIN” 语句，而 MongoDB 是不支持 “JOIN” 的，在应用内做这种查询的性能非常非常差，在这时使用嵌入式的文档其实就可以解决这种问题了，嵌入式的文档虽然可能会造成很多的数据冗余导致我们在更新时会很痛苦，但是查询时确实非常迅速。

```
{
  _id: <ObjectId1>,
  name: "draveness",
  books: [
    {
      _id: <ObjectId2>,
      name: "MongoDB: The Definitive Guide"
    },
    {
      _id: <ObjectId3>,
      name: "High Performance MySQL"
    }
  ]
}

```

在 MongoDB 的使用时，我们一定要忘记很多 RDBMS 中对于表设计的规则，同时想清楚 MongoDB 的优势，仔细思考如何对表进行设计才能利用 MongoDB 提供的诸多特性提升查询的效率。

## 数据模型

MongoDB 与 RDBMS 之间最大的不同，就是数据模型的设计有着非常明显的差异，数据模型的不同决定了它有着非常不同的特性，存储在 MongoDB 中的数据有着非常灵活的 Schema，我们不需要像 RDBMS 一样，在插入数据之前就决定并且定义表中的数据结构，MongoDB 的结合不对 Collection 的数据结构进行任何限制，但是在实际使用中，同一个 Collection 中的大多数文档都具有类似的结构。

![Different-Data-Structure](https://img.draveness.me/2017-09-06-Different-Data-Structure.jpg-1000width)

在为 MongoDB 应用设计数据模型时，如何表示数据模型之间的关系其实是需要开发者需要仔细考虑的，MongoDB 为表示文档之间的关系提供了两种不同的方法：引用和嵌入。

### 标准化数据模型

引用（Reference）在 MongoDB 中被称为标准化的数据模型，它与 MySQL 的外键非常相似，每一个文档都可以通过一个 `xx_id` 的字段『链接』到其他的文档：

![Reference-MongoDB](https://img.draveness.me/2017-09-06-Reference-MongoDB.jpg-1000width)

但是 MongoDB 中的这种引用不像 MySQL 中可以直接通过 JOIN 进行查找，我们需要使用额外的查询找到该引用对应的模型，这虽然提供了更多的灵活性，不过由于增加了客户端和 MongoDB 之间的交互次数（Round-Trip）也会导致查询变慢，甚至非常严重的性能问题。

MongoDB 中的引用并不会对引用对应的数据模型是否真正存在做出任何的约束，所以如果在应用层级没有对文档之间的关系有所约束，那么就可能会出现引用了指向不存在的文档的问题：

![Not-Found-Document](https://img.draveness.me/2017-09-06-Not-Found-Document.jpg-1000width)

虽然引用有着比较严重的性能问题并且在数据库层面没有对模型是否被删除加上限制，不过它提供的一些特点是嵌入式的文档无法给予了，当我们需要表示多对多关系或者更加庞大的数据集时，就可以考虑使用标准化的数据模型 — 引用了。

### 嵌入式数据模型

除了与 MySQL 中非常相似的引用，MongoDB 由于其独特的数据存储方式，还提供了嵌入式的数据模型，嵌入式的数据模型也被认为是不标准的数据模型：

![Embedded-Data-Models-MongoDB](https://img.draveness.me/2017-09-06-Embedded-Data-Models-MongoDB.jpg-1000width)

因为 MongoDB 使用 BSON 的数据格式对数据进行存储，而嵌入式数据模型中的子文档其实就是父文档中的另一个值，只是其中存储的是一个对象：

```
{
  _id: <ObjectId1>,
  username: "draveness",
  age: 20,
  contact: [
    {
      _id: <ObjectId2>,
      email: "i@draveness.me"
    }
  ]
}

```

嵌入式的数据模型允许我们将有相同的关系的信息存储在同一个数据记录中，这样应用就可以更快地对相关的数据进行查询和更新了；当我们的数据模型中有『包含』这样的关系或者模型经常需要与其他模型一起出现（查询）时，比如文章和评论，那么就可以考虑使用嵌入式的关系对数据模型进行设计。

总而言之，嵌入的使用让我们在更少的请求中获得更多的相关数据，能够为读操作提供更高的性能，也为在同一个写请求中同时更新相关数据提供了支持。

> MongoDB 底层的 WiredTiger 存储引擎能够保证对于同一个文档的操作都是原子的，任意一个写操作都不能原子性地影响多个文档或者多个集合。

## 主键和索引

在这一节中，我们将主要介绍 MongoDB 中不同类型的索引，当然也包括每个文档中非常重要的字段 `_id`，可以**理解**为 MongoDB 的『主键』，除此之外还会介绍单字段索引、复合索引以及多键索引等类型的索引。

MongoDB 中索引的概念其实与 MySQL 中的索引相差不多，无论是底层的数据结构还是基本的索引类型都几乎完全相同，两者之间的区别就在于因为 MongoDB 支持了不同类型的数据结构，所以也理所应当地提供了更多的索引种类。

![MongoDB-Indexes](https://img.draveness.me/2017-09-06-MongoDB-Indexes.jpg-1000width)

### 默认索引

MySQL 中的每一个数据行都具有一个主键，数据库中的数据都是按照以主键作为键物理地存储在文件中的；除了用于数据的存储，主键由于其特性也能够加速数据库的查询语句。

而 MongoDB 中所有的文档也都有一个唯一的 `_id` 字段，在默认情况下所有的文档都使用一个长 12 字节的 `ObjectId` 作为默认索引：

![MongoDB-ObjectId](https://img.draveness.me/2017-09-06-MongoDB-ObjectId.jpg-1000width)

前四位代表当前 `_id` 生成时的 Unix 时间戳，在这之后是三位的机器标识符和两位的处理器标识符，最后是一个三位的计数器，初始值就是一个随机数；通过这种方式代替递增的 `id` 能够解决分布式的 MongoDB 生成唯一标识符的问题，同时可以在一定程度上保证 `id` 的的增长是递增的。

### 单字段索引（Single Field）

除了 MongoDB 提供的默认 `_id` 字段之外，我们还可以建立其它的单键索引，而且其中不止支持顺序的索引，还支持对索引倒排：

```
db.users.createIndex( { age: -1 } )

```

MySQL8.0 之前的索引都只能是正序排列的，在 8.0 之后才引入了逆序的索引，单一字段索引可以说是 MySQL 中的辅助（Secondary）索引的一个子集，它只是对除了 `_id` 外的任意单一字段建立起正序或者逆序的索引树。

![Single-Field-Index](https://img.draveness.me/2017-09-06-Single-Field-Index.jpg-1000width)

### 复合索引（Compound）

除了单一字段索引这种非常简单的索引类型之外，MongoDB 还支持多个不同字段组成的复合索引（Compound Index），由于 MongoDB 中支持对同一字段的正逆序排列，所以相比于 MySQL 中的辅助索引就会出现更多的情况：

```
db.users.createIndex( { username: 1, age: -1 } )
db.users.createIndex( { username: 1, age: 1 } )

```

上面的两个索引是完全不同的，在磁盘上的 B+ 树其实也按照了完全不同的顺序进行存储，虽然 `username` 字段都是升序排列的，但是对于 `age` 来说，两个索引的处理是完全相反的：

![Compound-Index](https://img.draveness.me/2017-09-06-Compound-Index.jpg-1000width)

这也就造成了在使用查询语句对集合中数据进行查找时，如果约定了正逆序，那么其实是会使用不同的索引的，所以在索引创建时一定要考虑好使用的场景，避免创建无用的索引。

### 多键索引（Multikey）

由于 MongoDB 支持了类似数组的数据结构，所以也提供了名为多键索引的功能，可以将数组中的每一个元素进行索引，索引的创建其实与单字段索引没有太多的区别：

```
db.collection.createIndex( { address: 1 } )

```

如果一个字段是值是数组，那么在使用上述代码时会自动为这个字段创建一个多键索引，能够加速对数组中元素的查找。

### 文本索引（Text）

文本索引是 MongoDB 为我们提供的另一个比较实用的功能，不过在这里也只是对这种类型的索引提一下，也不打算深入去谈谈这东西的性能如何，如果真的要做全文索引的话，还是推荐使用 Elasticsearch 这种更专业的东西来做，而不是使用 MongoDB 提供的这项功能。

## 存储

如何存储数据就是一个比较重要的问题，在前面我们已经提到了 MongoDB 与 MySQL 一样都提供了插件化的存储引擎支持，作为 MongoDB 的主要组件之一，存储引擎全权负责了 MongoDB 对数据的管理。

![Multiple-Storage-Engines](https://img.draveness.me/2017-09-06-Multiple-Storage-Engines.jpg-1000width)

### WiredTiger

MongoDB3.2 之后 WiredTiger 就是默认的存储引擎了，如果对各个存储引擎并不了解，那么还是不要改变 MongoDB 的默认存储引擎；它有着非常多的优点，比如拥有效率非常高的缓存机制：

![WiredTiger-Cache](https://img.draveness.me/2017-09-06-WiredTiger-Cache.jpg-1000width)

WiredTiger 还支持在内存中和磁盘上对索引进行压缩，在压缩时也使用了前缀压缩的方式以减少 RAM 的使用，在后面的文章中我们会详细介绍和分析 WiredTiger 存储引擎是如何对各种数据进行存储的。

### Journaling

为了在数据库宕机保证 MongoDB 中数据的持久性，MongoDB 使用了 Write Ahead Logging 向磁盘上的 journal 文件预先进行写入；除了 journal 日志，MongoDB 还使用检查点（Checkpoint）来保证数据的一致性，当数据库发生宕机时，我们就需要 Checkpoint 和 journal 文件协作完成数据的恢复工作：

1. 在数据文件中查找上一个检查点的标识符；
2. 在 journal 文件中查找标识符对应的记录；
3. 重做对应记录之后的全部操作；

MongoDB 会每隔 60s 或者在 journal 数据的写入达到 2GB 时设置一次检查点，当然我们也可以通过在写入时传入 `j: true` 的参数强制 journal 文件的同步。

![Checkpoints-Conditions](https://img.draveness.me/2017-09-06-Checkpoints-Conditions.jpg-1000width)

这篇文章并不会介绍 Journal 文件的格式以及相关的内容，作者可能会在之后介绍分析 WiredTiger 的文章中简单分析其存储格式以及一些其它特性。

## 总结

这篇文章中只是对 MongoDB 的一些基本特性以及数据模型做了简单的介绍，虽然『无限』扩展是 MongoDB 非常重要的特性，但是由于篇幅所限，我们并没有介绍任何跟 MongoDB 集群相关的信息，不过会在之后的文章中专门介绍多实例的 MongoDB 是如何协同工作的。

在这里，我想说的是，如果各位读者接收到了类似 MongoDB 比 MySQL 性能好很多的断言，但是在使用 MongoDB 的过程中仍然遵循以往 RDBMS 对数据库的设计方式，那么我相信性能在最终也不会有太大的提升，反而可能会不升反降；只有真正理解 MongoDB 的数据模型，并且根据业务的需要进行设计才能很好地利用类似嵌入式文档等特性并提升 MongoDB 的性能。

# MySQL 索引设计概要

在关系型数据库中设计索引其实并不是复杂的事情，很多开发者都觉得设计索引能够提升数据库的性能，相关的知识一定非常复杂。

![Index-and-Performance](https://img.draveness.me/2017-09-11-Index-and-Performance.jpg-1000width)

然而这种想法是不正确的，索引其实并不是一个多么高深莫测的东西，只要我们掌握一定的方法，理解索引的实现就能在不需要 DBA 的情况下设计出高效的索引。

本文会介绍 [数据库索引设计与优化](https://www.amazon.cn/%E5%9B%BE%E4%B9%A6/dp/B00ZH27RH0) 中设计索引的一些方法，让各位读者能够快速的在现有的工程中设计出合适的索引。

## 磁盘 IO

一个数据库必须保证其中存储的所有数据都是可以随时读写的，同时因为 MySQL 中所有的数据其实都是以文件的形式存储在磁盘上的，而从磁盘上**随机访问**对应的数据非常耗时，所以数据库程序和操作系统提供了缓冲池和内存以提高数据的访问速度。

![Disk-IO](https://img.draveness.me/2017-09-11-Disk-IO.jpg-1000width)

除此之外，我们还需要知道数据库对数据的读取并不是以行为单位进行的，无论是读取一行还是多行，都会将该行或者多行所在的页全部加载进来，然后再读取对应的数据记录；也就是说，读取所耗费的时间与行数无关，只与页数有关。

![Page-DatabaseBufferPool](https://img.draveness.me/2017-09-11-Page-DatabaseBufferPool.jpg-1000width)

在 MySQL 中，页的大小一般为 16KB，不过也可能是 8KB、32KB 或者其他值，这跟 MySQL 的存储引擎对数据的存储方式有很大的关系，文中不会展开介绍，不过**索引或行记录是否在缓存池中极大的影响了访问索引或者数据的成本**。

### 随机读取

数据库等待一个页从磁盘读取到缓存池的所需要的成本巨大的，无论我们是想要读取一个页面上的多条数据还是一条数据，都需要消耗**约** 10ms 左右的时间：

![Disk-Random-IO](https://img.draveness.me/2017-09-11-Disk-Random-IO.jpg-1000width)

10ms 的时间在计算领域其实是一个非常巨大的成本，假设我们使用脚本向装了 SSD 的磁盘上顺序写入字节，那么在 10ms 内可以写入大概 3MB 左右的内容，但是数据库程序在 10ms 之内只能将一页的数据加载到数据库缓冲池中，从这里可以看出随机读取的代价是巨大的。

![Disk-IO-Total-Time](https://img.draveness.me/2017-09-11-Disk-IO-Total-Time.jpg-1000width)

这 10ms 的一次随机读取是按照每秒 50 次的读取计算得到的，其中等待时间为 3ms、磁盘的实际繁忙时间约为 6ms，最终数据页从磁盘传输到缓冲池的时间为 1ms 左右，在对查询进行估算时并不需要准确的知道随机读取的时间，只需要知道估算出的 10ms 就可以了。

### 内存读取

如果在数据库的**缓存池**中没有找到对应的数据页，那么会去内存中寻找对应的页面：

![Read-from-Memory](https://img.draveness.me/2017-09-11-Read-from-Memory.jpg-1000width)

当对应的页面存在于内存时，数据库程序就会使用内存中的页，这能够将数据的读取时间降低一个数量级，将 10ms 降低到 1ms；MySQL 在执行读操作时，会先从数据库的缓冲区中读取，如果不存在与缓冲区中就会尝试从内存中加载页面，如果前面的两个步骤都失败了，最后就只能执行随机 IO 从磁盘中获取对应的数据页。

### 顺序读取

从磁盘读取数据并不是都要付出很大的代价，当数据库管理程序一次性从磁盘中**顺序**读取大量的数据时，读取的速度会异常的快，大概在 40MB/s 左右。

![Sequential-Reads-from-Disk](https://img.draveness.me/2017-09-11-Sequential-Reads-from-Disk.jpg-1000width)

如果一个页面的大小为 4KB，那么 1s 的时间就可以读取 10000 个页，读取一个页面所花费的平均时间就是 0.1ms，相比随机读取的 10ms 已经降低了两个数量级，甚至比内存中读取数据还要快。

![Random-to-Sequentia](https://img.draveness.me/2017-09-11-Random-to-Sequential.jpg-1000width)

数据页面的顺序读取有两个非常重要的优势：

1. 同时读取多个界面意味着总时间的消耗会大幅度减少，磁盘的吞吐量可以达到 40MB/s；
2. 数据库管理程序会对一些即将使用的界面进行预读，以减少查询请求的等待和响应时间；

### 小结

数据库查询操作的时间大都消耗在从磁盘或者内存中读取数据的过程，由于随机 IO 的代价巨大，如何在一次数据库查询中减少随机 IO 的次数往往能够大幅度的降低查询所耗费的时间提高磁盘的吞吐量。

## 查询过程

在上一节中，文章从数据页加载的角度介绍了磁盘 IO 对 MySQL 查询的影响，而在这一节中将介绍 MySQL 查询的执行过程中以及数据库中的数据的特征对最终查询性能的影响。

### 索引片（Index Slices）

索引片其实就是 SQL 查询在执行过程中扫描的一个索引片段，在这个范围中的索引将被顺序扫描，根据索引片包含的列数不同，[数据库索引设计与优化](https://www.amazon.cn/%E5%9B%BE%E4%B9%A6/dp/B00ZH27RH0) 书中对将索引分为宽索引和窄索引：

![Thin-Index-and-Fat-Index](https://img.draveness.me/2017-09-11-Thin-Index-and-Fat-Index.jpg-1000width)

> 主键列 `id` 在所有的 MySQL 索引中都是一定会存在的。

对于查询 `SELECT id, username, age FROM users WHERE username="draven"` 来说，(id, username) 就是一个窄索引，因为该索引没有包含存在于 SQL 查询中的 age 列，而 (id, username, age) 就是该查询的一个宽索引了，它**包含这个查询中所需要的全部数据列**。

宽索引能够避免二次的随机 IO，而窄索引就需要在对索引进行顺序读取之后再根据主键 id 从主键索引中查找对应的数据：

![Thin-Index-and-Clustered-Index](https://img.draveness.me/2017-09-11-Thin-Index-and-Clustered-Index.jpg-1000width)

对于窄索引，每一个在索引中匹配到的记录行最终都需要执行另外的随机读取从聚集索引中获得剩余的数据，如果结果集非常大，那么就会导致随机读取的次数过多进而影响性能。

### 过滤因子

从上一小节对索引片的介绍，我们可以看到影响 SQL 查询的除了查询本身还与数据库表中的数据特征有关，如果使用的是窄索引那么对表的随机访问就不可避免，在这时如何让索引片变『薄』就是我们需要做的了。

一个 SQL 查询扫描的索引片大小其实是由过滤因子决定的，也就是满足查询条件的记录行数所占的比例：

![Filter-Facto](https://img.draveness.me/2017-09-11-Filter-Factor.jpg-1000width)

对于 users 表来说，sex=”male” 就不是一个好的过滤因子，它会选择整张表中一半的数据，所以**在一般情况下**我们最好不要使用 sex 列作为整个索引的第一列；而 name=”draven” 的使用就可以得到一个比较好的过滤因子了，它的使用能过滤整个数据表中 99.9% 的数据；当然我们也可以将这三个过滤进行组合，创建一个新的索引 (name, age, sex) 并同时使用这三列作为过滤条件：

![Combined-Filter-Facto](https://img.draveness.me/2017-09-11-Combined-Filter-Factor.jpg-1000width)

> 当三个过滤条件都是等值谓词时，几个索引列的顺序其实是无所谓的，索引列的顺序不会影响同一个 SQL 语句对索引的选择，也就是索引 (name, age, sex) 和 (age, sex, name) 对于上图中的条件来说是完全一样的，这两个索引在执行查询时都有着完全相同的效果。

组合条件的过滤因子就可以达到十万分之 6 了，如果整张表中有 10w 行数据，也只需要在扫描薄索引片后进行 6 次随机读取，这种直接使用乘积来计算组合条件的过滤因子其实有一个比较重要的问题：列与列之间不应该有太强的相关性，如果不同的列之间有相关性，那么得到的结果就会比直接乘积得出的结果大一些，比如：所在的城市和邮政编码就有非常强的相关性，两者的过滤因子直接相乘其实与实际的过滤因子会有很大的偏差，不过这在多数情况下都不是太大的问题。

对于一张表中的同一个列，不同的值也会有不同的过滤因子，这也就造成了同一列的不同值最终的查询性能也会有很大差别：

![Same-Columns-Filter-Facto](https://img.draveness.me/2017-09-11-Same-Columns-Filter-Factor.jpg-1000width)

当我们评估一个索引是否合适时，需要考虑极端情况下查询语句的性能，比如 0% 或者 50% 等；最差的输入往往意味着最差的性能，在平均情况下表现良好的 SQL 语句在极端的输入下可能就完全无法正常工作，这也是在设计索引时需要注意的问题。

总而言之，需要扫描的索引片的大小对查询性能的影响至关重要，而扫描的索引记录的数量，就是总行数与组合条件的过滤因子的乘积，索引片的大小最终也决定了从表中读取数据所需要的时间。

### 匹配列与过滤列

假设在 users 表中有 name、age 和 (name, sex, age) 三个辅助索引；当 WHERE 条件中存在类似 age = 21 或者 name = “draven” 这种**等值谓词**时，它们都会成为匹配列（Matching Column）用于选择索引树中的数据行，但是当我们使用以下查询时：

```
SELECT * FROM users
WHERE name = "draven" AND sex = "male" AND age > 20;

```

虽然我们有 (name, sex, age) 索引包含了上述查询条件中的全部列，但是在这里只有 name 和 sex 两列才是匹配列，MySQL 在执行上述查询时，会选择 name 和 sex 作为匹配列，扫描所有满足条件的数据行，然后将 age 当做过滤列（Filtering Column）：

![Match-Columns-Filter-Columns](https://img.draveness.me/2017-09-11-Match-Columns-Filter-Columns.jpg-1000width)

过滤列虽然不能够减少索引片的大小，但是能够减少从表中随机读取数据的次数，所以在索引中也扮演着非常重要的角色。

## 索引的设计

作者相信文章前面的内容已经为索引的设计提供了充足的理论基础和知识，从总体来看如何减少随机读取的次数是设计索引时需要重视的最重要的问题，在这一节中，我们将介绍 [数据库索引设计与优化](https://www.amazon.cn/%E5%9B%BE%E4%B9%A6/dp/B00ZH27RH0) 一书中归纳出的设计最佳索引的方法。

### 三星索引

三星索引是对于一个查询语句可能的最好索引，如果一个查询语句的索引是三星索引，那么它只需要进行**一次磁盘的随机读及一个窄索引片的顺序扫描**就可以得到全部的结果集；因此其查询的响应时间比普通的索引会少几个数量级；根据书中对三星索引的定义，我们可以理解为主键索引对于 `WHERE id = 1` 就是一个特殊的三星索引，我们只需要对主键索引树进行一次索引访问并且顺序读取一条数据记录查询就结束了。

![Three-Star-Index](https://img.draveness.me/2017-09-11-Three-Star-Index.jpg-1000width)

为了满足三星索引中的三颗星，我们分别需要做以下几件事情：

1. 第一颗星需要取出所有等值谓词中的列，作为索引开头的最开始的列（任意顺序）；
2. 第二颗星需要将 ORDER BY 列加入索引中；
3. 第三颗星需要将查询语句剩余的列全部加入到索引中；

> 三星索引的概念和星级的给定来源于 [数据库索引设计与优化](https://www.amazon.cn/%E5%9B%BE%E4%B9%A6/dp/B00ZH27RH0) 书中第四章三星索引一节。

如果对于一个查询语句我们依照上述的三个条件进行设计，那么就可以得到该查询的三星索引，这三颗星中的最后一颗星往往都是最容易获得的，满足第三颗星的索引也就是上面提到的宽索引，能够避免大量的随机 IO，如果我们遵循这个顺序为一个 SQL 查询设计索引那么我们就可以得到一个完美的索引了；这三颗星的获得其实也没有表面上这么简单，每一颗星都有自己的意义：

![Behind-Three-Star-Index](https://img.draveness.me/2017-09-11-Behind-Three-Star-Index.jpg-1000width)

1. 第一颗星不只是将等值谓词的列加入索引，它的作用是减少索引片的大小以减少需要扫描的数据行；
2. 第二颗星用于避免排序，减少磁盘 IO 和内存的使用；
3. 第三颗星用于避免每一个索引对应的数据行都需要进行一次随机 IO 从聚集索引中读取剩余的数据；

在实际场景中，问题往往没有这么简单，我们虽然可以总能够通过宽索引避免大量的随机访问，但是在一些复杂的查询中我们无法同时获得第一颗星和第二颗星。

```
SELECT id, name, age FROM users
WHERE age BETWEEN 18 AND 21
  AND city = "Beijing"
ORDER BY name;

```

在上述查询中，我们总可以通过增加索引中的列以获得第三颗星，但是如果我们想要获得第一颗星就需要最小化索引片的大小，这时索引的前缀必须为 (city, age)，在这时再想获得第三颗星就不可能了，哪怕在 age 的后面添加索引列 name，也会因为 name 在范围索引列 age 后面必须进行一次排序操作，最终得到的索引就是 (city, age, name, id)：

![Different-Stars-Index](https://img.draveness.me/2017-09-11-Different-Stars-Index.jpg-1000width)

如果我们需要在内存中避免排序的话，就需要交换 age 和 name 的位置了，在这时就可以得到索引 (city, name, age, id)，当一个 SQL 查询中**同时拥有范围谓词和 ORDER BY 时**，无论如何我们都是没有办法获得一个三星索引的，我们能够做的就是在这两者之间做出选择，是牺牲第一颗星还是第二颗星。

总而言之，在设计单表的索引时，首先把查询中所有的**等值谓词全部取出**以任意顺序放在索引最前面，在这时，如果索引中同时存在范围索引和 ORDER BY 就需要权衡利弊了，希望最小化扫描的索引片厚度时，应该将**过滤因子最小的范围索引列**加入索引，如果希望避免排序就选择 **ORDER BY 中的全部列**，在这之后就只需要将查询中**剩余的全部列**加入索引了，通过这种固定的方法和逻辑就可以最快地获得一个查询语句的二星或者三星索引了。

## 总结

在单表上对索引进行设计其实还是非常容易的，只需要遵循固定的套路就能设计出一个理想的三星索引，在这里强烈推荐 [数据库索引设计与优化](https://www.amazon.cn/%E5%9B%BE%E4%B9%A6/dp/B00ZH27RH0) 这本书籍，其中包含了大量与索引设计与优化的相关内容；在之后的文章中读者也会分析介绍书中提供的几种估算方法，来帮助我们通过预估问题设计出更高效的索引。

如果对文章内容的有疑问，可以在博客下面评论留言。

# MySQL 索引性能分析概要

上一篇文章 [MySQL 索引设计概要](https://draveness.me/sql-index-intro) 介绍了影响索引设计的几大因素，包括过滤因子、索引片的宽窄与大小以及匹配列和过滤列。在文章的后半部分介绍了 [数据库索引设计与优化](https://www.amazon.cn/%E5%9B%BE%E4%B9%A6/dp/B00ZH27RH0) 一书中，理想的三星索引的设计流程和套路，到目前为止虽然我们掌握了单表索引的设计方法，但是却没有分析预估索引耗时的能力。

![Proactive-Index-Design](https://img.draveness.me/2017-09-16-Proactive-Index-Design.jpg-1000width)

在本文中，我们将介绍书中提到的两种分析索引性能的方法：基本问题法（BQ）和快速估算上限法（QUBE），这两种方法能够帮助我们快速分析、估算索引的性能，及时发现问题。

## 基本问题法

当我们需要考虑对现有的 SELECT 查询进行分析时，哪怕没有足够的时间，也应该使用基本问题法对查询进行评估，评估的内容非常简单：现有的索引或者即将添加的索引是否包含了 WHERE 中使用的全部列，也就是对于当前查询来说，是否有一个索引是半宽索引。

![Semifat-Index-and-Fat-Index](https://img.draveness.me/2017-09-16-Semifat-Index-and-Fat-Index.jpg-1000width)

在上一篇文章中，我们介绍过宽索引和窄索引，窄索引 (username) 其实就叫做半宽索引，其中包含了 WHERE 中的全部的列 username，当前索引的对于该查询只有一颗星，它虽然避免了无效的回表查询造成的随机 IO，但是如果当前的索引的性能仍然无法满足需要，就可以添加 age 将该索引变成宽索引 (username, age) 以此来避免回表访问造成的性能影响；对于上图中的简单查询，索引 (username, age) 其实已经是一个三星索引了，但是对于包含 ORDER BY 或者更加复杂的查询，(username, age) 可能就只是二星索引：

![Complicated-Query-with-Order-By](https://img.draveness.me/2017-09-16-Complicated-Query-with-Order-By.jpg-1000width)

在这时如果该索引仍然不能满足性能的需要，就可以考虑按照上一篇文章 [MySQL 索引设计概要](https://draveness.me/sql-index-intro) 中提供的索引设计方法重新设计了。

> 虽然基本问题法能够快速解决一些由于索引造成的问题，但是它并不能保证足够的性能，当表中有 (city, username, age) 索引，谓词为 `WHERE username="draveness" AND age="21"` 时，使用基本问题法并不能得出正确的结果。

## 快速估算上限法

基本问题法非常简单，它能够最短的时间内帮助我们评估一个查询的性能，但是它并不能准确地反映一个索引相关的性能问题，而快速估算上限法就是一种更加准确、复杂的方法了；其目的在于在程序开发期间就能将访问路径缓慢的问题暴露出来，这个估算方法的输出就是本地响应时间（Local Response Time）：

![QUBE-LRT](https://img.draveness.me/2017-09-16-QUBE-LRT.jpg-1000width)

本地响应时间就是查询在数据库服务器中的耗时，不包括任何的网络延迟和多层环境的通信时间，仅包括执行查询任务的耗时。

### 响应时间

本地响应时间等于服务时间和排队时间的总和，一次查询请求需要在数据库中等待 CPU 以及磁盘的响应，也可能会因为其他事务正在对同样的数据进行读写，导致当前查询需要等待锁的获取，不过组成响应时间中的主要部分还是磁盘的服务时间：

![Local-Response-Time](https://img.draveness.me/2017-09-16-Local-Response-Time.jpg-1000width)

QUBE 在计算的过程中会忽略除了磁盘排队时间的其他排队时间，这样能够简化整个评估流程，而磁盘的服务时间主要还是包括同步读写以及异步读几个部分：

![Disk-Service-Time](https://img.draveness.me/2017-09-16-Disk-Service-Time.jpg-1000width)

在排除了上述多个部分的内容，我们得到了一个非常简单的估算过程，整个估算时间的输入仅为随机读和顺序读以及数据获取的三个输入，而它们也是影响查询的主要因素：

![Local-Response-Time-Calculation](https://img.draveness.me/2017-09-16-Local-Response-Time-Calculation.jpg-1000width)

其中数据获取的过程在比较不同的索引对同一查询的影响是不需要考虑的，因为同一查询使用不同的索引也会得到相同的结果集，获取的数据也是完全相同的。

### 访问

当 MySQL 读取一个索引行或者一个表行时，就会发生一次访问，当使用全表扫描或者扫描索引片时，读取的第一个行就是随机访问，随机访问需要磁盘进行寻道和旋转，所以其代价巨大，而接下来顺序读取的所有行都是通过顺序访问读取的，代价只有随机访问的千分之一。

如果大量的顺序读取索引行和表行，在原理上可能会造成一些额外的零星的随机访问，不过这对于整个查询的估算来说其实并不重要；在计算本地响应时间时，仍然会把它们当做顺序访问进行估算。

### 示例

在这里，我们简单地举一个例子来展示如何计算查询在使用某个索引时所需要的本地响应时间，假设我们有一张 `users` 表，其中有一千万条数据：

![User-Table](https://img.draveness.me/2017-09-16-User-Table.jpg-1000width)

在该 `users` 表中除了主键索引之外，还具有以下 (username, city)、(username, age) 和 (username) 几个辅助索引，当我们使用如下所示的查询时：

![Filter-Facto](https://img.draveness.me/2017-09-16-Filter-Factor.jpg-1000width)

两个查询条件分别有着 0.05% 和 12% 的过滤因子，该查询可以直接使用已有的辅助索引 (username, city)，接下来我们根据表中的总行数和过滤因子开始估算这一步骤 SQL 的执行时间：

![Index-Slice-Scan](https://img.draveness.me/2017-09-16-Index-Slice-Scan.jpg-1000width)

该查询在开始时会命中 (username, city) 索引，扫描符合条件的索引片，该索引总共会访问 10,000,000 * 0.05% * 12% = 600 条数据，其中包括 1 次的随机访问和 599 次的顺序访问，因为该索引中的列并不能满足查询的需要，所以对于每一个索引行都会产生一次表的随机访问，以获取剩余列 age 的信息：

![Index-Table-Touch](https://img.draveness.me/2017-09-16-Index-Table-Touch.jpg-1000width)

在这个过程中总共产生了 600 次随机访问，最后取回结果集的过程中也会有 600 次 FETCH 操作，从总体上来看这一次 SQL 查询共进行了 **601 次随机访问**、599 次顺序访问和 600 次 FETCH，根据上一节中的公式我们可以得到这个查询的用时约为 6075.99ms 也就是 6s 左右，这个时间对于绝大多数应用都是无法接受的。

![SQL-Query-Time](https://img.draveness.me/2017-09-16-SQL-Query-Time.jpg-1000width)

在整个查询的过程中，回表查询的 600 次随机访问成为了这个超级慢的查询的主要贡献，为了解决这个问题，我们只需要添加一个 (username, city, age) 索引或者在已有的 (username, city) 后添加新的 age 列就可以避免 600 次的随机访问：

![SQL-Query-Time-After-Optimization](https://img.draveness.me/2017-09-16-SQL-Query-Time-After-Optimization.jpg-1000width)

(username, city, age) 索引对于该查询其实就是一个三星索引了，有关索引设计的内容可以阅读上一篇文章 [MySQL 索引设计概要](https://draveness.me/sql-index-intro) 如果读者有充足的时间依然强烈推荐 [数据库索引设计与优化](https://www.amazon.cn/%E5%9B%BE%E4%B9%A6/dp/B00ZH27RH0) 这本书。

## 总结

这篇文章是这一年来写的最短的一篇文章了，本来想详细介绍一下 [数据库索引设计与优化](https://www.amazon.cn/%E5%9B%BE%E4%B9%A6/dp/B00ZH27RH0) 书中对于索引性能分析的预估方法，仔细想了一下这部分的内容实在太多，例子也非常丰富，只通过一篇文章很难完整地介绍其中的全部内容，所以只选择了其中的一部分知识点简单介绍，这也是这篇文章叫概要的原因。































































