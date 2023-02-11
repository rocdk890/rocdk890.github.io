---
layout:     post
title:      CentOS 7编译安装Omnicore USDT钱包
subtitle:   
date:       2023-02-11
author:     
header-img: img/post-bg-unix-linux.jpg
catalog: true
tags:
    - Linux
---

###  CentOS 7编译安装Omnicore USDT钱包

----

##### 系统配置:

> cpu: 8核
>
> 内存: 16G及以上
>
> 系统盘: 60G
>
> 数据盘: 2T及以上
>
> 系统: centos 7 (64位)
>
> 带宽: 独享 100M

---

##### 1.安装依赖

```
yum -y install nodejs git wget vim goaccess gcc gcc-c++ autoconf automake make pcre pcre-devel zlib zlib-devel libxml2 libxml2-devel openssl openssl-devel gd libjpeg libjpeg-devel libpng libpng-devel libaio libtool boost boost-devel libevent-devel
```

> 检查是否有安装*libdb4*,如果没有就执行
>
> yum -y install libdb4-*

##### 2.升级gcc版本

```
yum install centos-release-scl -y
yum install devtoolset-11-gcc* -y

scl enable devtoolset-11 bash
echo "source /opt/rh/devtoolset-11/enable" >>/etc/profile
```

##### 3.开始编译安装omnicore

```
mkdir /root/install && cd /root/install
git clone https://github.com/OmniLayer/omnicore.git

cd omnicore
./autogen.sh
./configure

make -j `cat /proc/cpuinfo | grep processor| wc -l`
make install
```

##### 4.配置文件

```
mkdir /data/omnicore/data -p
cd /data/omnicore

vim bitcoin.conf
datadir=/data/omnicore/data
testnet=0
txindex=1
rpcuser=admin
rpcpassword=mfkqxxr
daemon=1
server=1
gen=0
rpcport=8334
rpcbind=0.0.0.0:8332
rpcallowip=0.0.0.0/0
rpcconnect=127.0.0.1
timeout=50000
rpctimeout=100
```

##### 5.启动omnicore

```
omnicored --conf=/data/omnicore/bitcoin.conf
```

##### 6.停止omnicore

```
omnicore-cli stop
```

##### 7.编写监控脚本

```
mkdir /root/soft_shell

vim /root/soft_shell/check_omni.sh
#!/bin/bash

pid=`ps auxw|grep omnicored|grep -v grep|wc -l`

if [ $pid -eq 0 ];then
	echo "not running,start process....."
	omnicored --conf=/data/omnicore/bitcoin.conf > /dev/null 2 >&1 &
else
	echo "process is already running....."
fi
```

> 把监控脚本加入到crontab里,这样好在进程挂了后自动拉起来.
>
> crontab -e
>
> */3 * * * * /bin/bash /root/soft_shell/check_omni.sh

##### 8.查看同步状态

```
omnicore-cli "omni_getinfo"|grep block
```

---

- 本文参考:

  ##### [Omni全节点搭建](https://www.cnblogs.com/wf-l5201314/p/11969636.html)

- **相关链接**
  \- Omni 节点源码：https://github.com/OmniLayer/omnicore
  \- Omni 区块浏览器：[https://omniexplorer.info](https://omniexplorer.info/)
  \- Omni RPC 所以接口使用说明：https://github.com/OmniLayer/omnicore/blob/master/src/omnicore/doc/rpc-api.md

