---
title: Migrate Data to a Downstream TiDB Table with More Columns
summary: Learn how to migrate data to a downstream TiDB table with more columns than the corresponding upstream table.
aliases: ['/tidb/dev/usage-scenario-downstream-more-columns/']
---

# より多くの列を持つダウンストリームTiDBテーブルにデータを移行する {#migrate-data-to-a-downstream-tidb-table-with-more-columns}

このドキュメントでは、対応するアップストリームテーブルよりも多くの列を持つダウンストリームTiDBテーブルにデータを移行するときに実行する追加の手順について説明します。通常の移行手順については、次の移行シナリオを参照してください。

-   [小さなデータセットのMySQLをTiDBに移行する](/migrate-small-mysql-to-tidb.md)
-   [大規模なデータセットのMySQLをTiDBに移行する](/migrate-large-mysql-to-tidb.md)
-   [小さなデータセットのMySQLシャードをTiDBに移行およびマージする](/migrate-small-mysql-shards-to-tidb.md)
-   [大規模なデータセットのMySQLシャードをTiDBに移行およびマージする](/migrate-large-mysql-shards-to-tidb.md)

## DMを使用して、より多くの列を持つダウンストリームTiDBテーブルにデータを移行します {#use-dm-to-migrate-data-to-a-downstream-tidb-table-with-more-columns}

アップストリームのbinlogを複製する場合、DMはダウンストリームの現在のテーブルスキーマを使用してbinlogを解析し、対応するDMLステートメントを生成しようとします。アップストリームbinlogのテーブルの列番号が、ダウンストリームテーブルスキーマの列番号と一致しない場合、次のエラーが発生します。

```json
"errors": [
    {
        "ErrCode": 36027,
        "ErrClass": "sync-unit",
        "ErrScope": "internal",
        "ErrLevel": "high",
        "Message": "startLocation: [position: (mysql-bin.000001, 2022), gtid-set:09bec856-ba95-11ea-850a-58f2b4af5188:1-9 ], endLocation: [ position: (mysql-bin.000001, 2022), gtid-set: 09bec856-ba95-11ea-850a-58f2b4af5188:1-9]: gen insert sqls failed, schema: log, table: messages: Column count doesn't match value count: 3 (columns) vs 2 (values)",
        "RawCause": "",
        "Workaround": ""
    }
]
```

次に、アップストリームテーブルスキーマの例を示します。

```sql
# Upstream table schema
CREATE TABLE `messages` (
  `id` int(11) NOT NULL,
  PRIMARY KEY (`id`)
)
```

次に、ダウンストリームテーブルスキーマの例を示します。

```sql
# Downstream table schema
CREATE TABLE `messages` (
  `id` int(11) NOT NULL,
  `message` varchar(255) DEFAULT NULL, # This is the additional column that only exists in the downstream table.
  PRIMARY KEY (`id`)
)
```

DMがダウンストリームテーブルスキーマを使用してアップストリームによって生成されたbinlogイベントを解析しようとすると、DMは上記の`Column count doesn't match`のエラーを報告します。

このような場合、 `binlog-schema`コマンドを使用して、データソースから移行するテーブルのテーブルスキーマを設定できます。指定されたテーブルスキーマは、DMによって複製されるbinlogイベントデータに対応している必要があります。シャードテーブルを移行する場合は、シャードテーブルごとに、binlogイベントデータを解析するためにDMでテーブルスキーマを設定する必要があります。手順は次のとおりです。

1.  DMでSQLファイルを作成し、アップストリームテーブルスキーマに対応する`CREATE TABLE`ステートメントをファイルに追加します。たとえば、次のテーブルスキーマを`log.messages.sql`に保存します。

    ```sql
    # Upstream table schema
    CREATE TABLE `messages` (
    `id` int(11) NOT NULL,
    PRIMARY KEY (`id`)
    )
    ```

2.  `binlog-schema`コマンドを使用して、データソースから移行するテーブルのテーブルスキーマを設定します。このとき、上記の`Column count doesn't match`のエラーにより、データ移行タスクは一時停止状態になっているはずです。

    {{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

    ```
    tiup dmctl --master-addr ${advertise-addr} binlog-schema update -s ${source-id} ${task-name} ${database-name} ${table-name} ${schema-file}
    ```

    このコマンドのパラメーターの説明は次のとおりです。

    | パラメータ               | 説明                                                                                                       |
    | :------------------ | :------------------------------------------------------------------------------------------------------- |
    | `-master-addr`      | dmctlが接続されるクラスター内のDMマスターノードの`${advertise-addr}`を指定します。 `${advertise-addr}`は、DMマスターが外部にアドバタイズするアドレスを示します。 |
    | `binlog-schema set` | スキーマ情報を手動で設定します。                                                                                         |
    | `-s`                | ソースを指定します。 `${source-id}`はMySQLデータのソースIDを示します。                                                           |
    | `${task-name}`      | データ移行タスクの`task.yaml`構成ファイルで定義されている移行タスクの名前を指定します。                                                        |
    | `${database-name}`  | データベースを指定します。 `${database-name}`は、アップストリームデータベースの名前を示します。                                                |
    | `${table-name}`     | アップストリームテーブルの名前を指定します。                                                                                   |
    | `${schema-file}`    | 設定するテーブルスキーマファイルを指定します。                                                                                  |

    例えば：

    {{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

    ```
    tiup dmctl --master-addr 172.16.10.71:8261 binlog-schema update -s mysql-01 task-test -d log -t message log.message.sql
    ```

3.  `resume-task`コマンドを使用して、一時停止状態で移行タスクを再開します。

    {{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

    ```
    tiup dmctl --master-addr ${advertise-addr} resume-task ${task-name}
    ```

4.  `query-status`コマンドを使用して、データ移行タスクが正しく実行されていることを確認します。

    {{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

    ```
    tiup dmctl --master-addr ${advertise-addr} query-status resume-task ${task-name}
    ```
