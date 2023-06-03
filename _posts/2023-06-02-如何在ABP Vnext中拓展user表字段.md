---
layout:     post
title:      如何在ABP Vnext中拓展user表字段
subtitle:   如何在ABP Vnext中拓展user表字段
date:       2023-06-02
author:     BY
header-img: img/post-bg-desk.jpg
catalog: true
tags:
    - ABP Vnext
    - 拓展表字段
---

* 从ABP官网创建下载下来的代码中，有自带的表都是封装好的，如果个别业务需要去定制，如果不是很熟悉ABP的话就很难去做这个拓展的需求。

* 官方纵使也有这个方面的文档，但是对于柴点的朋友还是有点难度的，鄙人就是哪一位，因此研究了好些时间，在此记录一下获取能帮助到其他跟我一样的朋友。

1. 创建一个类，用于放置需要拓展的字段属性，根据规范这部分属于聚合根，放在Domain下即可，不需要去继承IdentityUser，血一样的教训····
2. 在 EF CORE下的 **项目名称+EfCoreEntityExtensionMappings** 对用户名新增字段。
    ``` C#
    ObjectExtensionManager.Instance
                .MapEfCoreProperty<IdentityUser, string>(
                    nameof(MyUser.Avatar),
                    (entityBuilder, propertyBuilder) =>
                    {
                        propertyBuilder.HasMaxLength(MyUser.MaxAvatarLength);
                    }
                );
    ```

3. 将第二步中的配置生效，则需要在EF CORE下的 **项目名称+EntityFrameworkCoreModule** 类中，找到 **PreConfigureServices** 方法，加上如下代码即可，但是一般都是默认都自带配置了的，只是出于谨慎最好检查一下。
    > 将上一步配置的内容，在模块中加载

    ```C#
    public override void PreConfigureServices(ServiceConfigurationContext context)
        {
            项目名称+EfCoreEntityExtensionMappings.Configure();

            base.PreConfigureServices(context);
        }
    ```

4. 配置 **项目名称+MigrationsDbContextFactory**类中的 **CreateDbContext**。
    > 同上一步，在DbContextFactory中进行同样配置，一些情况下我们需要通过DbContextFactory来获取DbContext，如不配置此项，将会导致获取的DbContext中内容不一致。
    ```C#
    public ExamineDbContext CreateDbContext(string[] args)
        {
            ExamineEfCoreEntityExtensionMappings.Configure();// 注意，只是这一行

            var configuration = BuildConfiguration();

            var builder = new DbContextOptionsBuilder<ExamineDbContext>()
                .UseSqlServer(configuration.GetConnectionString("Default")/*, MySqlServerVersion.LatestSupportedServerVersion*/);

            return new ExamineDbContext(builder.Options);
        }
    ```

5. 配置应用层Dto扩展
    > 因头像Avatar信息是我们追加给IdentityUser的，Abp Zero中内置的User相关Dto是没有此项内容的，我们需要对这些Dto做一些添加属性的操作。在Application.Contracts层，新建一个名为MyAppDtoExtensions的类（这里本人使用的ABP Vnext Pro在生成的代码中已经带有了，应该是官方自带的，所有我这里不需要去新建），当然类名称随意。

    ```C#
    public static class ExamineDtoExtensions
    {
        private static readonly OneTimeRunner OneTimeRunner = new OneTimeRunner();

        public static void Configure()
        {
            OneTimeRunner.Run(() =>
            {
                /* You can add extension properties to DTOs
                 * defined in the depended modules.
                 *
                 * Example:
                 *
                 * ObjectExtensionManager.Instance
                 *   .AddOrUpdateProperty<IdentityRoleDto, string>("Title");
                 *
                 * See the documentation for more:
                 * https://docs.abp.io/en/abp/latest/Object-Extensions
                 */

                ObjectExtensionManager.Instance
                .AddOrUpdateProperty<string>(
                    new[]
                    {
                        typeof(IdentityUserDto),
                        typeof(IdentityUserCreateDto),
                        typeof(IdentityUserUpdateDto),
                        typeof(ProfileDto),
                        typeof(UpdateProfileDto)
                    },
                    "Avatar"
                );
            });
        }
    }
    ```

6. 在EF CORE模块下的类ApplicationContractsModule中进行配置
    > 为了是上一步配置生效，还需要在 **项目名称+ApplicationContractsModule**中进行如下配置：
    
    ```C#
    public override void PreConfigureServices(ServiceConfigurationContext context)
    {
        ExamineDtoExtensions.Configure();

    }
    ```