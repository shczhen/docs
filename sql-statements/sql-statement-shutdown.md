---
title: SHUTDOWN
summary: An overview of the usage of SHUTDOWN for the TiDB database.
aliases: ['/docs/dev/sql-statements/sql-statement-shutdown/']
---

# シャットダウン {#shutdown}

`SHUTDOWN`ステートメントは、TiDBでシャットダウン操作を実行するために使用されます。 `SHUTDOWN`ステートメントを実行するには、ユーザーが`SHUTDOWN privilege`を持っている必要があります。

## あらすじ {#synopsis}

<strong>声明：</strong>

![Statement](/media/sqlgram/ShutdownStmt.png)

## 例 {#examples}

{{&lt;コピー可能な&quot;sql&quot;&gt;}}

```sql
SHUTDOWN;
```

```
Query OK, 0 rows affected (0.00 sec)
```

## MySQLの互換性 {#mysql-compatibility}

> <strong>ノート：</strong>
>
> TiDBは分散データベースであるため、TiDBでのシャットダウン操作は、TiDBクラスター全体ではなく、クライアントに接続されたTiDBインスタンスを停止します。

`SHUTDOWN`ステートメントはMySQLと部分的に互換性があります。互換性の問題が発生した場合は、問題を報告できます[GitHubで](https://github.com/pingcap/tidb/issues/new/choose) 。
