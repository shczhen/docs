---
title: TiDB Features
summary: Learn about the basic features of TiDB.
aliases: ["/docs/dev/basic-features/"]
---

# TiDB 機能 {#tidb-features}

このドキュメントでは、各 TiDB バージョンでサポートされる機能を示します。実験的機能のサポートは、最終リリースの前に変更される可能性があることに注意してください。

## データ型、関数、演算子 {#data-types-functions-and-operators}

| Data types, functions, and operators                                                 |     6.0      | 5.4          |     5.3      |     5.2      |     5.1      |     5.0      |     4.0      |
| ------------------------------------------------------------------------------------ | :----------: | ------------ | :----------: | :----------: | :----------: | :----------: | :----------: |
| [数値型](/data-type-numeric.md)                                                      |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [日付と時刻のタイプ](/data-type-date-and-time.md)                                    |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [文字列型](/data-type-string.md)                                                     |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [JSON タイプ](/data-type-json.md)                                                    | Experimental | Experimental | Experimental | Experimental | Experimental | Experimental | Experimental |
| [制御フロー機能](/functions-and-operators/control-flow-functions.md)                 |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [文字列関数](/functions-and-operators/string-functions.md)                           |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [数値関数と演算子](/functions-and-operators/numeric-functions-and-operators.md)      |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [日付と時刻の関数](/functions-and-operators/date-and-time-functions.md)              |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [ビット関数と演算子](/functions-and-operators/bit-functions-and-operators.md)        |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [キャスト関数と演算子](/functions-and-operators/cast-functions-and-operators.md)     |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [暗号化と圧縮機能](/functions-and-operators/encryption-and-compression-functions.md) |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [情報機能](/functions-and-operators/information-functions.md)                        |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [JSON 関数](/functions-and-operators/json-functions.md)                              | Experimental | Experimental | Experimental | Experimental | Experimental | Experimental | Experimental |
| [集計関数](/functions-and-operators/aggregate-group-by-functions.md)                 |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [ウィンドウ機能](/functions-and-operators/window-functions.md)                       |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [その他の機能](/functions-and-operators/miscellaneous-functions.md)                  |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [オペレーター](/functions-and-operators/operators.md)                                |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [文字セットと照合](/character-set-and-collation.md) \[^1]                            |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |

## 索引付けと制約 {#indexing-and-constraints}

| Indexing and constraints                                                         |     6.0      | 5.4          |     5.3      |     5.2      |     5.1      |     5.0      |     4.0      |
| -------------------------------------------------------------------------------- | :----------: | ------------ | :----------: | :----------: | :----------: | :----------: | :----------: |
| [式インデックス](/sql-statements/sql-statement-create-index.md#expression-index) | Experimental | Experimental | Experimental | Experimental | Experimental | Experimental | Experimental |
| [列指向ストレージ (TiFlash)](/tiflash/tiflash-overview.md)                       |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [RocksDB エンジン](/storage-engine/rocksdb-overview.md)                          |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [Titan プラグイン](/storage-engine/titan-overview.md)                            |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [不可視索引](/sql-statements/sql-statement-add-index.md)                         |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      N       |
| [複合 PLACEHOLDER-1-PLACEHOLDER-E}](/constraints.md)                             |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [ユニークインデックス](/constraints.md)                                          |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [整数 `PRIMARY KEY` のクラスター化インデックス](/constraints.md)                 |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [複合キーまたは非整数キーのクラスタ化インデックス](/constraints.md)              |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      N       |

## SQL ステートメント {#sql-statements}

| SQL statements \[^2]                                                                             |     6.0      | 5.4          |     5.3      |     5.2      |     5.1      |     5.0      |     4.0      |
| ------------------------------------------------------------------------------------------------ | :----------: | ------------ | :----------: | :----------: | :----------: | :----------: | :----------: |
| Basic `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `REPLACE`                                          |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| `INSERT ON DUPLICATE KEY UPDATE`                                                                 |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| `LOAD DATA INFILE`                                                                               |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| `SELECT INTO OUTFILE`                                                                            |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| `INNER JOIN`, `LEFT\|RIGHT [OUTER] JOIN`                                                         |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| `UNION`, `UNION ALL`                                                                             |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [`EXCEPT` および PLACEHOLDER-2-PLACEHOLDER-E} 演算子](/functions-and-operators/set-operators.md) |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      N       |
| `GROUP BY`, `ORDER BY`                                                                           |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [ウィンドウ機能](/functions-and-operators/window-functions.md)                                   |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [共通テーブル式 (CTE)](/sql-statements/sql-statement-with.md)                                    |      Y       | Y            |      Y       |      Y       |      Y       |      N       |      N       |
| `START TRANSACTION`, `COMMIT`, `ROLLBACK`                                                        |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [`EXPLAIN`](/sql-statements/sql-statement-explain.md)                                            |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [`EXPLAIN ANALYZE`](/sql-statements/sql-statement-explain-analyze.md)                            |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [ユーザー定義変数](/user-defined-variables.md)                                                   | Experimental | Experimental | Experimental | Experimental | Experimental | Experimental | Experimental |

## 高度な SQL 機能 {#advanced-sql-features}

| Advanced SQL features                                            | 6.0 | 5.4          |     5.3      |     5.2      |     5.1      |     5.0      |     4.0      |
| ---------------------------------------------------------------- | :-: | ------------ | :----------: | :----------: | :----------: | :----------: | :----------: |
| [準備済みステートメントキャッシュ](/sql-prepared-plan-cache.md)  |  Y  | Y            |      Y       | Experimental | Experimental | Experimental | Experimental |
| [SQL プラン管理 (SPM)](/sql-plan-management.md)                  |  Y  | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [コプロセッサーキャッシュ](/coprocessor-cache.md)                |  Y  | Y            |      Y       |      Y       |      Y       |      Y       | Experimental |
| [古い読み取り](/stale-read.md)                                   |  Y  | Y            |      Y       |      Y       |      Y       |      N       |      N       |
| [フォロワーの読み取り](/follower-read.md)                        |  Y  | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [履歴データの読み取り (tidb_snapshot)](/read-historical-data.md) |  Y  | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [オプティマイザーのヒント](/optimizer-hints.md)                  |  Y  | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [MPP 実行エンジン](/explain-mpp.md)                              |  Y  | Y            |      Y       |      Y       |      Y       |      Y       |      N       |
| [索引マージ](/explain-index-merge.md)                            |  Y  | Y            | Experimental | Experimental | Experimental | Experimental | Experimental |
| [SQL の配置ルール](/placement-rules-in-sql.md)                   |  Y  | Experimental | Experimental |      N       |      N       |      N       |      N       |

## データ定義言語 (DDL) {#data-definition-language-ddl}

| Data definition language (DDL)                                               |     6.0      | 5.4          |     5.3      |     5.2      |     5.1      |     5.0      |     4.0      |
| ---------------------------------------------------------------------------- | :----------: | ------------ | :----------: | :----------: | :----------: | :----------: | :----------: |
| Basic `CREATE`, `DROP`, `ALTER`, `RENAME`, `TRUNCATE`                        |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [生成された列](/generated-columns.md)                                        | Experimental | Experimental | Experimental | Experimental | Experimental | Experimental | Experimental |
| [ビュー](/views.md)                                                          |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [シーケンス](/sql-statements/sql-statement-create-sequence.md)               |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [自動インクリメント](/auto-increment.md)                                     |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [自動ランダム](/auto-random.md)                                              |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [DDL アルゴリズムアサーション](/sql-statements/sql-statement-alter-table.md) |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| Multi schema change: add column(s)                                           | Experimental | Experimental | Experimental | Experimental | Experimental | Experimental | Experimental |
| [列タイプを変更](/sql-statements/sql-statement-modify-column.md)             |      Y       | Y            |      Y       |      Y       |      Y       |      N       |      N       |
| [テンポラリテーブル](/temporary-tables.md)                                   |      Y       | Y            |      Y       |      N       |      N       |      N       |      N       |

## トランザクション {#transactions}

| Transactions                                                                          | 6.0 | 5.4 | 5.3 | 5.2 | 5.1 | 5.0 | 4.0 |
| ------------------------------------------------------------------------------------- | :-- | --- | :-: | :-: | :-: | :-: | :-: |
| [非同期コミット](/system-variables.md#tidb_enable_async_commit-new-in-v50)            | Y   | Y   |  Y  |  Y  |  Y  |  Y  |  N  |
| [1 個](/system-variables.md#tidb_enable_1pc-new-in-v50)                               | Y   | Y   |  Y  |  Y  |  Y  |  Y  |  N  |
| [大規模トランザクション (10GB)](/transaction-overview.md#transaction-size-limit)      | Y   | Y   |  Y  |  Y  |  Y  |  Y  |  Y  |
| [悲観的な取引](/pessimistic-transaction.md)                                           | Y   | Y   |  Y  |  Y  |  Y  |  Y  |  Y  |
| [楽観的な取引](/optimistic-transaction.md)                                            | Y   | Y   |  Y  |  Y  |  Y  |  Y  |  Y  |
| [繰り返し可能な読み取り分離 (スナップショット分離)](/transaction-isolation-levels.md) | Y   | Y   |  Y  |  Y  |  Y  |  Y  |  Y  |
| [読み取りコミット分離](/transaction-isolation-levels.md)                              | Y   | Y   |  Y  |  Y  |  Y  |  Y  |  Y  |

## パーティショニング {#partitioning}

| Partitioning                                                   |     6.0      | 5.4          |     5.3      |     5.2      |     5.1      |     5.0      | 4.0 |
| -------------------------------------------------------------- | :----------: | ------------ | :----------: | :----------: | :----------: | :----------: | :-: |
| [レンジ・パーティショニング](/partitioned-table.md)            |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |  Y  |
| [ハッシュパーティショニング](/partitioned-table.md)            |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |  Y  |
| [リスト分割](/partitioned-table.md)                            | Experimental | Experimental | Experimental | Experimental | Experimental | Experimental |  N  |
| [リスト COLUMNS パーティション](/partitioned-table.md)         | Experimental | Experimental | Experimental | Experimental | Experimental | Experimental |  N  |
| [`EXCHANGE PARTITION`](/partitioned-table.md)                  | Experimental | Experimental | Experimental | Experimental | Experimental | Experimental |  N  |
| [動的プルーニング](/partitioned-table.md#dynamic-pruning-mode) | Experimental | Experimental | Experimental | Experimental | Experimental |      N       |  N  |

## 統計情報 {#statistics}

| Statistics                                                |     6.0      | 5.4          |     5.3      |     5.2      |     5.1      |     5.0      |     4.0      |
| --------------------------------------------------------- | :----------: | ------------ | :----------: | :----------: | :----------: | :----------: | :----------: |
| [cmSketch](/statistics.md)                                |  Deprecated  | Deprecated   |  Deprecated  |  Deprecated  |  Deprecated  |  Deprecated  |      Y       |
| [ヒストグラム](/statistics.md)                            |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [拡張統計 (複数列)](/statistics.md)                       | Experimental | Experimental | Experimental | Experimental | Experimental | Experimental |      N       |
| [統計フィードバック](/statistics.md#automatic-update)     |  Deprecated  | Deprecated   | Experimental | Experimental | Experimental | Experimental | Experimental |
| [高速分析](/system-variables.md#tidb_enable_fast_analyze) | Experimental | Experimental | Experimental | Experimental | Experimental | Experimental | Experimental |

## セキュリティー {#security}

| Security                                                                     | 6.0 | 5.4 | 5.3 | 5.2 | 5.1 | 5.0 | 4.0 |
| ---------------------------------------------------------------------------- | :-: | --- | :-: | :-: | :-: | :-: | :-: |
| [透過層セキュリティ (TLS)](/enable-tls-between-clients-and-servers.md)       |  Y  | Y   |  Y  |  Y  |  Y  |  Y  |  Y  |
| [保管時の暗号化 (TDE)](/encryption-at-rest.md)                               |  Y  | Y   |  Y  |  Y  |  Y  |  Y  |  Y  |
| [ロールベース認証 (RBAC)](/role-based-access-control.md)                     |  Y  | Y   |  Y  |  Y  |  Y  |  Y  |  Y  |
| [証明書ベースの認証](/certificate-authentication.md)                         |  Y  | Y   |  Y  |  Y  |  Y  |  Y  |  Y  |
| `caching_sha2_password` authentication                                       |  Y  | Y   |  Y  |  Y  |  N  |  N  |  N  |
| [MySQL 互換 `GRANT` システム](/privilege-management.md)                      |  Y  | Y   |  Y  |  Y  |  Y  |  Y  |  Y  |
| [動的権限](/privilege-management.md#dynamic-privileges)                      |  Y  | Y   |  Y  |  Y  |  Y  |  N  |  N  |
| [セキュリティ強化モード](/system-variables.md#tidb_enable_enhanced_security) |  Y  | Y   |  Y  |  Y  |  Y  |  N  |  N  |
| [編集されたログファイル](/log-redaction.md)                                  |  Y  | Y   |  Y  |  Y  |  Y  |  Y  |  N  |

## データのインポートとエクスポート {#data-import-and-export}

| Data import and export                                                             |    6.0     |    5.4     |    5.3     |    5.2     |    5.1     |    5.0     |    4.0     |
| ---------------------------------------------------------------------------------- | :--------: | :--------: | :--------: | :--------: | :--------: | :--------: | :--------: |
| [高速インポーター (TiDB ライトニング)](/tidb-lightning/tidb-lightning-overview.md) |     Y      |     Y      |     Y      |     Y      |     Y      |     Y      |     Y      |
| mydumper logical dumper                                                            | Deprecated | Deprecated | Deprecated | Deprecated | Deprecated | Deprecated | Deprecated |
| [餃子論理ダンパー](/dumpling-overview.md)                                          |     Y      |     Y      |     Y      |     Y      |     Y      |     Y      |     Y      |
| [トランザクション `LOAD DATA`](/sql-statements/sql-statement-load-data.md)         |     Y      |     Y      |     Y      |     Y      |     Y      |     Y      |  N \[^3]   |
| [データベース移行ツールキット (DM)](/migration-overview.md)                        |     Y      |     Y      |     Y      |     Y      |     Y      |     Y      |     Y      |
| [TiDB ブログ](/tidb-binlog/tidb-binlog-overview.md)                                |     Y      |     Y      |     Y      |     Y      |     Y      |     Y      |     Y      |
| [変更データキャプチャ (CDC)](/ticdc/ticdc-overview.md)                             |     Y      |     Y      |     Y      |     Y      |     Y      |     Y      |     Y      |

## 管理、可観測性、ツール {#management-observability-and-tools}

| Management, observability, and tools                                                      |     6.0      | 5.4          |     5.3      |     5.2      |     5.1      |     5.0      |     4.0      |
| ----------------------------------------------------------------------------------------- | :----------: | ------------ | :----------: | :----------: | :----------: | :----------: | :----------: |
| [TiDB ダッシュボード UI](/dashboard/dashboard-intro.md)                                   |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [TiDB ダッシュボード継続的プロファイリング](/dashboard/continuous-profiling.md)           |      Y       | Experimental | Experimental |      N       |      N       |      N       |      N       |
| [TiDB ダッシュボードトップ SQL](/dashboard/top-sql.md)                                    |      Y       | Experimental |      N       |      N       |      N       |      N       |      N       |
| [TiDB ダッシュボード SQL 診断](/information-schema/information-schema-sql-diagnostics.md) | Experimental | Experimental | Experimental | Experimental | Experimental | Experimental | Experimental |
| [情報スキーマ](/information-schema/information-schema.md)                                 |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [メトリックスキーマ](/metrics-schema.md)                                                  |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [ステートメントの概要表](/statement-summary-tables.md)                                    |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [スロー・クエリー・ログ](/identify-slow-queries.md)                                       |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [tiUP デプロイ](/tiup/tiup-overview.md)                                                   |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| Ansible deployment                                                                        |      N       | N            |      N       |      N       |      N       |      N       |  Deprecated  |
| [Kubernetes オペレーター](https://docs.pingcap.com/tidb-in-kubernetes/)                   |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [組み込みの物理バックアップ](/br/backup-and-restore-use-cases.md)                         |      Y       | Y            |      Y       |      Y       |      Y       |      Y       |      Y       |
| [グローバルキル](/sql-statements/sql-statement-kill.md)                                   | Experimental | Experimental | Experimental | Experimental | Experimental | Experimental | Experimental |
| [ビューをロック](/information-schema/information-schema-data-lock-waits.md)               |      Y       | Y            |      Y       |      Y       | Experimental | Experimental | Experimental |
| [`SHOW CONFIG`](/sql-statements/sql-statement-show-config.md)                             | Experimental | Experimental | Experimental | Experimental | Experimental | Experimental | Experimental |
| [`SET CONFIG`](/dynamic-config.md)                                                        | Experimental | Experimental | Experimental | Experimental | Experimental | Experimental | Experimental |
| [DM WebUI](/dm/dm-webui-guide.md)                                                         | Experimental | N            |      N       |      N       |      N       |      N       |      N       |

\[^1]: TiDB は latin1 を utf8 のサブセットとして誤って扱います。詳細については、[TiDB #18955](https://github.com/pingcap/tidb/issues/18955) を参照してください。

\[^2]: サポートされている SQL ステートメントの完全なリストについては、[ステートメントリファレンス](/sql-statements/sql-statement-select.md) を参照してください。

\[^3]: TiDB v4.0 では、`LOAD DATA` トランザクションはアトミック性を保証しません。
