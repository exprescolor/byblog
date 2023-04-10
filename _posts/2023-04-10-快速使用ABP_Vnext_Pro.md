---
layout:     post
title:      快速使用ABP Vnext Pro
subtitle:   快速使用ABP Vnext Pro
date:       2023-04-10
author:     BY
header-img: img/post-bg-desk.jpg
catalog: true
tags:
    - ABP Vnext Pro
---

# 如何快速使用ABP Vnext Pro
> ABP Vnext，入门还是有点门槛的，仅从技术和时间成本这两项而定对我这种渣渣小白来说，还是有点挑战的。
> 相比Pro这个版本，前端UI是可以直接用于业务开发的，这套完善度更高；而后端代码也经过一些修改封装后，增加了一些功能和中间件的使用，更加全面（PS：作者国内的对于小白请教问题还是更加方便快捷的，23333）。

## 先决条件
> 下列不一定都需要，看个人需求，如果不用前端就是只要.NET6+、数据库、Redis即可。
* .net 6+
* nodejs 16+
* pnpm
* mysql（或其他数据库）
* redis
* rabbitmq（可选）

## 创建项目

### 安装Cli工具
[项目仓库](https://github.com/WangJunZzz/Lion.AbpPro.Cli)

```
dotnet tool install Lion.AbpPro.Cli -g
```

### 生成项目
**提供了三个模板生成**
> 考虑到需要用到网关，就直接用第二个版本。
> 
> 语句：lion.abp new abp-vnext-pro-basic -c MayMix -p Examine
* 生成源码版本
```
//lion.abp new abp-vnext-pro -c MayMix -p Examine
lion.abp new abp-vnext-pro -c 公司名称 -p 项目名称 -v 版本号(默认LastRelease)
```
* nuget 包形式的基础版本,包括 abp 自带的所有模块，已经 pro 的通知模块，数据字典模块 以及 ocelot 网关
```
lion.abp new abp-vnext-pro-basic -c 公司名称 -p 项目名称 -v 版本(默认LastRelease)
```
* nuget 包形式的基础版本,包括 abp 自带的所有模块，已经 pro 的通知模块，数据字典模块 无 ocelot 网关
```
lion.abp new abp-vnext-pro-basic-no-ocelot -c 公司名称 -p 项目名称 -v 版本(默认LastRelease)
```

### 后端
> DB、Redis等配置需要修改

* 修改 HttpApi.Host-> appsettings.json 配置
* Mysql（或其他DB） 连接字符串
* Redis 连接字符串
* RabbitMq(如果不需要启用设置为 false)
* Es 地址即可(如果没有 es 也可以运行,只是前端 es 日志页面无法使用而已，不影响后端项目启动)
* 修改 DbMigrator-> appsettings.json 数据库连接字符串
* 右键单击.DbMigrator 项目,设置为启动项目运行，按 F5(或 Ctrl + F5) 运行应用程序. 