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

### 创建数据库
> PL/SQL 登录sys账户执行下方语句即可
1. 创建表空间
    ``` SQL
    ---创建表空间（自增长）
    create tablespace MayMixExamineDB
    datafile 'E:\lld\Files\DB_Documents\MayMixExamineDB.dbf' 
    size 3200M autoextend on next 5M maxsize unlimited; 
    ```
2. 创建用户
    ``` SQL
    --创建用户和密码，并指向表空间
    CREATE USER C##MAYMIXEXAMINEDB IDENTIFIED BY MAYMIXEXAMINEDB2023--"c##"是12c的新特性，不加会报错无法执行，登录也是如此
    PROFILE DEFAULT
    DEFAULT TABLESPACE MayMixExamineDB 
    ACCOUNT UNLOCK;
    ```
3. 角色赋予用户
   ```
    -- 将角色赋予用户
    GRANT DBA TO C##MAYMIXEXAMINEDB;--"c##"是12c的新特性，不加会报错无法执行，登录也是如此
   ```

4. PL/SQL 使用新创建好的账户登录即可。



### 后端代码
> DB、Redis等配置需要修改

1. 修改HttpApi.Host-> appsettings.json 配置，其中包括DB、redis、Hangfire，说明一下Hangfire密码是要设置同redisd 的密码一样的，具体为何，还需要深入学习一下Hangfire。
2. 修改 DbMigrator-> appsettings.json 数据库连接字符串
3. 修改连接的DB引用，如果是用的mysql则不需要修改，否则就需要从NuGet添加对应使用的DB引用，如Oracle等。（最好在操作前，先找到原先DB引用并移除，这样就知道哪里有需要更新代码了）
    > 移除MySQL引用，EntityFrameworkCore、FreeSqlRepository 两个下的引用。
    > 
    > ExamineMigrationsDbContextFactory.cs -> (修改 UseMySQL -> UseOracle) -> 注释掉MySqlServerVersion.LatestSupportedServerVersion

    >ExamineEntityFrameworkCoreModule -> ( typeof(AbpEntityFrameworkCoreMySQLModule) -> typeof(AbpEntityFrameworkCoreOracleModule) )

    > Migrations -> GlobalUsings.cs ，注释掉global using MySqlConnector; 将global using Volo.Abp.EntityFrameworkCore.MySQL 修改为global using Volo.Abp.EntityFrameworkCore.Oracle;;

4. 重新生成整个解决方案，通过说明修改之后没有错误了。
5. 将******.EntityFrameworkCore.DbMigrations–Migrations下的所有文件全部删掉，然后执行迁移
6. VS 搜索栏，输入**程序包控制器控制台** ，然后在控制台上方选择 EntityFrameworkCore 为默认项目
7. dotnet ef migrations add xxxx(名称)  ----pro作者提供的语句，但是是不能使用的，输入后会生成一堆乱码，改用**add-migration xxx**（名称），就可以了
8. 生成文件成功后，再执行 **update-database**，提示成功就好了，这时候去数据库中就能看到表
9. 将.DbMigrator 项目设为启动项，按 F5(或 Ctrl + F5) 运行应用程序，成功后
    > 不出意外的话就要出意外了。

    > PL/SQL 登录对应账户后，找到任意创建的ABP...表，打开表，然后竟然提示**表或者试图不存在**。

    > 啊啊啊啊，随机抽查了一些键、表都是这样在 Tables文件夹下看到的，怎么会找不到或者不存在呢？

    > 原因就是因为Oracle对大小写敏感，通常在创建和查询时对名称数据库会自动转为大写，但语句中有引号时会按引号中的内容保留。

    > 所以就是在项目代码中EF 生成的表、字段、键名称信息时，生成的SQL代码是带有 " 名称 " 的情况导致的。

    > 这种情况下虽然创建表成功了，但是不能通过正常的SQL语句直接查询表等信息，凡是涉及到**表、字段、键名称**的都要用双引号括上才能找到对应的表、字段、键等信息。

    > 解决办法有两种，一：改源码，把实体表的名称全部手动改为大写，从源头上解决，但是我不会。二：在代码种改，EF **下的ExamineDbContext** 类下 **的OnModelCreating** 方法加上如下代码：

    ``` C#
            #region 表、字段、键名称转大写

            // 循环所有表名称，并转换大写
            foreach (var entityType in builder.Model.GetEntityTypes())
            {
                int num = entityType.Name.LastIndexOf('.');
                string tableName = entityType.Name.Substring(num + 1);
                entityType.SetTableName($"ABP{tableName.ToUpper()}");
            }

            // 字段
            foreach (var property in builder.Model.GetEntityTypes().SelectMany(e => e.GetProperties()))
            {
                property.SetColumnName(property.Name.ToUpper());
            }

            // 键
            foreach (var index in builder.Model.GetEntityTypes().SelectMany(e => e.GetIndexes()))
            {
                if (index?.Name != null)
                {
                    index.SetDatabaseName(index.Name.ToUpper());
                }
            }

            #endregion 表、字段、键名称转大写
    ```

    > 但是，用了第二种方法后，生成的表等等是大写了，结果运行 **.DbMigrator** 又提示表或视图不存在（代码报错），啊啊啊啊啊

    > 试过官方生成的项目，同样转换了大写后，报的一样的错误，所以定位就是因为改大写导致这个错误，目前这个按时不会，搁置··········

    > 这里注意一下官方生成的项目生成迁移文件时，要在EF下 **右键** -> **在终端中打开** 才能正确执行生成文件脚本（ **dotnet ef migrations add xxx(名称)** ），更新语句是 **dotnet ef database update** ； 若要撤销此操作（生成文件）则使用 **ef migrations remove** 。

10. 移除 **.FreeSqlRepository** 下的 FreeSQL 的引用，重新添加nuget 引用，搜索 **FreeSql.Provider.Oracle** 安装；再修改 **ExamineFreeSqlModule.cs** 下的DataType引用到Oracle即可运行成功(基于迁移时产生大小写的方案，SQL 查询表时要带 双引号 才能查到表)。
    > 这里需要注意一下，在还没做业务表映射、业务服务编写时，使用 **add-migration xxx** 是没问题，但是当添加表映射后，要生成文件时，是要使用 **dotnet ef migrations add xxx(名称)** 才可以的，如果使用前一个脚本语句会报错无法生成文件；后续更新也是用的  dotnet ef····脚本语句

> 这里补充一下，数据迁移流程：apb pro 项目中，在 EF 下 右键打开终端，使用  **dotnet ef migrations add xxx(名称)** ， 然后 **dotnet ef database update**，最后执行 **.DbMigrator**（作用是写入种子数据），这样才避免控制台输出一些表或视图不存在的错误提示（原因未知）。
   
#### 自定义表映射实体
> 在 EF 下的 **ExamineDbContext.cs**  中，找到 **OnModelCreating**  方法，在配置表映射部分中增加 **b.HasKey(x => x.字段名称);** 即可。

### 后端 for Oracle 10g

#### 重命名超出长度的名称：
> Oracle的表、键长度都有规定，比几个常用的DB的长度都要短，所以做数据迁移的时候，就会报字段长度过长的错误提示，想到的办法就是改对应的表、键名称长度。

> 只能一次生成后，把提示到过长的名称重新命名，达到规范长度范围内，这里就是要一次有一次的启动、修改了。

在 EF 下，新增如下类：
```C#
using Lion.AbpPro.DataDictionaryManagement;
using Lion.AbpPro.DataDictionaryManagement.DataDictionaries.Aggregates;
using Lion.AbpPro.NotificationManagement;
using Lion.AbpPro.NotificationManagement.Notifications.Aggregates;
using Volo.Abp.AuditLogging.EntityFrameworkCore;
using Volo.Abp.BackgroundJobs.EntityFrameworkCore;
using Volo.Abp.EntityFrameworkCore.Modeling;
using Volo.Abp.FeatureManagement.EntityFrameworkCore;
using Volo.Abp.Identity.EntityFrameworkCore;
using Volo.Abp.PermissionManagement.EntityFrameworkCore;
using Volo.Abp.SettingManagement.EntityFrameworkCore;
using Volo.Abp.TenantManagement.EntityFrameworkCore;
using Volo.Abp.Users.EntityFrameworkCore;

namespace MM.Examine.EntityFrameworkCore;

public static class BasicManagementOracleDbContextModelCreatingExtensions
{
    public static void ConfigurePermissionManagementForOracle(this ModelBuilder builder)
    {
        Check.NotNull(builder, nameof(builder));

        builder.Entity<PermissionGrant>(b =>
        {
            b.ToTable(AbpPermissionManagementDbProperties.DbTablePrefix + "PermissionGrants", AbpPermissionManagementDbProperties.DbSchema);
            b.ConfigureByConvention();
            b.Property(x => x.Name).HasMaxLength(PermissionDefinitionRecordConsts.MaxNameLength).IsRequired();
            b.Property(x => x.ProviderName).HasMaxLength(PermissionGrantConsts.MaxProviderNameLength).IsRequired();
            b.Property(x => x.ProviderKey).HasMaxLength(PermissionGrantConsts.MaxProviderKeyLength).IsRequired();
            b.HasIndex(x => new { x.TenantId, x.Name, x.ProviderName, x.ProviderKey }).IsUnique().HasDatabaseName("IX_PermissionGrant_UnionKey");
            b.ApplyObjectExtensionMappings();
        });

        builder.Entity<PermissionGroupDefinitionRecord>(b =>
        {
            b.ToTable(AbpPermissionManagementDbProperties.DbTablePrefix + "PermissionGroups", AbpPermissionManagementDbProperties.DbSchema);

            b.ConfigureByConvention();

            b.Property(x => x.Name).HasMaxLength(PermissionGroupDefinitionRecordConsts.MaxNameLength).IsRequired();
            b.Property(x => x.DisplayName).HasMaxLength(PermissionGroupDefinitionRecordConsts.MaxDisplayNameLength).IsRequired();

            b.HasIndex(x => new { x.Name }).IsUnique().HasDatabaseName("IX_PermissionGroups_Key");

            b.ApplyObjectExtensionMappings();
        });

        builder.Entity<PermissionDefinitionRecord>(b =>
        {
            b.ToTable(AbpPermissionManagementDbProperties.DbTablePrefix + "Permissions", AbpPermissionManagementDbProperties.DbSchema);

            b.ConfigureByConvention();

            b.Property(x => x.GroupName).HasMaxLength(PermissionGroupDefinitionRecordConsts.MaxNameLength).IsRequired();
            b.Property(x => x.Name).HasMaxLength(PermissionDefinitionRecordConsts.MaxNameLength).IsRequired();
            b.Property(x => x.ParentName).HasMaxLength(PermissionDefinitionRecordConsts.MaxNameLength);
            b.Property(x => x.DisplayName).HasMaxLength(PermissionDefinitionRecordConsts.MaxDisplayNameLength).IsRequired();
            b.Property(x => x.Providers).HasMaxLength(PermissionDefinitionRecordConsts.MaxProvidersLength);
            b.Property(x => x.StateCheckers).HasMaxLength(PermissionDefinitionRecordConsts.MaxStateCheckersLength);

            b.HasIndex(x => new { x.Name }).IsUnique().HasDatabaseName("IX_Permissions_Name_Key");
            b.HasIndex(x => new { x.GroupName }).HasDatabaseName("IX_Permission_Groups_Key");

            b.ApplyObjectExtensionMappings();
        });

        builder.TryConfigureObjectExtensions<PermissionManagementDbContext>();
    }

    public static void ConfigureSettingManagementForOracle(this ModelBuilder builder)
    {
        Check.NotNull(builder, nameof(builder));

        if (builder.IsTenantOnlyDatabase())
        {
            return;
        }

        builder.Entity<Setting>(b =>
        {
            b.ToTable(AbpSettingManagementDbProperties.DbTablePrefix + "Settings", AbpSettingManagementDbProperties.DbSchema);

            b.ConfigureByConvention();

            b.Property(x => x.Name).HasMaxLength(SettingConsts.MaxNameLength).IsRequired();

            if (builder.IsUsingOracle())
            {
                SettingConsts.MaxValueLengthValue = 2000;
            }

            b.Property(x => x.Value).HasMaxLength(SettingConsts.MaxValueLengthValue).IsRequired();

            b.Property(x => x.ProviderName).HasMaxLength(SettingConsts.MaxProviderNameLength);
            b.Property(x => x.ProviderKey).HasMaxLength(SettingConsts.MaxProviderKeyLength);

            b.HasIndex(x => new { x.Name, x.ProviderName, x.ProviderKey }).IsUnique(true).HasDatabaseName("IX_Settings_Union_Key");

            b.ApplyObjectExtensionMappings();
        });

        builder.TryConfigureObjectExtensions<SettingManagementDbContext>();
    }

    public static void ConfigureBackgroundJobsForOracle(this ModelBuilder builder)
    {
        Check.NotNull(builder, nameof(builder));

        if (builder.IsTenantOnlyDatabase())
        {
            return;
        }

        builder.Entity<BackgroundJobRecord>(b =>
        {
            b.ToTable(AbpBackgroundJobsDbProperties.DbTablePrefix + "BackgroundJobs", AbpBackgroundJobsDbProperties.DbSchema);

            b.ConfigureByConvention();

            b.Property(x => x.JobName).IsRequired().HasMaxLength(BackgroundJobRecordConsts.MaxJobNameLength);
            b.Property(x => x.JobArgs).IsRequired().HasMaxLength(BackgroundJobRecordConsts.MaxJobArgsLength);
            b.Property(x => x.TryCount).HasDefaultValue(0);
            b.Property(x => x.NextTryTime);
            b.Property(x => x.LastTryTime);
            b.Property(x => x.IsAbandoned).HasDefaultValue(false);
            b.Property(x => x.Priority).HasDefaultValue(BackgroundJobPriority.Normal);

            b.HasIndex(x => new { x.IsAbandoned, x.NextTryTime }).HasDatabaseName("IX_BackgroundJobs_Union_Key");

            b.ApplyObjectExtensionMappings();
        });

        builder.TryConfigureObjectExtensions<BackgroundJobsDbContext>();
    }

    public static void ConfigureAuditLoggingForOracle(this ModelBuilder builder)
    {
        Check.NotNull(builder, nameof(builder));

        builder.Entity<AuditLog>(b =>
        {
            b.ToTable(AbpAuditLoggingDbProperties.DbTablePrefix + "AuditLogs", AbpAuditLoggingDbProperties.DbSchema);

            b.ConfigureByConvention();

            b.Property(x => x.ApplicationName).HasMaxLength(AuditLogConsts.MaxApplicationNameLength).HasColumnName(nameof(AuditLog.ApplicationName));
            b.Property(x => x.ClientIpAddress).HasMaxLength(AuditLogConsts.MaxClientIpAddressLength).HasColumnName(nameof(AuditLog.ClientIpAddress));
            b.Property(x => x.ClientName).HasMaxLength(AuditLogConsts.MaxClientNameLength).HasColumnName(nameof(AuditLog.ClientName));
            b.Property(x => x.ClientId).HasMaxLength(AuditLogConsts.MaxClientIdLength).HasColumnName(nameof(AuditLog.ClientId));
            b.Property(x => x.CorrelationId).HasMaxLength(AuditLogConsts.MaxCorrelationIdLength).HasColumnName(nameof(AuditLog.CorrelationId));
            b.Property(x => x.BrowserInfo).HasMaxLength(AuditLogConsts.MaxBrowserInfoLength).HasColumnName(nameof(AuditLog.BrowserInfo));
            b.Property(x => x.HttpMethod).HasMaxLength(AuditLogConsts.MaxHttpMethodLength).HasColumnName(nameof(AuditLog.HttpMethod));
            b.Property(x => x.Url).HasMaxLength(AuditLogConsts.MaxUrlLength).HasColumnName(nameof(AuditLog.Url));
            b.Property(x => x.HttpStatusCode).HasColumnName(nameof(AuditLog.HttpStatusCode));

            b.Property(x => x.Comments).HasMaxLength(AuditLogConsts.MaxCommentsLength).HasColumnName(nameof(AuditLog.Comments));
            b.Property(x => x.ExecutionDuration).HasColumnName(nameof(AuditLog.ExecutionDuration));
            b.Property(x => x.ImpersonatorTenantId).HasColumnName(nameof(AuditLog.ImpersonatorTenantId));
            b.Property(x => x.ImpersonatorUserId).HasColumnName(nameof(AuditLog.ImpersonatorUserId));
            b.Property(x => x.ImpersonatorTenantName).HasMaxLength(AuditLogConsts.MaxTenantNameLength).HasColumnName(nameof(AuditLog.ImpersonatorTenantName));
            b.Property(x => x.ImpersonatorUserName).HasMaxLength(AuditLogConsts.MaxUserNameLength).HasColumnName(nameof(AuditLog.ImpersonatorUserName));
            b.Property(x => x.UserId).HasColumnName(nameof(AuditLog.UserId));
            b.Property(x => x.UserName).HasMaxLength(AuditLogConsts.MaxUserNameLength).HasColumnName(nameof(AuditLog.UserName));
            b.Property(x => x.TenantId).HasColumnName(nameof(AuditLog.TenantId));
            b.Property(x => x.TenantName).HasMaxLength(AuditLogConsts.MaxTenantNameLength).HasColumnName(nameof(AuditLog.TenantName));

            b.HasMany(a => a.Actions).WithOne().HasForeignKey(x => x.AuditLogId).HasConstraintName("IX_Action_HasForeignKey").IsRequired();
            b.HasMany(a => a.EntityChanges).WithOne().HasForeignKey(x => x.AuditLogId).HasConstraintName("IX_EntityChange_ForeignKey").IsRequired();

            b.HasIndex(x => new { x.TenantId, x.ExecutionTime }).HasDatabaseName("IX_AuditLogs_Union_Key");
            b.HasIndex(x => new { x.TenantId, x.UserId, x.ExecutionTime }).HasDatabaseName("IX_AuditLogs_Union_Name_Key");

            b.ApplyObjectExtensionMappings();
        });

        builder.Entity<AuditLogAction>(b =>
        {
            b.ToTable(AbpAuditLoggingDbProperties.DbTablePrefix + "AuditLogActions", AbpAuditLoggingDbProperties.DbSchema);

            b.ConfigureByConvention();

            b.Property(x => x.AuditLogId).HasColumnName(nameof(AuditLogAction.AuditLogId));
            b.Property(x => x.ServiceName).HasMaxLength(AuditLogActionConsts.MaxServiceNameLength).HasColumnName(nameof(AuditLogAction.ServiceName));
            b.Property(x => x.MethodName).HasMaxLength(AuditLogActionConsts.MaxMethodNameLength).HasColumnName(nameof(AuditLogAction.MethodName));
            b.Property(x => x.Parameters).HasMaxLength(AuditLogActionConsts.MaxParametersLength).HasColumnName(nameof(AuditLogAction.Parameters));
            b.Property(x => x.ExecutionTime).HasColumnName(nameof(AuditLogAction.ExecutionTime));
            b.Property(x => x.ExecutionDuration).HasColumnName(nameof(AuditLogAction.ExecutionDuration));

            b.HasIndex(x => new { x.AuditLogId }).HasDatabaseName("IX_AuditLogActions_Key");
            b.HasIndex(x => new { x.TenantId, x.ServiceName, x.MethodName, x.ExecutionTime }).HasDatabaseName("IX_AuditLogActions_Union_Key");

            b.ApplyObjectExtensionMappings();
        });

        builder.Entity<EntityChange>(b =>
        {
            b.ToTable(AbpAuditLoggingDbProperties.DbTablePrefix + "EntityChanges", AbpAuditLoggingDbProperties.DbSchema);

            b.ConfigureByConvention();

            b.Property(x => x.EntityTypeFullName).HasMaxLength(EntityChangeConsts.MaxEntityTypeFullNameLength).IsRequired().HasColumnName(nameof(EntityChange.EntityTypeFullName));
            b.Property(x => x.EntityId).HasMaxLength(EntityChangeConsts.MaxEntityIdLength).IsRequired().HasColumnName(nameof(EntityChange.EntityId));
            b.Property(x => x.AuditLogId).IsRequired().HasColumnName(nameof(EntityChange.AuditLogId));
            b.Property(x => x.ChangeTime).IsRequired().HasColumnName(nameof(EntityChange.ChangeTime));
            b.Property(x => x.ChangeType).IsRequired().HasColumnName(nameof(EntityChange.ChangeType));
            b.Property(x => x.TenantId).HasColumnName(nameof(EntityChange.TenantId));

            b.HasMany(a => a.PropertyChanges).WithOne().HasForeignKey(x => x.EntityChangeId).HasConstraintName("IX_EntityChangeId_Key");

            b.HasIndex(x => new { x.AuditLogId }).HasDatabaseName("IX_EntityChanges_Key");
            b.HasIndex(x => new { x.TenantId, x.EntityTypeFullName, x.EntityId }).HasDatabaseName("IX_EntityChanges_Union_Key");

            b.ApplyObjectExtensionMappings();
        });

        builder.Entity<EntityPropertyChange>(b =>
        {
            b.ToTable(AbpAuditLoggingDbProperties.DbTablePrefix + "EntityPropertyChanges", AbpAuditLoggingDbProperties.DbSchema);

            b.ConfigureByConvention();

            b.Property(x => x.NewValue).HasMaxLength(EntityPropertyChangeConsts.MaxNewValueLength).HasColumnName(nameof(EntityPropertyChange.NewValue));
            b.Property(x => x.PropertyName).HasMaxLength(EntityPropertyChangeConsts.MaxPropertyNameLength).IsRequired().HasColumnName(nameof(EntityPropertyChange.PropertyName));
            b.Property(x => x.PropertyTypeFullName).HasMaxLength(EntityPropertyChangeConsts.MaxPropertyTypeFullNameLength).IsRequired()
                .HasColumnName(nameof(EntityPropertyChange.PropertyTypeFullName));
            b.Property(x => x.OriginalValue).HasMaxLength(EntityPropertyChangeConsts.MaxOriginalValueLength).HasColumnName(nameof(EntityPropertyChange.OriginalValue));

            b.HasIndex(x => new { x.EntityChangeId }).HasDatabaseName("IX_EntityPropertyChanges_Key");

            b.ApplyObjectExtensionMappings();
        });

        builder.TryConfigureObjectExtensions<AbpAuditLoggingDbContext>();
    }

    public static void ConfigureIdentityForOracle(this ModelBuilder builder)
    {
        Check.NotNull(builder, nameof(builder));

        builder.Entity<IdentityUser>(b =>
        {
            b.ToTable(AbpIdentityDbProperties.DbTablePrefix + "Users", AbpIdentityDbProperties.DbSchema);

            b.ConfigureByConvention();
            b.ConfigureAbpUser();

            b.Property(u => u.NormalizedUserName).IsRequired()
                .HasMaxLength(IdentityUserConsts.MaxNormalizedUserNameLength)
                .HasColumnName(nameof(IdentityUser.NormalizedUserName));
            b.Property(u => u.NormalizedEmail).IsRequired()
                .HasMaxLength(IdentityUserConsts.MaxNormalizedEmailLength)
                .HasColumnName(nameof(IdentityUser.NormalizedEmail));
            b.Property(u => u.PasswordHash).HasMaxLength(IdentityUserConsts.MaxPasswordHashLength)
                .HasColumnName(nameof(IdentityUser.PasswordHash));
            b.Property(u => u.SecurityStamp).IsRequired().HasMaxLength(IdentityUserConsts.MaxSecurityStampLength)
                .HasColumnName(nameof(IdentityUser.SecurityStamp));
            b.Property(u => u.TwoFactorEnabled).HasDefaultValue(false)
                .HasColumnName(nameof(IdentityUser.TwoFactorEnabled));
            b.Property(u => u.LockoutEnabled).HasDefaultValue(false)
                .HasColumnName(nameof(IdentityUser.LockoutEnabled));

            b.Property(u => u.IsExternal).IsRequired().HasDefaultValue(false)
                .HasColumnName(nameof(IdentityUser.IsExternal));

            b.Property(u => u.AccessFailedCount)
                .If(!builder.IsUsingOracle(), p => p.HasDefaultValue(0))
                .HasColumnName(nameof(IdentityUser.AccessFailedCount));

            b.HasMany(u => u.Claims).WithOne().HasForeignKey(uc => uc.UserId).HasConstraintName("IX_Claims_UserId_Key").IsRequired();
            b.HasMany(u => u.Logins).WithOne().HasForeignKey(ul => ul.UserId).HasConstraintName("IX_Logins_UserId_Key").IsRequired();
            b.HasMany(u => u.Roles).WithOne().HasForeignKey(ur => ur.UserId).HasConstraintName("IX_Roles_UserId_Key").IsRequired();
            b.HasMany(u => u.Tokens).WithOne().HasForeignKey(ur => ur.UserId).HasConstraintName("IX_Tokens_UserId_Key").IsRequired();
            b.HasMany(u => u.OrganizationUnits).WithOne().HasForeignKey(ur => ur.UserId).HasConstraintName("IX_Org_UserId_ForeignKey").IsRequired();

            b.HasIndex(u => u.NormalizedUserName).HasDatabaseName("IX_NormalizedUserName_Key");
            b.HasIndex(u => u.NormalizedEmail).HasDatabaseName("IX_NormalizedEmail_Key");
            b.HasIndex(u => u.UserName).HasDatabaseName("IX_UserName_Key");
            b.HasIndex(u => u.Email).HasDatabaseName("IX_Email_Key");

            b.ApplyObjectExtensionMappings();
        });

        builder.Entity<IdentityUserClaim>(b =>
        {
            b.ToTable(AbpIdentityDbProperties.DbTablePrefix + "UserClaims", AbpIdentityDbProperties.DbSchema);

            b.ConfigureByConvention();

            b.Property(x => x.Id).ValueGeneratedNever();

            b.Property(uc => uc.ClaimType).HasMaxLength(IdentityUserClaimConsts.MaxClaimTypeLength).IsRequired();
            b.Property(uc => uc.ClaimValue).HasMaxLength(IdentityUserClaimConsts.MaxClaimValueLength);

            b.HasIndex(uc => uc.UserId).HasDatabaseName("IX_UserClaims_Key");

            b.ApplyObjectExtensionMappings();
        });

        builder.Entity<IdentityUserRole>(b =>
        {
            b.ToTable(AbpIdentityDbProperties.DbTablePrefix + "UserRoles", AbpIdentityDbProperties.DbSchema);

            b.ConfigureByConvention();

            b.HasKey(ur => new { ur.UserId, ur.RoleId });

            b.HasOne<IdentityRole>().WithMany().HasForeignKey(ur => ur.RoleId).HasConstraintName("IX_Role_ForeignKey").IsRequired();
            b.HasOne<IdentityUser>().WithMany(u => u.Roles).HasForeignKey(ur => ur.UserId).HasConstraintName("IX_User_ForeignKey").IsRequired();

            b.HasIndex(ur => new { ur.RoleId, ur.UserId }).HasDatabaseName("IX_UserRoles_Key");

            b.ApplyObjectExtensionMappings();
        });

        builder.Entity<IdentityUserLogin>(b =>
        {
            b.ToTable(AbpIdentityDbProperties.DbTablePrefix + "UserLogins", AbpIdentityDbProperties.DbSchema);

            b.ConfigureByConvention();

            b.HasKey(x => new { x.UserId, x.LoginProvider });

            b.Property(ul => ul.LoginProvider).HasMaxLength(IdentityUserLoginConsts.MaxLoginProviderLength)
                .IsRequired();
            b.Property(ul => ul.ProviderKey).HasMaxLength(IdentityUserLoginConsts.MaxProviderKeyLength)
                .IsRequired();
            b.Property(ul => ul.ProviderDisplayName)
                .HasMaxLength(IdentityUserLoginConsts.MaxProviderDisplayNameLength);

            b.HasIndex(l => new { l.LoginProvider, l.ProviderKey }).HasDatabaseName("IX_UserLogins_Key");

            b.ApplyObjectExtensionMappings();
        });

        builder.Entity<IdentityUserToken>(b =>
        {
            b.ToTable(AbpIdentityDbProperties.DbTablePrefix + "UserTokens", AbpIdentityDbProperties.DbSchema);

            b.ConfigureByConvention();

            b.HasKey(l => new { l.UserId, l.LoginProvider, l.Name }).HasName("IX_UserTokens_Key");

            b.Property(ul => ul.LoginProvider).HasMaxLength(IdentityUserTokenConsts.MaxLoginProviderLength)
                .IsRequired();
            b.Property(ul => ul.Name).HasMaxLength(IdentityUserTokenConsts.MaxNameLength).IsRequired();

            b.ApplyObjectExtensionMappings();
        });

        builder.Entity<IdentityRole>(b =>
        {
            b.ToTable(AbpIdentityDbProperties.DbTablePrefix + "Roles", AbpIdentityDbProperties.DbSchema);

            b.ConfigureByConvention();

            b.Property(r => r.Name).IsRequired().HasMaxLength(IdentityRoleConsts.MaxNameLength);
            b.Property(r => r.NormalizedName).IsRequired().HasMaxLength(IdentityRoleConsts.MaxNormalizedNameLength);
            b.Property(r => r.IsDefault).HasColumnName(nameof(IdentityRole.IsDefault));
            b.Property(r => r.IsStatic).HasColumnName(nameof(IdentityRole.IsStatic));
            b.Property(r => r.IsPublic).HasColumnName(nameof(IdentityRole.IsPublic));

            b.HasMany(r => r.Claims).WithOne().HasForeignKey(rc => rc.RoleId).HasConstraintName("IX_RoleId_Key").IsRequired();

            b.HasIndex(r => r.NormalizedName).HasDatabaseName("IX_Roles_Key");

            b.ApplyObjectExtensionMappings();
        });

        builder.Entity<IdentityRoleClaim>(b =>
        {
            b.ToTable(AbpIdentityDbProperties.DbTablePrefix + "RoleClaims", AbpIdentityDbProperties.DbSchema);

            b.ConfigureByConvention();

            b.Property(x => x.Id).ValueGeneratedNever();

            b.Property(uc => uc.ClaimType).HasMaxLength(IdentityRoleClaimConsts.MaxClaimTypeLength).IsRequired();
            b.Property(uc => uc.ClaimValue).HasMaxLength(IdentityRoleClaimConsts.MaxClaimValueLength);

            b.HasIndex(uc => uc.RoleId).HasDatabaseName("IX_RoleClaims_Key");

            b.ApplyObjectExtensionMappings();
        });

        if (builder.IsHostDatabase())
        {
            builder.Entity<IdentityClaimType>(b =>
            {
                b.ToTable(AbpIdentityDbProperties.DbTablePrefix + "ClaimTypes", AbpIdentityDbProperties.DbSchema);

                b.ConfigureByConvention();

                b.Property(uc => uc.Name).HasMaxLength(IdentityClaimTypeConsts.MaxNameLength)
                    .IsRequired(); // make unique
                b.Property(uc => uc.Regex).HasMaxLength(IdentityClaimTypeConsts.MaxRegexLength);
                b.Property(uc => uc.RegexDescription).HasMaxLength(IdentityClaimTypeConsts.MaxRegexDescriptionLength);
                b.Property(uc => uc.Description).HasMaxLength(IdentityClaimTypeConsts.MaxDescriptionLength);

                b.ApplyObjectExtensionMappings();
            });
        }

        builder.Entity<OrganizationUnit>(b =>
        {
            b.ToTable(AbpIdentityDbProperties.DbTablePrefix + "OrganizationUnits", AbpIdentityDbProperties.DbSchema);

            b.ConfigureByConvention();

            b.Property(ou => ou.Code).IsRequired().HasMaxLength(OrganizationUnitConsts.MaxCodeLength)
                .HasColumnName(nameof(OrganizationUnit.Code));
            b.Property(ou => ou.DisplayName).IsRequired().HasMaxLength(OrganizationUnitConsts.MaxDisplayNameLength)
                .HasColumnName(nameof(OrganizationUnit.DisplayName));

            //b.HasMany<OrganizationUnit>().WithOne().HasForeignKey(ou => ou.ParentId).HasConstraintName("IX_ParentId_Key");
            b.HasMany(ou => ou.Roles).WithOne().HasForeignKey(our => our.OrganizationUnitId).HasConstraintName("IX_Org_Role_ForeignKey").IsRequired();

            b.HasIndex(ou => ou.Code).HasDatabaseName("IX_Org_Code_Key");

            b.ApplyObjectExtensionMappings();
        });

        builder.Entity<OrganizationUnitRole>(b =>
        {
            b.ToTable(AbpIdentityDbProperties.DbTablePrefix + "OrganizationUnitRoles", AbpIdentityDbProperties.DbSchema);

            b.ConfigureByConvention();

            b.HasKey(ou => new { ou.OrganizationUnitId, ou.RoleId }).HasName("IX_Org_RoleId_Key");

            b.HasOne<IdentityRole>().WithMany().HasForeignKey(ou => ou.RoleId).HasConstraintName("IX_RoleId_ForeignKey").IsRequired();

            b.HasIndex(ou => new { ou.RoleId, ou.OrganizationUnitId }).HasDatabaseName("IX_Org_Union_Key");

            b.ApplyObjectExtensionMappings();
        });

        builder.Entity<IdentityUserOrganizationUnit>(b =>
        {
            b.ToTable(AbpIdentityDbProperties.DbTablePrefix + "UserOrganizationUnits", AbpIdentityDbProperties.DbSchema);

            b.ConfigureByConvention();

            b.HasKey(ou => new { ou.OrganizationUnitId, ou.UserId }).HasName("IX_Org_Id_UserId_Key");

            b.HasOne<OrganizationUnit>().WithMany().HasForeignKey(ou => ou.OrganizationUnitId).HasConstraintName("IX_Org_ForeignKey").IsRequired();

            b.HasIndex(ou => new { ou.UserId, ou.OrganizationUnitId }).HasDatabaseName("IX_Org_Id_Key");

            b.ApplyObjectExtensionMappings();
        });

        builder.Entity<IdentitySecurityLog>(b =>
        {
            b.ToTable(AbpIdentityDbProperties.DbTablePrefix + "SecurityLogs", AbpIdentityDbProperties.DbSchema);

            b.ConfigureByConvention();

            b.Property(x => x.TenantName).HasMaxLength(IdentitySecurityLogConsts.MaxTenantNameLength);

            b.Property(x => x.ApplicationName).HasMaxLength(IdentitySecurityLogConsts.MaxApplicationNameLength);
            b.Property(x => x.Identity).HasMaxLength(IdentitySecurityLogConsts.MaxIdentityLength);
            b.Property(x => x.Action).HasMaxLength(IdentitySecurityLogConsts.MaxActionLength);

            b.Property(x => x.UserName).HasMaxLength(IdentitySecurityLogConsts.MaxUserNameLength);

            b.Property(x => x.ClientIpAddress).HasMaxLength(IdentitySecurityLogConsts.MaxClientIpAddressLength);
            b.Property(x => x.ClientId).HasMaxLength(IdentitySecurityLogConsts.MaxClientIdLength);
            b.Property(x => x.CorrelationId).HasMaxLength(IdentitySecurityLogConsts.MaxCorrelationIdLength);
            b.Property(x => x.BrowserInfo).HasMaxLength(IdentitySecurityLogConsts.MaxBrowserInfoLength);

            b.HasIndex(x => new { x.TenantId, x.ApplicationName }).HasDatabaseName("IX_Org_Name_Key");
            b.HasIndex(x => new { x.TenantId, x.Identity }).HasDatabaseName("IX_Org_Identity_Key");
            b.HasIndex(x => new { x.TenantId, x.Action }).HasDatabaseName("IX_Org_Action_Key");
            b.HasIndex(x => new { x.TenantId, x.UserId }).HasDatabaseName("IX_Org_UserId_Key");

            b.ApplyObjectExtensionMappings();
        });

        if (builder.IsHostDatabase())
        {
            builder.Entity<IdentityLinkUser>(b =>
            {
                b.ToTable(AbpIdentityDbProperties.DbTablePrefix + "LinkUsers", AbpIdentityDbProperties.DbSchema);

                b.ConfigureByConvention();

                b.HasIndex(x => new
                {
                    UserId = x.SourceUserId,
                    TenantId = x.SourceTenantId,
                    LinkedUserId = x.TargetUserId,
                    LinkedTenantId = x.TargetTenantId
                }).IsUnique().HasDatabaseName("IX_LinkUsers_Key");

                b.ApplyObjectExtensionMappings();
            });
        }

        builder.TryConfigureObjectExtensions<IdentityDbContext>();
    }

    public static void ConfigureFeatureManagementForOracle(this ModelBuilder builder)
    {
        Check.NotNull(builder, nameof(builder));

        if (builder.IsTenantOnlyDatabase())
        {
            return;
        }

        builder.Entity<FeatureValue>(b =>
        {
            b.ToTable(AbpFeatureManagementDbProperties.DbTablePrefix + "FeatureValues", AbpFeatureManagementDbProperties.DbSchema);

            b.ConfigureByConvention();

            b.Property(x => x.Name).HasMaxLength(FeatureValueConsts.MaxNameLength).IsRequired();
            b.Property(x => x.Value).HasMaxLength(FeatureValueConsts.MaxValueLength).IsRequired();
            b.Property(x => x.ProviderName).HasMaxLength(FeatureValueConsts.MaxProviderNameLength);
            b.Property(x => x.ProviderKey).HasMaxLength(FeatureValueConsts.MaxProviderKeyLength);

            b.HasIndex(x => new { x.Name, x.ProviderName, x.ProviderKey }).IsUnique().HasDatabaseName("IX_FeatureValues_Key");

            b.ApplyObjectExtensionMappings();
        });
        builder.Entity<FeatureGroupDefinitionRecord>(b =>
        {
            b.ToTable(AbpFeatureManagementDbProperties.DbTablePrefix + "FeatureGroups", AbpFeatureManagementDbProperties.DbSchema);

            b.ConfigureByConvention();

            b.Property(x => x.Name).HasMaxLength(FeatureGroupDefinitionRecordConsts.MaxNameLength).IsRequired();
            b.Property(x => x.DisplayName).HasMaxLength(FeatureGroupDefinitionRecordConsts.MaxDisplayNameLength).IsRequired();

            b.HasIndex(x => new { x.Name }).IsUnique().HasDatabaseName("IX_FeatureGroups_Key");

            b.ApplyObjectExtensionMappings();
        });

        builder.Entity<FeatureDefinitionRecord>(b =>
        {
            b.ToTable(AbpFeatureManagementDbProperties.DbTablePrefix + "Features", AbpFeatureManagementDbProperties.DbSchema);

            b.ConfigureByConvention();

            b.Property(x => x.GroupName).HasMaxLength(FeatureGroupDefinitionRecordConsts.MaxNameLength).IsRequired();
            b.Property(x => x.Name).HasMaxLength(FeatureDefinitionRecordConsts.MaxNameLength).IsRequired();
            b.Property(x => x.ParentName).HasMaxLength(FeatureDefinitionRecordConsts.MaxNameLength);
            b.Property(x => x.DisplayName).HasMaxLength(FeatureDefinitionRecordConsts.MaxDisplayNameLength).IsRequired();
            b.Property(x => x.Description).HasMaxLength(FeatureDefinitionRecordConsts.MaxDescriptionLength);
            b.Property(x => x.DefaultValue).HasMaxLength(FeatureDefinitionRecordConsts.MaxDefaultValueLength);
            b.Property(x => x.AllowedProviders).HasMaxLength(FeatureDefinitionRecordConsts.MaxAllowedProvidersLength);
            b.Property(x => x.ValueType).HasMaxLength(FeatureDefinitionRecordConsts.MaxValueTypeLength);

            b.HasIndex(x => new { x.Name }).IsUnique().HasDatabaseName("IX_Feature_Name_Key");
            b.HasIndex(x => new { x.GroupName }).HasDatabaseName("IX_Feature_Group_Key");

            b.ApplyObjectExtensionMappings();
        });

        builder.TryConfigureObjectExtensions<FeatureManagementDbContext>();
    }

    public static void ConfigureTenantManagementForOracle(this ModelBuilder builder)
    {
        Check.NotNull(builder, nameof(builder));

        if (builder.IsTenantOnlyDatabase())
        {
            return;
        }

        builder.Entity<Tenant>(b =>
        {
            b.ToTable(AbpTenantManagementDbProperties.DbTablePrefix + "Tenants", AbpTenantManagementDbProperties.DbSchema);

            b.ConfigureByConvention();

            b.Property(t => t.Name).IsRequired().HasMaxLength(TenantConsts.MaxNameLength);

            b.HasMany(u => u.ConnectionStrings).WithOne().HasForeignKey(uc => uc.TenantId).HasConstraintName("IX_TenantId_Key").IsRequired();

            b.HasIndex(u => u.Name).HasDatabaseName("IX_Tenants_Name_Key");

            b.ApplyObjectExtensionMappings();
        });

        builder.Entity<TenantConnectionString>(b =>
        {
            b.ToTable(AbpTenantManagementDbProperties.DbTablePrefix + "TenantConnectionStrings", AbpTenantManagementDbProperties.DbSchema);

            b.ConfigureByConvention();

            b.HasKey(x => new { x.TenantId, x.Name }).HasName("IX_ConnectionStrings_Key");

            b.Property(cs => cs.Name).IsRequired().HasMaxLength(TenantConnectionStringConsts.MaxNameLength);
            b.Property(cs => cs.Value).IsRequired().HasMaxLength(TenantConnectionStringConsts.MaxValueLength);

            b.ApplyObjectExtensionMappings();
        });

        builder.TryConfigureObjectExtensions<TenantManagementDbContext>();
    }

    public static void ConfigureDataDictionaryManagementForOracle(this ModelBuilder builder)
    {
        Check.NotNull(builder, nameof(builder));

        builder.Entity<DataDictionary>(b =>
        {
            b.ToTable(DataDictionaryManagementDbProperties.DbTablePrefix + nameof(DataDictionary), DataDictionaryManagementDbProperties.DbSchema);
            b.HasMany(u => u.Details).WithOne().HasForeignKey(uc => uc.DataDictionaryId).HasConstraintName("IX_DataDictionary_Key").IsRequired();
            b.ConfigureByConvention();
        });

        builder.Entity<DataDictionaryDetail>(b =>
        {
            b.ToTable(DataDictionaryManagementDbProperties.DbTablePrefix + nameof(DataDictionaryDetail),
                DataDictionaryManagementDbProperties.DbSchema);
            b.HasIndex(e => e.DataDictionaryId).HasDatabaseName("IX_DataDictionaryId_Key");
            b.ConfigureByConvention();
        });
    }

    public static void ConfigureNotificationManagementForOracle(this ModelBuilder builder)
    {
        Check.NotNull(builder, nameof(builder));

        builder.Entity<Notification>(b =>
        {
            b.ToTable(NotificationManagementDbProperties.DbTablePrefix + "Notification", NotificationManagementDbProperties.DbSchema);
            b.HasMany(u => u.NotificationSubscriptions).WithOne().HasForeignKey(uc => uc.NotificationId).HasConstraintName("IX_Notification_Key").IsRequired();

            b.ConfigureByConvention();
        });

        builder.Entity<NotificationSubscription>(b =>
        {
            b.ToTable(NotificationManagementDbProperties.DbTablePrefix + "NotificationSubscription", NotificationManagementDbProperties.DbSchema);
            b.HasIndex(e => e.NotificationId).HasDatabaseName("IX_NotificationId_Key");
            b.ConfigureByConvention();
        });
    }
}
```

* 修改 HttpApi.Host-> appsettings.json 配置
* Mysql（或其他DB） 连接字符串
* Redis 连接字符串
* RabbitMq(如果不需要启用设置为 false)，暂时用操作，没找到哪里修改配置
* Es 地址即可(如果没有 es 也可以运行,只是前端 es 日志页面无法使用而已，不影响后端项目启动)
* 修改 DbMigrator-> appsettings.json 数据库连接字符串
* 右键单击.DbMigrator 项目,设置为启动项目运行，按 F5(或 Ctrl + F5) 运行应用程序. 



### DB相关语句（非流程）
``` SQL
    --查找表空间及文件位置
    select file_name from dba_data_files where tablespace_name = 'EXAMINE';

    ---删除表空间及物理文件
    drop tablespace EXAMINE including contents and datafiles;

    ---创建表空间（自增长）
    create tablespace EXAMINE
    datafile 'E:\lld\Files\DB_Documents\EXAMINE.dbf' 
    size 3200M autoextend on next 5M maxsize unlimited; 
    
    -- 删除用户
    drop user c##EXAMINE cascade;

    --查询当前数据库名称
    show con_name;
    --是否处于CDB下
    select name,cdb from v$database;


    --创建用户和密码，并指向表空间
    CREATE USER c##EXAMINE IDENTIFIED BY EXAMINE2022--"c##"是12c的新特性，不加会报错无法执行，登录也是如此
    PROFILE DEFAULT
    DEFAULT TABLESPACE EXAMINE 
    ACCOUNT UNLOCK;
    
    --如过上方步骤出现错误提示包含 不存在表空间 之类的说明，则只想下方操作成功后再次执行上方步骤
    -- 1,尝试重启数据库 cmd
    SQL> sqlplus / as sysdba
    SQL> shutdown immediate 　　--关闭数据库
    SQL> startup 　　　　　　--数据库启动
    SQL> exit
    --重新连接即可


    --查看所有角色
    select * from dba_roles;
    --查看用户角色
    select * from dba_role_privs where GRANTEE = 'EXAMINE';
    -- 将角色赋予用户
    GRANT DBA TO c##EXAMINE;--"c##"是12c的新特性，不加会报错无法执行，登录也是如此


    --链接创建的用户
    conn EXAMINE/EXAMINE2022;

    --备份文件的映射目录
    create directory dump_dir as 'D:\name_20230301.dmp';--.dmp文件所在的目录

    --导入logfile=name_20230301.log
    impdp EXAMINE/EXAMINE2022@127.0.0.1/orcl directory=dump_dir dumpfile=name_20230301.dmp  schemas=EXAMINE remap_schema=EXAMINE:EXAMINE2022;--失败
    impdp c##EXAMINE/EXAMINE2022@ORCL DIRECTORY=dump_dir DUMPFILE=name_20230321.DMP SCHEMAS=EXAMINE remap_schema=EXAMINE:c##EXAMINE;--失败
    imp EXAMINE/EXAMINE2022@ORCL fromuser=EXAMINE touser=EXAMINE file=name_20230301.dmp;--失败
    impdp c##EXAMINE/EXAMINE2022@orcl directory=dump_dir  dumpfile=name_20230321.dmp logfile=name_20230321.log remap_tablespace=EXAMINE:EXAMINE  remap_schema=EXAMINE:c##EXAMINE;--  parallel=4
    impdp c##EXAMINE/EXAMINE2022@orcl directory=dump_dir  dumpfile=name_20230321.dmp remap_tablespace=EXAMINE:EXAMINE  remap_schema=EXAMINE:c##EXAMINE;
    --最终成功脚本命令
    imp c##EXAMINE/EXAMINE2022@orcl file=E:\lld\Files\name_20230321.dmp full=y;--成功，无日志，推荐下方语句

    imp c##EXAMINE/EXAMINE2022@orcl rows=y indexes=n commit=y buffer=65536 feedback=100000 ignore=y full=y file=E:\lld\Files\name_20230321.dmp log=E:\lld\Files\name_20230321.log;--成功

```