---
title: Spark local& stand-alone配置
date: 2022-05-22 02:47:40
tags: Spark local& stand-alone配置
---



##### *博主的本地环境为：Macbok pro , macOS Monterey 12.3.1*

----

## 安装配置Spark

***Spark是专为大规模数据处理而设计的快速通用的计算引擎，其提供了一个全面、统一的框架用于管理各种不同性质的数据集和数据源的大数据处理的需求，大数据开发需掌握Spark基础、SparkJob、Spark RDD、spark job部署与资源分配、Spark shuffle、Spark内存管理、Spark广播变量、Spark SQL、Spark Streaming以及Spark ML等相关知识。***

-----



## 一、Spark-local模式

***local模式是以一个单独的进程，通过其内部的多个线程来模拟整个spark***

#### 1.安装Anaconda（首先将Anaconda3-2021.05-Linux-x86_64.sh上传到/export/server目录，并运行）————（此模式只需在node1上安装即可）

```shell
sh Anaconda3-2021.05-Linux-x86_64.sh

#执行过程中需注意：
...
# 出现内容选 yes
Please answer 'yes' or 'no':'
>>> yes
...
# 出现添加路径：/export/server/anaconda3
...
[/root/anaconda3] >>> /export/server/anaconda3
PREFIX=/export/server/anaconda3
...
```

安装成功后，现推出，再重新登录终端

```shell
exit

(base) [root@node1 ~]# 
```

⚠️：出现（base）则代表安装成功！

#### 2.创建基于python3.8虚拟环境的pyspark

```shell
conda create -n pyspark python=3.8 
```

#### 3.切换到pyspark虚拟环境内 

```shell
(base) [root@node1 ~]# conda activate pyspark  
(pyspark) [root@master ~]# 
```

⚠️：出现（pyspark）则代表成功！

#### 4.在虚拟环境内安装所需包

```shell
pip install pyhive pyspark jieba -i https://pypi.tuna.tsinghua.edu.cn/simple
```

⚠️：中间过程可能会出现WARNING 不管即可

#### 5. 安装配置Spark（首先将spark-3.2.0-bin-hadoop3.2.tgz上传到/export/server目录，并解压）

```shell
tar -zxvf spark-3.2.0-bin-hadoop3.2.tgz -C /export/server/
```

#### 6.创建软连接

```shell
ln -s /export/server/spark-3.2.0-bin-hadoop3.2 /export/server/spark
```

#### 7.配置全局环境变量,并重新加载

```shell
vim /etc/profile

#JAVA_HOME							JAVA_HOME: 告知Spark Java的所在路径位置
export JAVA_HOME=/export/server/jdk
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
#HADOOP_HOME						HADOOP_HOME: 告知Spark  Hadoop的所在路径位置
export HADOOP_HOME=/export/server/hadoop-3.3.0
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
#ZOOKEEPER_HOME
export ZOOKEEPER_HOME=/export/server/zookeeper
export PATH=$PATH:$ZOOKEEPER_HOME/bin
#SPARK_HOME      				SPARK_HOME: 表示Spark安装路径
export SPARK_HOME=/export/server/spark
#HADOOP_CONF_DIR				告知Spark Hadoop的配置文件的路径位置
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
#PYSPARK_PYTHON      		PYSPARK_PYTHON: 表示Spark将要运行Python程序, python解释器的位置
export PYSPARK_PYTHON=/export/server/anaconda3/envs/pyspark/bin/python


#重新加载
source /etc/profile
```

```shell
vim .bashrc

#JAVA_HOME
export JAVA_HOME=/export/server/jdk1.8.0_241  
#PYSPARK_PYTHON
export PYSPARK_PYTHON=/export/server/anaconda3/envs/pyspark/bin/python


#重新加载
source ~/.bashrc
```

#### 8.开启

```shell
cd /export/server/anaconda3/envs/pyspark/bin/
./pyspark
```

![4.png](./Spark-local-stand-alone配置/4.png)

#### 9.查看web UI界面



#### 10.退出

```shell
 conda deactivate
```

-----



## 二、Spark-stand-alone模式

***Stand-alone模式中各个角色以独立个体进程的方式存在，一起组成Spark集群***

#### 1.安装Anaconda（首先将Anaconda3-2021.05-Linux-x86_64.sh上传到/export/server目录，并运行）————（此模式需要全部安装，因node1已部署，所以在node2 node3全部单独部署）

```shell
sh Anaconda3-2021.05-Linux-x86_64.sh

#执行过程中需注意：
...
# 出现内容选 yes
Please answer 'yes' or 'no':'
>>> yes
...
# 出现添加路径：/export/server/anaconda3
...
[/root/anaconda3] >>> /export/server/anaconda3
PREFIX=/export/server/anaconda3
...
```

安装成功后，现推出，再重新登录终端

```shell
exit

(base) [root@node1 ~]# 
```

⚠️：node2 node3 分别全部执行！！！

#### 2.把所需的全局环境变量全部传输到node2 node3

```shell
#传输 .bashrc :
scp ~/.bashrc root@node2:~/
scp ~/.bashrc root@node3:~/

#传输 profile :
scp /etc/profile/ root@node2:/etc/
scp /etc/profile/ root@node3:/etc/
```

#### 3.创建基于python3.8虚拟环境的pyspark 

```shell
conda create -n pyspark python=3.8 
```

#### 4.切换到pyspark虚拟环境内 

```shell
conda activate pyspark 

(base) [root@node2 ~]# conda activate pyspark  
(pyspark) [root@node2 ~]# 
```

#### 5.在虚拟环境内安装所需包

```shell
pip install pyhive pyspark jieba -i https://pypi.tuna.tsinghua.edu.cn/simple
```

#### 6.配置相关文件

```shell
cd /export/server/spark/conf
```

⚠️：此操作在node1上执行

* 将文件 workers.template 改名为 workers，并配置文件

  ```shell
  mv workers.template workers
  
  vim workers
  
  # localhost删除，内容追加文末：
  node1
  node2
  node3
  ```

* 将文件 spark-env.sh.template 改名为 spark-env.sh，并配置文件

  ```shell
  mv spark-env.sh.template spark-env.sh
  
  vim spark-env.sh
  
  文末追加内容：
  
  ## 设置JAVA安装目录
  JAVA_HOME=/export/server/jdk
  
  ## HADOOP软件配置文件目录，读取HDFS上文件和运行YARN集群
  HADOOP_CONF_DIR=/export/server/hadoop/etc/hadoop
  YARN_CONF_DIR=/export/server/hadoop/etc/hadoop
  
  ## 指定spark老大Master的IP和提交任务的通信端口
  export SPARK_MASTER_HOST=node1
  # 告知sparkmaster的通讯端口
  export SPARK_MASTER_PORT=7077
  # 告知spark master的 webui端口
  SPARK_MASTER_WEBUI_PORT=8080
  
  # worker cpu可用核数
  SPARK_WORKER_CORES=1
  # worker可用内存
  SPARK_WORKER_MEMORY=1g
  # worker的工作通讯地址
  SPARK_WORKER_PORT=7078
  # worker的 webui地址
  SPARK_WORKER_WEBUI_PORT=8081
  
  ## 设置历史服务器 ------将spark上运行的日志记录储存到hdfs中的sparklog中
  # 配置的意思是  将spark程序运行的历史日志 存到hdfs的/sparklog文件夹中
  SPARK_HISTORY_OPTS="-Dspark.history.fs.logDirectory=hdfs://node1:8020/sparklog/ -Dspark.history.fs.cleaner.enabled=true"
  ```

  * 此刻打开一个新的shell窗口，然后开启Hadoop

    ```shell
    start-all.sh
    ```

    ⚠️：不建议 在启动单独集群时 需要执行单独对应的.sh文件

  * 在HDFS上创建可存放历史日志的文件夹

    ```shell
    hadoop fs -mkdir /sparklog
    
    hadoop fs -chmod 777 /sparklog
    ```

* 将 spark-defaults.conf.template改为 spark-defaults.conf ，并配置文件

  ```shell
  mv spark-defaults.conf.template spark-defaults.conf
  
  vim spark-defaults.conf
  
  # 开启spark的日期记录功能
  spark.eventLog.enabled 	true
  # 设置spark日志记录的路径
  spark.eventLog.dir	 hdfs://node1:8020/sparklog/ 
  # 设置spark日志是否启动压缩
  spark.eventLog.compress 	true
  ```

* 将og4j.properties.template改为log4j.properties，并配置文件 (修改第十九行：将INFO改为WARN)

```shell
mv log4j.properties.template log4j.properties

vim log4j.properties

 18 # Set everything to be logged to the console
 19 log4j.rootCategory=WARN, console
 20 log4j.appender.console=org.apache.log4j.ConsoleAppender
 21 log4j.appender.console.target=System.err
 22 log4j.appender.console.layout=org.apache.log4j.PatternLayout
 23 log4j.appender.console.layout.ConversionPattern=%d{yy/MM/dd HH:mm:ss} %p %c{1}: %m%n
```

**此步骤的目的是让输出的日志之输出警告和错误的日志 **   *INFO表示输出所有日志*

#### 7.将Spark文件夹全部传输到node2 node3上

```shell
scp -r /export/server/spark-3.2.0-bin-hadoop3.2/ node2:$PWD
scp -r /export/server/spark-3.2.0-bin-hadoop3.2/ node3:$PWD
```

#### 8.创建软连接

```shell
ln -s /export/server/spark-3.2.0-bin-hadoop3.2 /export/server/spark
```

#### 9.重新加载环境变量

```shell
source /etc/profile
```

⚠️：因为在步骤2中已经将node1中的全局环境变量传输到node2 node3 中，所以执行即可

#### 10.启动history-server服务

```shell
cd /export/server/spark/sbin 

./start-history-server.sh

(base)[root@node1 sbin]# jps
7393 HistoryServer
6034 NameNode
7012 NodeManager
2603 DataNode
6699 ResourceManager
4911 QuorumPeerMain
7439 Jps
```

#### 11.查看 webUI

![5.png](./Spark-local-stand-alone配置/5.png)

#### 12.启动/关闭spark 的主从节点 master worker进程指令

```shell
sbin/start-all.sh 			#启动全部的master和worke    在node1上执行则默认node1为master
sbin/start-master.sh		#只启动执行主机上master
sbin/start-worker.sh		#只启动执行主机上worker
sbin/stop-all.sh				#停止所有
sbin/stop-master.sh			#只停止执行主机上的master
sbin/stop-worker.sh			#只停止执行主机上的worker
```

#### 13. 查看webUI

![6.png](./Spark-local-stand-alone配置/6.png)
