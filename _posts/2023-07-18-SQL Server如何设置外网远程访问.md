---
layout:     post
title:      Oracle 迁移至 SQL Server
subtitle:   Oracle 迁移至 SQL Server
date:       2023-07-18
author:     BY
header-img: img/post-bg-desk.jpg
catalog: true
tags:
    - SQL Server
---

#### 步骤

[参考连接](https://www.yii666.com/article/451316.html)

#### 上述参考连接中少了一项，没有设置的话就会连接不了，请注意

**巨坑**

在文章的 **图7** 中，网络配置---协议---TCP/IP下，右键 **属性** ，然后在属性中找到 **127.0.0.1**，把这个改成公网IP，不然你始终都是只能本地连接，外网都连接不上。

最后检查防火墙是否屏蔽数据的端口，如果屏蔽要接触即可。

#### 结束