>Ubuntu16 Desktop 
----
# Hadoop
## 1. Hadoop安装
### 0. 准备
1. **SSH配置：**

使用 apt-get 获取 SSH 服务器端
```
sudo apt-get install openssh-server
```
利用 ssh-keygen 生成密钥，并将密钥加入到授权中，实现免密码登录
```
cd ~/.ssh/
ssh-keygen -t rsa
cat ./id_rsa.pub >> ./authorized_keys
```
登录本机进行测试。
```
ssh localhost 
```

> 如果需要安装分布式环境，需要交换Namenode的公钥。将Namenode的公钥复制到各个节点上，并且追加到authorized_keys中：
>
> 1. 在各个datanode节点上执行：
>
>    ```shell
>    scp hadoop@master:~/.ssh/id_rsa.pub ~/.ssh/master_rsa.pub
>    sat ~/.ssh/master_rsa.pub >> ~/.ssh/authorized_keys
>    ```

2. **Java环境配置**

ubuntu 可能预装了 openjdk，此处我安装 oracle java 可能会报错，建议先卸载 openjdk，再使用 apt-get安装 Java8
```
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
sudo apt-get install oracle-java8-set-default
```
打开`/etc`下的 environment 文件，设置 JAVA_HOME
```
JAVA_HOME = "{你的JAVA安装路径}"
```
保存后，使用`source /etc/environment`命令更新环境变量，最后检验是否安装成功
```
echo $JAVA_HOME
java -version
```

3. **新建用户**

   > 如果需要安装分布式环境，我们需要各个节点上都有一个相同的用户，也方便各个节点之间SSH的通信

   1. 新建组 ``sudo addgroup hadoop``
   2. 新建用户 ``sudo adduser –ingroup hadoop hadoop ``
   3. 用户提权 ``sudo adduser hadoop sudo``



### 1. 下载

安装[镜像](http://hadoop.apache.org/releases.html)可以在Apache的官方网站获取,当前稳定版为2.9
> 版本选择：因为我先装的HBase-1.2.6，在考虑[兼容性](http://hbase.apache.org/book.html#configuration)时只能选用Hadoop2.7版本。
### 2. 安装
我选择解压到`/usr/local`下
```
sudo tar xzf {hadoop镜像存放目录} -C /usr/local/
cd /usr/local/
sudo mv ./hadoop-2.7.6/ ./hadoop  # 将文件夹名改为hadoop
sudo chown -R {你的用户名} ./hadoop   # 修改文件权限
```
完成后进入目录下`/usr/local/hadoop`进行测试
```
./bin/hadoop version
```

## 2. 伪分布式模式
### 1. 伪分布式配置
修改``hadoop-env.sh``文件，确保java home的设置

```shell
export JAVA_HOME=${your JAVA_HOME_PATH}
```

修改配置文件`./etc/hadoop/core-site.xml`

```shell
<configuration>
        <property>
             <name>hadoop.tmp.dir</name>
             <value>/usr/local/hadoop/tmp</value>
             <description>Abase for other temporary directories.</description>
        </property>
        <property>
             <name>fs.defaultFS</name>
             <value>hdfs://localhost:9000</value>
        </property>
</configuration>
```
同样修改`./etc/hadoop/hdfs-site.xml`，这里需要确保``dfs.namenode.name.dir``和``dfs.namenode.data.dir``的文件路径存在
```
<configuration>
        <property>
             <name>dfs.replication</name>
             <value>1</value>
        </property>
        <property>
             <name>dfs.namenode.name.dir</name>
             <value>file:/usr/local/hadoop/tmp/dfs/name</value>
        </property>
        <property>
             <name>dfs.datanode.data.dir</name>
             <value>file:/usr/local/hadoop/tmp/dfs/data</value>
        </property>
</configuration>
```
> 对于分布式环境，还需要修改slave文件，填上对应节点名字

修改后执行 NameNode 的格式化:

```
./bin/hdfs namenode -format
```
接着开启NameNode和DataNode的守护进程
```
./sbin/start-dfs.sh     #stop-dfs.sh可关闭进程
```
若出现如下SSH提示，输入yes即可。如果反复要求输入ssh密码，解决方法：

1. 确定是否用ssh生成了密钥对
2. Hadoop 安装文件夹及其所属，他们的所有者是否是 hadoop 用户，如果不是，执行 ``sudo chown –R hadoop:hadoop {hadoop 安装文件夹} ``
3. 
   若都不行，输入 ``cd ~/.ssh``，然后``ssh-add –K id_rsa`` 


启动完成后，可以通过命令 jps 来判断是否成功启动，若成功启动则会列出如下进程: “NameNode”、”DataNode” 和 “SecondaryNameNode”。

成功启动后，可以访问 Web 界面 http://localhost:50070 查看 NameNode 和 Datanode 信息，还可以在线查看 HDFS 中的文件。

### 伪分布式运行
1. Word count小栗子

进入`/usr/lcoal/hadoop`目录下
```shell
./bin/hadoop jar {你要部署的jar包路径}
./bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-${your hadoop version}.jar wordcount ${your input file path} ${your output file path}
```















