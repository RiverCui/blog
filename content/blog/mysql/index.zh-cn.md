---
title: 'MySQL 安装和基础使用'
date: 2024-06-15
description: "MySQL 的安装、环境变量配置以及基础命令行使用"
summary: "MySQL 的安装、环境变量配置以及基础命令行使用"
tags: ["MySQL", "Database"]
---

# MySQL安装和使用

## 安装

### MySQL Community Server

MySQL Community Server 是数据库服务器，它负责实际存储和管理数据。

下载地址：https://dev.mysql.com/downloads/mysql/

选择版本和操作系统，点击 Download 按钮下载。我是 macOS 14 系统，M1 芯片，这里就选择了 `macOS 14 (ARM, 64-bit), DMG Archive` 下载。

![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202406220105311.png)

安装完成后，打开系统设置，拉到最下方就能看到 MySQL，点击按钮运行 MySQL 服务。

![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202406220209009.png)

### MySQL Workbench

MySQL Workbench 是一个客户端应用程序，你可以用它来连接到 MySQL Community Server，进行数据库设计、开发和管理等操作。

要注意版本保持一致，下载地址：https://dev.mysql.com/downloads/workbench/

安装后打开，点击 Connection 或者新建一个，输入密码就OK了。

![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202406220214784.png)

## 环境变量

查看当前 shell
```zsh
echo $SHELL
```

如果是 `/bin/bash`，说明用的是 bash，如果是 `/bin/zsh`，说明用的是 zsh。

如果是 bash:

```bash
# 1. 更改
vim ~/.bash_profile

# 2. 添加
export PATH=${PATH}:/usr/local/mysql/bin

# 3. 更新
source ~/.bash_profile
```

如果是 zsh：
```zsh
# 1. 更改
vim ~/.zshrc

# 2. 添加
export PATH=${PATH}:/usr/local/mysql/bin

# 3. 更新
source ~/.zshrc
```

## 进入 MySQL CLI

```zsh
mysql -u root -p
```

输入安装时密码，即可成功进入 MySQL CLI。


## 命令行工具

全部命令行请查看官方文档：[mysql Client Commands](https://dev.mysql.com/doc/refman/8.0/en/mysql-commands.html)

下面列举几种常用的：

`\g` 和 `\G` 都是命令的结束符，除此之外，也可以直接使用 `;` 来结束命令。
```bash
\g # 将命令发送到 MySQL 服务器
\G # 将命令发送到 MySQL 服务器，并且格式化输出
```

```bash
\c # 清除当前输入
\q # 退出 MySQL
```

## 基础使用

### 连接 MySQL
```bash
mysql -u root -p
Enter password:******
```

### 基本命令
#### 查看数据库列表
```sql
SHOW DATABASES;
```

#### 创建数据库
```sql
CREATE DATABASE testdb;
```

#### 使用数据库
```sql
USE testdb;
```

#### 创建表
```sql
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);
```

#### 查看表结构
```sql
DESCRIBE users;
```

### 数据操作
#### 插入数据
```sql
INSERT INTO users (name, email) VALUES ('John Doe', 'john@example.com');
```

#### 查询数据
```sql
SELECT * FROM users;
```

#### 更新数据
```sql
UPDATE users SET email = 'john.doe@example.com' WHERE name = 'John Doe';
```

#### 删除数据
```sql
DELETE FROM users WHERE name = 'John Doe';
```

### 常用功能
#### 条件查询
```sql
SELECT * FROM users WHERE email LIKE '%@example.com';
```

#### 排序查询
```sql
SELECT * FROM users ORDER BY name ASC;
```

#### 聚合函数
```sql
SELECT COUNT(*) FROM users;
SELECT AVG(id) FROM users;
```

### 多表操作
#### 创建第二个表
```sql
CREATE TABLE orders (
    order_id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT,
    product_name VARCHAR(100),
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

#### 连接查询
```sql
SELECT users.name, orders.product_name
FROM users
JOIN orders ON users.id = orders.user_id;
```
`SELECT users.name, orders.product_name` 子句指定我们希望从数据库中提取的列，这里会提取到两个列：
- users 表中的 name 列(users.name)
- orders 表中的 product_name 列(orders.product.name)
`FROM users` 子句指定我们查询的第一个表，即 users 表。
`JOIN orders ON users.id = orders.user_id` 子句用于将两个表连接起来。即使用内连接(INNER JOIN)将 users 表和 orders 表关联起来。连接条件是 users.id = orders.user_id。

内连接(INNER JOIN)会返回两个表中满足条件的所有记录。如果一条记录在 users 表和 orders 表中都有对应的匹配记录，那么这条记录会被包括在结果集中。简单来说，这个查询会返回所有有订单的用户及其订单的产品名称。

举个例子，假如有以下两个表：

users 表：

| id            | name          |
| ------------- | ------------- |
| 1 | Alice |
| 2 | Bob |
| 3 | Charlie |

orders 表：
| order_id | user_id | product_name |
| ------------- | ------------- | ------------- |
| 101 | 1 | Laptop |
| 102 | 1 | SmartPhone |
| 103 | 2 | Tablet |

执行上述 SQL 语句后，查询结果是：

| name | product_name |
| ------------- | ------------- |
| Alice | Laptop |
| Alice | Smartphone |
| Bob | Tablet |

# 常见问题

## 用 mysql workbench 新建 table 时没有 apply 按钮

8.0.36 版本有问题，替换成 8.0.34 就正常使用了。

## ERROR 1819 (HY000): Your password does not satisfy the current policy requirements

运行下方代码会报错
```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'admin'
```

错误信息：ERROR 1819 (HY000): Your password does not satisfy the current policy requirements。密码不符合安全要求，改下密码的安全要求就可以了。解决步骤：

登录 MySQL
```
mysql -u root -p
```

检查当前密码策略
```
SHOW VARIABLES LIKE 'validate_password%';

```

```
+-------------------------------------------------+--------+
| Variable_name                                   | Value  |
+-------------------------------------------------+--------+
| validate_password.changed_characters_percentage | 0      |
| validate_password.check_user_name               | ON     |
| validate_password.dictionary_file               |        |
| validate_password.length                        | 8      |
| validate_password.mixed_case_count              | 1      |
| validate_password.number_count                  | 1      |
| validate_password.policy                        | MEDIUM |
| validate_password.special_char_count            | 1      |
+-------------------------------------------------+--------+
```

修改当前密码策略

```
SET GLOBAL validate_password.length=4;
SET GLOBAL validate_password.policy=LOW;
```

现在密码策略已经修改了
```
+-------------------------------------------------+-------+
| Variable_name                                   | Value |
+-------------------------------------------------+-------+
| validate_password.changed_characters_percentage | 0     |
| validate_password.check_user_name               | ON    |
| validate_password.dictionary_file               |       |
| validate_password.length                        | 4     |
| validate_password.mixed_case_count              | 1     |
| validate_password.number_count                  | 1     |
| validate_password.policy                        | LOW   |
| validate_password.special_char_count            | 1     |
+-------------------------------------------------+-------+
```

刷新权限并再次尝试修改密码

```
FLUSH PRIVILEGES
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'admin';
```
