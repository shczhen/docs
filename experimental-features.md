---
title: TiDB Experimental Features
summary: Learn the experimental features of TiDB.
aliases: ["/tidb/dev/experimental-features-4.0/"]
---

# TiDB 実験的機能 {#tidb-experimental-features}

このドキュメントでは、TiDB の実験的な機能をさまざまなバージョンで紹介します。**ない** は、本番環境でこれらの機能を使用することをお勧めします。

## パフォーマンス {#performance}

- [ラフトエンジン](/tikv-configuration-file.md#raft-engine)。(v5.4 で導入)
- [`PREDICATE COLUMNS` の統計収集をサポート](/statistics.md#collect-statistics-on-some-columns) (v5.4 で導入)
- [統計の同期ロードをサポート](/statistics.md#load-statistics)。(v5.4 で導入)

## 安定性 {#stability}

- TiFlash は、データを圧縮またはソートすることによって I/O リソースの使用を制限し、バックグラウンドタスクとフロントエンドデータの読み取り/書き込みの間での I/O リソースの競合を軽減します（v5.0 で導入）
- オプティマイザが選択したインデックスの安定性を向上させる (v5.0 で導入)
  - 複数列の順序依存情報を収集して、統計機能を拡張します。
  - `CMSKetch` とヒストグラムから `TopN` 値を削除したり、各テーブルインデックスのヒストグラムバケットの NDV 情報を追加したりするなど、統計モジュールをリファクタリングします。
- TikV が限られたリソースでデプロイされている場合、TikV のフォアグラウンドが処理する読み取りおよび書き込み要求が多すぎると、バックグラウンドで使用される CPU リソースがそのような要求の処理を支援するために占有され、TikV のパフォーマンスの安定性に影響します。この状況を回避するには、[クォータリミッター](/tikv-configuration-file.md#quota) を使用して、フォアグラウンドで使用される CPU リソースを制限できます。(v6.0 で導入)

## スケジューリング {#scheduling}

- カスケード配置ルール機能。これは、PD がさまざまなタイプのデータに対応するスケジュールを生成するように誘導するレプリカルールシステムです。異なるスケジューリングルールを組み合わせることで、レプリカの数、保存場所、ホストタイプ、Raft 選挙に参加するかどうか、Raft リーダーとして機能するかどうかなど、連続するデータ範囲の属性を細かく制御できます。詳細については [カスケード配置ルール](/configure-placement-rules.md) を参照してください。(v4.0 で導入)
- 弾力性のあるスケジューリング機能。これにより、TiDB クラスターはリアルタイムワークロードに基づいて Kubernetes 上で動的にスケールアウトおよびスケールインできます。これにより、アプリケーションのピーク時のストレスを効果的に軽減し、オーバーヘッドを節約できます。詳細については [TIDBcluster 自動スケーリングを有効にする](https://docs.pingcap.com/tidb-in-kubernetes/stable/enable-tidb-cluster-auto-scaling) を参照してください。(v4.0 で導入)

## SQL {#sql}

- リストパーティション (v5.0 で導入)
- 列パーティションを一覧表示 (v5.0 で導入)
- [分割テーブルの動的プルーニングモード](/partitioned-table.md#dynamic-pruning-mode)。(v5.1 で導入)
- 式インデックス機能。式インデックスは、関数ベースインデックスとも呼ばれます。インデックスを作成する場合、インデックスフィールドは特定の列である必要はありませんが、1 つ以上の列から計算される式にすることができます。この機能は、計算ベースのテーブルにすばやくアクセスするのに役立ちます。詳細については [式インデックス](/sql-statements/sql-statement-create-index.md) を参照してください。(v4.0 で導入)
- [生成された列](/generated-columns.md) (v2.1 で導入)
- [ユーザー定義変数](/user-defined-variables.md) (v2.1 で導入)
- [JSON データタイプ](/data-type-json.md) と [JSON 関数](/functions-and-operators/json-functions.md) (v2.1 で導入)
- [表示](/information-schema/information-schema-views.md) (v2.1 で導入)

## 構成管理 {#configuration-management}

- 構成パラメータを PD に永続的に保存し、構成項目の動的な変更をサポートします。(v4.0 で導入)
- [コンフィグを表示](/sql-statements/sql-statement-show-config.md) (v4.0 で導入)

## データ共有と購読 {#data-sharing-and-subscription}

- [TicDC を Kafka Connect と統合する (Confluent プラットフォーム)](/ticdc/integrate-confluent-using-ticdc.md) (v5.0 で導入)

## ストレージ {#storage}

- [タイタンを無効にする](/storage-engine/titan-configuration.md#disable-titan-experimental) (v4.0 で導入)
- [タイタンレベルマージ](/storage-engine/titan-configuration.md#level-merge-experimental) (v4.0 で導入)
- TiFlash は、ストレージエンジンの新しいデータを複数のハードドライブに分散して I/O プレッシャーを分担することをサポートしています。(v4.0 で導入)

## バックアップと復元 {#backup-and-restoration}

- [Raw KV をバックアップする](/br/use-br-command-line-tool.md#back-up-raw-kv-experimental-feature) (v3.1 で導入)

## データ移行 {#data-migration}

- [WebUI を使う](/dm/dm-webui-guide.md) を使用して、DM の移行タスクを管理します。(v6.0 で導入)

## ガベージコレクション {#garbage-collection}

- [グリーン GC](/system-variables.md#tidb_gc_scan_lock_mode-new-in-v50) (v5.0 で導入)

## 診断 {#diagnostics}

- [SQL 診断](/information-schema/information-schema-sql-diagnostics.md) (v4.0 で導入)
- [クラスター診断](/dashboard/dashboard-diagnostics-access.md) (v4.0 で導入)
- [オンライン安全でないリカバリ](/online-unsafe-recovery.md) (v5.3 で導入)
