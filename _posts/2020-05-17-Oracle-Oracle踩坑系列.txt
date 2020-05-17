---
layout:     post
title:      Oracle踩坑系列
subtitle:   Oracle 2020踩坑 全记录
date:       2020-05-17
author:     BY
header-img: img/post-bg-cook.jpg
catalog: true
tags:
    - iOS
---

# Oracle踩坑系列
> 入职新公司，使用的是Oracle，开始初次使用基本上是小白使用系列之各种懵逼各种坑····。
> 谨记系列。
[TOC]

## Oracle体系结构
* Oracle数据库的体系结构的组成部分
* Oracle数据库和Oracle实例的含义
* Oracle数据库的组成和Oracle实例的组成

---
## 常用命令
* Connect：切换连接用户，简写形式conn
* Show user：显示当前登陆的用户
* Host<dos命令>：执行操作系统命令
* Spool：导出记录到文本
* Clear screen：清屏
* Start d:\test.sql 执行文件系统中的SQL语句（**注：start命令等同于@，即：@d:\test.sql**）
* Desc：显示表结构
* Show error：显示错误信息
* Exit：退出

**注：** 在连接sys用户的时候，切记一定要带上用户权限，如： conn sys/Ld123456 as sysdba。

**解锁示例用户scott锁定**
```sql
--先连接到某一用户，sys或任意一个，示例连的是sys
conn sys/Ld123456 as sysdba

--再进行操作解锁命令，要带上最后的分号
alter user scott account unlock;
```

---
## 安装
>安装的话基本上就是下一步下一步，使用默认的即可，但是有些步骤还是要注意看完再点下一步的好，避免不必要的坑，这里就不提了，度娘一大把。

用户及口令(HOME)：
1. orcl
    123(因不符合口令规则提示后，执意执行确定下一步，导致登陆提示账户/密码不对，无解)


2. SYS
    Ld123456


3. SYSTEM
    Ld123456



**注：** 如果电脑是第一安装话基本上默认安装的都会自己生成Oracle的实例，新手基本可以不用管；为什么这里要提，是因为安装可以跟是否安装实例这两个是可以分开的，具体没有去深究；因为是前辈有问到我在安装过程中有没有安装Oracle的实例，因为我只用过MY SQL和SQL SERVER两个数据库（安装基本没啥需要注意的，嗖嗖嗖的）；但是Oracle不同之处也在于此，后面还有创建表空间、创建用户、权限等等都与其他两个主流的数据库不同，这些是需要注意的。

* 在安装过程中会有一步是必须要用户自己设置一个用户及口令的，这个是没法像之前的步骤能跳过的；这里就是一个坑，往往默认会生成一个用户名为**orcl**的用户，后面就是你自己设置口令及确认口令了，这里就需要注意，一定要按照Oracle的口令规则来设定，不然即使你设置了口令，只要不符合规范，设置了其实也没用，它只是警告你，没有做必须让你做这个操作。所以这里就需要注意了，因为你设置了不规范的口令，后面在DOM中无论你怎么输入之前刚刚设定好的用户和口令，都是提示错误，这个错误包含用户及口令这两个提示的错误，简直无解。
* 在安装完成后先不要着急关闭最后的提示框，这里可以进行设置系统用户的口令信息，在口令管理中，当然你也可以关闭出来再在DOM下进行设置系统用户的口令信息，个人喜好，但在安装过程中去设置操作起来相对简单一点吧。

**注：即使在安装结束后设置的后面两个账号的密码还是没法直接在DOM下进行正确的登陆用户及口令。但是等在数据库控制URL（https://localhost:1158/em）中进行登陆SYS/SYSTEM账户,SYS要选SYSDBA身份连接，SYSTEM则选择（只能）普通用户连接。**

**安装设置完成后提示：**

![d96658012723a292aa62e0781e86f1df.png](en-resource://database/1008:1)


Database Control URL 为 https://localhost:1158/em

管理资料档案库已置于安全模式下, 在此模式下将对 Enterprise Manager 数据进行加密。加密密钥已放置在文件 G:/Tool/Oracle/product/11.2.0/dbhome_1/localhost_orcl/sysman/config/emkey.ora 中。请务必备份此文件, 因为如果此文件丢失, 则加密数据将不可用。

---
## 创建/查询表空间
**数据库与表空间：**
* 表空间实际上是数据库上的逻辑存储结构，可以把表空间理解为在数据库中开辟的一个空间，用于存放我们数据库的对象，一个数据库可以由多个表空间构成。（这也是区别于其他数据的一点）

**表空间与数据文件：**
* 表空间实际上是由一个或多个数据文件构成的，数据文件的位置和大小可以由我们用户自己来定义。我们所操作的一些表，一些其他的数据对象都是存放在数据文件里的。那么数据文件是物理存储结构，真正可以看到的，而表空间是逻辑存储结构，如下图：
```mermaid
graph TD
C{数据库}
C --> D[表空间1]
C --> E[表空间2]
C --> F[表空间3]
D --> G[数据文件1]
D --> H[数据文件2]
```

**表空间的分类**
* 永久表空间
* 临时表空间
* UNDO表空间（删错了数据，可以回退回来之前的，那那个之前的数据就是存放在这个空间里的）

1. 登陆用户。
2. 创建表空间（含临时表空间）。
    ```sql
        --不设定创建表空的文件存放的目录
        create tablespace test1_tablespace datafile 'test1file.dbf' size 10m;
    ```
    
    ``` sql
        --创建临时表空间，不设定创建表空的文件存放的目录
         create temporary tablespace temptest1_tablespace tempfile 'tempfile1.dbf' size 10m;
    ```
 3. 查找表空间及文件位置

    ```sql
        --查找表空间
        select file_name from dba_data_files where tablespace_name = 'TEST1_TABLESPACE';
    ```
    
     ```sql
        --查找临时表空间
         select file_name from dba_temp_files where tablespace_name = 'TEMPTEST1_TABLESPACE';
    ```


---
## 用户管理
> 创建用户的作用就是起到一个保险，在这个用户下的所有操作、数据的产生都跟这个用户绑定，设置权限什么的也方便，还有就是删除这个用户后也会连带这个用户的所有**东西**都会被删除。会省事很多。


**创建用户的语法格式：**
```sql
    -- 创建用户
    create user <user_name>
    --设置用户口令
    identified by <password>
    --创建表空间（即永久表空间）
    default tablespace <default tablespace>
    --创建临时表空间
    temporary tablespace <temporary tablespace>;
```
**举例：**
```sql
    --创建用户及口令
    create user yan identified by test
    --创建永久表空间
    default tablespace test1_tablespace
    --创建临时表空间
    temporary tablespace temptest1_tablespace;
```

**查看创建的用户：**
```sql
Select username from dba_users;
```

**给创建的用户授权：**
grant 权限 to 用户名
```sql
grant connect to yan;
```

**更改用户密码：**
* Alter user 用户名 identified by 新密码；

**不希望某用户登陆，而又不删除其用户，可以将用户锁定：**
* Alter user 用户名 account lock;

**删除用户：**
* drop user 用户名 cascade;
* //加上cascade则将用户连同其创建的东西全部删除

**下面演示连贯上面的操作**
```sql
--第一，创建用户
create user yan identified by test default tablespace test1_tablespace temporary tablespace temptest1_tablespace;

--第二，查看用户是否存在
 select username from dba_users;
 
 --第三，授权用户
  grant connect to yan;
  
  --第四，更改用户密码
  conn yan/test--先连接当前需要更改密码的用户
  alter user yan identified by t123;
  
  --第五，尝试连接更改密码过后的用户
   conn yan/t123
   
   --第六，删除用户
   conn sys/Ld123456 as sysdba;--先连接到管理员权限用户
   
   --删除用户，及用户所有的东西
    alter user yan account lock;
```


## 角色管理
**三种标准的角色**
1. CONNECT（连接角色）：拥有Connect权限的用户只能登陆Oracle，不可以创建实体（表），不可以创建表结构（如用户）。
2. RESOURCE（资源角色）：拥有Resource权限的用户只可以创建实体（表），不可以创建数据库结构（如用户）。
3. DBA（数据库管理员角色）：拥有全部特权，是系统最高权限，只有DBA才可以创建数据库结构。

>对于普通用户：授予connect，resource权限。
>对于DBA管理用户：授予dba权限。

**创建角色**
* 语法格式：
* CREATE ROLE 角色名;
* 例如：
* CREATE ROLE manager;

**为角色赋予权限**
* 语法格式
* GRANT 权限 TO 角色；
* 例如，给角色赋予创建表、视图的权限：
* CRANT create table, create view TO manager;

**将角色赋予用户**
* 语法格式：
* GRANT 角色 TO 用户；

* 例如，将一个角色赋予两个用户：
* GRANT manager TO user01, user02;

**收回权限**
* 例如：
* revoke manager from user02;

**删除角色**
* 语法格式：
* drop role manager;

## 权限
> 权限指的是执行特定命令或访问数据库对象的权力。

**权限的作用：**
* 主要保证数据库的安全性，其中有两个，分别是**系统的安全性**和**数据安全性**

**权限的分类**
* 系统权限：运行用户执行特定的数据库动作，如创建表、创建索引、连接实例等。
* 对象（实体）权限：允许用户操作一些特定的对象，如读取视图，可更新某些列】执行存储过程等。


**系统权限**
* 查询Oracle所有的系统权限：
* SELECT * FROM SYSTEM_PRIVIVLEGE_MAP;

**常用的系统权限如：**
* CREATE SESSION 创建会话
* CREATE SEQUENCE 创建序列
* CREATE TABLE 创建表
* CREATE USER 创建用户
* ALTER USER 更改用户
* DROP USER 删除用户
* CREATE VIEW 创建视图


**授予系统权限的语法格式**
* GRANT privilege [, privilege...] TO user [, user | role, PUBLIC...]

**举例：**
``` sql
--注意：这句话的意思是对用户user01，user02分配了创建表，创建序列的权限。
grant create table, create sequence to manager;
grant manager to user01, user02;
```

**回收系统权限的语法格式：**
* REVOKE {privilege | role} FROM {user_name | role_name | PUBLIC}
* 举例：
    ```sql
        revoke manager from user01;
        revoke create table, create sequence from manager;
    ```


**对象权限**
* 查询Oracle所有的对象权限
* select * from table_privilege_map
* 常用的对象权限如：
    >select, update, insert, delete, all 等
    >//all包括所有的权限

**授予对象权限的语法格式**
* GRANT object_priv | ALL [(columns)] ON object TO {user | role | PUBLIC}
* 举例：
    * grant select, update, insert on scott.emp to manager2;
    * grant manager2 to user03;
    * grant all on scott.emp to user04; 

**回收对象权限的语法格式：**
* REVOKE{privilege [, privilege...] | ALL} ON object FROM {user[, user...] | role | PUBLIC}
* 举例：
    * revoke all on scott.emp from user04;

**综合权限的知识点操作实例代码如下**
```sql
    --查看所有系统权限
select * from system_privilege_map;

--创建用户
create user user02 identified by pass02;

--给用户赋予一个创建会话的权限
grant create session to user01;

--通过角色给用户赋予一个系统权限
create role manager;

grant create table, create sequence to manager;

grant manager to user01;


--查看所有对象权限
select * from table_privilege_map;


--通过角色给用户赋予一个对象权限
create role manager01;

grant select,update,insert on scott.emp to manager01;

grant manager01 to user01;

--测试对象权限
conn user01/pass01

select * from scott.emp;(成功)

select * from scott.dept;（失败)

--回收对象权限
revoke select,update,insert on scott.emp from manager01;


```


---
## 还原备份
> 数据库备份后还原是经常做的事情，且对于新人，公司的东西不可能立马让你去服务器上**玩耍**，需要本地去跑起来自己去倒腾是最安全的。

> 创建过程： 表空间--->用户--->表;
> 所属关系： 表空间 包含 用户 包含 表；


1. 创建表空间
* 在PL/SQL中执行
    ``` sql
    --空间名：HISWW
    create tablespace CICI
    logging
    datafile
    'G:\Tool\Oracle\product\11.2.0\dbhome_1\oradata\orcl\CICI.DBF'
    size 32m
    autoextend on
    next 32m maxsize 2048m
    extent management local;
    ```

2. 创建用户并指定表空间
    * 可在DOS命令也可以在PL/SQL中执行，本例是在PL/SQL中执行的
    ``` sql
    CREATE USER cici IDENTIFIED BY cici
    PROFILE DEFAULT
    DEFAULT TABLESPACE CICI 
    ACCOUNT UNLOCK;
    ```
    
3. 为用户赋予权限
* 本例是在DOS命令下执行的。
    ```sql
     grant dba TO cici;
      grant create session to cici;--这一步如果是DBA以外的授权目测是必须要有的，但DBA好像不是必须的。
    ```
4. 连接创建的用户
    * 本例是在DOS命令下执行
    ```sql
    conn cici/cici
    ```
5. 创建映射目录，
即oracle机器上放置备份文件的路径， 如下图（第一次还原必须执行，以后不要执行）
    ```sql
    create directory dir_dp as 'D:\data';--.dmp文件所在的目录
    ```

6. 赋予用户目录权限
    *先退出当前用户
    *在连接SYS用户去赋予CICI用户权限，因为不可以自己给自己赋予权限。
    ```sql
        Grant read,write on directory dir_dp to cici;
    ```
5. 导入数据库
    ```sql
    --这里会报错，所有用不了imp
    --在imp 导入数据库的时候出现问题； 这个问题是 你用 expdp导出的 却用客户端的 imp 导入；换成impdp导入即可
    imp cici/cici@orcl file=I:\南方数码\南方数码\cyqkdata\CYQK20200313.DMP from user=expuser to user=cici

    --第二次
    --文件位置不需要指出，impdp程序会自动去之前创建的directory中查找impdp命令中指定的文件名是否存在
    impdp cici/cici@orcl file=CYQK20200313.DMP full=y
    ```


## 纠正
>要先创建一个一样的表空间，还有用户（可以不用一样，但是在还原的时候需要写上原用户和当前用户信息），新建路径（日后再次备份文件数据是会直接存放再该目录下）
1. 还原
    ``` 
    impdp ihap2_cy/south@127.0.0.1/orcl directory=dump_dir dumpfile=CYQK20200313.dmp logfile=cy20200316.log schemas=ihap2_cy_qk remap_schema=ihap2_cy_qk:ihap2_cy

    ```
    **注：**
    @后面的IP是这个数据库的地址IP（有端口则带端口，本地可以不用写，会默认本地的端口）；
    orcl是数据库；
    directory={目录名（用于再次备份的数据文件、日志存放的地址，需要自己手动去新建目录，这个最好是必须要有。）}，dump_dir目录名可以在pl/sql中简历，方便简单；
    logfile={日志文件，用于当次还原数据库的过程中出现的日志信息，如果在DOS命令下会出现很多日志信息展示不全无法完整的查看}，可以不用去手动建这个文件，直接命名文件名即可；
    schemas={备份数据库的用户名，一定要带有，否则会报错无法还原}；
    remap_schema={原来备份数据的用户名 ：本地还原的用户名}。

    **表空间最好是一致的名称，不然会报错**


2. 赋予用户权限
    ```sql
        grant create session to Ihap2_cy;--该权限不足以还原数据
        grant dba session to Ihap2_cy;--为了省事使用了DBA的权限，还没有尝试connect，resource权限

    ```

3. 执行导入语句
    ```
        impdp ihap2_cy/south@127.0.0.1/orcl directory=dump_dir dumpfile=CYQK20200313.dmp logfile=cy20200316.log schemas=ihap2_cy_qk remap_schema=ihap2_cy_qk:ihap2_cy
    ```

4. 成功
![6eb541bceee134f803465ec8d6219133.png](en-resource://database/1022:1)

5. 创建日志用户并给予权限
    * 在PL/SQL中创建用户，日志用户的默认表空间指向创建的表空间下，临时表空间可指向TEMP下（不是很重要，但却不可获取的要带有）；临时表空间一般是数据库上的用户共用一个。
    ![6a3da5f1604270f70a65639760145006.png](en-resource://database/1023:1)
    
        
    * 在DOS命令行中进行日志用户授权（因为对PL/SQL工具不是熟悉使用）
        ```sql
             grant create session to ihaplog_cy;
        ```

6. 执行SQL脚本，导入到这个日志用户下
    >* 登录日志用户，使用PL/SQL导入（导入相当与执行了SQL语句）日志用户的SQL语句。
    ![b09c248a6e8f533190dbc5e8b1ad8411.png](en-resource://database/1024:1)
    
    
 7. 完成，重新选择一下当前用户（要点击选择，不然tables文件夹下会没有刷新）刷新一下tables文件。
![e9dd7379582557d94ccd648eb02b9583.png](en-resource://database/1025:1)
![e5807c13ab53eec6b4e3138ca0624a8f.png](en-resource://database/1026:1)

**注：**
* 导入的时候出现一闪而过的情况，太快的话就是有问题的，因为有些表还没导入成功，导致到了登录页，可是在登录的时候报错误，原因就是报了LOG_TRYUSER这个错误（通过日志查看到的，以后一定要养成查看日志的习惯啊！！！）；
* 在导入日志SQL是出现了一闪而过的现象，代表失败了，要重新删掉该用户和日志表，PL和命令行删除提示“无法删除当前已连接的用户”，说明有在连接，要断开方可删除。
    1. 查看用户的连接状况
        select username,sid,serial# from v$session;
     2. 找到日志表，找到y要删除用户的SID和SERIAL，并删除，如图：
        ![07dfd311ee05dd1cb020c72e1c7a362a.png](en-resource://database/1056:1)
        ![45213e901e383939100830b4cf4e82dc.png](en-resource://database/1057:1)
     3. 再删除用户
        ![7f201296588a6e80ead0140aa3fe2363.png](en-resource://database/1058:1)
      **完成**
        
        
* 最后还是重新删了原来的日志用户，不用PL/SQL手动操作新建的方式了，改用在PL/SQL中新建一个SQL语句新建的方式，如下：
    ```sql
    CREATE USER ihaplog_cy  PROFILE "DEFAULT" IDENTIFIED BY "south" DEFAULT TABLESPACE "IHAP" 
    TEMPORARY TABLESPACE "TEMP" ACCOUNT UNLOCK;

    GRANT "AQ_ADMINISTRATOR_ROLE" TO ihaplog_cy WITH ADMIN OPTION;

    GRANT "CONNECT" TO ihaplog_cy WITH ADMIN OPTION;

    GRANT "DBA" TO ihaplog_cy WITH ADMIN OPTION;

    ```
 * 再重新在PL/SQL导入SQL语句，步骤一样，但是一定要注意一闪而过的弹窗有没有一些提示，太快的就需要重新导入了，稍微有那么一秒多两秒即可。
 * 一闪而过问题不大，但是导入的时候必须要登录的是**ihaplog_cy**的用户导入才可以，一闪而过后如果在tables中看到只要有表就代表成功了。

**完成**

---
## 最终导入的结果：
![9dedffdd3fdf87957560a0b508e4afee.png](en-resource://database/1018:1)

![9066eeeda881274d37dd6fef88787d5b.png](en-resource://database/1016:1)

![999c16b80cdc361431cabcdad97d8c6b.png](en-resource://database/1014:1)

![1d1e3f19dc1571f696d56acbcced5924.png](en-resource://database/1012:1)

![192a30a307c701f46203b9cf5b348e92.png](en-resource://database/1010:1)



## Oracle 64与32修复
>背景：本地装了64位后又装了32位的，之所以装了32的是用于office插件的使用。
>之后本地部署的系统就上不去了，提示“ORACLE初始化或关闭正在进行中”。

**原因：1. 可能在装两个oracle的时候误删某些文件；
2. 并没有删除某些文件，可能是非法关机或是断电造成的**

### 64位
1. 登录dba权限
![e2b2cb0cdbc3f68640b252ef7d80f27d.png](en-resource://database/1050:1)

2. 关闭数据库：
shutdown immediate
![b10a9d0462943149d633120a87734f22.png](en-resource://database/1052:1)


3. 启动数据库
startup
启动的过程中发现出了问题
![036cd6f625fa9a688132a06cf7534fab.png](en-resource://database/1051:1)

4. 再次关闭数据库
![0d850c53d6e31967981057eb0fb640bb.png](en-resource://database/1053:1)

5. 启动实例
startup mount
![a508040ebec306ad26089e6a39a7871d.png](en-resource://database/1054:1)

6. 修复（失败）
recover datafile 'D:\APP\MURPHY\ORADATA\ORACL\RED002.LOG'
![c4889388f5f6e6b4ae77cfbdb1e55daf.png](en-resource://database/1055:1)
原因：这里目测是缺失文件，经过百度看到解决方案如下

    1. 从备份或者dataguard上取。
    2. 是文件拷贝过程中出现了问题。重新拷贝下文件就好了。
 **注：这里是确实文件，跟参考的文章损坏不太一样，且应该是文件缺失导致，所以应该是不能直接通过命令行进行修复，只能去找源头复制过来才可以。**
 
 ### 32位
 32位的直接就不能登录原来在64位创建的sys用户，直接无解。

[参考文章1](https://blog.csdn.net/jeryjeryjery/article/details/70510862)
[参考文章2](http://www.itpub.net/thread-1905195-1-1.html)

[参考文章3_未果](https://blog.csdn.net/qq_31250157/article/details/54340792)
[参考文章4_未果](
https://www.koubeiblog.com/inteqa/53676.html)


## T-SQL调取存储过程中的某个业务逻辑

>示例存储过程：PRO_SJSB_GETSUMDATA；类型：7


```sql
declare

sDateTime varchar2(50);

strOutMsg varchar2(2000);

begin

  sDateTime := '2020-04-01';

  pro_sjsb_getsumdata(iType => 7,iDateTime => sDateTime,iUserID => -1,iSystemID => 
-1,strOutMsg => strOutMsg);

  dbms_output.put_line(strOutMsg);

  

end;


```