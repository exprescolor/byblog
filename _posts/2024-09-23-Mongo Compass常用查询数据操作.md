---
layout:     post
title:      Mongo Compass常用查询操作
subtitle:   Mongo 常用查询操作
date:       2024-09-23
author:     BY
header-img: img/post-bg-desk.jpg
catalog: true
tags:
    - Mongo
    - Mongo Compass
    - 查询
---

# 在 MongoDB Compass 中可以使用正则表达式进行模糊查询。

### 数据示例：
```json
{
  "_id": "3bc22878-e***baaf8b42d3a",
  "UserName": "458****r",
  "FirstName": "+86138****12",
  "LastName": null,
  "Email": null,
  "Phone": "+861381****2",
  "Salt": "9a2b2010****439c8",
  "Password": "455c44****6afe",
  "Source": "internal",
  "ExternalId": null,
  "Type": "client",
  "Role": "user",
  "VerificationCode": "142382",
  "Verified": false,
  "CreatedTime": {
    "$date": "2024-09-21T06:21:48.389Z"
  },
  "UpdatedTime": {
    "$date": "2024-09-21T06:21:48.389Z"
  }
}
```

### 多条件1
* 怎么查询出Verified 为false，并且Phone包含"+86"或者Phone不能为null或者”“的数据？

```json
  {
  "Verified": false,
  "$or": [
    {
      "Phone": {
        "$regex": "\\+86"
      }
    },
    {
      "Phone": {
        "$ne": null
      }
    },
    {
      "Phone": {
        "$ne": ""
      }
    }
  ]
}
```

### 解释：
* "Verified": false 表示查询 Verified 字段为 false 的文档。
* "$or" 表示满足其中任一条件即可。
* "Phone": {"$regex": "\\+86"} 表示 Phone 字段中包含 "+86" 的文档。
* "Phone": {"$ne": null} 表示 Phone 字段不为 null 的文档。
* "Phone": {"$ne": ""} 表示 Phone 字段不为空字符串的文档。

---

### 多条件2
* Verified等于false，并且Phone包含+86
```json
{
  "Verified": false,
  "Phone": {
    "$regex": "\\+86"
  }
}
```
### 解释
* "Verified": false 表示查询 Verified 字段为 false 的文档。
* "Phone": {"$regex": "\\+86"} 表示 Phone 字段中包含 "+86" 的文档。