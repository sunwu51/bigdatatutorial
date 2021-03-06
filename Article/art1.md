# 大数据分析的两大核心：存储构架和计算构架
在大数据领域的技术无非是在两个领域进行着探索和改进，一个是存储构架，另一个是计算构架。  
## 存储
随着互联网的发展，数据量越来越大从GB到TB再到PB，对于数据的存储就出现了问题。**单机存储**的情况下，如果机器损坏则数据全部丢失，再者TB级别的数据单机磁盘容量吃紧。**主从式的分布式存储**在一定程度上解决了机器损坏导致数据丢失的问题，但是主从一致性非常麻烦，从节点全都copy主节点磁盘性能没有利用起来。`HDFS`是一种**分布式文件系统**，通过将文件变成块状均匀的分布到每台分布式节点上的方式充分利用了每台机器的磁盘，并且设置了备份策略对每一个文件块都备份3份（默认设置）。这样即使一台机器损坏，上面的所有文件块都丢失了，也能保证在其他两台机子上总能找到每个文件块的备份。`HDFS`是谷歌分布式文件系统的开源实现，具有非常好的特性，直到现在仍然是最受欢迎的开源分布式文件系统，在各大计算框架中都有对`HDFS`的支持。`HDFS`尤其适合存储TB甚至PB级别的数据量，集群规模可以成千上万。而且对于存储的机器没有要求，甚至是性能落后的机子也可以参与到存储中来，贡献自己的一份力量，`HDFS`会根据每台机子的状态进行存储和读取。在数据存储部分。<br>

本系列除了介绍了`HDFS`后续还会介绍一些sql和nosql的框架，一般的场景是小型数据量例如GB级别的数据存储可以直接放到`Postgresql` `Mysql`这样的关系型数据库中，而TB级别数据则可以放到`GreenPlum` `Mongodb`这样的分布式数据库，如果是PB级别的数据就可以放到`HDFS`（当然TB GB也可以应对）。<br>

这是因为如果数据量较少的时候传统关系型数据库提供了强大的查询功能，存储和查询响应都很快，如果在较小的数据量情况下用`HDFS`反而因为网络IO的开销太大读写速度远低于前者。而TB级别的数据如果也想支持关系型数据库的操作则可以放到`GreenPlum`（简称GPDB）上面，对于该数据库的操作就和操作postgresql一样，后续会介绍，他是MPP的一种实现，支持分布式事务。而对于TB级别的数据，`GreenPlum`则会有些吃力，因为事务过程中开销可能过大，所以一般这个级别都会转向`HDFS`。当然`HDFS`只是一个文件系统，本身和前面的数据库是有不同的，他可以存储任何格式的文件，这就导致查询数据的时候非常麻烦。给出的解决方案有很多，比如在文件存储的时候按照一定的结构进行存储（如逗号隔开每列），然后用`Hive`对文件进行"表化"，用sql查询文件内容。再比如`HBase`基于键值对的存储数据在`HDFS`上，不过最终都没有实现关系型数据库的全部功能。
## 计算
大数据分析除了能把文件完好的存下来，还要提供分析的功能，这就是计算框架要实现的。在传统关系型数据sql查询本身就是一种计算框架（**查询本质就是计算**），比如选出最大的10个这种查询。我们也看到`Hive`在`HDFS`文件上实现了sql的部分常用功能，这对于只会sql的数据分析师来说更容易操作了。不可否认的是，sql是最好的数据统计查询工具了，所以有一种说法是`SQL on EveryThing`。到后面我们发现直接将关系型数据库性能加强，支持分布式也是一种大数据分析的解决方案，`Pipelinedb`和`GPDB`就是这样的产物。不过大数据分析除了分析这种结构化的数据还应该能够挖掘更灵活的数据类型，比如WordCount程序（数一篇文章的每个单词出现次数）就是一种灵活的数据类型（纯文本）上进行的灵活查询。这是sql不能实现的，这时候就需要写程序来实现，这里的程序就是要借助计算框架。<br><br>
Hadoop提供的计算框架叫`MapReduce`，我在前面都没有仔细讲这个计算框架，是因为大多数情况下MR的表现没有`Spark`好，所以直接就用更好的了。至于为啥有这种性能差异，会在下一篇浅谈中仔细讲述。Spark本身其实就是计算框架，有说法说Spark完全可以取代Hadoop，这是不准确的，因为HDFS也是Hadoop的重要组成，而Spark的大数据分析也可能是基于HDFS上的数据进行的分析。<br><br>
## 小结
对于存储框架除了`HDFS`后面还会介绍基于`Postgresql`改进的数据库存储方式，以及`NoSQL`的存储方式，还有强大的分布式内存存储方式。而计算框架则会介绍数据库的sql方言，Spark以及一些快速查询框架、全文查询框架。



