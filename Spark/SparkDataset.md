# Dataset
# 1 概述
一开始我们介绍了`RDD`的使用，后来在`SparkSQL`中我们又介绍了`Dataset`。并且我们知道`Dataset`比`Dataframe`有更好的性质，所以已经替代掉了后者。这一节我们来对比下RDD和DS，看看两者的不同和使用环境。  
  
首先你可以读一下[这篇文章](http://blog.csdn.net/wo334499/article/details/51689549)、和[这篇文章](http://www.jianshu.com/p/c0181667daa0)。
# 2 对比
# 2.1 创建
JavaRDD的创建入口是`JavaSparkContext`  
RDD的创建入口是`SparkContext`  
如果用Java写则最好用JavaRDD会方便些
```java
SparkConf conf = new SparkConf().setAppName("app").setMaster("local");
JavaSparkContext sparkContext = new JavaSparkContext(conf);

//List变量转化为JavaRDD
List<String> list= Arrays.asList("r","vd","azhao","hia zo");
JavaRDD<String> rdd = sc.parallelize(list);

//从文件中读取为JavaRDD
JavaRDD textRdd = sc.textFile("text");
JavaRDD textRdd2 = rdd.objectFile("object");

//从DS(DF)转化为JavaRDD
ds.javaRdd();
df.javaRdd();
```
   
<br/>DS的创建入口是`SparkSession`(ss)
```java
SparkSession sparkSession = SparkSession.builder().appName("app").master("local").getOrCreate();

//从List变量转化为DS 注意后面Encoder必须写这个否则报错
//目前提供了基本类型的Encoder
sparkSession.createDataset(Arrays.asList(1,2,3),Encoders.INT());

//从文件中读取为DS(DF)  DS<Row>以前叫DF
Dataset<Row> df = spark.read().textFile("text");//纯文本Row一列字符串
Dataset<Row> df2 = spark.read().json("1.json");//json格式Row分多列
Dataset<Row> df3 = spark.read().load("parquet");//Row也是分多列的

//从JavaRDD<Person>转化为DS
Dataset<String> ds1= sparkSession.createDataset(javardd.rdd(),Encoders.STRING());
Dataset<Person> ds2= sparkSession.createDataset(javardd.rdd(),Encoders.javaSerialization(Person.class));
//注意Person需要实现Serializable接口
```
上面的Person类
```java
//必须有getter setter否则转为df是空
//必须实现Serializable否则无法转ds
class Person implements Serializable{
    long group;
    long age;
    String name;
    public long getAge() {
        return age;
    }

    public void setAge(long age) {
        this.age = age;
    }

    public long getGroup() {
        return group;
    }

    public void setGroup(long group) {
        this.group = group;
    }

    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }

    public Person(long age, long group, String name){
        this.group=group;this.name=name;this.age=age;
    }
}
```
# 2.2 API
RDD的API在之前已经介绍了，更多的方法可以参考[RddDemo](https://github.com/sunwu51/SparkDemo/blob/master/src/main/java/RddDemo.java)。<br>
DS的部分功能我们在SQL中也有所介绍，最大的特点就是DS封装了sql查询的相关方法，可以组合出任意sql(hql)语句。尤其是在查询方面，sql语句比rdd中调用filter来实现易读性要好很多。<br>
例如选出22-36岁名字中含有John的行，分别用DS和RDD实现<br>
DS:
```java
//ds类型是Dataset<Person>
Dataset<Person> sqlDF = ds.where(col("age").between("22","36"))
    .where(col("name").like("%John%")).select(col("*"));
```
RDD:
```java
//rdd类型是JavaRDD<Person>
JavaRDD<Person> newRdd = rdd.filter(new Function<Person, Boolean>() {
    @Override
    public Boolean call(Person person) throws Exception {
        return person.getAge()>22 && person.getAge()<36 &&person.getName().contains("John");
    }
});
```
DS的where可以用RDD的filter实现，DS的group等某些聚合类型的函数则需要RDD进行PairRDD变换后才能实现，所以在一些情况下简化了代码开发。使对SparkRDD API并不那么熟悉的只懂sql的程序员，也能写出高效的代码。而且DS通过显示的Encoder的声明方式，在序列化上取得了性能优势（相比RDD），而且因为ds操作过程中较少的中间项创建（相比RDD）GC压力稍小些。<br>
注:DS也有`map`和`flatMap`方法可以进行泛型类型转化。
# 4 小结
rdd的优点是高度的灵活性，高度的可自定义编程。缺点是网络IO中序列化反序列化消耗高，GC高。<br>
ds的优点是sql的易读易上手，显式Encoder序列化反序列化消耗稍小，GC稍小。<br>
在一般的应用中我们建议使用ds进行操作，实在没法实现的则转换成rdd操作。
