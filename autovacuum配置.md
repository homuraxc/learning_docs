## Prompt:
启动autovacuum需要做的配置

## Response:
要启动（启用）PostgreSQL 的 **autovacuum（自动清理）** 功能，一般只需要确保几个关键参数的设置正确。以下是启动 autovacuum 所需要做的配置：



✅ 一、确认 autovacuum 是否启用
----------------------

PostgreSQL 默认是开启 autovacuum 的，但建议你确认以下设置：

| 参数名 | 推荐设置 | 说明 |
| --- | --- | --- |
| `autovacuum` | `on` | 全局开关，是否启用自动清理功能。 |
| `track_counts` | `on` | 必须为 `on`，用于收集表和索引的统计信息，autovacuum 依赖这些信息。 |

> 可以通过 SQL 查看当前设置：

```sql
SHOW autovacuum;
SHOW track_counts;
```



✅ 二、autovacuum 的行为控制参数
----------------------

| 参数名 | 默认值 | 说明 |
| --- | --- | --- |
| `autovacuum_naptime` | `1min` | 后台进程多久检查一次需要清理的表。 |
| `autovacuum_vacuum_threshold` | `50` | 每张表至少有多少行更新/删除后才触发 vacuum。 |
| `autovacuum_vacuum_scale_factor` | `0.2` | 表中被更新/删除的比例超过该值时触发 vacuum。 |
| `autovacuum_analyze_threshold` | `50` | 每张表至少有多少行被修改后触发 analyze。 |
| `autovacuum_analyze_scale_factor` | `0.1` | 表中被修改的比例超过该值时触发 analyze。 |

例如，某张表有 10,000 行数据，触发 autovacuum 的条件是：

```text
autovacuum_vacuum_threshold + autovacuum_vacuum_scale_factor × 表的行数
= 50 + 0.2 × 10000 = 2050 行被更新或删除
```



✅ 三、autovacuum 资源限制参数（可调优）
--------------------------

| 参数名 | 默认值 | 说明 |
| --- | --- | --- |
| `autovacuum_max_workers` | `3` | 同时运行的 autovacuum 进程数上限。 |
| `autovacuum_work_mem` | `-1`（即使用 `work_mem`） | 每个 autovacuum 进程使用的内存上限。 |
| `autovacuum_vacuum_cost_limit` | `-1`（使用 `vacuum_cost_limit`） | 限制 I/O 资源使用，防止 autovacuum 占用太多资源。 |
| `autovacuum_vacuum_cost_delay` | `20ms` | 每消耗一定“成本”后暂停的时间，减缓对系统的冲击。 |



✅ 四、表级别的 autovacuum 设置（可选）
--------------------------

你可以通过 `ALTER TABLE` 为某张表单独配置 autovacuum 的行为。例如：

```sql
ALTER TABLE my_table SET (autovacuum_enabled = true);
ALTER TABLE my_table SET (autovacuum_vacuum_scale_factor = 0.05);
ALTER TABLE my_table SET (autovacuum_vacuum_threshold = 100);
```



✅ 五、重启或reload配置
---------------

*   修改 `postgresql.conf` 后，如果更改了 `autovacuum`、`track_counts` 等需要 **重启 PostgreSQL**。
*   修改 autovacuum 的行为参数后（比如 `autovacuum_naptime`），只需要 **reload** 即可：

```bash
pg_ctl reload -D /your/data/dir
```



✅ 六、确认 autovacuum 是否在运行
-----------------------

可以通过以下命令查看 autovacuum 是否运行中：

```sql
SELECT * FROM pg_stat_activity WHERE query LIKE '%autovacuum%';
```

也可以通过监控表：

```sql
SELECT relname, n_dead_tup, last_autovacuum
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC;
```



如需我帮你根据具体场景（比如数据库负载、表大小）推荐配置值，请告诉我你当前的使用环境。



---
Powered by [ChatGPT Exporter](https://www.chatgptexporter.com)