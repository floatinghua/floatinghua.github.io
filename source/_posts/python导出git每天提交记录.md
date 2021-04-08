---
title: python导出git每天提交记录
date: 2020-11-16 10:37:06
keyword: python 导出git提交日志
description: 利用python导出每天的git提交日志为excel
cover: http://cdn.wutang.cc/20201116104445.png
tags:
- python
- git
categories:
- python
---

##### 前言：

> 和前面的文章是一样的，程序员是最懒的人
>
> 之前写了一个爬虫导出每天的禅道bug，现在写的这个是导出每天git的提交记录，因为懒

还是很简单

##### 准备工作

1. python3

2. git

<font color='cornflowerblue'>首先需要将git添加到环境变量path中</font>

![image-20201116105447347](http://cdn.wutang.cc/20201116105447.png)



然后你需要了解[git](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E6%9F%A5%E7%9C%8B%E6%8F%90%E4%BA%A4%E5%8E%86%E5%8F%B2)的命令，用到了下面的这些选项

![](http://cdn.wutang.cc/20201116105628.png)

然后看下代码就行了：

```python
# -*- coding: utf-8 -*-
import os
import datetime
from easyxlsx import SimpleWriter


def main():
    # 切换git的目录
    os.chdir("E:\\Project\\xxx")
    cur = str(datetime.date.today())
    path = 'C:\\Users\\Administrator\\Desktop\\公司项目\\bugs\\' + cur + '.log'
    if os.path.exists(path):
        os.remove(path)
    os.system(
        'git log --pretty=format:"%s。"  --author tangfan --after=\'2020-11-15\' >> C:\\Users\\Administrator\\Desktop\\公司项目\\bugs\\' + cur + '.log')
    # 读取file文件
    log = open(path, 'r', encoding='utf-8')
    content = log.read()
    li = content.split("。")
    wri = []
    for i in li:
        if len(i) > 0:
            li = []
            li.append('项目名称')
            li.append(i)
            li.append("1")
            li.append("唐帆")
            li.append(cur)
            li.append(cur)
            wri.append(li)
    if len(wri) > 0:
        # 导出
        SimpleWriter(wri, headers=('项目名称', '任务名称', '权重', '研发人员', "预计时间", "完成时间"),
                     bookname="C:\\Users\\Administrator\\Desktop\\公司项目\\bugs\\daily.xlsx").export()


if __name__ == '__main__':
    main()

```

