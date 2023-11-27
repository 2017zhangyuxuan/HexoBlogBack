---
title: 【骚操作】制作一个炫酷的Github个人主页
date: 2023-11-19 15:40:58
excerpt: 本期骚操作将会介绍如何制作一个炫酷的Github个人主页，妈妈再也不用担心我的介绍不够花里胡哨了（大雾）。
index_img: https://kingofdark-blog.oss-cn-beijing.aliyuncs.com/picture_backend/picture_backend/img/202311201717417.jpg
categories: 
- [计算机知识, 骚操作系列]
tags:
- Github
---

# 前言

废话不多说，直接上图。



![](https://kingofdark-blog.oss-cn-beijing.aliyuncs.com/picture_backend/picture_backend/img/202311201610533.png)


<center>
👆<em>图片来源：itgoyo 个人主页[^1]</em>
</center>


![](https://kingofdark-blog.oss-cn-beijing.aliyuncs.com/picture_backend/picture_backend/img/202311201612706.png)
<center>
👆<em>图片来源：二丫讲梵 个人主页[^2]</em>
</center>



![](https://kingofdark-blog.oss-cn-beijing.aliyuncs.com/picture_backend/picture_backend/img/202311201618499.png)
<center>
👆<em>图片来源：Hein Thant 个人主页[^3]</em>
</center>

---

怎么样，看到这些风格各异的个人主页，是不是大为震撼，~~够不够炫酷，够不够花哨~~，如果你也想包装下自己，好好打扮下个人主页，那就请继续阅读下文实操部分。

# 实操

其实修改的原理很简单，就是创建一个与自己Github账号同名的一个仓库，然后添加一个<text>README.md</text>文件，在该文件中编辑填写自己的个人介绍，就像下面提示的那样。而markdown文件支持内嵌html，因此借助一些大佬提供的开源http服务，请求对应的资源，就可以排版设计出漂亮美观的个人主页。

![创建同名仓库](https://kingofdark-blog.oss-cn-beijing.aliyuncs.com/picture_backend/picture_backend/img/202311201638069.png)

那么，上哪去找美观炫酷的模板呢？可以看看这个仓库 https://github.com/LHRUN/bubble[^5]，这个仓库提供了一个素材库链接[^4]，里面包含各种组件，可以形象生动地展示你的Github数据，像是commit，stars，pr等。此外，上面这个仓库也收集了很多Github用户的个人主页，你也可以从中挑选自己中意的排版，然后参考对应的代码进行设计。更简单粗暴的方式，就是通过fork他/她的仓库，然后将仓库名改为自己的Github名称，就可以拥有与之相同的个人主页。

# 结束语

虽然炫酷花哨的个人主页，的确能令人耳目一新，但也别忘了Github社区的初衷，多多参与学习优质的开源项目，贡献自己的知识和方案，在各种思想交流碰撞中，成长为一名Geek。切忌因小失大，沉迷折腾些边角料的事，就像这次修改自己的个人主页，没有实质性的内容和出彩项目，没有实打实的数据支撑，包装得再如何美观，都只是流于表面罢了。



[^1]: https://github.com/itgoyo
[^2]: https://github.com/eryajf
[^3]: https://github.com/IndieCoderMM

[^4]:https://bubble-awesome-profile.vercel.app/
[^5]: [Github Profile素材仓库](https://github.com/LHRUN/bubble)



