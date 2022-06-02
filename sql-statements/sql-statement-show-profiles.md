---
title: プロファイルを表示
summary: TiDBデータベースのSHOWPROFILESの使用法の概要。
---

# プロファイルを表示 {#show-profiles}

`SHOW PROFILES`ステートメントは現在、空の結果のみを返します。

## あらすじ {#synopsis}

**ShowStmt：**

![ShowStmt](/media/sqlgram/ShowStmt.png)

## 例 {#examples}

{{< copyable "" >}}

```sql
SHOW PROFILES
```

```
Empty set (0.00 sec)
```

## MySQLの互換性 {#mysql-compatibility}

このステートメントは、MySQLとの互換性のためにのみ含まれています。 `SHOW PROFILES`を実行すると、常に空の結果が返されます。
