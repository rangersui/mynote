[toc]
# 概述
使用一个系统赖管理多台机器上的文件，HDFS是分布式文件管理系统中的一种。适合一次写入，多次读出的场景。
## 优缺点
### 高容错性
数据自动保存多个副本，通过增加副本提高容错性。某个副本丢失后自动回复。
### 适合处理大数据
- 数据规模：能够处理数据GB、TB甚至PB级别
- 文件规模：能够处理百万规模以上的文件数量
### 构建在廉价机器上
通过多副本机制提高可靠性
### 不适合低延时访问
如毫秒级存储数据灯场景不合适
### 无法高效对大量小文件进行存储
占用NameNode大量的内存赖存储文件目录和块信息。
寻址时间可能超过读取时间，违反HDFS的设计目标。
### 不支持并发写入，文件随机修改
一个文件只能有一个写，不允许同时写。
只能数据追加，不能对数据随机修改。
# HDFS的shell相关操作
基本语法`hadoop fs 具体命令`或`hdfs dfs 具体命令`
## 启动hadoop集群
```
sbin/start-dfs.sh
sbin/start-yarn.sh
```
## 上传
`hadoop fs -mkdir /sanguo`创建文件夹

`hadoop fs -moveFromLocal ./shuguo.txt /sanguo`剪切文件到指定目录

`hadoop fs -copyFromLocal ./weiguo.txt /sanguo`复制文件到指定目录，该命令等价于`hadoop fs -put ./weiguo.txt /sanguo`

`hadoop fs -appendToFile liubei.txt /sanguo/shuguo.txt`追加文件内容到已有的文件末尾。

## 下载
`hadoop fs -copyToLocal /sanguo/shuguo.txt ./`等同于`hadoop fs -get /sanguo/shuguo.txt ./`

## 支持的常用其他命令
基本和linux文件系统命令一致
|指令|功能|
|:-|:-|
|ls|查看文件列表|
|cat|查看文件内容|
|chgrp，chmod，chown|修改文件权限|
|mkdir|创建路径|
|cp|从HDFS路径复制到另一个HDFS路径|
|mv|在HDFS中移动文件|
|tail|显示末尾1kb的数据|
|rm|删除文件或文件夹|
|du -s -h|统计文件夹的大小信息,第一个数字是文件大小，第二个数字是文件大小乘以副本个数.-s代表统计文件夹|
|setrep|设置HDFS文件的副本数量，上限为节点数量|

# HDFS API
在Windows编写HDFS客户端代码。其实就是把指令转化为代码形式，具体参考[链接](https://www.bilibili.com/video/BV1Qp4y1n7EN?p=48)。
```java
package com.example.hdfs;
/*
1. 获取一个客户端对象
2. 执行操作命令
3. 关闭资源
HDFS ZOOKEEPER
*/
public class HdfsClient{

    private FileSystem fs;

    @Before
    public void init() throws Exception{
        //客户端和NameNode打交道,连接集群的NN地址
        URI uri = new URI("hdfs://hadoop102:8020");
        //创建配置文件
        Configuration configuration  = new Configuration();
        //用户
        String user = "user";

        Filesystem fs = FileSystem.get(uri,configuration,user);
    }

    @After
    public void close() throws Exception{
        fs.close();
    }

    @Test
    public void testmkdir(){

        fs. mkdirs(new Path("/test/"));
    }
}
```