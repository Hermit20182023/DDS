# MongoDB 版本管理

发行说明 &gt; MongoDB 版本管理

## MongoDB 版本管理[¶](https://docs.mongodb.com/v4.2/reference/versioning/#mongodb-versioning)

重要提示

始终升级到所发布系列的最新稳定版本。

MongoDB的版本管理按照`X.Y.Z`的形式，其中`X.Y`是发行版本序列号或者开发版本序列号，`Z`是版本号或者修订号。

* 如果`Y`是偶数，则`X.Y`为发行版本序号；例如，`4.0`是一个发行版本序列号，`4.2`也是一个发行版本序列号。发行版本通常比较稳定，可用于生产环境。
* 如果`Y`是奇数, 则`X.Y`为开发版本；例如，`4.1`是一个开发版本序列号，`4.3`也是一个开发版本序列号。开发版本应该**仅用于测试，不能用于生产环境**。

例如，MongoDB版本号`4.0.12`，`4.0`是发行版本序列号，`.12`是此发行版本的修订号。

### 新版本

发行版本系列号的改变（如`4.0`变成`4.2`）通常标志着新的特性引入，这些新特性通常无法向后兼容。

### 补丁发布

修订号的改变（例如`4.0.11`到`4.0.12`）通常标志着bug的修复，并且可以向后兼容。

### 驱动程序版本

MongoDB的版本编号系统与用于MongoDB驱动程序的版本编号系统不同。

← [MongoDB 1.2.x 发行版本说明](https://docs.mongodb.com/v4.2/release-notes/1.2/) [技术支持](https://docs.mongodb.com/v4.2/support/) →

原文链接：[https://docs.mongodb.com/v4.2/reference/versioning/](https://docs.mongodb.com/v4.2/reference/versioning/)

译者: 罗治军

