---
title: TiDB Experimental Features
summary: Learn the experimental features of TiDB.
aliases: ["/tidb/dev/experimental-features-4.0/"]
---

# TiDB の実験的特徴 {#tidb-experimental-features}

このドキュメントでは、さまざまなバージョンの TiDB の実験的な機能を紹介します。これらの機能を実稼働環境で使用することはお勧めし<strong>ません</strong>。

## パフォーマンス {#performance}

- [いかだエンジン](/tikv-configuration-file.md#raft-engine)。 （v5.4 で導入）
- [<code>PREDICATE COLUMNS</code>の統計収集をサポート](/statistics.md#collect-statistics-on-some-columns)（v5.4 で導入）
- [統計の同期ロードをサポート](/statistics.md#load-statistics)。 （v5.4 で導入）

## 安定 {#stability}

- TiFlash は、データの圧縮または並べ替えによって I / O リソースの使用を制限し、バックグラウンドタスクとフロントエンドデータの読み取りおよび書き込みの間の I / O リソースの競合を軽減します（v5.0 で導入）
- オプティマイザーによるインデックスの選択の安定性を向上させます（v5.0 で導入）
  - 複数列の順序依存関係情報を収集して、統計機能を拡張します。
  - `CMSKetch`とヒストグラムから`TopN`値を削除し、各テーブルのヒストグラムバケットの NDV 情報を追加するなど、統計モジュールをリファクタリングします。索引。
- TiKV が限られたリソースでデプロイされている場合、TiKV のフォアグラウンドが処理する読み取りおよび書き込み要求が多すぎると、バックグラウンドで使用される CPU リソースがそのような要求の処理に使用され、TiKV のパフォーマンスの安定性に影響します。この状況を回避するには、[クォータリミッター](/tikv-configuration-file.md#quota)を使用して、フォアグラウンドで使用される CPU リソースを制限します。 （v6.0 で導入）

## スケジューリング {#scheduling}

- カスケード配置ルール機能。これは、PD がさまざまなタイプのデータに対応するスケジュールを生成するようにガイドするレプリカルールシステムです。さまざまなスケジューリングルールを組み合わせることで、レプリカの数、保存場所、ホストタイプ、Raft 選挙に参加するかどうか、Raft リーダーとして機能するかどうかなど、任意の連続データ範囲の属性を細かく制御できます。詳細については、[カスケード配置ルール](/configure-placement-rules.md)を参照してください。 （v4.0 で導入）
- エラスティックスケジューリング機能。これにより、TiDB クラスターがリアルタイムワークロードに基づいて Kubernetes で動的にスケールアウトおよびスケールインできるようになり、アプリケーションのピーク時のストレスが効果的に軽減され、オーバーヘッドが節約されます。詳細については、[TidbCluster 自動スケーリングを有効にする](https://docs.pingcap.com/tidb-in-kubernetes/stable/enable-tidb-cluster-auto-scaling)を参照してください。 （v4.0 で導入）

## SQL {#sql}

- リストパーティション（v5.0 で導入）
- COLUMNS パーティションの一覧表示（v5.0 で導入）
- [パーティション化されたテーブルの動的プルーニングモード](/partitioned-table.md#dynamic-pruning-mode)。 （v5.1 で導入）
- 式インデックス機能。式インデックスは、関数ベースのインデックスとも呼ばれます。インデックスを作成する場合、インデックスフィールドは特定の列である必要はありませんが、1 つ以上の列から計算された式にすることができます。この機能は、計算ベースのテーブルにすばやくアクセスするのに役立ちます。詳細については、[式インデックス](/sql-statements/sql-statement-create-index.md)を参照してください。 （v4.0 で導入）
- [生成された列](/generated-columns.md)（v2.1 で導入）
- [ユーザー定義変数](/user-defined-variables.md)（v2.1 で導入）
- [JSON データ型](/data-type-json.md)および[JSON 関数](/functions-and-operators/json-functions.md)（v2.1 で導入）
- [意見](/information-schema/information-schema-views.md)（v2.1 で導入）

## 構成管理 {#configuration-management}

- 構成パラメーターを PD に永続的に保存し、構成アイテムの動的な変更をサポートします。 （v4.0 で導入）
- [設定を表示](/sql-statements/sql-statement-show-config.md)（v4.0 で導入）

## データ共有とサブスクリプション {#data-sharing-and-subscription}

- [TiCDC を KafkaConnect（Confluent Platform）と統合する](/ticdc/integrate-confluent-using-ticdc.md)（v5.0 で導入）

## ストレージ {#storage}

- [Titan を無効にする](/storage-engine/titan-configuration.md#disable-titan-experimental)（v4.0 で導入）
- [タイタンレベルマージ](/storage-engine/titan-configuration.md#level-merge-experimental)（v4.0 で導入）
- TiFlash は、ストレージエンジンの新しいデータを複数のハードドライブに分散して、I/O 圧力を共有することをサポートしています。 （v4.0 で導入）

## バックアップと復元 {#backup-and-restoration}

- [RawKV をバックアップする](/br/use-br-command-line-tool.md#back-up-raw-kv-experimental-feature)（v3.1 で導入）

## データ移行 {#data-migration}

- [WebUI を使用する](/dm/dm-webui-guide.md)DM で移行タスクを管理します。 （v6.0 で導入）

## ガベージコレクション {#garbage-collection}

- [グリーン GC](/system-variables.md#tidb_gc_scan_lock_mode-new-in-v50)（v5.0 で導入）

## 診断 {#diagnostics}

- [SQL 診断](/information-schema/information-schema-sql-diagnostics.md)（v4.0 で導入）
- [クラスター診断](/dashboard/dashboard-diagnostics-access.md)（v4.0 で導入）
- [オンラインの安全でない回復](/online-unsafe-recovery.md)（v5.3 で導入）
