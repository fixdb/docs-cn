---
title: TiDB 主从集群的数据校验
aliases: ['/docs-cn/dev/sync-diff-inspector/upstream-downstream-diff/','/docs-cn/dev/reference/tools/sync-diff-inspector/tidb-diff/']
---

# TiDB 主从集群的数据校验

在你使用 TiDB Binlog 搭建 TiDB 的主从集群，Drainer 在把数据同步到 TiDB 时，保存 checkpoint 的同时也会将上下游的 TSO 对应关系保存为 `ts-map`。在 sync-diff-inspector 中配置 `snapshot` 即可对 TiDB 主从集群的数据进行校验。

## 获取 ts-map

在下游 TiDB 中执行以下 SQL 语句：

{{< copyable "sql" >}}

```sql
select * from tidb_binlog.checkpoint;
```

```
+---------------------+--------------------------------------------------------------------------------------------------------------+
| clusterID           | checkPoint                                                                                                   |
+---------------------+--------------------------------------------------------------------------------------------------------------+
| 6711243465327639221 | {"commitTS":409622383615541249,"ts-map":{"primary-ts":409621863377928194,"secondary-ts":409621863377928345}} |
+---------------------+--------------------------------------------------------------------------------------------------------------+
```

从结果中可以获取 ts-map 信息。

## 配置 snapshot

使用上一步骤获取的 ts-map 信息来配置上下游数据库的 snapshot 信息。其中的 `Datasource config` 部分示例配置如下：

```toml
######################### Datasource config ########################
[data-sources.uptidb]
    host = "172.16.0.1"
    port = 4000
    user = "root"
    password = ""
    snapshot = "409621863377928194"

[data-sources.downtidb]
    host = "172.16.0.2"
    port = 4000
    user = "root"
    password = ""
    snapshot = "409621863377928345"
```

## 注意事项

- Drainer 的 `db-type` 需要设置为 `tidb`，这样才会在 checkpoint 中保存 `ts-map`。
- 需要调整 TiKV 的 GC 时间，保证在校验时 snapshot 对应的历史数据不会被执行 GC。建议调整为 1 个小时，在校验后再还原 GC 设置。
- 以上配置只展示 `Datasource config` 部分，并不完全。完整配置请参考 [sync-diff-inspector 用户文档](/sync-diff-inspector/sync-diff-inspector-overview.md)。