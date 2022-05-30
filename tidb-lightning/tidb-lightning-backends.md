---
title: TiDB Lightning Import Mode
summary: Learn how to choose different import modes of TiDB Lightning.
aliases:
  [
    "/docs/dev/tidb-lightning/tidb-lightning-tidb-backend/",
    "/docs/dev/reference/tools/tidb-lightning/tidb-backend/",
    "/tidb/dev/tidb-lightning-tidb-backend",
    "/docs/dev/loader-overview/",
    "/docs/dev/reference/tools/loader/",
    "/docs/dev/load-misuse-handling/",
    "/docs/dev/reference/tools/error-case-handling/load-misuse-handling/",
    "/tidb/dev/load-misuse-handling",
    "/tidb/dev/loader-overview/",
  ]
---

# TiDBLightning インポートモード {#tidb-lightning-import-modes}

TiDB Lightning は、2 つの[バックエンド](/tidb-lightning/tidb-lightning-glossary.md#back-end)で 2 つのインポートモードをサポートします。バックエンドは、TiDBLightning がデータをターゲットクラスターにインポートする方法を決定します。

- <strong>ローカルバックエンド</strong>：TiDB Lightning は、最初にデータをキーと値のペアにエンコードし、並べ替えてローカルの一時ディレクトリに保存し、これらのキーと値のペアを各 TiKV ノードに<em>アップロード</em>します。次に、TiDB Lightning は TiKV 取り込みインターフェイスを呼び出して、TiKV の RocksDB にデータを書き込みます。初期化されたデータのインポートについては、インポート速度が速いため、ローカルバックエンドを検討してください。

- <strong>TiDB バックエンド</strong>：TiDB Lightning は、最初にデータを SQL ステートメントにエンコードし、次にこれらのステートメントを実行してデータをインポートします。ターゲットクラスターが実稼働環境にある場合、またはターゲットテーブルにすでにデータがある場合は、TiDB バックエンドを検討してください。

| バックエンド                                | ローカルバックエンド   | TiDB-バックエンド    |
| :------------------------------------------ | :--------------------- | :------------------- |
| スピード                                    | 高速（〜500GB /時）    | 遅い（〜50 GB / hr） |
| リソースの使用                              | 高い                   | 低い                 |
| ネットワーク帯域幅の使用                    | 高い                   | 低い                 |
| インポート中の ACID コンプライアンス        | いいえ                 | はい                 |
| ターゲットテーブル                          | 空である必要があります | 移入可能             |
| サポートされている TiDB バージョン          | = v4.0.0               | 全て                 |
| TiDB はインポート中にサービスを提供できます | いいえ                 | はい                 |

> <strong>注</strong>：
>
> - ローカルバックエンドモードで本番環境の TiDB クラスターにデータをインポートしないでください。これは、オンラインアプリケーションに深刻な影響を及ぼします。
> - デフォルトでは、複数の TiDB Lightning インスタンスを起動して、同じ TiDB クラスターにデータをインポートすることはできません。代わりに、 [並列インポート](/tidb-lightning/tidb-lightning-distributed-import.md)つの機能を使用する必要があります。
> - 複数の TiDBLightning インスタンスを使用して同じターゲットデータベースにデータをインポートする場合は、複数のバックエンドを使用しないでください。たとえば、ローカルバックエンドと TiDB バックエンドの両方を使用してデータを TiDB クラスターにインポートしないでください。

## ローカルバックエンド {#local-backend}

TiDB Lightning では、TiDBv4.0.3 にローカルバックエンドが導入されています。ローカルバックエンドを使用すると、データを TiDB クラスター&gt;=v4.0.0 にインポートできます。

### 構成と例 {#configuration-and-examples}

```toml
[Lightning]
# Specifies the database to store the execution results. If you do not want to create this schema, set this value to an empty string.
# task-info-schema-name = 'lightning_task_info'

[tikv-importer]
backend = "local"
# When the backend is 'local', whether to detect and resolve conflicting records (unique key conflict).
# The following three resolution strategies are supported:
#  - none: does not detect duplicate records, which has the best performance in the three
#    strategies, but might lead to inconsistent data in the target TiDB.
#  - record: only records conflicting records to the `lightning_task_info.conflict_error_v1`
#    table on the target TiDB. Note that the required version of the target TiKV is not
#    earlier than v5.2.0; otherwise, it falls back to 'none'.
#  - remove: records all conflicting records, like the 'record' strategy. But it removes all
#    conflicting records from the target table to ensure a consistent state in the target TiDB.
# duplicate-resolution = 'none'

# The directory of local KV sorting in the local-backend mode. SSD is recommended, and the
# directory should be set on a different disk from `data-source-dir` to improve import
# performance.
# The sorted-kv-dir directory should have free space greater than the size of the largest
# table in the upstream. If the space is insufficient, the import will fail.
sorted-kv-dir = ""
# The concurrency that TiKV writes KV data in the local-backend mode. When the network
# transmission speed between TiDB Lightning and TiKV exceeds 10 Gigabit, you can increase
# this value accordingly.
# range-concurrency = 16
# The number of KV pairs sent in one request in the local-backend mode.
# send-kv-pairs = 32768

[tidb]
# The target cluster information. The address of any tidb-server from the cluster.
host = "172.16.31.1"
port = 4000
user = "root"
# Configure the password to connect to TiDB. Either plaintext or Base64 encoded.
password = ""
# Required in the local-backend mode. Table schema information is fetched from TiDB via this status-port.
status-port = 10080
# Required in the local-backend mode. The address of any pd-server from the cluster.
pd-addr = "172.16.31.4:2379"
```

### 紛争解決 {#conflict-resolution}

`duplicate-resolution`つの構成は、競合する可能性のあるデータを解決するための 3 つの戦略を提供します。

- `none` （デフォルト）：重複レコードを検出しません。これは、3 つの戦略で最高のパフォーマンスを発揮しますが、ターゲット TiDB のデータに一貫性がなくなる可能性があります。
- `record` ：競合するレコードのみをターゲット TiDB の`lightning_task_info.conflict_error_v1`テーブルに記録します。ターゲット TiKV の必要なバージョンは v5.2.0 より前ではないことに注意してください。それ以外の場合は、「none」にフォールバックします。
- `remove` ：「レコード」戦略のように、競合するすべてのレコードを記録します。ただし、ターゲットテーブルから競合するすべてのレコードを削除して、ターゲット TiDB で一貫した状態を確保します。

データソースに競合するデータがあるかどうかわからない場合は、 `remove`の戦略をお勧めします。 `none`と`record`の戦略では、競合するデータがターゲットテーブルから削除されません。つまり、TiDBLightning によって生成された一意のインデックスがデータと矛盾している可能性があります。

## TiDB-バックエンド {#tidb-backend}

### 構成と例 {#configuration-and-examples}

```toml
[tikv-importer]
# The backend mode. To use TiDB-backed, set it to "tidb".
backend = "tidb"

# Action to do when trying to insert a conflicting data.
# - replace: use new record to replace the existing record.
# - ignore: keep the existing record, and ignore the new record.
# - error: abort the import and report an error.
# on-duplicate = "replace"
```

### 紛争解決 {#conflict-resolution}

TiDB バックエンドは、すでに入力されている（空でない）テーブルへのインポートをサポートします。ただし、新しいデータにより、古いデータとの一意のキーの競合が発生する可能性があります。 `on-duplicate`の構成を使用して、競合を解決する方法を制御できます。

| 価値      | 競合時のデフォルトの動作                   | SQL ステートメント       |
| :-------- | :----------------------------------------- | :----------------------- |
| `replace` | 新しいレコードが古いレコードを置き換えます | `REPLACE INTO ...`       |
| `ignore`  | 古い記録を保持し、新しい記録を無視します   | `INSERT IGNORE INTO ...` |
| `error`   | インポートを中止する                       | `INSERT INTO ...`        |

## も参照してください {#see-also}

- [データを並行してインポートする](/tidb-lightning/tidb-lightning-distributed-import.md)
