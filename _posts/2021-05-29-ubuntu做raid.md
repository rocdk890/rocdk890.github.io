---
layout:     post
title:      Ubuntu配置raid
subtitle:   
date:       2021-05-29
author:     rocdk890
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - Linux
    - Ubuntu
---

### 准备工作

> 先查看服务器上有几块磁盘,命令是 lsblk  
> 要把硬盘格式化为xfs,然后挂载到/opt/hdd_pool.

#### 1.先清掉磁盘之前的raid

`sudo mdadm --stop /dev/md127`

#### 2.再格式化准备做raid 的每一个硬盘

```
for i in `lsblk -d | grep -v NAME | grep sd |awk '{print $1}'` ; do mkfs.xfs -f /dev/$i ; done
```

#### 3.创建raid

- raid 0  

> RAID 0 即Data Stripping（数据分条技术）。整个逻辑盘的数据是被分条（stripped）分布在多个物理磁盘上，可以并行读/写，提供最快的速度，但没有冗余能力。要求至少两个磁盘。我们通过RAID 0可以获得更大的单个逻辑盘的容量，且通过对多个磁盘的同时读取获得更高的存取速度。RAID 0首先考虑的是磁盘的速度和容量，忽略了安全，只要其中一个磁盘出了问题，那么整个阵列的数据都会不保了。

`mdadm --create --verbose /dev/md127 --level=0 --raid-devices=8 /dev/sd[a-l]` 

- raid 1

> RAID 1是两组以上的N个磁盘相互作镜像，在一些多线程操作系统中能有很好的读取速度，理论上读取速度等于硬盘数量的倍数，另外写入速度有微小的降低。只要一个磁盘正常即可维持运作，可靠性最高。其原理为在主硬盘上存放数据的同时也在镜像硬盘上写一样的数据。当主硬盘（物理）损坏时，镜像硬盘则代替主硬盘的工作。因为有镜像硬盘做数据备份，所以RAID 1的数据安全性在所有的RAID级别上来说是最好的。但无论用多少磁盘做RAID 1，仅算一个磁盘的容量，是所有RAID中磁盘利用率最低的一个级别。
磁盘利用率为50％。也就是说，如果我们有两个磁盘每个500GB，总共是1TB，但在镜像中它只会显示500GB。
在镜像如果一个磁盘发生故障不会有数据丢失，因为两个磁盘中的内容相同。
读取性能会比写入性能更好。
创建 RAID 1 至少要有两个磁盘，你也可以添加更多的磁盘，磁盘数需为2，4，6，8等偶数。要添加更多的磁盘，你的系统必须有 RAID 物理适配器（硬件卡）。

`mdadm --create --verbose /dev/md127 --level=1 --raid-devices=8 /dev/sd[a-l]`

- raid 5

> RAID 5是一种储存性能、数据安全和存储成本兼顾的存储解决方案。它使用的是Disk Striping（硬盘分区）技术。RAID 5至少需要三块硬盘，RAID 5不是对存储的数据进行备份，而是把数据和相对应的奇偶校验信息存储到组成RAID5的各个磁盘上，并且奇偶校验信息和相对应的数据分别存储于不同的磁盘上。当RAID5的一个磁盘数据发生损坏后，可以利用剩下的数据和相应的奇偶校验信息去恢复被损坏的数据。RAID 5可以理解为是RAID 0和RAID 1的折衷方案。RAID 5可以为系统提供数据安全保障，但保障程度要比镜像低而磁盘空间利用率要比镜像高。RAID 5具有和RAID 0相近似的数据读取速度，只是因为多了一个奇偶校验信息，写入数据的速度相对单独写入一块硬盘的速度略慢，若使用“回写缓存”可以让性能改善不少。同时由于多个数据对应一个奇偶校验信息，RAID 5的磁盘空间利用率要比RAID 1高，存储成本相对较便宜。

`mdadm --create --verbose /dev/md127 --level=5 --raid-devices=8 /dev/sd[a-l]`

- raid 6

> 与RAID 5相比，RAID 6增加第二个独立的奇偶校验信息块。两个独立的奇偶系统使用不同的算法，数据的可靠性非常高，任意两块磁盘同时失效时不会影响数据完整性。RAID 6需要分配给奇偶校验信息更大的磁盘空间和额外的校验计算，相对于RAID 5有更大的IO操作量和计算量，其“写性能”强烈取决于具体的实现方案，因此RAID6通常不会通过软件方式来实现，而更可能通过硬件/固件方式实现。
同一数组中最多容许两个磁盘损坏。更换新磁盘后，数据将会重新算出并写入新的磁盘中。依照设计理论，RAID 6必须具备四个以上的磁盘才能生效。

`mdadm --create --verbose /dev/md127 --level=6 --raid-devices=8 /dev/sd[a-l]`

- raid 10

> RAID 10是先镜射再分区数据，再将所有硬盘分为两组，视为是RAID 0的最低组合，然后将这两组各自视为RAID 1运作.
RAID 10(又叫RAID 1+0)特点：
最少需要4块磁盘
先按RAID 0分成两组，再分别对两组按RAID 1方式镜像
兼顾冗余(提供镜像存储)和性能(数据条带形分布)
在实际应用中较为常用.

`mdadm --create --verbose /dev/md127 --level=10 --raid-devices=8 /dev/sd[a-l]`

#### 4.格式化raid

`mkfs.xfs -f /dev/md127`

#### 5.挂载到盘符

`mount /dev/md127 /opt/hdd_pool`

#### 6. 最后验证raid  

`cat /proc/mdstat`   

![img](/assets/blog_images/20210106163114.png)


可以看到图中我们已经做好了raid 0.

#### 7.最后记得一定要在 /etc/fstab 配置文件内加入自动挂载的设置:  

`blkid`  

![img](/assets/blog_images/20210106164416.png)

`echo 'UUID=e49026f0-caeb-4adc-aeab-db59d09d6e32 /opt/hdd_pool xfs defaults,nofail,discard 0 0' | sudo tee -a /etc/fstab`  

这样做raid就完成了.