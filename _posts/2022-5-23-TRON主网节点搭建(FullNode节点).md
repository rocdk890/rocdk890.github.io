---
layout:     post
title:      TRON主网节点搭建(FullNode节点)
subtitle:   
date:       2022-05-23
author:     
header-img: img/post-bg-unix-linux.jpg
catalog: true
tags:
    - Linux
---

### TRON主网节点搭建(FullNode节点)

***

##### 系统配置

> cpu: 8核
>
> 内存: 16G
>
> 系统盘: 60G
>
> 数据盘:: 2T及以上
>
> 系统: centos 7 (64位)
>
> JDK: 1.8
>
> 带宽: 独享 100M

***

- tron快照地址: http://47.74.159.117/
- tron浏览器: https://tronscan.org/#/

***

##### 1.安装jdk

```
mkdir /root/{soft_shell,install} -p

wget --no-cookie --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F" https://download.oracle.com/otn/java/jdk/8u333-b02/2dee051a5d0647d5be72a7c0abff270e/jdk-8u333-linux-x64.tar.gz?AuthParam=1652943729_30b1f7b34474a4d8ca546e030f123e4f

mv jdk-8u333-linux-x64.tar.gz?AuthParam=1652943729_30b1f7b34474a4d8ca546e030f123e4f jdk-8u333-linux-x64.tar.gz

tar zxf jdk-8u333-linux-x64.tar.gz
mkdir -p /usr/java
mv jdk1.8.0_333/ /usr/java/

JDKPATH=`ls /usr/java`

echo " ">>/etc/profile
echo "# JDK ENV">>/etc/profile
echo "JAVA_HOME=/usr/java/${JDKPATH}">>/etc/profile
echo 'PATH=$JAVA_HOME/bin:$PATH'>>/etc/profile
echo 'CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar'>>/etc/profile
echo 'export JAVA_HOME'>>/etc/profile
echo 'export PATH'>>/etc/profile
echo 'export CLASSPATH'>>/etc/profile

source /etc/profile
```

```
java -version
```

> java version "1.8.0_333" <br>
> Java(TM) SE Runtime Environment (build 1.8.0_333-b02) <br>
> Java HotSpot(TM) 64-Bit Server VM (build 25.333-b02, mixed mode)



##### 2.下载快照

```
cd /data

wget http://47.89.178.46/backup20220519/FullNode_output-directory.tgz

tar xf FullNode_output-directory.tgz
```



##### 3.快速部署FullNode节点

```
cd /data

wget https://raw.githubusercontent.com/tronprotocol/TronDeployment/master/deploy_tron.sh -O deploy_tron.sh 

chmod +x deploy_tron.sh 

./deploy_tron.sh
```

- 安装完成后,先结束掉进程,接下来使用快照来进行同步.

```
cd ~/soft_shell
```

```
vim stop.sh
```

```
#!/bin/bash
while true; do
  pid=`ps -ef |grep FullNode.jar |grep -v grep |awk '{print $2}'`
  if [ -n "$pid" ]; then
    kill -15 $pid
    echo "The java-tron process is exiting, it may take some time, forcing the exit may cause damage to the database, please wait patiently..."
    sleep 1
  else
    echo "java-tron killed successfully!"
    break
  fi
done
```



```
chmod +x stop.sh
./stop.sh
```

##### 4.使用快照进行同步

```
cd /data/FullNode

rm -fr output-directory

mv /data/output-directory /data/FullNode/
```

```
cd ~/soft_shell/
```

```
vim start.sh
```

```
#!/bin/bash
export LD_PRELOAD="/usr/lib64/libtcmalloc.so.4"
export TCMALLOC_RELEASE_RATE=10

APP_HOME="/data/FullNode"
APP_PID=$APP_HOME/pid
START_LOG="/data/FullNode/start.log"
LOG_FILE="/data/FullNode/pid.log"

if [ -f "$APP_PID" ]; then
	PID=`cat $APP_PID`
	rm -rf $APP_PID
	kill -9 $PID > /dev/null
fi

cd $APP_HOME
nohup java -Xmx12g -XX:+HeapDumpOnOutOfMemoryError -jar $APP_HOME/FullNode.jar  -c $APP_HOME/main_net_config.conf  >> $START_LOG 2>&1 &

PID=$!
echo $PID > ${APP_PID}
if [ ! -f $APP_HOME/${LOG_FILE} ]; then
	echo "begin....." >>${LOG_FILE}
fi
```

- 如果没有装tcmalloc的话,那么执行下面这条命令再执行start.sh

```
yum install gperftools-libs -y
```

```
chmod +x start.sh

./start.sh
```



##### 5.查看同步高度

```
tail -n 100 /data/FullNode/logs/tron.log  -f|grep "Success process block"

tail -n 100 /data/FullNode/logs/tron.log  -f|grep "pushBlock block number:"
```

- 这两个都可以查看tron的同步高度

***

###### 如果使用快照同步一段时间后,发现不再同步,修改配置文件参数

```
vim /data/FullNode/main_net_config.conf
```

- 参数修改： maxTimeRatio 修改到20.0

```
vm = {
supportConstant = false
minTimeRatio = 0.0
maxTimeRatio = 20.0
saveInternalTx = false
}
```

***

本文参考:<br>
[tron波场节点搭建](https://blog.csdn.net/dandelionLYY/article/details/120097346)

[波场中文文档](https://tronprotocol.github.io/documentation-zh/developers/deployment/)

[波场TRC20钱包-搭建私有超级节点和FullNode节点](https://blog.csdn.net/u012387141/article/details/108658091)