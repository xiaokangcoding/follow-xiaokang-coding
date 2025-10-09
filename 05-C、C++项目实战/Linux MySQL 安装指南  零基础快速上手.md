 大家好，我是小康，今天咱们一起来搞定Ubuntu上的MySQL安装！  

> **适用人群**：Linux小白、Ubuntu初学者、第一次接触MySQL的同学  
**系统版本**：Ubuntu 20.04 LTS  
**预计用时**：5-10分钟
>

你是不是也遇到过这样的困扰：看着网上各种MySQL安装教程，要么步骤不全，要么版本太老，要么就是一堆专业术语看得云里雾里？

别慌！今天这篇教程专门为小白量身定制，**保证你跟着做一遍就能成功**！我们用最简单的APT包管理器来安装，就像在手机上装APP一样简单。

## 📋 安装前的准备工作
### 第一步：确认你的Ubuntu版本
首先，让我们确认一下系统版本。打开终端（按`Ctrl + Alt + T`），输入：

```bash
lsb_release -a
```

你应该看到类似这样的输出：

```plain
Distributor ID: Ubuntu
Description:    Ubuntu 20.04.x LTS
Release:        20.04
Codename:       focal
```

如果看到`20.04`，那就对了！

### 第二步：更新系统包列表
这一步很重要，就像给你的"软件商店"刷新一下最新的软件列表：

```bash
sudo apt update
```

**小贴士**：输入这个命令后，系统可能会要求你输入密码，这是正常的。输入时屏幕上不会显示密码字符，直接输入完按回车就行。

## 🔧 开始安装MySQL
### 第三步：安装MySQL服务器
现在是关键时刻！输入这个"魔法咒语"：

```bash
sudo apt install mysql-server
```

系统会问你是否继续安装（Do you want to continue? [Y/n]），直接按回车或者输入`Y`然后回车。

**这里会发生什么？**

+ 系统开始下载MySQL相关文件
+ 自动安装和配置
+ 你会看到一堆滚动的文字，这是正常的，耐心等待

### 第四步：检查MySQL是否安装成功
安装完成后，让我们检查一下MySQL服务是否正在运行：

```bash
sudo systemctl status mysql
```

如果看到绿色的`active (running)`字样，恭喜你，安装成功了！

如果没有自动启动，可以手动启动：

```bash
sudo systemctl start mysql
```

### 第四步补充：安装MySQL开发库（重要）
如果你后续需要开发MySQL相关的程序（比如用C/C++、Python等语言连接MySQL），建议现在就安装MySQL的开发库：

```bash
sudo apt install libmysqlclient-dev -y
```

**这个库的作用**：

+ 提供MySQL的C语言头文件和库文件
+ 支持多种编程语言的MySQL连接器编译
+ C/C++程序连接MySQL必备

**什么时候需要**：

+ 编写C/C++程序连接MySQL
+ 安装一些Python包（如mysqlclient）时需要
+ 编译其他需要MySQL支持的软件

**安装很快**，现在装上以后就不用担心了，避免后续开发时再去找这个依赖。

## 🔐 MySQL安全配置（重要！）
### 第五步：运行安全配置脚本
MySQL安装后默认设置比较宽松，我们需要加强一下安全性。MySQL 8.0的安全配置脚本运行起来很简单：

```bash
sudo mysql_secure_installation
```

**运行过程说明**：

+ MySQL 8.0会自动检测当前的认证方式
+ 如果root用户使用auth_socket认证，脚本会正常运行
+ 直接按回车通常不会有问题

这个命令会启动一个交互式的配置过程，**跟着我的步骤来**：

#### 配置过程详解：
**1. 是否安装密码验证插件？**

```plain
Would you like to setup VALIDATE PASSWORD plugin? (Press y|Y for Yes, any other key for No):
```

+ **新手建议**：输入`n`（不安装）
+ **为什么**：这个插件会强制要求复杂密码，对新手来说可能比较麻烦

**2. 删除匿名用户？**

```plain
Remove anonymous users? (Press y|Y for Yes, any other key for No):
```

+ **建议**：输入`y`（删除匿名用户，提高安全性）

**3. 禁止root远程登录？**

```plain
Disallow root login remotely? (Press y|Y for Yes, any other key for No):
```

+ **建议**：输入`y`（禁止远程登录，提高安全性）

**4. 删除test数据库？**

```plain
Remove test database and access to it? (Press y|Y for Yes, any other key for No):
```

+ **建议**：输入`y`（删除测试数据库）

**5. 重新加载权限表？**

```plain
Reload privilege tables now? (Press y|Y for Yes, any other key for No):
```

+ **建议**：输入`y`（立即生效配置）

## 🎯 测试MySQL安装
### 第六步：登录MySQL并配置root用户认证方式
#### 6.1 首次登录MySQL
MySQL 8.0安装后，root用户默认使用`auth_socket`认证方式，这意味着你可以直接用sudo登录，无需密码：

```bash
sudo mysql
```

**成功的标志**：你会看到类似这样的欢迎信息：

```plain
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.xx-Ubuntu

mysql>
```

#### 6.2 查看当前root用户认证方式
在MySQL命令行中，输入以下命令查看root用户的认证信息：

```sql
SELECT user, host, plugin, authentication_string FROM mysql.user WHERE user='root';
```

你会看到类似这样的输出：

```plain
+------+-----------+-------------+-----------------------+
| user | host      | plugin      | authentication_string |
+------+-----------+-------------+-----------------------+
| root | localhost | auth_socket |                       |
+------+-----------+-------------+-----------------------+
```

**解释**：

+ `plugin`显示为`auth_socket`，表示使用socket认证
+ `authentication_string`为空，表示没有设置密码

#### 6.3 修改root用户认证方式（推荐）
为了方便日常使用和提高安全性，建议将root用户的认证方式改为传统的密码认证(mysql_native_password)：

```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'your_password';
```

**重要说明**：

+ 将`your_password`替换为你想要设置的密码
+ 密码要足够强壮，建议包含大小写字母、数字和特殊字符

#### 6.4 验证修改结果
再次查看root用户的认证信息：

```sql
SELECT user, host, plugin, authentication_string FROM mysql.user WHERE user='root';
```

现在你应该看到：

```plain
+------+-----------+-----------------------+-------------------------------------------+
| user | host      | plugin                | authentication_string                     |
+------+-----------+-----------------------+-------------------------------------------+
| root | localhost | mysql_native_password | *加密后的密码字符串*                      |
+------+-----------+-----------------------+-------------------------------------------+
```

#### 6.5 退出并重新登录测试
退出MySQL：

```sql
EXIT;
```

现在用新的密码登录：

```bash
mysql -u root -p
```

系统会提示输入密码，输入你刚才设置的密码。

**为什么要做这个修改？**

+ **方便性**：可以直接用密码登录，不需要sudo权限
+ **兼容性**：某些应用程序可能不支持auth_socket认证
+ **安全性**：明确的密码认证更符合数据库安全规范

看到`mysql>`这个提示符，说明你已经成功进入MySQL了！

### 第七步：创建普通用户（推荐）
在实际项目中，我们通常不直接使用root用户，而是创建专门的普通用户。这是一个好习惯！

#### 7.1 创建新用户
在MySQL命令行中执行：

```sql
-- 创建新用户（将username和password替换为你想要的用户名和密码）
CREATE USER 'username'@'localhost' IDENTIFIED BY 'password';
```

**举个例子**：

```sql
-- 创建一个名为myapp的用户，密码是myapp123
CREATE USER 'myapp'@'localhost' IDENTIFIED BY 'myapp123';
```

#### 7.2 给用户授权
新创建的用户默认没有任何权限，我们需要给它分配权限：

```sql
-- 给用户授予某个数据库的所有权限
GRANT ALL PRIVILEGES ON database_name.* TO 'username'@'localhost';

-- 如果要给用户授予所有数据库的权限（谨慎使用）
GRANT ALL PRIVILEGES ON *.* TO 'username'@'localhost';
```

**实际例子**：

```sql
-- 先创建一个测试数据库
CREATE DATABASE testdb;

-- 给myapp用户授予testdb数据库的所有权限
GRANT ALL PRIVILEGES ON testdb.* TO 'myapp'@'localhost';

-- 刷新权限
FLUSH PRIVILEGES;
```

#### 7.3 测试新用户
退出当前连接：

```sql
EXIT;
```

用新用户登录：

```bash
mysql -u myapp -p
```

输入密码后，试试查看数据库：

```sql
SHOW DATABASES;
```

你应该能看到testdb数据库。

### 第八步：简单测试
在MySQL命令行中，试试这些基本命令：

```sql
-- 查看所有数据库
SHOW DATABASES;

-- 查看MySQL版本
SELECT VERSION();

-- 退出MySQL
EXIT;
```

**注意**：每个SQL命令后面要加分号`;`

## 🎉 设置MySQL开机自启动
让MySQL在每次开机时自动启动：

```bash
sudo systemctl enable mysql
```

检查是否设置成功：

```bash
sudo systemctl is-enabled mysql
```

如果返回`enabled`，就成功了！

## 🛠️ 常用MySQL管理命令
作为小白，记住这几个命令就够用了：

```bash
# 启动MySQL服务
sudo systemctl start mysql

# 停止MySQL服务
sudo systemctl stop mysql

# 重启MySQL服务
sudo systemctl restart mysql

# 查看MySQL服务状态
sudo systemctl status mysql

# 连接MySQL
mysql -u root -p
```

## 🔍 可能遇到的问题及解决方案
### 问题1：忘记root密码怎么办？
**情况1：如果你使用的是默认的auth_socket认证** 可以直接用sudo登录：

```bash
sudo mysql
```

然后重新设置密码：

```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '新密码';
EXIT;
```

**情况2：如果你已经设置了mysql_native_password认证但忘记密码** 可以这样重置：

```bash
# 停止MySQL服务
sudo systemctl stop mysql

# 安全模式启动MySQL
sudo mysqld_safe --skip-grant-tables &

# 无密码登录
mysql -u root

# 在MySQL中执行
USE mysql;
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '新密码';
FLUSH PRIVILEGES;
EXIT;

# 重启MySQL
sudo systemctl restart mysql
```

### 问题2：无法连接MySQL
如果遇到连接问题，检查服务是否运行：

```bash
sudo systemctl status mysql
```

如果服务没运行，启动它：

```bash
sudo systemctl start mysql
```

### 问题3：端口被占用
MySQL默认使用3306端口，检查端口是否被占用：

```bash
sudo netstat -tlnp | grep 3306
```

## 💡 给初学者的小建议
1. **记住你的密码**：MySQL的root密码一定要记住，建议写在安全的地方
2. **定期备份**：养成备份数据库的好习惯
3. **学习基础SQL**：安装只是第一步，学会基本的SQL语句才能真正使用MySQL
4. **使用图形化工具**：推荐安装Navicat、DBeaver或MySQL Workbench，图形界面更友好

## 🎊 总结
恭喜你！如果按照以上步骤操作，你现在已经：

✅ 成功在Ubuntu 20.04上安装了MySQL  
✅ 完成了基本的安全配置  
✅ 学会了基本的MySQL管理命令  
✅ 知道了常见问题的解决方法

MySQL已经准备好为你的项目服务了！接下来你可以开始学习SQL语句，创建数据库和表，开始你的数据库之旅。

**记住**：遇到问题不要慌，Google是你最好的朋友，Stack Overflow上有很多热心的开发者会帮助你。

---

**写在最后**：这篇教程专门为MySQL初学者准备，如果你觉得有帮助，别忘了**点赞、在看**哦！

## 🚀 Hey！学会MySQL安装只是第一步

恭喜你成功安装了MySQL！不过说实话，会安装MySQL和会**高效使用**MySQL，这中间还隔着一座大山。

特别是当你开始写C++后端项目时，会发现一个残酷的现实：

**直接连接MySQL = 性能灾难！**

我之前遇到过一个真实案例：同事写的服务接口响应时间要50-100ms，CPU使用率却只有20%。问题出在哪？每次请求都要**重新建立数据库连接**！

## 💡 数据库连接池：C++后端的必备技能

如果你想往C++后端方向发展，或者想提升现有项目的性能，**数据库连接池**绝对是你绕不过的核心技术：

✅ **性能提升10-50倍**：复用连接，避免重复建立连接的开销  
✅ **大厂面试必考**：阿里、腾讯几乎100%会问连接池设计  
✅ **架构设计基础**：微服务、分布式系统的核心组件  
✅ **实际项目必需**：任何上规模的C++后端项目都需要连接池

## 🎯 我的MySQL连接池实战项目

最近我从零设计实现了一个**基本生产可用**的MySQL连接池项目：

**项目亮点：**
- **4200+行代码**：核心实现2500+行，不是玩具demo
- **智能重连机制**：比MySQL官方重连更智能稳定
- **负载均衡支持**：支持多数据库，三种负载均衡算法
- **性能监控系统**：实时统计10+项关键指标
- **纯C++11实现**：无第三方依赖，可直接集成

**8天实战课程，每天1小时：**
- 第1天：基础架构 + 日志系统
- 第2天：连接封装 + 配置管理  
- 第3天：智能重连机制
- 第4天：连接池核心逻辑
- 第5天：负载均衡算法
- 第6天：健康检查系统
- 第7天：性能监控模块
- 第8天：压力测试 + 性能调优

**对这个项目感兴趣的朋友可以看这篇详细介绍**：



## 🤝 如何报名？

1. 扫下方二维码或者微信搜索添加：**jkfwdkf**
2. 备注「**连接池**」
3. 了解详情后付款即可加项目实战群

**名额有限，仅20人！**

---

**最后说句心里话**：很多人学MySQL就停留在会安装、会写SQL的层面，但真正的技术壁垒在于**系统设计能力**。

会用开源库的程序员遍地都是，但能从零设计实现核心组件的工程师，这才是真正的稀缺人才！

MySQL装好了，下一步就是学会高效使用它。连接池技术，等你来挑战！💪

---

![](https://files.mdnice.com/user/48364/643d0a06-bef7-49f1-b311-5bc0cf5ae9e7.png)
