# Hbase
# 1 简介
Hadoop DataBase的简称，不是有了Hive了吗，为什么又有Hbase呢？两者都是基于HadoopHDFS存储的能进行查询的数据仓库。
不过Hive针对的是结构化的已经存在的文件导入表中，对这个文件进行查询。其数据本身还是文件本身，而查询语句则是按照表结构封装成MapReduce程序去执行，响应没有那么即时。
Hbase则是一种非关系型数据库，不能将现成的文件转化，而是必须通过自己的API插入数据。数据格式是键值对的方式，不支持复杂条件查询如where语句（和`Redis`很像），设置过滤器可进行有限的条件查询。可以认为是海量键值对数据的存储，键值对的查找速度会很快（通过键找值），因而支持即时响应的查询。
##1.1 存储特点
传统数据库一个表有很多行，也有很多列。而Hbase则将多个列的基础上加了一个列族（cf）并对每一行自动添加行键（row-key）和时间戳。参考[这里](https://www.ibm.com/developerworks/cn/analytics/library/ba-cn-bigdata-hbase/index.html)。


# 2 搭建[1.2.6]
Hbase依赖于HDFS，请先完成[HDFS搭建](Hadoop.md#2.2 搭建HDFS)
这里我们搭建`单机运行`的Hbase，注意Hbase存储虽然在hdfs但是Hbase的服务器是和HDFS独立的，我们接入Hbase服务器可以查询其存在HDFS上的数据，这里并不矛盾，也就是说Hbase是单机的，HDFS是集群的这样的情况也是可以的。
1 官网下载安装包，解压配置环境变量
2 修改配置文件
  [hbase-env.sh](conf/hbase-env.sh)
  [hbase-site.xml](conf/hbase-site.xml)
3 启动hbase
```
start-hbase.sh
```
4 客户端连接
```
hbase shell
```

注：单机版的ZooKeeper是内置的，不用自己安装ZooKeeper。
# 3 使用Hbase
和前面一样Hbase提供了shell客户端和JavaAPI这里我们只讲前者。
常用的指令如创建表和增删改查：
```
//test是表名 info和pwd都是列族
//创建
create 'test','info','pwd'
//test是表 1是行键 pwd是列族 p是列族中一列 123是值 相当于1|pwd|p三个部分组成的键指向一个值
//增(已存在就是改)
put 'test','1','pwd:p','123'
//删
delete 'test','1','pwd:p'
//查 可以查整个表 也可以查表中一行 也可以是一列族 或一列下面4个参数可以有1-4个
get 'test'[,'1','pwd:p']
```
FILTER过滤器的使用
参考[这里](http://blog.csdn.net/qq_27078095/article/details/56482010)，如下是一个Row-Key过滤器过滤含有1的行数据。过滤器有行过滤、值过滤、列族过滤，这也就要求我们在命名的时候有一定的讲究才方便以后查询。

```
scan 'test', FILTER=>"RowFilter(=,'substring:1')"
```
![image](img/hbase.gif)
# 4 小结
Hbase是非关系型数据库，用于存储海量数据，因为键值对策略能达到即时查询的效果。

