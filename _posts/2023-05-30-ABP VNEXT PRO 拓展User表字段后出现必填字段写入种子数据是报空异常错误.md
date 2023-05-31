---
layout:     post
title:      ABP VNEXT PRO 拓展User表字段后出现必填字段写入种子数据是报空异常错误
subtitle:   ABP VNEXT PRO 拓展User表字段后出现必填字段写入种子数据是报空异常错误
date:       2023-05-30
author:     BY
header-img: img/post-bg-desk.jpg
catalog: true
tags:
    - EF CORE
    - 问题
---

> 在拓展了ABP 原生user表字段后，如果之前已经做过迁移了，那么后续迁移不会出现下图错误内容；但是如果清除掉迁移记录文件、表后，重新迁移就会出现如下图错误。



![](../img/post-bg-abp05.png)