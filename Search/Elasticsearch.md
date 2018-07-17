# ElasticSearch
# 简介
ElasticSearch后面简称es，从名字中可以看出是一个搜索引擎。其本身也是一个数据(仓)库。有以下几个特点
- 1 存储json格式的数据，可以是非结构化，和mongo很像
- 2 支持Rest方式查询数据，在搜索上性能突出
- 3 支持集群化

其中第二个特点是最突出的。
# 安装
Es的安装非常简单，只需要官方下载，然后直接shell/cmd运行即可。我们这里同时下载es公司另一款产品Kibana，用于后续的图形化操作。同样是下载后直接shell/cmd运行。

或者docker compose的方式
```yaml
version: '3.1'

services:
  kibana:
    image: kibana
    ports:
      - 5601:5601

  elasticsearch:
    image: elasticsearch
```
# 概念
`index`索引，索引和mysql数据库中索引可不是一个东西，可以认为这是一个mysql中的库database。  
`type`类型，对应mysql中的table。最新版的es中一个index中只允许有一个type了。
`document`文档，相当于mysql中一条数据。

# 增删改查操作
Rest方式则是直接通过http访问9200（默认）端口：
## 1 添加/修改一条数据
```
PUT /alibaba/employee/1
{
  "name":"wuyao",
  "age":25,
  "interests":["music","sport"]
}
```
PUT方法后面要有id，如果没有则插入一条，如果有了则修改这条数据内容。同样没有index和type叫这个名字，也会创建。返回create字段是true则是创建，false则是修改。
## 2 添加一条随机id数据
```
POST /alibaba/employee
{
  "name":"wuyao"
}
```
POST添加数据，不需要最后跟id，会自动生成_id字段。
## 3 查询数据
```
GET /alibaba/employee/1
```
根据id查询这条数据，没有id返回404

```
GET /alibaba/employee/_search
```
查询所有数据，GET也可以改为POST和PUT也能查询。
```
GET /alibaba/employee/_search?q=age:25
```
条件查询age为25的数据
## 4 删除一条数据
```
DELETE /alibaba/employee/1
```
删除成功200，没有该id返回404

