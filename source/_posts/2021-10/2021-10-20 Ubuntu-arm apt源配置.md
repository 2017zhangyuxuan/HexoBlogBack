---
title: Ubuntu-arm apt源配置
date: 2021-10-20 15:32:24
excerpt: 本文主要介绍如何在M1 MacBook Air上使用Ubuntu虚拟机配置apt源。
index_img: https://kingofdark-blog.oss-cn-beijing.aliyuncs.com/picture_backend/picture_backend/hexo_img/ubuntu.jpeg
categories: 
- [计算机知识,Linux]
tags: 
- Ubuntu
---

# 配置环境

PC：M1 MacBook Air

虚拟机软件：Parallel Desktop 16.5

虚拟机操作系统：[Ubuntu Server 20.04 TLS](https://ubuntu.com/download/server/arm)



# 遇到问题

首先就是随便搜索了一篇网上教程，来配置apt源，比如这篇

https://blog.csdn.net/p1279030826/article/details/111640455

<p class='note note-info'>要注意自己的ubuntu版本和apt源设置的版本一致</p>



但是然后呢，满怀期待执行`sudo apt-get update` 却报错了。。。

![apt-get update报错 ](https://kingofdark-blog.oss-cn-beijing.aliyuncs.com/picture_backend/picture_backend/img/202110201532536.png)

当时很郁闷啊，因为之前在windows虚拟机上配置的时候都好使的，怎么到你这就拉胯了呢？

后来在网上找各种资料、解决办法，有的说是网络的问题，有的说是DNS解析的问题，但经过各种尝试都没有效果，后来无奈放弃，就此作罢。



# 解决方案

但是天无绝人之路，当我第二次再次尝试配置的时候，终于让我找到了正确答案：[解决方案](https://zongxp.blog.csdn.net/article/details/90604966?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1.no_search_link&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1.no_search_link)

重点就是后缀要加上-ports，这样对应的镜像源才是arm源，才能适配M1的mac系统。

```shell
deb http://mirrors.aliyun.com/ubuntu-ports/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu-ports/ focal main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu-ports/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu-ports/ focal-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu-ports/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu-ports/ focal-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu-ports/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu-ports/ focal-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu-ports/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu-ports/ focal-backports main restricted universe multiverse
```

​         

换源之后执行成功，nice！

![apt-get update成功](https://kingofdark-blog.oss-cn-beijing.aliyuncs.com/picture_backend/picture_backend/img/202110201533176.png)



# 小结

总的来说踩了个坑，而究其原因，还是自己对这个apt配置参数没有理解到位，仅仅只是照猫画虎，复制教程，而没有领会其中的真意。同时对这个系统架构arm 和 x86的差异没有深刻认识，不同架构之间软件肯定是需要进行适配的，没有很清楚的认识到这一点。所以呢，学海无涯，我还差得远呢~
<div align='center'>
<img src="https://kingofdark-blog.oss-cn-beijing.aliyuncs.com/picture_backend/picture_backend/img/202110251023808.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg" width="50%" height="50%" align="middle" >
</div>
