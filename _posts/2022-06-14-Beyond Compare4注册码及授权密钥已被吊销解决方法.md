---
layout:     post
title:      Beyond Compare4注册码及授权密钥已被吊销解决方法
subtitle:   
date:       2022-06-14
author:     
header-img: img/tag-bg-o.jpg
catalog: true
tags:
    - Linux
---

### Beyond Compare4注册码及授权密钥已被吊销解决方法

---

##### windwos系统下Beyond Compare破解方式

- 错误提示： 这个授权密钥已被吊销.

  > 解决方法： 删除以下目录中的所有文件即可。（目录根据当前windows登录用户对应的目录）
  > 例如：C:\Users\Administrator\AppData\Roaming\Scooter Software\Beyond Compare 4
  > 新建文本文件黏贴如下代码，另存为.bat的所有格式的utf-8编码文件。加入windows的定时执行任务

```
@echo off
rem ******************************
rem *解决Beyond Compare 4秘钥吊销问题* 
rem ****************************** 
::注释：删除Beyond Compare 4下所有文件，不包含目录
del/f/s/q　 "C:\Users\Administrator\AppData\Roaming\Scooter Software\Beyond Compare 4\*.*"   
::注释：删除Beyond Compare 4目录
 rd/s/q       "C:\Users\Administrator\AppData\Roaming\Scooter Software\Beyond Compare 4"
```

windows + R输入：taskschd.msc，创建基本任务

##### Mac OS系统下Beyond Compare破解方式

1. 进入Beyond Compare应用程序MacOS目录下(/Applications/Beyond Compare.app/Contents/MacOS)
2. 将主启动程序BCompare重命名为BCompare.real
3. 在同级目录下新建一个脚本文件命名为BCompare，文件内容往下看
4. 给新建的文件BCompare，授权文件执行权限

**接下来，就是破解重头戏了：**

- 创建BCompare文件命令如下：
- 在这个脚本里面写如下代码，第一行是注明解释器，第二行是删除注册信息，第三行是启动真正的主程序

```
#!/bin/bash
rm "/Users/$(whoami)/Library/Application Support/Beyond Compare/registry.dat"
"`dirname "$0"`"/BCompare.real $@
```

**授权文件执行权限**

```
chmod a+x /Applications/Beyond\ Compare.app/Contents/MacOS/BCompare
```

这样我们每次打开软件的时候,都会先自动删掉注册信息,也就是永久免费试用了.



###### 原来链接

[windwos和mac下Beyond Compare破解方法笔记](https://shaohualee.com/article/976)