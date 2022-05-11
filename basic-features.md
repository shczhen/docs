---
title: TiDB Features
summary: Learn about the basic features of TiDB.
aliases: ["/docs/dev/basic-features/"]
---

# TiDB の機能 {#tidb-features}

このドキュメントには、各 TiDB バージョンでサポートされている機能がリストされています。実験的な機能のサポートは、最終リリースの前に変更される可能性があることに注意してください。

## データ型、関数、および演算子 {#data-types-functions-and-operators}

| データ型、関数、および演算子                                                             |  6.0   | 5.4    |  5.3   |  5.2   |  5.1   |  5.0   | 4.0 4.0 |
| ---------------------------------------------------------------------------------------- | :----: | ------ | :----: | :----: | :----: | :----: | :-----: |
| [数値タイプ](/data-type-numeric.md)                                                      |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [日付と時刻のタイプ](/data-type-date-and-time.md)                                        |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [文字列タイプ](/data-type-string.md)                                                     |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [JSON タイプ](/data-type-json.md)                                                        | 実験的 | 実験的 | 実験的 | 実験的 | 実験的 | 実験的 | 実験的  |
| [制御フロー機能](/functions-and-operators/control-flow-functions.md)                     |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [文字列関数](/functions-and-operators/string-functions.md)                               |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [数値関数と演算子](/functions-and-operators/numeric-functions-and-operators.md)          |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [日付と時刻の関数](/functions-and-operators/date-and-time-functions.md)                  |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [ビット関数と演算子](/functions-and-operators/bit-functions-and-operators.md)            |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [キャスト関数と演算子](/functions-and-operators/cast-functions-and-operators.md)         |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [暗号化および圧縮機能](/functions-and-operators/encryption-and-compression-functions.md) |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [情報機能](/functions-and-operators/information-functions.md)                            |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [JSON 関数](/functions-and-operators/json-functions.md)                                  | 実験的 | 実験的 | 実験的 | 実験的 | 実験的 | 実験的 | 実験的  |
| [集計関数](/functions-and-operators/aggregate-group-by-functions.md)                     |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [ウィンドウ関数](/functions-and-operators/window-functions.md)                           |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [その他の機能](/functions-and-operators/miscellaneous-functions.md)                      |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [オペレーター](/functions-and-operators/operators.md)                                    |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [キャラクターセットと照合](/character-set-and-collation.md) [^ 1]                        |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |

## インデックス作成と制約 {#indexing-and-constraints}

| インデックス作成と制約                                                             |  6.0   | 5.4    |  5.3   |  5.2   |  5.1   |  5.0   | 4.0 4.0 |
| ---------------------------------------------------------------------------------- | :----: | ------ | :----: | :----: | :----: | :----: | :-----: |
| [式のインデックス](/sql-statements/sql-statement-create-index.md#expression-index) | 実験的 | 実験的 | 実験的 | 実験的 | 実験的 | 実験的 | 実験的  |
| [列型ストレージ（TiFlash）](/tiflash/tiflash-overview.md)                          |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [RocksDB エンジン](/storage-engine/rocksdb-overview.md)                            |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [Titan プラグイン](/storage-engine/titan-overview.md)                              |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [見えないインデックス](/sql-statements/sql-statement-add-index.md)                 |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    N    |
| [複合<code>PRIMARY KEY</code>](/constraints.md)                                    |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [一意のインデックス](/constraints.md)                                              |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [整数の<code>PRIMARY KEY</code>のクラスター化インデックス](/constraints.md)        |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [複合キーまたは非整数キーのクラスター化されたインデックス](/constraints.md)        |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    N    |

## SQL ステートメント {#sql-statements}

| SQL ステートメント[^2]                                                                             |  6.0   | 5.4    |  5.3   |  5.2   |  5.1   |  5.0   | 4.0 4.0 |
| -------------------------------------------------------------------------------------------------- | :----: | ------ | :----: | :----: | :----: | :----: | :-----: |
| 基本`SELECT`、`INSERT`、`UPDATE`、`DELETE`、`REPLACE`                                              |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| `INSERT ON DUPLICATE KEY UPDATE`                                                                   |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| `LOAD DATA INFILE`                                                                                 |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| `SELECT INTO OUTFILE`                                                                              |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| `INNER JOIN`、`LEFT\|RIGHT [OUTER] JOIN`                                                           |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| `UNION`、`UNION ALL`                                                                               |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [<code>EXCEPT</code>および<code>INTERSECT</code>演算子](/functions-and-operators/set-operators.md) |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    N    |
| `GROUP BY`、`ORDER BY`                                                                             |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [ウィンドウ関数](/functions-and-operators/window-functions.md)                                     |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [共通テーブル式（CTE）](/sql-statements/sql-statement-with.md)                                     |   Y    | Y      |   Y    |   Y    |   Y    |   N    |    N    |
| `START TRANSACTION`、`COMMIT`、`ROLLBACK`                                                          |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [<code>EXPLAIN</code>](/sql-statements/sql-statement-explain.md)                                   |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [<code>EXPLAIN ANALYZE</code>](/sql-statements/sql-statement-explain-analyze.md)                   |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [ユーザー定義変数](/user-defined-variables.md)                                                     | 実験的 | 実験的 | 実験的 | 実験的 | 実験的 | 実験的 | 実験的  |

## 高度な SQL 機能 {#advanced-sql-features}

| 高度な SQL 機能                                                   | 6.0 | 5.4    |  5.3   |  5.2   |  5.1   |  5.0   | 4.0 4.0 |
| ----------------------------------------------------------------- | :-: | ------ | :----: | :----: | :----: | :----: | :-----: |
| [プリペアドステートメントキャッシュ](/sql-prepared-plan-cache.md) |  Y  | Y      |   Y    | 実験的 | 実験的 | 実験的 | 実験的  |
| [SQL 計画管理（SPM）](/sql-plan-management.md)                    |  Y  | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [コプロセッサーキャッシュ](/coprocessor-cache.md)                 |  Y  | Y      |   Y    |   Y    |   Y    |   Y    | 実験的  |
| [古い読み取り](/stale-read.md)                                    |  Y  | Y      |   Y    |   Y    |   Y    |   N    |    N    |
| [フォロワーの読み取り](/follower-read.md)                         |  Y  | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [履歴データの読み取り（tidb_snapshot）](/read-historical-data.md) |  Y  | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [オプティマイザーのヒント](/optimizer-hints.md)                   |  Y  | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [MPP Exection Engine](/explain-mpp.md)                            |  Y  | Y      |   Y    |   Y    |   Y    |   Y    |    N    |
| [インデックスマージ](/explain-index-merge.md)                     |  Y  | Y      | 実験的 | 実験的 | 実験的 | 実験的 | 実験的  |
| [SQL の配置ルール](/placement-rules-in-sql.md)                    |  Y  | 実験的 | 実験的 |   N    |   N    |   N    |    N    |

## データ定義言語（DDL） {#data-definition-language-ddl}

| データ定義言語（DDL）                                                        |  6.0   | 5.4    |  5.3   |  5.2   |  5.1   |  5.0   | 4.0 4.0 |
| ---------------------------------------------------------------------------- | :----: | ------ | :----: | :----: | :----: | :----: | :-----: |
| 基本`CREATE`、`DROP`、`ALTER`、`RENAME`、`TRUNCATE`                          |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [生成された列](/generated-columns.md)                                        | 実験的 | 実験的 | 実験的 | 実験的 | 実験的 | 実験的 | 実験的  |
| [ビュー](/views.md)                                                          |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [シーケンス](/sql-statements/sql-statement-create-sequence.md)               |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [自動増加](/auto-increment.md)                                               |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [自動ランダム](/auto-random.md)                                              |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [DDL アルゴリズムアサーション](/sql-statements/sql-statement-alter-table.md) |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| マルチスキーマの変更：列を追加                                               | 実験的 | 実験的 | 実験的 | 実験的 | 実験的 | 実験的 | 実験的  |
| [列タイプを変更する](/sql-statements/sql-statement-modify-column.md)         |   Y    | Y      |   Y    |   Y    |   Y    |   N    |    N    |
| [一時的なテーブル](/temporary-tables.md)                                     |   Y    | Y      |   Y    |   N    |   N    |   N    |    N    |

## トランザクション {#transactions}

| トランザクション                                                                      | 6.0 | 5.4 | 5.3 | 5.2 | 5.1 | 5.0 | 4.0 4.0 |
| ------------------------------------------------------------------------------------- | :-- | --- | :-: | :-: | :-: | :-: | :-----: |
| [非同期コミット](/system-variables.md#tidb_enable_async_commit-new-in-v50)            | Y   | Y   |  Y  |  Y  |  Y  |  Y  |    N    |
| [1 個](/system-variables.md#tidb_enable_1pc-new-in-v50)                               | Y   | Y   |  Y  |  Y  |  Y  |  Y  |    N    |
| [大規模なトランザクション（10GB）](/transaction-overview.md#transaction-size-limit)   | Y   | Y   |  Y  |  Y  |  Y  |  Y  |    Y    |
| [悲観的な取引](/pessimistic-transaction.md)                                           | Y   | Y   |  Y  |  Y  |  Y  |  Y  |    Y    |
| [楽観的な取引](/optimistic-transaction.md)                                            | Y   | Y   |  Y  |  Y  |  Y  |  Y  |    Y    |
| [繰り返し可能-読み取り分離（スナップショット分離）](/transaction-isolation-levels.md) | Y   | Y   |  Y  |  Y  |  Y  |  Y  |    Y    |
| [読み取り-コミットされた分離](/transaction-isolation-levels.md)                       | Y   | Y   |  Y  |  Y  |  Y  |  Y  |    Y    |

## パーティショニング {#partitioning}

| パーティショニング                                                  |  6.0   | 5.4    |  5.3   |  5.2   |  5.1   |  5.0   | 4.0 4.0 |
| ------------------------------------------------------------------- | :----: | ------ | :----: | :----: | :----: | :----: | :-----: |
| [範囲分割](/partitioned-table.md)                                   |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [ハッシュ分割](/partitioned-table.md)                               |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [リストのパーティション化](/partitioned-table.md)                   | 実験的 | 実験的 | 実験的 | 実験的 | 実験的 | 実験的 |    N    |
| [COLUMNS パーティショニングを一覧表示します](/partitioned-table.md) | 実験的 | 実験的 | 実験的 | 実験的 | 実験的 | 実験的 |    N    |
| [<code>EXCHANGE PARTITION</code>](/partitioned-table.md)            | 実験的 | 実験的 | 実験的 | 実験的 | 実験的 | 実験的 |    N    |
| [動的剪定](/partitioned-table.md#dynamic-pruning-mode)              | 実験的 | 実験的 | 実験的 | 実験的 | 実験的 |   N    |    N    |

## 統計学 {#statistics}

| 統計学                                                    |  6.0   | 5.4    |  5.3   |  5.2   |  5.1   |  5.0   | 4.0 4.0 |
| --------------------------------------------------------- | :----: | ------ | :----: | :----: | :----: | :----: | :-----: |
| [CMSketch](/statistics.md)                                | 非推奨 | 非推奨 | 非推奨 | 非推奨 | 非推奨 | 非推奨 |    Y    |
| [ヒストグラム](/statistics.md)                            |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [拡張統計（複数の列）](/statistics.md)                    | 実験的 | 実験的 | 実験的 | 実験的 | 実験的 | 実験的 |    N    |
| [統計フィードバック](/statistics.md#automatic-update)     | 非推奨 | 非推奨 | 実験的 | 実験的 | 実験的 | 実験的 | 実験的  |
| [高速分析](/system-variables.md#tidb_enable_fast_analyze) | 実験的 | 実験的 | 実験的 | 実験的 | 実験的 | 実験的 | 実験的  |

## 安全 {#security}

| 安全                                                                                        | 6.0 | 5.4 | 5.3 | 5.2 | 5.1 | 5.0 | 4.0 4.0 |
| ------------------------------------------------------------------------------------------- | :-: | --- | :-: | :-: | :-: | :-: | :-----: |
| [トランスペアレントレイヤーセキュリティ（TLS）](/enable-tls-between-clients-and-servers.md) |  Y  | Y   |  Y  |  Y  |  Y  |  Y  |    Y    |
| [保管時の暗号化（TDE）](/encryption-at-rest.md)                                             |  Y  | Y   |  Y  |  Y  |  Y  |  Y  |    Y    |
| [役割ベースの認証（RBAC）](/role-based-access-control.md)                                   |  Y  | Y   |  Y  |  Y  |  Y  |  Y  |    Y    |
| [証明書ベースの認証](/certificate-authentication.md)                                        |  Y  | Y   |  Y  |  Y  |  Y  |  Y  |    Y    |
| `caching_sha2_password`認証                                                                 |  Y  | Y   |  Y  |  Y  |  N  |  N  |    N    |
| [MySQL 互換<code>GRANT</code>システム](/privilege-management.md)                            |  Y  | Y   |  Y  |  Y  |  Y  |  Y  |    Y    |
| [動的特権](/privilege-management.md#dynamic-privileges)                                     |  Y  | Y   |  Y  |  Y  |  Y  |  N  |    N    |
| [セキュリティ強化モード](/system-variables.md#tidb_enable_enhanced_security)                |  Y  | Y   |  Y  |  Y  |  Y  |  N  |    N    |
| [編集されたログファイル](/log-redaction.md)                                                 |  Y  | Y   |  Y  |  Y  |  Y  |  Y  |    N    |

## データのインポートとエクスポート {#data-import-and-export}

| データのインポートとエクスポート                                                     |  6.0   |  5.4   |  5.3   |  5.2   |  5.1   |  5.0   | 4.0 4.0 |
| ------------------------------------------------------------------------------------ | :----: | :----: | :----: | :----: | :----: | :----: | :-----: |
| [Fast Importer（TiDB Lightning）](/tidb-lightning/tidb-lightning-overview.md)        |   Y    |   Y    |   Y    |   Y    |   Y    |   Y    |    Y    |
| mydumper 論理ダンパー                                                                | 非推奨 | 非推奨 | 非推奨 | 非推奨 | 非推奨 | 非推奨 | 非推奨  |
| [餃子論理ダンパー](/dumpling-overview.md)                                            |   Y    |   Y    |   Y    |   Y    |   Y    |   Y    |    Y    |
| [トランザクション<code>LOAD DATA</code>](/sql-statements/sql-statement-load-data.md) |   Y    |   Y    |   Y    |   Y    |   Y    |   Y    | N [^ 3] |
| [データベース移行ツールキット（DM）](/migration-overview.md)                         |   Y    |   Y    |   Y    |   Y    |   Y    |   Y    |    Y    |
| [TiDB Binlog](/tidb-binlog/tidb-binlog-overview.md)                                  |   Y    |   Y    |   Y    |   Y    |   Y    |   Y    |    Y    |
| [変更データキャプチャ（CDC）](/ticdc/ticdc-overview.md)                              |   Y    |   Y    |   Y    |   Y    |   Y    |   Y    |    Y    |

## 管理、可観測性、およびツール {#management-observability-and-tools}

| 管理、可観測性、およびツール                                                              |  6.0   | 5.4    |  5.3   |  5.2   |  5.1   |  5.0   | 4.0 4.0 |
| ----------------------------------------------------------------------------------------- | :----: | ------ | :----: | :----: | :----: | :----: | :-----: |
| [TiDB ダッシュボード UI](/dashboard/dashboard-intro.md)                                   |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [TiDB ダッシュボードの継続的なプロファイリング](/dashboard/continuous-profiling.md)       |   Y    | 実験的 | 実験的 |   N    |   N    |   N    |    N    |
| [TiDB ダッシュボードトップ SQL](/dashboard/top-sql.md)                                    |   Y    | 実験的 |   N    |   N    |   N    |   N    |    N    |
| [TiDB ダッシュボード SQL 診断](/information-schema/information-schema-sql-diagnostics.md) | 実験的 | 実験的 | 実験的 | 実験的 | 実験的 | 実験的 | 実験的  |
| [情報スキーマ](/information-schema/information-schema.md)                                 |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [メトリクススキーマ](/metrics-schema.md)                                                  |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [ステートメント要約表](/statement-summary-tables.md)                                      |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [遅いクエリログ](/identify-slow-queries.md)                                               |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [TiUP の展開](/tiup/tiup-overview.md)                                                     |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| Ansible デプロイメント                                                                    |   N    | N      |   N    |   N    |   N    |   N    | 非推奨  |
| [Kubernetes オペレーター](https://docs.pingcap.com/tidb-in-kubernetes/)                   |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [組み込みの物理バックアップ](/br/backup-and-restore-use-cases.md)                         |   Y    | Y      |   Y    |   Y    |   Y    |   Y    |    Y    |
| [グローバルキル](/sql-statements/sql-statement-kill.md)                                   | 実験的 | 実験的 | 実験的 | 実験的 | 実験的 | 実験的 | 実験的  |
| [ビューをロックする](/information-schema/information-schema-data-lock-waits.md)           |   Y    | Y      |   Y    |   Y    | 実験的 | 実験的 | 実験的  |
| [<code>SHOW CONFIG</code>](/sql-statements/sql-statement-show-config.md)                  | 実験的 | 実験的 | 実験的 | 実験的 | 実験的 | 実験的 | 実験的  |
| [<code>SET CONFIG</code>](/dynamic-config.md)                                             | 実験的 | 実験的 | 実験的 | 実験的 | 実験的 | 実験的 | 実験的  |
| [DM WebUI](/dm/dm-webui-guide.md)                                                         | 実験的 | N      |   N    |   N    |   N    |   N    |    N    |

[^1]: TiDB は、latin1 を utf8 のサブセットとして誤って扱います。詳細については、[TiDB＃18955](https://github.com/pingcap/tidb/issues/18955)を参照してください。
[^2]: サポートされている SQL ステートメントの完全なリストについては、[ステートメントリファレンス](/sql-statements/sql-statement-select.md)を参照してください。
[^3]: TiDB v4.0 の場合、`LOAD DATA`トランザクションはアトミック性を保証しません。
