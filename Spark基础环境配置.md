---
title: Spark基础环境配置
date: 2022-05-22 02:50:33
tags: Spark基础环境配置
---



##### *博主的本地环境为：Macbok pro , macOS Monterey 12.3.1*

* Mac 用户注意需要注意，虚拟机使用VMware Fusion，镜像为CentOS7.

-----



## 一、基础配置

#### **1.配置网络适配**

Mac用户只需点击VMware Fusion > 偏好设置 > 网络，并点击左下角“ + ”即可添加虚拟机上所用的子网ip（例：192.168.138.0）。

![1.png](./Spark基础环境配置/1.png)

*注：Mac用户若省略此处步骤可能会导致时间的推移而导致ping不动网络（血的教训😭）*

* 分别修改三台虚拟机的主机名

  ```shell
  # node1
  echo "node1" >/etc/hostname
  # node2
  echo "node2" >/etc/hostname
  # node3
  echo "node3" >/etc/hostname
  ```

* 更该网络配置 （node1 node2 node3）

  ```shell
  vim /etc/sysconfig/network-scripts/ifcfg-ens33(node2,node3中的IPADDR只需在151上递加即可)
  
  TYPE="Ethernet"
  PROXY_METHOD="none"
  BROWSER_ONLY="no"
  BOOTPROTO="static"# 配置BOOTPROTO=static：表示静态路由协议，可以保持IP固定
  DEFROUTE="yes"
  IPV4_FAILURE_FATAL="no"
  IPV6INIT="yes"
  IPV6_AUTOCONF="yes"
  IPV6_DEFROUTE="yes"
  IPV6_FAILURE_FATAL="no"
  IPV6_ADDR_GEN_MODE="stable-privacy"
  NAME="ens33"
  UUID="7815751b-505d-4ae2-b2d4-aa39591dc6aa"
  DEVICE="ens33"
  ONBOOT="yes"# 配置ONBOOT=yes:表示启动这块网卡
  IPADDR="192.168.138.151"# 配置IPADDR:表示虚拟机的IP地址，这里设置的IP地址要与前面IP映射配置时的IP地址一致，否则无法通过主机名找到对应IP;
  PREFIX="24"
  GATEWAY="192.168.138.2"# 配置GATEWAY:表示虚拟机网关,通常都是将IP地址最后一个位数变为2;
  DNS1="192.168.138.2"# 配置DNS1:表示域名解析器，此处采用Google提供的免费DNS服务器88.8.8(也可以设置为PC端电脑对应的DNS)。
  DOMAIN="114.114.114.114"
  IPV6_PRIVACY="no"
  ```

* 更改完成后对三台虚拟机进行重启

  ```shell
  shutdown -r 0
  ```

* 重启成功后进行一次连通外网的测试

  ```shell
  ping baidu.com -c3
  ```

#### 2.修改host映射

修改/etc/hosts映射文件，为的是将来对相关联的虚拟机操作更加便捷，其中添加相互对应的IP地址和主机名。

```shell
vim /etc/hosts

127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.138.151 node1 node1.itcast.cn
192.168.138.152 node2 node2.itcast.cn
192.168.138.153 node3 node3.itcast.cn                                 
```

新手须知：推出并保存的方式可为 ‘esc + x‘，’esc + shift+zz‘

#### 3.集群的时间同步

我们有的时候在使用虚拟机时遇到数据误差的时候，总是不清楚哪里出错，原因就在可能没有进行时间同步。在整个集群当中，时间同步充当十分重要的角色。不要存在侥幸心理，如若因为时间不同步造成了损失，即使是进行了数据备份，那么你可能无法在正确的时间将正确的数据进行备份。

```shell
ntpdate ntp5.aliyun.com
```

建议：在每次执行poweroff之后的再启动后执行。

#### 4. SSH免密钥登陆

* 在node1上生成公钥私钥

  ```shell
  ssh-keygen
  
  Generating public/private rsa key pair.
  Enter file in which to save the key (/root/.ssh/id_rsa): 
  Enter passphrase (empty for no passphrase): 
  Enter same passphrase again: 
  Your identification has been saved in /root/.ssh/id_rsa.
  Your public key has been saved in /root/.ssh/id_rsa.pub.
  The key fingerprint is:
  SHA256:QUAgFH5KBc/Erlf1JWSBbKeEepPJqMBqpWbc02/uFj8 root@master
  The key's randomart image is:
  +---[RSA 2048]----+
  | .=++oo+.o+.     |
  | . *. ..*.o .    |
  |. o.++ *.+ o     |
  |.o ++ B ...      |
  |o.=o.o .S        |
  |.*oo.. .         |
  |+  .. . o        |
  |       + E       |
  |      =o  .      |
  +----[SHA256]-----+
  ```

* 在node1上配置免密登陆node1 node2 node3

  ```shell
  ssh-copy-id master
  ssh-copy-id slave1
  ssh-copy-id slave2
  ```

-----



## 二、安装配置JDK

#### 1.创建编译环境软件所需的根目录

```shell
mkdir -p /export/server
```

#### 2.安装JDK1.8 （首先将 jdk-8u241-linux-x64.tar.gz压缩包上传到/export/server目录，并解压）

```shell
tar -zxvf jdk-8u241-linux-x64.tar.gz
```

#### 3.配置全局环境变量

```shell
vim /etc/profile

export JAVA_HOME=/export/server/jdk1.8.0_241
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```

#### 4.加载环境变量

```shell
source /etc/profile
```

#### 5.查看java版本

```shell
java -version
(base) [root@node1 ~]# java -version
java version "1.8.0_241"
Java(TM) SE Runtime Environment (build 1.8.0_241-b07)
Java HotSpot(TM) 64-Bit Server VM (build 25.241-b07, mixed mode)
```

出现以下内容则代表安装成功

#### 6.将java文件从node1中传输到node2 node3中，传输完成后，配置方法参考 3. 4.

```shell
scp -r /export/server/jdk1.8.0_241/ root@node2:/export/server/
scp -r /export/server/jdk1.8.0_241/ root@node3:/export/server/
```

#### 7. 在三台主机分别创建软连接（为了方便）

```shell
cd /export/server
ln -s jdk1.8.0_241/jdk
```

----



## 三、安装配置Hadoop

#### 1.安装Hadoop （首先将hadoop-3.3.0-Centos7-with-wnappy.tar.gz压缩包上传到/export/server目录，并解压）

```shell
tar -zxvf hadoop-3.3.0-Centos7-with-wnappy.tar.gz
```

#### 2.创建软连接

```shell
ln -s jhadoop-3.3.0/ hadoop
```

#### 3.修改配置文件

```shell
cd  /export/server/hadoop/etc/hadoop
```

* 修改Hadoop-env.sh文件 

  ```shell
  vim hadoop-env.sh
  
  #末尾添加
  export JAVA_HOME=/export/server/jdk
  
  export HDFS_NAMENODE_USER=root
  export HDFS_DATANODE_USER=root
  export HDFS_SECONDARYNAMENODE_USER=root
  export YARN_RESOURCEMANAGER_USER=root
  export YARN_NODEMANAGER_USER=root
  ```

* 修改core-site.xml文件

  * 设置默认系统

    ```shell
        <property>
            <name>fs.defaultFS</name>
            <value>hdfs://node1:8020</value>
        </property>
    ```

  * 设置hadoop本地保存数据路径

    ```
        <property>
            <name>hadoop.tmp.dir</name>
            <value>/export/data/hadoop-3.3.0</value>
        </property>
    ```

  * 设置HDFS web UI用户身份

    ```
        <property>
            <name>hadoop.http.staticuser.user</name>
            <value>root</value>
        </property>
    ```

  * 整合hive用户代理设置

    ```
        <property>
            <name>hadoop.proxyuser.root.hosts</name>
            <value>*</value>
        </property>
    
        <property>
            <name>hadoop.proxyuser.root.groups</name>
            <value>*</value>
        </property>
    ```

  * 设置文件系统垃圾桶保存时间

    ```
        <property>
            <name>fs.trash.interval</name>
            <value>1440</value>
        </property>
    ```

* 配置mapred-site.xml

  * 设置MR程序默认运行模式：yarn集群模式 local本地模式

    ```
        <property>
          <name>mapreduce.framework.name</name>
          <value>yarn</value>
        </property>
    ```

  * 设置MR程序历史服务地址

    ```
        <property>
          <name>mapreduce.jobhistory.address</name>
          <value>node1:10020</value>
        </property>
    ```

  * 设置MR程序历史服务器web端地址

    ```
        <property>
          <name>mapreduce.jobhistory.webapp.address</name>
          <value>node1:19888</value>
        </property>
    
        <property>
          <name>yarn.app.mapreduce.am.env</name>
          <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
        </property>
    
        <property>
          <name>mapreduce.map.env</name>
          <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
        </property>
    
        <property>
          <name>mapreduce.reduce.env</name>
          <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
        </property>
    ```

* 修改yarn-site.xml

  * 设置YARN集群主角色运行机器位置

    ```
        <property>
            <name>yarn.resourcemanager.hostname</name>
            <value>node1</value>
        </property>
    
        <property>
            <name>yarn.nodemanager.aux-services</name>
            <value>mapreduce_shuffle</value>
        </property>
    ```

  * 是否将对容器实施物理内存限制

    ```
        <property>
            <name>yarn.nodemanager.pmem-check-enabled</name>
            <value>false</value>
        </property>
        <property>
            <name>yarn.nodemanager.vmem-check-enabled</name>
            <value>false</value>
        </property>
        <property>
          <name>yarn.log-aggregation-enable</name>
          <value>true</value>
        </property>
    ```

  * 设置yarn历史服务器地址

    ```
        <property>
            <name>yarn.log.server.url</name>
            <value>http://node1:19888/jobhistory/logs</value>
        </property>
    ```

  * 历史日志保存的时间 7天

    ```
        <property>
          <name>yarn.log-aggregation.retain-seconds</name>
          <value>604800</value>
        </property>
    ```

#### 4. 将Hadoop文件从node1中传输到node2 node3中

```shell
scp -r /export/server/hadoop/ root@node2:/export/server/
scp -r /export/server/hadoop/ root@node3:/export/server/
```

#### 5. 三台主机分别配置全局变量

```shell
export HADOOP_HOME=/export/server/hadoop-3.3.0
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

#### 6.重新加载全局环境变量

```shell
source /etc/profile
```

#### 7.Hadoop集群启动前需要进行格式化

```shell
hdfs namenode -format
```

#### 8.启动hadoop集群

```shell
cd /export/server/hadoop/bin

start-dfs.sh
start-yarn.sh
```

#### 9.查看当前所有进程服务

```shell
(base) [root@node1 ~]# jps
3040 ResourceManager
3713 Jps
3240 NodeManager
2603 DataNode
2335 NameNode

(base) [root@node2 ~]# jps
2322 Jps
2101 SecondaryNameNode
2199 NodeManager
1978 DataNode
You have new mail in /var/spool/mail/root

(base) [root@node3 ~]# jps
2225 Jps
1972 DataNode
2101 NodeManager
You have new mail in /var/spool/mail/root
```

#### 10.进入hdfs yarn web UI页面

![2.png](./Spark基础环境配置/2.png)

![3.png](./Spark基础环境配置/3.png)

-----



## 四、安装配置zookeeper

#### 1.安装zookeeper （首先将zookeeper-3.4.10.tar.gz压缩包上传到/export/server目录，并解压）

```shell
tar -zxvf zookeeper-3.4.10.tar.gz
```

#### 2.创建软连接

```shell
ln -s zookeeper-3.4.10/ zookeeper
```

#### 3.修改配置文件

```shell
cd /export/server/zookeeper/conf/ 
```

* 将 zoo_sample.cfg 文件复制为新文件 zoo.cfg 

  ```shell
  cp zoo_sample.cfg zoo.cfg
  ```

* 配置zoo.cfg文件

  ```shell
  vim zoo.cfg
  
  #Zookeeper的数据存放目录
  dataDir=/export/server/zookeeper/zkdatas
  # 保留多少个快照
  autopurge.snapRetainCount=3
  # 日志多少小时清理一次
  autopurge.purgeInterval=1
  # 集群中服务器地址
  server.1=node1:2888:3888
  server.2=node2:2888:3888
  server.3=node3:2888:3888
  ```

#### 4.进入 /export/server/zookeeper/zkdatas 目录在此目录下创建 myid 文件，将 1 写入进去

```shell
cd /export/server/zookeeper/zkdata

touch myid

echo '1' > myid
```

#### 5.将zookeeper文件从node1中传输到node2 node3中

```shell
scp -r /export/server/zookeeper-3.4.10/ node2:$PWD
scp -r /export/server/zookeeper-3.4.10/ node3:$PWD
```

#### 6.创建软连接

```shell
ln -s zookeeper-3.4.10/ zookeeper
```

#### 7.node2 node3分别进入 /export/server/zookeeper/zkdatas 目录在此目录下创建 myid 文件，将 2 ，3 写入进去

```shell
cd /export/server/zookeeper/zkdatas/

[root@node2 zkdatas]# vim myid 
[root@node2 zkdatas]# more myid 
2

[root@node3 zkdatas]# vim myid 
[root@node3 zkdatas]# more myid 
3
```

#### 8.配置全局环境变量

```shell
vim /etc/profile

# zookeeper 环境变量
export ZOOKEEPER_HOME=/export/server/zookeeper
export PATH=$PATH:$ZOOKEEPER_HOME/bin
```

#### 9.重新加载环境变量

```shell
source /etc/profile
```

#### 10.启动zookeeper服务

* 普通启动

  ```shell
  cd  /export/server/zookeeper-3.4.10/bin 
  
  zkServer.sh start
  ```

* 编写脚本进行一键启动

  ```shell
  cd /bin
  
  vim zkall.sh
  
  #!/bin/bash
  if [ $# -eq 0 ]
  then
  echo "please input param:start stop"
  else
  if [ $1 = start  ]
  then
  for i in {1..3}
  do
  echo "${1}ing node${i}"
  ssh node${i} "source /etc/profile;/export/server/zookeeper/bin/zkServer.sh start"
  done
  fi
  
  if [ $1 = stop ]
  then
  for i in {1..3}
  do
  echo "${1}ping node${i}"
  ssh node${i} "source /etc/profile;/export/server/zookeeper/bin/zkServer.sh stop"
  done
  fi
  
  if [ $1 = status ]
  then
  for i in {1..3}
  do
  echo "node${i} status:"
  ssh node${i} "source /etc/profile;/export/server/zookeeper/bin/zkServer.sh status"
  done
  fi
  
  fi
  ```

  * 给编写的脚本赋予可执行权限

    ```
    chmod +x zkall.sh
    ```

  **注：启动时，无需进入任意文件目录下，直接执行zkall.sh start 即可**

#### 11.查看进程

```shell
(base) [root@node1 ~]# jps
4957 Jps
4911 QuorumPeerMain

(base) [root@node2 ~]# jps
2736 QuorumPeerMain
2795 Jps
You have new mail in /var/spool/mail/root

(base) [root@node3 ~]# jps
2576 Jps
2533 QuorumPeerMain
You have new mail in /var/spool/mail/root
```

