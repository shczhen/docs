---
title: CRUD SQL in TiDB
summary: A brief introduction to TiDB's CURD SQL.
---

# TiDBのCRUDSQL {#crud-sql-in-tidb}

このドキュメントでは、TiDBのCURDSQLの使用方法を簡単に紹介します。

## 始める前に {#before-you-start}

TiDBクラスターに接続していることを確認してください。そうでない場合は、 [TiDBクラウドでTiDBクラスターを構築する（DevTier）](/develop/dev-guide-build-cluster-in-cloud.md#step-1-create-a-free-cluster)を参照して無料のクラスターを作成します。

## TiDBでSQLを探索する {#explore-sql-with-tidb}

> <strong>ノート：</strong>
>
> このドキュメントでは、 [TiDBでSQLを探索する](/basic-sql-operations.md)を参照して簡略化しています。詳細については、 [TiDBでSQLを探索する](/basic-sql-operations.md)を参照してください。

TiDBはMySQLと互換性があり、ほとんどの場合、MySQLステートメントを直接使用できます。サポートされていない機能については、 [MySQLとの互換性](/mysql-compatibility.md#unsupported-features)を参照してください。

SQLを試し、MySQLクエリとのTiDBの互換性をテストするには、次のようにします[TiDBをインストールせずに、Webブラウザで直接実行します](https://tour.tidb.io/) 。また、最初にTiDBクラスターをデプロイしてから、そのクラスターでSQLステートメントを実行することもできます。

このページでは、DDL、DML、CRUD操作などの基本的なTiDBSQLステートメントについて説明します。 TiDBステートメントの完全なリストについては、 [TiDBSQL構文図](https://pingcap.github.io/sqlgram/)を参照してください。

## カテゴリー {#category}

SQLは、その機能に応じて次の4つのタイプに分類されます。

-   <strong>DDL（データ定義言語）</strong> ：データベース、テーブル、ビュー、インデックスなどのデータベースオブジェクトを定義するために使用されます。

-   <strong>DML（データ操作言語）</strong> ：アプリケーション関連のレコードを操作するために使用されます。

-   <strong>DQL（データクエリ言語）</strong> ：条件付きフィルタリング後にレコードをクエリするために使用されます。

-   <strong>DCL（データ制御言語）</strong> ：アクセス権限とセキュリティレベルを定義するために使用されます。

以下は主にDMLとDQLを紹介します。 DDLおよびDCLの詳細については、 [TiDBでSQLを探索する](/basic-sql-operations.md)または[TiDBSQL構文の詳細な説明](https://pingcap.github.io/sqlgram/)を参照してください。

## データ操作言語 {#data-manipulation-language}

一般的なDML機能は、テーブルレコードの追加、変更、および削除です。対応するコマンドは`INSERT` 、および`UPDATE` `DELETE` 。

テーブルにデータを挿入するには、次の`INSERT`ステートメントを使用します。

{{&lt;コピー可能な&quot;sql&quot;&gt;}}

```sql
INSERT INTO person VALUES(1,'tom','20170912');
```

一部のフィールドのデータを含むレコードをテーブルに挿入するには、次の`INSERT`ステートメントを使用します。

{{&lt;コピー可能な&quot;sql&quot;&gt;}}

```sql
INSERT INTO person(id,name) VALUES('2','bob');
```

テーブル内のレコードの一部のフィールドを更新するには、 `UPDATE`ステートメントを使用します。

{{&lt;コピー可能な&quot;sql&quot;&gt;}}

```sql
UPDATE person SET birthday='20180808' WHERE id=2;
```

テーブル内のデータを削除するには、次の`DELETE`ステートメントを使用します。

{{&lt;コピー可能な&quot;sql&quot;&gt;}}

```sql
DELETE FROM person WHERE id=2;
```

> <strong>ノート：</strong>
>
> フィルタとして`WHERE`句を含まない`UPDATE`および`DELETE`ステートメントは、テーブル全体で機能します。

## データクエリ言語 {#data-query-language}

DQLは、1つまたは複数のテーブルから目的のデータ行を取得するために使用されます。

テーブル内のデータを表示するには、 `SELECT`ステートメントを使用します。

{{&lt;コピー可能な&quot;sql&quot;&gt;}}

```sql
SELECT * FROM person;
```

特定の列を照会するには、 `SELECT`キーワードの後に列名を追加します。

{{&lt;コピー可能な&quot;sql&quot;&gt;}}

```sql
SELECT name FROM person;
```

結果は次のとおりです。

```
+------+
| name |
+------+
| tom  |
+------+
1 rows in set (0.00 sec)
```

`WHERE`句を使用して、条件に一致するすべてのレコードをフィルタリングし、結果を返します。

{{&lt;コピー可能な&quot;sql&quot;&gt;}}

```sql
SELECT * FROM person WHERE id < 5;
```
