---
layout:     post
title:      ABP Vnext Pro 中如何根据查到的数据集之后再进行分页
subtitle:   PagedResultDto 作为返回体时如何根据数据集再去做分页返回数据
date:       2023-06-28
author:     BY
header-img: img/post-bg-desk.jpg
catalog: true
tags:
    - ABP Vnext
    - 分页
    - PagedResultDto
---



#### 正常操作查询时，一般都会在执行查询时带分页条件一起做查询，如何下基于EF CORE做分页查询示例：
```C#
    public async Task<List<Roles>> GetListAsync(string roleName, int maxResultCount = 10, int skipCount = 0)
        {
            return await (await GetDbSetAsync())
                .WhereIf(roleName.IsNotNullOrWhiteSpace(), e => e.Name.Contains(roleName))
                //.Where(e => e.Enabled == 1)
                .OrderByDescending(e => e.CreationTime)
                .PageBy(skipCount, maxResultCount)
                .ToListAsync();
        }
```

#### 但是也有一种情况是查询数据时无法控制分页的话，需要在返回时才做分页的情况，可以如下示例写：
1. 先查询出数据。
    ```C#
    var devices = await _deviceManager.FindByDeviceIds(deviceIds);
    ```
2. 在基于查询后的数据做分页。
```C#
    var result = new PagedResultDto<PageDeviceOutput>();
    var items = devices.Skip(input.SkipCount).Take(input.PageSize).ToList();
    result.Items = ObjectMapper.Map<List<DeviceDto>, List<PageDeviceOutput>>(items);
```
注意：分页的返回体是基于 **PagedResultDto** 的，如果用其他则不做参考。

