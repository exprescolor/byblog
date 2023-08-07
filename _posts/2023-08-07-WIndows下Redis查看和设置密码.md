---
layout:     post
title:      WIndows下Redis查看和设置密码
subtitle:   WIndows下Redis查看和设置密码
date:       2023-08-07
author:     BY
header-img: img/post-bg-desk.jpg
catalog: true
tags:
    - Redis
---

#### 背景
* 在一直存在redis服务的情况下，如何查看其密码/端口，或者给redis设置新密码。

#### 查看/修改密码操作步骤
1. 打开服务，找到redis的服务，右键属性查看redis路径并打开。

2. 找到conf文件并打开。

3. **修改密码：** 搜索 **requirepass** ，然后在其后面就是密码，如果是之前没设置密码，注意要先取消掉注释标识符（#）才会生效。

4. 回到服务，右键redis服务，重新启动。


#### 查看/修改端口操作步骤
1. 同密码一样，只是要在conf文件中查找 **port** ，后面带有数字的（且是单独一行的）基本上就是其端口号。
2. 重启服务即可生效。

#### 结束