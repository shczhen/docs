---
title: Create a Database
summary: Learn steps, rules, and examples to create a database.
---

# データベースを作成する {#create-a-database}

このドキュメントでは、SQLとさまざまなプログラミング言語を使用してデータベースを作成する方法について説明し、データベース作成のルールを示します。このドキュメントでは、 [書店](/develop/dev-guide-bookshop-schema-design.md)のアプリケーションを例として取り上げ、データベース作成の手順を説明します。

## 始める前に {#before-you-start}

データベースを作成する前に、次の手順を実行します。

-   [TiDBクラウド開発者層でTiDBクラスターを構築する](/develop/dev-guide-build-cluster-in-cloud.md) 。
-   [スキーマ設計の概要](/develop/dev-guide-schema-design-overview.md)を読んでください。

## データベースとは {#what-is-database}

TiDBの[データベース](/develop/dev-guide-schema-design-overview.md)個のオブジェクトには、<strong>テーブル</strong>、<strong>ビュー</strong>、<strong>シーケンス</strong>、およびその他のオブジェクトが含まれています。

## データベースを作成する {#create-databases}

データベースを作成するには、 `CREATE DATABASE`ステートメントを使用できます。

たとえば、データベースが存在しない場合に`bookshop`という名前のデータベースを作成するには、次のステートメントを使用します。

{{&lt;コピー可能な&quot;sql&quot;&gt;}}

```sql
CREATE DATABASE IF NOT EXISTS `bookshop`;
```

`CREATE DATABASE`ステートメントの詳細と例については、 [`CREATE DATABASE`](/sql-statements/sql-statement-create-database.md)ドキュメントを参照してください。

ライブラリビルドステートメントを`root`ユーザーとして実行するには、次のコマンドを実行します。

{{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

```shell
mysql
    -u root \
    -h {host} \
    -P {port} \
    -p {password} \
    -e "CREATE DATABASE IF NOT EXISTS bookshop;"
```

## データベースを表示する {#view-databases}

クラスタ内のデータベースを表示するには、 [`SHOW DATABASES`](/sql-statements/sql-statement-show-databases.md)ステートメントを使用します。

例えば：

{{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

```shell
mysql
    -u root \
    -h {host} \
    -P {port} \
    -p {password} \
    -e "SHOW DATABASES;"
```

次に、出力例を示します。

```
+--------------------+
| Database           |
+--------------------+
| INFORMATION_SCHEMA |
| PERFORMANCE_SCHEMA |
| bookshop           |
| mysql              |
| test               |
+--------------------+
```

## データベース作成のルール {#rules-in-database-creation}

-   [データベースの命名規則](/develop/dev-guide-object-naming-guidelines.md)に従い、データベースに意味のある名前を付けます。
-   TiDBには、 `test`という名前のデフォルトデータベースが付属しています。ただし、必要がない場合は、実稼働環境で使用することはお勧めしません。 SQLセッションで`CREATE DATABASE`ステートメントを使用して独自のデータベースを作成し、 [`USE {databasename};`](/sql-statements/sql-statement-use.md)ステートメントを使用して現在のデータベースを変更できます。
-   `root`人のユーザーを使用して、データベース、ロール、ユーザーなどのオブジェクトを作成します。ロールとユーザーに必要な特権のみを付与します。
-   ベストプラクティスとして、データベーススキーマの変更を実行するには、ドライバーまたはORMの代わりに<strong>MySQLコマンドラインクライアント</strong>または<strong>MySQLGUI</strong>クライアントを使用することをお勧めします。

## 次のステップ {#next-step}

データベースを作成したら、それに<strong>テーブル</strong>を追加できます。詳細については、 [テーブルを作成する](/develop/dev-guide-create-table.md)を参照してください。
