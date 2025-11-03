## Prompt:
介绍下postgres中domain的用法

## Response:
`DOMAIN` 是 PostgreSQL 中一个非常实用但常被忽略的特性，它允许你在**类型层面上定义带有约束的自定义数据类型**。  
简单说，它是“带规则的类型模板”。你可以把它理解成“带默认值和检查条件的字段类型”。



### 一、基本概念

`DOMAIN` 本质上是**基于已有数据类型**（如 `integer`, `text`, `date` 等）创建的**受限版本**。

它可以：

*   指定默认值（`DEFAULT`）
*   定义 `NOT NULL` 约束
*   定义 `CHECK` 条件（用于限制取值范围或格式）

创建后，这个域类型就能像内置类型一样，用在表结构定义里。



### 二、语法格式

```sql
CREATE DOMAIN domain_name AS base_type
  [ DEFAULT default_value ]
  [ NOT NULL ]
  [ CHECK (expression) ];
```



### 三、使用示例

#### 1\. 创建一个“非负整数”类型

```sql
CREATE DOMAIN positive_int AS integer
  CHECK (VALUE >= 0);
```

> 注意：`VALUE` 是关键字，代表该域的当前取值。

然后你可以像这样使用它：

```sql
CREATE TABLE account (
  id serial PRIMARY KEY,
  balance positive_int
);
```

插入数据时会自动检查条件：

```sql
INSERT INTO account (balance) VALUES (100);  -- ✅ OK
INSERT INTO account (balance) VALUES (-50);  -- ❌ ERROR: value for domain positive_int violates check constraint
```



#### 2\. 创建带默认值和非空约束的域

```sql
CREATE DOMAIN status_text AS text
  DEFAULT 'active'
  NOT NULL
  CHECK (VALUE IN ('active', 'inactive', 'suspended'));
```

使用：

```sql
CREATE TABLE users (
  user_id serial PRIMARY KEY,
  status status_text
);

INSERT INTO users DEFAULT VALUES;  -- 自动使用 'active'
```



#### 3\. 组合多个规则

```sql
CREATE DOMAIN limited_string AS varchar(20)
  NOT NULL
  CHECK (length(VALUE) >= 3);
```

使用时会同时检查长度和非空：

```sql
INSERT INTO items (name) VALUES ('ok');     -- ✅
INSERT INTO items (name) VALUES ('no');     -- ❌ 太短
INSERT INTO items (name) VALUES (NULL);     -- ❌ NOT NULL 违反
```



### 四、修改与删除

```sql
ALTER DOMAIN domain_name DROP CONSTRAINT constraint_name;
ALTER DOMAIN domain_name ADD CONSTRAINT ...;
ALTER DOMAIN domain_name SET DEFAULT 'xxx';
DROP DOMAIN domain_name;
```

例如：

```sql
ALTER DOMAIN positive_int ADD CONSTRAINT must_be_small CHECK (VALUE < 10000);
```



### 五、典型应用场景

1.  **多表共享相同约束规则**
    *   如邮箱字段格式校验、电话号码长度校验等。
    *   定义一次 `CREATE DOMAIN email_address ...`，多表共用。
2.  **强化数据一致性**
    *   把逻辑放在类型层面而非表层面，可减少重复 `CHECK`。
3.  **代码规范化**
    *   团队开发时，用统一域类型可以规范字段设计。



### 六、注意事项

*   域不能引用表中的列（它是类型级别的，不依赖具体表）。
*   域可以嵌套在数组、复合类型中使用。
*   触发器、约束、默认值等仍可在表层面覆盖。



简而言之：

> `DOMAIN` 是 PostgreSQL 提供的“自定义带约束类型”，让你可以用类型系统来表达业务规则，从而减少重复、提高一致性。



如果你愿意，我可以给你一个“邮箱地址 domain + 检查正则”的完整例子，让它在任何表中都能自动验证邮箱格式。是否展示？



---
Powered by [ChatGPT Exporter](https://www.chatgptexporter.com)