# 培训供给 SQL 视图的增量维护与换版

各记录目录保存独立的合成业务资料。SQL 文件仅包含原始关系表与已有行。manifest.json 记录库角色、表、行数及运行参数。timeline.json 记录后续客户端行为、来源提交或故障，用 pos 表示可比较的到达次序。同一 pos 按数组顺序。提交位置由相应记录的业务字段指定。

## 接入协议

源表可在任一提交内同时修改。timeline 的 tx 是事务标识，source_pos 是该事务提交位置，ops 内 update 同时带修改前键、旧值与新值。源位置用于对外说明哪批数据已可见。源表 workshops 与 trade_tracks 多对多，licenses 可为同一人和工种多次产生匹配，allocated 每行的 places 是占用数。布尔标记的 1 表示生效。
服务可登记 SQL 子集定义，包括 SELECT、别名、等值 INNER JOIN、LEFT JOIN 或 NOT EXISTS、常量比较、GROUP BY、SUM 与 COUNT(DISTINCT 单列表达式)。NULL 不匹配等值连接，COUNT(DISTINCT) 忽略 NULL；没有输入时全局计数为零，有 GROUP BY 的空结果无分组，SUM 无非空输入时为 NULL。query_requests.json 记录业务方已有定义或注册请求的部分文档，可更换参数及在支持范围内登记新定义。

## 本地资料

使用 Python 3.11.2 和内置 SQLite 3.40.1。以下命令从记录创建关系库，目标文件需要尚不存在。各角色可以作为独立 SQLite 文件读取。

~~~sh
python3 seed_db.py fixtures/anti-join-zero-and-concurrent-view-build main-local.db --role main
~~~

使用 Python 的 sqlite3 打开新文件即可访问原始表。用 python3 -m unittest -v 核对资料可装载、表与原始记录一致、SQLite 可同时打开读写连接。后续事件保存在各自的原始记录中。
