---
title: SHOW STATS_HISTOGRAMS
summary: An overview of the usage of SHOW HISTOGRAMS for TiDB database.
aliases: ['/docs/dev/sql-statements/sql-statement-show-histograms/']
---

# SHOW STATS_HISTOGRAMS {#show-stats-histograms}

このステートメントは、 `ANALYZE`ステートメントによって収集されたヒストグラム情報を示しています。

## あらすじ {#synopsis}

<strong>ShowStmt</strong>

![ShowStmt](/media/sqlgram/ShowStmt.png)

<strong>ShowTargetFiltertable</strong>

![ShowTargetFilterable](/media/sqlgram/ShowTargetFilterable.png)

<strong>ShowLikeOrWhereOpt</strong>

![ShowLikeOrWhereOpt](/media/sqlgram/ShowLikeOrWhereOpt.png)

## 例 {#examples}

{{&lt;コピー可能な&quot;sql&quot;&gt;}}

```sql
show stats_histograms;
```

```sql
+---------+------------+----------------+-------------+----------+---------------------+----------------+------------+--------------+-------------+
| Db_name | Table_name | Partition_name | Column_name | Is_index | Update_time         | Distinct_count | Null_count | Avg_col_size | Correlation |
+---------+------------+----------------+-------------+----------+---------------------+----------------+------------+--------------+-------------+
| test    | t          |                | a           |        0 | 2020-05-25 19:20:00 |              7 |          0 |            1 |           1 |
| test    | t2         |                | a           |        0 | 2020-05-25 19:20:01 |              6 |          0 |            8 |           0 |
| test    | t2         |                | b           |        0 | 2020-05-25 19:20:01 |              6 |          0 |         1.67 |           1 |
+---------+------------+----------------+-------------+----------+---------------------+----------------+------------+--------------+-------------+
3 rows in set (0.00 sec)
```

{{&lt;コピー可能な&quot;sql&quot;&gt;}}

```sql
show stats_histograms where table_name = 't2';
```

```sql
+---------+------------+----------------+-------------+----------+---------------------+----------------+------------+--------------+-------------+
| Db_name | Table_name | Partition_name | Column_name | Is_index | Update_time         | Distinct_count | Null_count | Avg_col_size | Correlation |
+---------+------------+----------------+-------------+----------+---------------------+----------------+------------+--------------+-------------+
| test    | t2         |                | b           |        0 | 2020-05-25 19:20:01 |              6 |          0 |         1.67 |           1 |
| test    | t2         |                | a           |        0 | 2020-05-25 19:20:01 |              6 |          0 |            8 |           0 |
+---------+------------+----------------+-------------+----------+---------------------+----------------+------------+--------------+-------------+
2 rows in set (0.00 sec)
```

## MySQLの互換性 {#mysql-compatibility}

このステートメントは、MySQL構文のTiDB拡張です。

## も参照してください {#see-also}

-   [分析する](/sql-statements/sql-statement-analyze-table.md)
-   [統計入門](/statistics.md)
