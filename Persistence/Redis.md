# Redis
# 简介
`redis`是内存存储的键值对数据库，可以认为是将编程语言中的一些数据存储类型做成了数据库。`redis`支持`String`、`Hash`、`List`、`Set`、`ZSet`这几种数据类型的存储。他将数据存到内存，支持一定策略的磁盘持久化，所以也不用太担心数据丢失的问题。
# String
String类型就是一个键对应一个字符串值，这个值是按照String存储的。读写命令为
```
get key
set key value
mset k1 v1 k2 v2
mget k1 k2
```
# Hash
Hash类型就是一个键对应一个HashMap，例如`key:{name:'xx',age:'xx'}`。常用读写命令为
```
hget k field 
hset k field value 
hmget k f1 f2 ...
hmset k f1 v1 f2 v2 ...
hgetall k
```
# List
List类型就是一个键对应一个双向链表，可以类似的看成是java中的`LinkedList`，当做双向链表来用。常用的压入和弹出指令为：
```
lpush k v1 v2 ...
rpush k v1 v2 ...
lpop k
rpop k
```
# Set
Set类型就是一个键对应一个String集合，集合插入已有的值不会有任何反应，常用的集合操作增删有：
```
sadd k v
srem k v
```
# ZSet
ZSet是有序集合，在存储数据的时候，需要为元素指定`Order`序号，在某些特定场景下有用。
# 小结
从上面来看Redis的使用是非常简单的，他就像操作编程语言中的数据结构一样。当然上面的描述是非常简单的，除了数据读写redis还可以配置分布式主从模式，简单的事务以及持久化配置，当然还提供了一个简单的订阅发布服务器。更多地用法和操作可以参照官网或者参考这个[使用手册](http://microfrank.top/redis/)
