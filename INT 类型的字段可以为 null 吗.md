# INT 类型中插入 null

Postgre的INT类型中不能插入null吗

## Response:
PostgreSQL 中的 `INT` 类型（包括 `INTEGER`, `SMALLINT`, `BIGINT` 等）**是可以插入 `NULL` 值的**，除非你**显式加了 `NOT NULL` 约束**。



### ✅ 示例：可以插入 NULL

```sql
CREATE TABLE test (
    id SERIAL PRIMARY KEY,
    age INT
);

INSERT INTO test (age) VALUES (NULL);  -- ✅ 没问题
```

上面建了一个 `age` 字段，它是 `INT` 类型，没有设置 `NOT NULL`，所以可以插入 `NULL`。



### ❌ 示例：加了 `NOT NULL` 后就不行了

```sql
CREATE TABLE test2 (
    id SERIAL PRIMARY KEY,
    age INT NOT NULL
);

INSERT INTO test2 (age) VALUES (NULL);  -- ❌ 报错：违反非空约束
```



### 🔎 补充：`SERIAL` 不等于 `NOT NULL`

很多人误会 `SERIAL` 字段（常用于主键）默认就是 `NOT NULL`。其实它的定义会自动添加 `NOT NULL`：

```sql
id SERIAL  -- 等价于：id INTEGER NOT NULL DEFAULT nextval('some_sequence')
```



### 小结

| 类型 | 默认能否插入 NULL | 是否要显式限制 |
| --- | --- | --- |
| `INT` | ✅ 可以 | 需加 `NOT NULL` |
| `SERIAL` | ❌ 默认不行 | 因为自动加了 `NOT NULL` |

如果你的 `INT` 字段不能插入 NULL，很可能是这个字段建表时加了 `NOT NULL`。你可以用下面 SQL 查看表结构确认：

```sql
\d+ your_table_name
```

