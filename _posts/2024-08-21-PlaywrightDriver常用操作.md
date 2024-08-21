---
layout:     post
title:      PlaywrightDriver常用操作
subtitle:   PlaywrightDriver常用操作
date:       2024-08-21
author:     BY
header-img: img/post-bg-desk.jpg
catalog: true
tags:
    - Playwright
    - PlaywrightDriver
---

* PlaywrightDriver 是一个基于 Playwright 的 WebDriver 实现，用于自动化 Web 浏览器。下面是一些 PlaywrightDriver 操作方式和相关案例代码：

#### 打开浏览器 ：
``` C#
    var browser = await PlaywrightDriver.CreateAsync();
```

#### 导航到 URL：
``` C#
    var testResult = await browser.GoToPage(msgInfo, new PageActionArgs
    {
        Url = $"https://news.qq.com/",
        OpenNewTab = true
    });
```

#### 定位元素：
``` C#
    var element = await browser.LocateElement(msgInfo, new ElementLocatingArgs
    {
        Selector = "input[placeholder='Search by invitation name']",
        Highlight = true
    });
```

#### 输入文本：
``` C#
    testResult = await browser.ActionOnElement(msgInfo, new ElementLocatingArgs
    {
        Selector = "input[placeholder='Search by invitation name']",
        Highlight = true
    }, new ElementActionArgs(BroswerActionEnum.InputText, task.Content.Name));
```

#### 点击元素：
``` C#
    testResult = await browser.ActionOnElement(msgInfo, new ElementLocatingArgs
    {
        Selector = "input[placeholder='Search by invitation name']",
        Highlight = true
    }, new ElementActionArgs(BroswerActionEnum.Click));
```

#### 关闭浏览器：
``` C#
    await browser.CloseAsync();
```
