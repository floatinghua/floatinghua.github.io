---
title: picgo处理hexo博客图片问题
date: 2020-08-28 16:52:09
cover: https://gitee.com/float97/wutang-imgs/raw/master/20200908135842.png
keyword: hexo picgo 图片
description: 免费用picgo处理hexo的图片问题
tags:
- 工具
categories: 
- 工具
---

#### picgo处理hexo博客图片问题

> 免费！速度快！简单！
>
> 前提之前写博客都是先写好，发表到csdn，然后再发表到自己的博客地址，因为csdn大家都懂，所以今天解决下一直没处理的图片存储问题。

##### 效果

技术上因为用的是hexo，所以盯上了picgo，typor自己就支持所以配置起来很方便

![image-20200828165505909](https://gitee.com/float97/wutang-imgs/raw/master/20200828165506.png)

**可以看到下图粘贴图片的一瞬间就上传图片完成了，速度极快**

![image-20200828165549692](https://gitee.com/float97/wutang-imgs/raw/master/20200828165549.png)



##### 方法

首先需要你下载 [picgo](https://github.com/Molunerfinn/PicGo/releases) ， [typora](https://www.typora.io/)  应该做开发的都知道吧，一款很好用的md编辑器,然后需要你建一个码云仓库，当然你可以选择github（速度慢，容易加载失败），或者七牛云，因为我的七牛云拿去做其他事情了，所以就演示码云。

###### 步骤

安装好 先下载一个插件 搜索gitee就行

![image-20200828170010029](https://gitee.com/float97/wutang-imgs/raw/master/20200828170010.png)

建好码云仓库，要是开源的，master分支可选，等下配置到master就行

![image-20200828170153136](https://gitee.com/float97/wutang-imgs/raw/master/20200828170153.png)

**然后生成自己的令牌一定要记住令牌，因为点击了就不会再次展示了**

![image-20200828170303361](https://gitee.com/float97/wutang-imgs/raw/master/20200828170303.png)

然后打开picgo选到gitee 填入自己的仓库地址（**注意仓库地址是你选择克隆下载的那个地址后面的部分**），token信息

![image-20200828170400477](https://gitee.com/float97/wutang-imgs/raw/master/20200828170400.png)

![image-20200828170514110](https://gitee.com/float97/wutang-imgs/raw/master/20200828170514.png)

然后打开typora 偏好设置，配置好就行了

![image-20200828165505909](https://gitee.com/float97/wutang-imgs/raw/master/20200828165506.png)



##### 总结

总体来说，很简单，也很便宜，当然你可以选择其他方式，多动脑子。