# 免费软件

# Mysql GUI

|名称|下载地址|授权说明|
|:---|:---|:---|
|MySQL Workbench|`https://dev.mysql.com/downloads/workbench/`|Permission to use, modify, and distribute this software and its documentation for any purpose without fee is hereby granted|

# 文本编辑器

|名称|下载地址|授权说明|
|:---|:---|:---|
|Notepad++|`https://notepad-plus-plus.org/`|GNU General Public License|

# Office办公软件

|名称|下载地址|授权说明|
|:---|:---|:---|
|WPS Office|`https://www.wps.cn/`|个人版最终用户许可协议` https://www.wps.cn/privacy/useragreement/`|

# Markdown

|名称|地址|授权|
|:---|:---|:---|
|MarkdownPad|`http://www.markdownpad.com/download.html`|`http://www.markdownpad.com/privacy.html`|

# 数据库

## MongoDB

|名称|地址|授权|
|:---|:---|:---|
|MongoDB Windows版本|`https://fastdl.mongodb.org/win32/mongodb-win32-x86_64-2012plus-4.2.15.zip`||
|MongoDB Linux版本|`https://repo.mongodb.org/yum/amazon/2013.03/mongodb-org/4.2/x86_64/RPMS/mongodb-org-server-4.2.15-1.amzn1.x86_64.rpm`||

## PostgreSQL

|名称|地址|授权|
|:---|:---|:---|
|PostgreSQL Windows版本|`https://www.postgresql.org/download/windows/`|`https://www.postgresql.org/about/licence/`|
|pgAdmin|`https://www.pgadmin.org/download/`|`https://www.pgadmin.org/licence/`|

## Oracle

下载客户端需要注意

- 下载`Instant Client`，最新版不支持win7，如果是win7需要选择适合的版本`18.5.0.0.0`
- 下载页面最下方有`Instant Client`的安装说明
- 解压下载的压缩包，比如：`D:\ProgramFiles\instantclient_18_5`
- 把这个目录添加到环境变量`PATH`中
- `Instant Client`不同版本依赖不同版本的`VC++ Redistributable`，需要下载正确的`VC++ Redistributable`版本
- 添加Oracle配置文件目录，比如：`D:\ProgramFiles\instantclient_18_5\network\admin`
- 可以使用环境变量`TNS_ADMIN`指定`\network\admin`这个目录的路径
- 下载`PL/SQL Developer`
- 修改`tnsnames.ora`

  ```
    // OTD开发/测试环境
    OTD-test =
    (description =
    (address = (protocol = tcp)(host = 139.224.234.170)(port = 1521))
    (connect_data = (sid = orcl)(SERVICE_NAME = orcl))
    )
    
    // OTD生产环境
    OTD-production =
    (description =
    (address = (protocol = tcp)(host = 172.19.201.134)(port = 1521))
    (connect_data = (sid = ORCLPDB1)(SERVICE_NAME = ORCLPDB1))
    )
  ```

|名称|地址|授权|
|:---|:---|:---|
|Instant Client|`https://www.oracle.com/cn/database/technology/instant-client.html`|`https://www.oracle.com/downloads/licenses/distribution-license.html`|
|支持的操作系统版本|`https://docs.oracle.com/en/database/oracle/oracle-database/19/ntcli/operating-system-checklist-for-oracle-database-client-installation-on-microsoft-windows.html#GUID-DB33F444-969C-48EC-A6CC-83E938499DCB`||
|Microsoft Visual C++ Redistributable|`https://docs.microsoft.com/en-US/cpp/windows/latest-supported-vc-redist?view=msvc-170`||

### PL/SQL Developer
|名称|地址|授权|
|:---|:---|:---|
|PL/SQL Developer|`https://www.allroundautomations.com/try-it-free/`||

### 注册/激活码

- PL/SQL 14
    - product code: ke4tv8t5jtxz493kl8s2nn3t6xgngcmgf3
    - serial Number: 264452
    - password: xs374ca

# 脑图
|名称|地址|授权|
|:---|:---|:---|
|FreeMind|`https://sourceforge.net/projects/freemind/`|GNU General Public License version 2.0 (GPLv2)|

# MQ

## Kafka

|名称|地址|授权|
|:---|:---|:---|
|Kafka|`http://kafka.apache.org/downloads`|`https://www.apache.org/licenses/LICENSE-2.0.html`|

# UML工具

|名称|地址|授权|
|:---|:---|:---|
|Open ModelSphere|`http://www.modelsphere.com/org/`|GPL (GNU Public License)|
|Modelio|`https://www.modelio.org/downloads/download-modelio.html`|GPL (GNU Public License)|
|Enterprise Architect|`https://sparxsystems.cn/products/ea/trial/request.php`|Not Free|

# IDE

##　Eclipse

|名称|地址|授权|
|:---|:---|:---|
|Eclipse|`https://www.eclipse.org/downloads/packages/`|`https://www.eclipse.org/legal/privacy.php`|

## IntelliJ IDEA Community

|名称|地址|授权|
|:---|:---|:---|
|Eclipse|`https://www.jetbrains.com/idea/download/#section=windows`|`Community Edition is free to use for personal and commercial development. The IDE and most of it bundled plugins are open-source, licensed under Apache 2.0.`|

# nginx

|名称|地址|授权|
|:---|:---|:---|
|nginx|`http://nginx.org/en/download.html`||