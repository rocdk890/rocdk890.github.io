---
layout:     post
title:      python对文件夹下xml内容批量替换、删除
subtitle:   针对要重复修改的文件.
date:       2021-06-09
author:     
header-img: img/post-bg-unix-linux.jpg
catalog: true
tags:
    - Python
    - Linux
---

```
#-*- coding:utf-8 -*-
 
#将a替换成b
 
import os
xmldir=''
savedir=''
xmllist=os.listdir(xmldir)
for xml in xmllist:
    if '.xml' in xml:
      fo=open(savedir+'/'+'new_{}'.format(xml),'w')#不要new前缀可以在这改
      print('{}'.format(xml))
      fi=open(xmldir+'/'+'{}'.format(xml),'r')
      content=fi.readlines()
      for line in content:
         line=line.replace('a','b')#例：将a替换为b
         fo.write(line)
      fo.close()
      print('替换成功')
 
#如通b为空字符串，就是删除
```