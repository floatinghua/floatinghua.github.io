---
title: hexo博客同步迁移
date: 2021-04-08 16:26:17
cover: http://cdn.wutang.cc/image-20210408165705517.png
keyword: hexo 使用github博客同步迁移
description: hexo 使用github博客同步迁移
tags:
- hexo
categories: 
- hexo 
---



>当你更换工作环境的时候，博客也需要同步更新，当然你可以完全拷贝文件夹，所以这边的做法就是上传到github上，因为本身就是部署到github上的静态页面

#### 上传项目

建立hexo（随便你取名）的分支

![image-20210408163845707](http://cdn.wutang.cc/image-20210408163845707.png)

在本地盘下（位置任意）右键Git bash here，执行以下指令，把 git clone xxx.github.io 项目文件克隆到本地,**删除除了 .git 以外的所有文件夹**

![image-20210408164126648](http://cdn.wutang.cc/image-20210408164126648.png)

然后将你博客文件夹内的所有文件复制到这个xxx.github.io 中

注意自己的gitignore文件是不是如图所示，一般默认就是这种

![image-20210408164338333](http://cdn.wutang.cc/image-20210408164338333.png)



然后在 xxxx.github.io根目录下创建一个分支，并且切换到这个分支

```shell
git checkout -b hexo
```

​    

添加文件到暂存区，这个地方有个`坑`

```shell
git add --all
```



`坑:`

如果是有主题的，你上传到git上可能主题文件夹就是空的，导致你拉下来的时候，主题都不见了，可能是该子文件夹下有.git文件夹导致无法上传

删除子文件夹下.git后，依然无法提交子文件夹下的文件。

就在 xxx.github.io 根目录下执行,反正就是要多添加一次，把主题文件夹添加进去

```shell
git rm --cached themes/xxx主题

git add .

git commit -m "xxx"
```



提交

```shell
git commit -m ""
```



推送到仓库

```shell
git push --set-upstream origin hexo
```



最后记得检查下git上，主题文件夹是不是有东西，没有的就看下6里面的坑

#### 新电脑\新环境

> 首先安装npm hexo就不多介绍了



安装好之后，进入到你新建的博客目录安装模块

```shell
 npm install
 npm install hexo-deployer-git --save
 npm install hexo-generator-feed --save
 npm install hexo-generator-sitemap --save
```

然后就可以继续写博客了

#### 总结

总的来说步骤很简单，就是遇见的坑不少

