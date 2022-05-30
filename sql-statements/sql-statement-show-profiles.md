---
title: SHOW PROFILES
summary: An overview of the usage of SHOW PROFILES for the TiDB database.
aliases: ['/docs/dev/sql-statements/sql-statement-show-profiles/']
---

# プロファイルを表示 {#show-profiles}

`SHOW PROFILES`ステートメントは現在、空の結果のみを返します。

## あらすじ {#synopsis}

<strong>ShowStmt：</strong>

![ShowStmt](/media/sqlgram/ShowStmt.png)

## 例 {#examples}

{{&lt;コピー可能な&quot;sql&quot;&gt;}}

```sql
SHOW PROFILES
```

```
Empty set (0.00 sec)
```

## MySQLの互換性 {#mysql-compatibility}

このステートメントは、MySQLとの互換性のためにのみ含まれています。 `SHOW PROFILES`を実行すると、常に空の結果が返されます。
