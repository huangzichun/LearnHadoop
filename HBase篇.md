>Ubuntu16 Desktop
# HBase
## 1. HBase安装
### 0. 准备
同 [Hadoop](/Hadoop篇.md)
### 1.下载&安装
安装[镜像](http://mirror.bit.edu.cn/apache/hbase/)可以在Apache的官方网站获取,当前稳定版为1.2.6。
>此处我选择1.2.6版本
解压安装过程类似Hadoop，同样放在`/usr/local/`目录下，记得修改目录名和权限。
### 2.伪分布式配置

>单机模式
>1) Hbase不使用HDFS,仅使用本地文件系统
>2) ZooKeeper与Hbase运行在同一个JVM中
> – 伪分布式模式
>1) 所有进程运行在同一个节点上,不同进程运行在不同的JVM当中
>2) 比较适合实验测试
– 完全分布式模式
1> 进程运行在多个服务器集群中
2> 分布式依赖于HDFS系统，因此布署Hbase之前一定要有一个正常工作的HDFS集群
>
首先配置根目录，进入`/usr/local/hbase/conf/`，配置 hbase-site.xml 文件
```
<configuration>  
  <property>  
    <name>hbase.rootdir</name>  
    <value>hdfs://localhost:9000/hbase</value> #此处注意端口号按需更改 
  </property>
</configuration>  
```
### 3.伪分布式运行

