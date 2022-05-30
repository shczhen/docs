---
title: SHOW CONFIG
summary: Overview of the use of SHOW CONFIG in the TiDB database
aliases: ['/docs/dev/sql-statements/sql-statement-show-config/']
---

# 設定を表示 {#show-config}

> <strong>警告：</strong>
>
> この機能は現在実験的な機能です。この機能を実稼働環境で使用することはお勧めしません。

`SHOW CONFIG`ステートメントは、TiDBのさまざまなコンポーネントの現在の構成を示すために使用されます。構成変数とシステム変数は異なる次元で機能するため、混同しないでください。システム変数情報を取得する場合は、 [変数を表示](/sql-statements/sql-statement-show-variables.md)の構文を使用します。

## あらすじ {#synopsis}

<strong>ShowStmt：</strong>

![ShowStmt](/media/sqlgram/ShowStmt.png)

<strong>ShowTargetFilterable：</strong>

![ShowTargetFilterable](/media/sqlgram/ShowTargetFilterable.png)

## 例 {#examples}

すべての構成を表示：

{{&lt;コピー可能な&quot;sql&quot;&gt;}}

```sql
SHOW CONFIG;
```

```
+------+----------------+-------------------------------------------------+---------------------------------------------------------------------+
| Type | Instance       | Name                                            | Value                                                               |
+------+----------------+-------------------------------------------------+---------------------------------------------------------------------+
| tidb | 127.0.0.1:4000 | advertise-address                               | 127.0.0.1                                                           |
| tidb | 127.0.0.1:4000 | binlog.binlog-socket                            |                                                                     |
| tidb | 127.0.0.1:4000 | binlog.enable                                   | false                                                               |
...
120 rows in set (0.01 sec)
```

`type`が`tidb`である構成を表示します。

{{&lt;コピー可能な&quot;sql&quot;&gt;}}

```sql
SHOW CONFIG WHERE type = 'tidb' AND name = 'advertise-address';
```

```
+------+----------------+-------------------+-----------+
| Type | Instance       | Name              | Value     |
+------+----------------+-------------------+-----------+
| tidb | 127.0.0.1:4000 | advertise-address | 127.0.0.1 |
+------+----------------+-------------------+-----------+
1 row in set (0.05 sec)
```

`LIKE`句を使用して、 `type`が`tidb`である構成を表示することもできます。

{{&lt;コピー可能な&quot;sql&quot;&gt;}}

```sql
SHOW CONFIG LIKE 'tidb';
```

```
+------+----------------+-------------------------------------------------+---------------------------------------------------------------------+
| Type | Instance       | Name                                            | Value                                                               |
+------+----------------+-------------------------------------------------+---------------------------------------------------------------------+
| tidb | 127.0.0.1:4000 | advertise-address                               | 127.0.0.1                                                           |
| tidb | 127.0.0.1:4000 | binlog.binlog-socket                            |                                                                     |
| tidb | 127.0.0.1:4000 | binlog.enable                                   | false                                                               |
...
40 rows in set (0.01 sec)
```

## MySQLの互換性 {#mysql-compatibility}

このステートメントは、MySQL構文のTiDB拡張です。

## も参照してください {#see-also}

-   [変数を表示](/sql-statements/sql-statement-show-variables.md)
