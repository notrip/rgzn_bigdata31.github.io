---
title: Spark HA & Yarn配置
date: 2022-05-22 02:45:21
tags: Spark HA & Yarn配置
---



##### *博主的本地环境为：Macbok pro , macOS Monterey 12.3.1*

-----

### 一、Spark-Standalone-HA模式

***Standalone集群是Master-Slaves架构的集群模式，和大部分的Master-Slaves结构集群一样，存在着Master单点故障的问题。 如何解决这个单点故障的问题，Spark提供了两种方案：1.基于文件系统的单点恢复(Single-Node Recovery with Local File System)–只能用于开发或测试环境。2.基于zookeeper的Standby Masters(Standby Masters with ZooKeeper)–可以用于生产环境。***

⚠️：主机中的zookeeper的版本需要喝spark版本兼容，否则会出现错误。若不兼容则重新下载与之兼容的zookeeper，并删除之前的软连接，再重新配置，最后分配给node2 node3即可。

#### 1.配置修改 spark-env.sh 文件内容

```shell
cd /export/server/spark/conf 

vim spark-env.sh

##为 83 行内容加上注释
82 # 告知Spark的master运行在哪个机器上
83 # export SPARK_MASTER_HOST=master

##文末填写以下内容
105 SPARK_DAEMON_JAVA_OPTS="-Dspark.deploy.recoveryMode=ZOOKEEPER -Dspark.deploy    .zookeeper.url=node1:2181,node2:2181,node3:2181 -Dspark.deploy.zookeeper.dir    =/spark-ha"
# spark.deploy.recoveryMode 指定HA模式 基于Zookeeper实现
# 指定Zookeeper的连接地址
# 指定在Zookeeper中注册临时节点的路径
```

#### 2.将spark-env.sh传输到node2 node3上

```shell
scp spark-env.sh node2:/export/server/spark/conf/

scp spark-env.sh node3:/export/server/spark/conf/
```

#### 3.启动集群

⚠️：在启动集群前需确保相关zookeeper和Hadoop均已启动

```shell
##首先在node1上启动master 和所有主机上的worker
cd /export/server/spark/sbin
./start-all.sh

##再到node2上启动 node2上的master作为备用master
cd /export/server/spark/sbin
./start-master.sh
```

```shell
(base) [root@node1 sbin]# jps
7393 HistoryServer
6034 NameNode
7012 NodeManager
9749 Worker
9639 Master
2603 DataNode
6699 ResourceManager
9805 Jps
4911 QuorumPeerMain

(base) [root@node2 sbin]# jps
2736 QuorumPeerMain
4130 Jps
4071 Master
3480 NodeManager
3386 SecondaryNameNode
3263 DataNode
3711 Worker
```

#### 4.访问WebUI

![7.png](./Spark-HA-Yarn配置/7.png)

![8.png](./Spark-HA-Yarn配置/8.png)

#### 5.为验证功能，我们kill掉node1中master的进程号，来模拟node1作为我们正进行操作的主机突然宕机

```shell
(base) [root@node1 ~]# kill -9 9639
(base) [root@node1 ~]# jps
7393 HistoryServer
6034 NameNode
7012 NodeManager
9940 Jps
9749 Worker
2603 DataNode
6699 ResourceManager
4911 QuorumPeerMain
```

#### 6.此刻我们访问备用master（node2）的WebUI

![9.png](./Spark-HA-Yarn配置/9.png)

> 由此可得出结论
>
> > 在Spark-Standalone-HA模式下，主备切换不会出现任何错误，在其中的一个master宕机后，也会有备用的master继续工作。

-----



### 二、Spark On YARN模式

*** spark on yarn架构有两种模式，分为Yarn-client模式和Yarn-cluster模式***

#### 1. 配置spark-env.sh

```shell
##添加

# HADOOP软件配置文件目录，读取HDFS上文件和运行YARN集群
HADOOP_CONF_DIR=/export/server/hadoop/etc/hadoop
YARN_CONF_DIR=/export/server/hadoop/etc/hadoop
```

#### 2.启动pyspark --master yarn,并测试

```shell
cd /export/server/spark/bin

Python 3.8.12 (default, Oct 12 2021, 13:49:34) 
[GCC 7.5.0] :: Anaconda, Inc. on linux
Type "help", "copyright", "credits" or "license" for more information.

22/05/28 02:06:36 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
22/05/28 02:06:39 WARN Client: Neither spark.yarn.jars nor spark.yarn.archive is set, falling back to uploading libraries under SPARK_HOME.
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /__ / .__/\_,_/_/ /_/\_\   version 3.2.0
      /_/

Using Python version 3.8.12 (default, Oct 12 2021 13:49:34)
Spark context Web UI available at http://node1:4040
Spark context available as 'sc' (master = yarn, app id = application_1653500749142_0001).
SparkSession available as 'spark'.
>>> sc.parallelize([1,2,3,4,5]).map(lambda x: x*10).collect()
[10, 20, 30, 40, 50] 
```

访问node1:8088查看：

![10.png](./Spark-HA-Yarn配置/10.png)

#### 4.验证client模式

```shell
(base) [root@node1 ~]# /export/server/spark/bin/spark-submit --master yarn --deploy-mode client --driver-memory 512m --executor-memory 512m --num-executors 3 --total-executor-cores 3 /export/server/spark/examples/src/main/python/pi.py 3

22/05/28 02:22:18 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
22/05/28 02:22:19 WARN Utils: Service 'SparkUI' could not bind on port 4040. Attempting port 4041.
22/05/28 02:22:20 WARN Client: Neither spark.yarn.jars nor spark.yarn.archive is set, falling back to uploading libraries under SPARK_HOME.
Pi is roughly 3.146480
```

#### 5.验证cluster模式

```shell
(base) [root@node1 ~]# /export/server/spark/bin/spark-submit --master yarn --deploy-mode cluster --driver-memory 512m --executor-memory 512m --num-executors 3 --total-executor-cores 3 /export/server/spark/examples/src/main/python/pi.py 3

22/05/28 02:24:00 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
22/05/28 02:24:01 WARN Client: Neither spark.yarn.jars nor spark.yarn.archive is set, falling back to uploading libraries under SPARK_HOME.
You have new mail in /var/spool/mail/root
```

> ⚠️：为什么两者的输出一个有结果一个没有结果？
>
> > 是因为clint（客户端）模式和cluster模式是有区别的：
> >
> > client模式：Driver运行在Client上，应用程序运行结果会在客户端显示，所有适合运行结果有输出的应用程序（如spark-shell）
> >
> > cluster模式：Driver程序在YARN中运行，Driver所在的机器是随机的，应用的运行结果不能在客户端显示只能通过yarn查看，所以最好运行那些将结果最终保存在外部存储介质（如HDFS、Redis、Mysql）而非stdout输出的应用程序，客户端的终端显示的仅是作为YARN的job的简单运行状况。
