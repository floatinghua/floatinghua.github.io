---
title: win10使用docker搭建jenkins实现gitee自动化部署
date: 2021-03-23 10:57:22
keyword: jenkins自动化部署
description: win10使用docker搭建jenkins实现gitee自动化部署到服务器
cover: http://cdn.wutang.cc/image-20210323150949654.png
tags:
- git
- maven
- jenkins
categories:
- 运维
---

#### 环境准备：

> 1. win10
> 2. docker
> 3. gitee
> 4. 一台服务器

##### 安装jenkins配置环境

首先你自己的需要在电脑上搭建docker的环境，这边就不介绍docker了。

接下来就是搜索镜像，安装jenkins，直接看图吧

![image-20210323110132481](http://cdn.wutang.cc/image-20210323110132481.png)



安装好后启动的时候需要指定下挂载目录，方便后续的修改和查看

```bash
docker run -u root -d -p 8080:8080 -p 50000:50000 -v F:\tmp\jenkins_data:/var/jenkins_home  -v /var/run/docker.sock:/var/run/docker.sock --name jenkins-test jenkinsci/blueocean
```



等待一会，打开浏览器 localhost:8080就可以看到以下画面

![image-20210323111450859](http://cdn.wutang.cc/image-20210323111450859.png)

然后去我们刚刚挂载的目录下找到secrets下的初始密码：

![image-20210323111551321](http://cdn.wutang.cc/image-20210323111551321.png)

登录进去初始化一会，根据你的网络状况看是否能加载出来这个页面

![image-20210323111706797](http://cdn.wutang.cc/image-20210323111706797.png)



安装推荐的插件就行，`要是没加载出来，就跳过等下去，手动安装插件`，默认安装的插件如图所示:

![image-20210323111925779](http://cdn.wutang.cc/image-20210323111925779.png)



安装完成就会让你创建一个账户,创建完成会点击重启，等待几分钟就好,登录进去就行。

![image-20210323112017754](http://cdn.wutang.cc/image-20210323112017754.png)

![image-20210323112141242](http://cdn.wutang.cc/image-20210323112141242.png)

##### 配置：

##### 进来就先进去,系统管理->插件管理->高级

将http://updates.jenkins.io/update-center.json 改为http的，记得点击下面的立即获取，等待验证一下



![image-20210323112412393](http://cdn.wutang.cc/image-20210323112412393.png)

##### gitee插件

然后就是安装所需要的插件，这边我用的是**gitee**，所以需要下载gitee的，等待安装重启

![image-20210323112628200](http://cdn.wutang.cc/image-20210323112628200.png)

##### ssh插件

**安装ssh插件**，需要上传到服务器运行项目

![image-20210323113333145](http://cdn.wutang.cc/image-20210323113333145.png)

##### maven任务插件

**安装maven任务插件**

![image-20210323115528920](http://cdn.wutang.cc/image-20210323115528920.png)



##### maven外部配置插件

**安装maven外部配置插件**



![image-20210323113818613](http://cdn.wutang.cc/image-20210323113818613.png)

#### jenkins配置

##### maven的配置

刚刚maven的插件下好后，在系统管理中就多了managed files

![image-20210323114400773](http://cdn.wutang.cc/image-20210323114400773.png)

![image-20210323114625830](http://cdn.wutang.cc/image-20210323114625830.png)

点进去后，左侧选择add 选择maven settings.xml 更改自己想要的配置就行。

##### 全局配置

maven就选择刚刚新建的xml文件，这里的jdk，是选择的jenkins默认安装的，使用docker exec 进入jenkins的容器，使用which java

查看

![image-20210323115157785](http://cdn.wutang.cc/image-20210323115157785.png)



![image-20210323115051821](http://cdn.wutang.cc/image-20210323115051821.png)

##### 后续的git和maven我都是用的自动安装，安装完成重启jenkins

![image-20210323115246088](http://cdn.wutang.cc/image-20210323115246088.png)

`但是git的自动安装完成也要配置一次，进入jenkins容器 which git`

![image-20210323145610802](http://cdn.wutang.cc/image-20210323145610802.png)

![image-20210323145627159](http://cdn.wutang.cc/image-20210323145627159.png)

##### 在全局设置中，需要添加一个gitee的令牌![image-20210323135722713](http://cdn.wutang.cc/image-20210323135722713.png)



![image-20210323134025140](http://cdn.wutang.cc/image-20210323134025140.png)

##### ssh上传到服务器配置 ,在全局配置里面的选项

![image-20210323141012656](http://cdn.wutang.cc/image-20210323141012656.png)

#### 新增maven任务

![image-20210323120033469](http://cdn.wutang.cc/image-20210323120033469.png)

##### 	选择刚刚配置的gitee地址,配置项目地址和登录gitee的账号密码

![image-20210323135940187](http://cdn.wutang.cc/image-20210323135940187.png)



##### 选择构建触发器

注意下面的gitee webhook填写的是localhost 但是我这边用了花生壳做了映射，所以我在gitee中的配置是外网的

花生壳的映射

![image-20210323141722324](http://cdn.wutang.cc/image-20210323141722324.png)

##### 构建触发器的配置

![image-20210323140046691](http://cdn.wutang.cc/image-20210323140046691.png)



##### 仓库的webhook配置

![image-20210323140213226](http://cdn.wutang.cc/image-20210323140213226.png)









##### 执行构建

![image-20210323145249767](http://cdn.wutang.cc/image-20210323145249767.png)



##### 构建后上传到服务器 执行停止和启动脚本

![image-20210323145303152](http://cdn.wutang.cc/image-20210323145303152.png)



![image-20210323145343636](http://cdn.wutang.cc/image-20210323145343636.png)

![image-20210323145404837](http://cdn.wutang.cc/image-20210323145404837.png)

`脚本注意`

start.sh 一定要加这个 不然你的脚本可能不会被执行，后台也会被杀掉

source /etc/profile
BUILD_ID=dontKillMe 

```sh
source /etc/profile
#!/bin/bash
export JAVA_HOME=/home/tools/jdk1.8.0_251
echo ${JAVA_HOME}
echo 'Start the program : mall-0.0.1-SNAPSHOT.war'
chmod 777 /root/tmp/mall-0.0.1-SNAPSHOT.war
echo '-------Starting-------'
cd /root/tmp/
source /etc/profile
BUILD_ID=dontKillMe 
nohup /home/tools/jdk1.8.0_251/bin/java -Dhudson.util.ProcessTree.disable=true -jar mall-0.0.1-SNAPSHOT.war > /dev/null 2>&1 >> test.log &
sleep 2
pid=`ps -ef |grep java|grep mall-0.0.1-SNAPSHOT.war|awk '{print $2}'`
echo 'pid: ' $pid
echo 'start success'
```



stop.sh

```sh
source /etc/profile
#!/bin/bash

echo "Stop Procedure : mall-0.0.1-SNAPSHOT.war"
pid=`ps -ef |grep java|grep mall-0.0.1-SNAPSHOT.war|awk '{print $2}'`
echo 'old Procedure pid:'$pid
if [ -n "$pid" ]
then
kill -9 $pid
fi
```

