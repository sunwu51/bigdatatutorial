# Hadoop
# 1.介绍
Hadoop家族的族长，最初的1.x版本的Hadoop由两大部分组成：`HDFS`和`MapReduce`。
## 1.1 HDFS
Hadoop分布式文件系统，就是将文件存储到多台计算机上面，例如一个1PB大的文件，一台机子存不过来，可以将其存到多台机子（如100台）上面。HDFS帮我们做的就是实际存储到了多台机子上，但是对外提供访问的时候是一个文件例如`hdfs://myserver:9000/myfile`。这得益于文件的分块功能，默认将每个文件按照128M分成一个文件块，例如1PB就有1024x1024x8个文件块，文件块均匀放到100台机子上面进行存储。

除了可以存储海量数据，HDFS还提供了备份策略，上述场景下如果存储文件的机器有一台坏掉了，则无法复原整个文件，所以备份策略是为了更高的可靠性，默认会将每个文件块在3台机器上进行存储。即上述的文件块中每一块都必须在三台机子上面进行分别存储。这样虽然占用了三倍的存储空间，但是换来了可靠性。

名词解释
**NameNode**：hdfs访问的入口，从这里访问集群中的文件，也肩负着管理和调度文件块的作用，是hdfs系统的"管理员"。
**SecondNameNode**：管理员万一遇到不幸，需要有个能接替者，相当于NameNode的冷备份。
**DataNode**：存储数据的机子，管理员的下属，文件最终存储在这些节点上面。
三者可以分开，也可以是同一台机子兼职多个角色。一般学习过程中由于机器缺乏，往往将三者全都放到一台机子上面。
## 1.2 MapReduce
一种计算思想：一个人的力量再强也比不过一个军队。举个简单的例子数红楼梦中<林黛玉>三个字出现的次数，给一个来数，可能需要数很久，但是如果把这本书平均分给一个班的所有同学，第一个人数1-100页，第二个人101-200页，第三个人.......这样分配下去，很快每个人都数出了自己负责的部分的林黛玉数，汇报到一起运行一步加法运算，就完成了任务。

上面例子中每个人相当于进行Map运算，这个运算是并行的，而最后汇报到一起的加法相当于Reduce运算。不过值得一提的是Hadoop的MapReduce运算是基于文件系统的，即需要磁盘读取来进行操作。因而速度并不快，所以Hadoop的MapReduce用的越来越少了，一些基于内存操作的执行器如Spark，体现出了更高的性能。
## 1.3 Yarn
一个平台，为了海乃百川。正如上面所说Spark等更高性能的执行器逐渐取代着MapReduce部分，但是HDFS却没有过时，其他分析和执行的框架想要建立在HDFS这样的分布式文件系统上面。于是有了Yarn，Yarn是一个平台，他允许其他框架如Spark、Storm等在Yarn调度下运行在HDFS上面。
名词解释
**ResourceManager**：Yarn调度中的管理员
**NodeManager**：Yarn中的节点，管理员下属
# 2.搭建[2.8.0]
## 2.1 配置SSH密钥
在集群中需要能互相免密钥访问，因而需要配置SSH密钥。就算是伪集群也最好配置下，总之是有益无害的配置。
```
ssh-keygen -trsa -P ''
```
生成的密钥在用户目录的`.ssh`文件夹下，`id_rsa.pub`文件中的内容就是公钥，需要追加到同级目录下的`authorized_keys`文件中，如果是多台机子集群则需要将每台机子产生的pub文件写到一个`authorized_keys`中，然后将这个文件复制到每台机子上面。集群模式最好每台机子都有个域名，放到hosts文件中，方便后续配置。

验证免密钥配置成功，直接ssh server1，如果不用输账密直接连上这说明配置成功。


## 2.2 搭建HDFS
HDFS和Yarn是分开的，他们可以独立运行，没有互相依赖。例如如果不常使用Yarn，则可以只搭建HDFS。  

1 到官网下载Hadoop安装包，并解压  
2 修改配置文件在安装包下的`etc/hadoop`目录下，主要修改四个配置文件：  
 [hadoop-env.sh](conf/hadoop-env.sh)  
 [core-site.xml](conf/core-site.xml)  
 [hdfs-site.xml](conf/hdfs-site.xml)  
 [slaves](conf/slaves)指定哪些节点作为DataNode  
3 初始化NameNode
```
hadoop namenode -format
```
4 开启HDFS
```
sbin/start-dfs.sh
```
5 验证是否启动
通过JPS查看是否有NameNode DataNode和SecondNameNode。
## 2.3 搭建Yarn
如果你不确定需要Yarn，就先不用搭建Yarn等用到的时候再来搭建也不迟。  
1.修改配置文件同样在`etc/hadoop`路径下，主要修改俩文件：   
 [mapred-site.xml](conf/mapred-site.xml)只有ResourceManager节点才能有这个文件    
 [yarn-site.xml](conf/yarn-site.xml)  
2.启动yarn
```
sbin/start-yarn.sh
```
3.验证启动
jps中有ResourceManager和NodeManager
# 3 操作HDFS
既然是文件系统，那一定可以像文件一样的操作，HDFS和windows下的NTFS都是文件系统，我们的磁盘中文件可以读写，同样HDFS也应该能进行读写操作。hadoop提供了操作文件的shell和Java API，我们这里只介绍前者。我们先将Hadoop目录的bin和sbin目录加到环境变量PATH中。

列出hdfs根目录下的文件：
```
hadoop fs -ls /
```
创建目录和复制本地文件到hdfs
```
hadoop fs -mkdir /test
hadoop fs -put /root/1.txt /test
```
读取hdfs中的文件内容
```
hadoop fs -cat /test/1.txt
```
![image](img/hadoop.gif)
# 4 小结
Hadoop的HDFS是非常重要的分布式文件系统，在大数据分析领域很多实现都是基于HDFS的，所以是必须掌握和会使用的内容。MapReduce的思想也必须要理解，在后续可能使用的较少。Yarn则是在HadoopMapReduce程序的时候或者用到其他框架的时候需要使用的平台。本文中没有用MapReduce进行编程，主要是考虑到MapReduce程序较为难懂，且效率不高，在后来的工具和框架中完全可以替代。
文章篇幅有限，希望能帮助理解，想要更深入的理解HDFS的工作原理和MapReduce的原理还需要查询其他资料。
