---
title: SHOW PLUGINS
summary: An overview of the usage of SHOW PLUGINS for the TiDB database.
aliases: ['/docs/dev/sql-statements/sql-statement-show-plugins/']
---

# プラグインを表示する {#show-plugins}

`SHOW PLUGINS`は、各プラグインのステータスとバージョン情報を含む、TiDBにインストールされているすべてのプラグインを示します。

## あらすじ {#synopsis}

<strong>ShowStmt：</strong>

![ShowStmt](/media/sqlgram/ShowStmt.png)

<strong>ShowTargetFilterable：</strong>

![ShowTargetFilterable](/media/sqlgram/ShowTargetFilterable.png)

## 例 {#examples}

{{&lt;コピー可能な&quot;sql&quot;&gt;}}

```sql
SHOW PLUGINS;
```

```
+-------+--------------+-------+-----------------------------+---------+---------+
| Name  | Status       | Type  | Library                     | License | Version |
+-------+--------------+-------+-----------------------------+---------+---------+
| audit | Ready-enable | Audit | /tmp/tidb/plugin/audit-1.so |         | 1       |
+-------+--------------+-------+-----------------------------+---------+---------+
1 row in set (0.000 sec)
```

{{&lt;コピー可能な&quot;sql&quot;&gt;}}

```sql
SHOW PLUGINS LIKE 'a%';
```

```
+-------+--------------+-------+-----------------------------+---------+---------+
| Name  | Status       | Type  | Library                     | License | Version |
+-------+--------------+-------+-----------------------------+---------+---------+
| audit | Ready-enable | Audit | /tmp/tidb/plugin/audit-1.so |         | 1       |
+-------+--------------+-------+-----------------------------+---------+---------+
1 row in set (0.000 sec)
```

## MySQLの互換性 {#mysql-compatibility}

このステートメントは、MySQLと完全に互換性があると理解されています。互換性の違いは、GitHubでは[問題を介して報告](https://github.com/pingcap/tidb/issues/new/choose)である必要があります。
