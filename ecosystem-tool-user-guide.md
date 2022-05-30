---
title: TiDB Tools Overview
aliases: ['/docs/dev/ecosystem-tool-user-guide/','/docs/dev/reference/tools/user-guide/','/docs/dev/how-to/migrate/from-mysql/','/docs/dev/how-to/migrate/incrementally-from-mysql/','/docs/dev/how-to/migrate/overview/']
---

# TiDBツールの概要 {#tidb-tools-overview}

TiDBは、展開操作、データ管理（インポートとエクスポート、データ移行、バックアップとリカバリなど）、および複雑なOLAPクエリを支援する豊富なツールセットを提供します。必要に応じて適切なツールを選択できます。

## 展開および操作ツール {#deployment-and-operation-tools}

さまざまなシステム環境での展開と運用のニーズを満たすために、TiDBには、TiUPとTiDBOperatorの2つの導入および運用ツールが用意されています。

### 物理マシンまたは仮想マシンにTiDBを展開して運用する {#deploy-and-operate-tidb-on-physical-or-virtual-machines}

[TiUP](/tiup/tiup-overview.md)は、物理マシンまたは仮想マシン上のTiDBパッケージマネージャーです。 TiUPは、TiDB、PD、TiKVなどの複数のTiDBコンポーネントを管理できます。 TiDBエコシステムのコンポーネントを開始するには、1つのTiUPコマンドを実行するだけです。

TiUPは、Golangで記述されたクラスター管理コンポーネントである[TiUPクラスター](https://github.com/pingcap/tiup/tree/master/components/cluster)を提供します。 TiUPクラスターを使用すると、TiDBクラスターの展開、開始、停止、破棄、スケーリング、アップグレードなどの日常のデータベース操作を簡単に実行し、TiDBクラスターパラメーターを管理できます。

TiUPの基本は次のとおりです。

-   [用語と概念](/tiup/tiup-terminology-and-concepts.md)
-   [TiUPを使用してTiDBクラスターをデプロイする](/production-deployment-using-tiup.md)
-   [TiUPコマンドを使用したTiUPコンポーネントの管理](/tiup/tiup-component-management.md)
-   該当するTiDBバージョン：v4.0以降

### KubernetesでTiDBをデプロイして運用する {#deploy-and-operate-tidb-in-kubernetes}

[TiDBオペレーター](https://github.com/pingcap/tidb-operator)は、KubernetesのTiDBクラスターの自動オペレーティングシステムです。展開、アップグレード、スケーリング、バックアップ、フェイルオーバー、構成変更など、TiDBの完全なライフサイクル管理を提供します。 TiDB Operatorを使用すると、TiDBはパブリッククラウドまたはプライベートクラウドにデプロイされたKubernetesクラスターでシームレスに実行できます。

TiDBOperatorの基本は次のとおりです。

-   [TiDBオペレーターアーキテクチャ](https://docs.pingcap.com/tidb-in-kubernetes/stable/architecture)
-   [KubernetesでTiDBOperatorを使い始める](https://docs.pingcap.com/tidb-in-kubernetes/stable/get-started/)
-   該当するTiDBバージョン：v2.1以降

## データ管理ツール {#data-management-tools}

TiDBは、インポートとエクスポート、バックアップと復元、データレプリケーション、データ移行、増分同期、データ検証などの複数のデータ管理ツールを提供します。

### 完全なデータエクスポート {#full-data-export}

[団子](/dumpling-overview.md)は、MySQLまたはTiDBから論理的に完全なデータをエクスポートするためのツールです。

餃子の基本は次のとおりです。

-   入力：MySQL/TiDBクラスター
-   出力：SQL/CSVファイル
-   サポートされているTiDBバージョン：すべてのバージョン
-   Kubernetesのサポート：いいえ

> <strong>ノート：</strong>
>
> PingCAPは、以前はTiDBに固有の拡張機能を備えた[mydumperプロジェクト](https://github.com/maxbube/mydumper)のフォークを維持していました。その後、このフォークは[団子](/dumpling-overview.md)に置き換えられました。これは、Goで書き直され、TiDBに固有のより多くの最適化をサポートします。 mydumperの代わりにDumplingを使用することを強くお勧めします。

### 完全なデータのインポート {#full-data-import}

[TiDB Lightning](/tidb-lightning/tidb-lightning-overview.md) （Lightning）は、TiDBクラスターに大量のデータを完全にインポートするために使用されるツールです。現在、TiDB Lightningは、ダンプリングまたはCSVデータソースを介してエクスポートされたSQLダンプの読み取りをサポートしています。

TiDB Lightningは、次の3つのモードをサポートしています。

-   `local` ：TiDB Lightningは、データを順序付けられたキーと値のペアに解析し、それらをTiKVに直接インポートします。このモードは通常、大量のデータ（TBレベル）を新しいクラスターにインポートするためのものです。インポート中、クラスターはサービスを提供できません。
-   `importer` ：このモードは`local`モードに似ています。このモードを使用するには、キーと値のペアのインポートを支援する追加のコンポーネント`tikv-importer`をデプロイする必要があります。ターゲットクラスターがv4.0以降のバージョンの場合は、 `local`モードを使用することをお勧めします。
-   `tidb` ：このモードはバックエンドとしてTiDB / MySQLを使用します。これは、 `local`モードおよび`importer`モードよりも低速ですが、オンラインで実行できます。また、MySQLへのデータのインポートもサポートしています。

TiDBLightningの基本は次のとおりです。

-   入力データソース：
    -   餃子の出力ファイル
    -   その他の互換性のあるCSVファイル
-   サポートされているTiDBバージョン：v2.1以降
-   Kubernetesのサポート：はい。詳細については、 [TiDB Lightningを使用して、KubernetesのTiDBクラスターにデータをすばやく復元する](https://docs.pingcap.com/tidb-in-kubernetes/stable/restore-data-using-tidb-lightning)を参照してください。

> <strong>ノート：</strong>
>
> ローダーツールはメンテナンスされなくなりました。ローダーに関連するシナリオでは、代わりにTiDBバックエンドを使用することをお勧めします。

### バックアップと復元 {#backup-and-restore}

[復元する](/br/backup-and-restore-tool.md) （BR）は、TiDBクラスターデータの分散バックアップと復元のためのコマンドラインツールです。 BRは、膨大なデータ量のTiDBクラスターを効果的にバックアップおよび復元できます。

BRの基本は次のとおりです。

-   [入力および出力データソース](/br/backup-and-restore-tool.md#types-of-backup-files) ：SST+ `backupmeta`ファイル
-   サポートされているTiDBバージョン：v3.1およびv4.0
-   Kubernetesのサポート：はい。詳細については、 [BRを使用してS3互換ストレージにデータをバックアップする](https://docs.pingcap.com/tidb-in-kubernetes/stable/backup-to-aws-s3-using-br)と[BRを使用してS3互換ストレージからデータを復元する](https://docs.pingcap.com/tidb-in-kubernetes/stable/restore-from-aws-s3-using-br)を参照してください。

### インクリメンタルデータレプリケーション {#incremental-data-replication}

[TiDB Binlog](/tidb-binlog/tidb-binlog-overview.md)は、TiDBクラスターのbinlogを収集し、ほぼリアルタイムの同期とバックアップを提供するツールです。これは、TiDBクラスターをプライマリTiDBクラスターのセカンダリクラスターにするなど、TiDBクラスター間の増分データレプリケーションに使用できます。

TiDBBinlogの基本は次のとおりです。

-   入出力：
    -   入力：TiDBクラスター
    -   出力：TiDBクラスター、MySQL、Kafkaまたは増分バックアップファイル
-   サポートされているTiDBバージョン：v2.1以降
-   Kubernetesのサポート：はい。詳細については、 [TiDBBinlogクラスターの操作](https://docs.pingcap.com/tidb-in-kubernetes/stable/deploy-tidb-binlog)と[KubernetesでのTiDBBinlogDrainerの構成](https://docs.pingcap.com/tidb-in-kubernetes/stable/configure-tidb-binlog-drainer)を参照してください。

### データ移行 {#data-migration}

[TiDBデータ移行](/dm/dm-overview.md) （DM）は、MySQL/MariaDBからTiDBへの完全なデータ移行と増分データ複製をサポートする統合データ複製タスク管理プラットフォームです。

DMの基本は次のとおりです。

-   入力：MySQL / MariaDB
-   出力：TiDBクラスター
-   サポートされているTiDBバージョン：すべてのバージョン
-   Kubernetesのサポート：いいえ、開発中です

データ量がTBレベルを下回っている場合は、DMを使用してMySQL/MariaDBからTiDBに直接データを移行することをお勧めします。移行プロセスには、完全なデータのインポートとエクスポート、および増分データレプリケーションが含まれます。

データ量がTBレベルの場合は、次の手順を実行します。

1.  [団子](/dumpling-overview.md)を使用して、MySQL/MariaDBから完全なデータをエクスポートします。
2.  [TiDB Lightning](/tidb-lightning/tidb-lightning-overview.md)を使用して、手順1でエクスポートしたデータをTiDBクラスターにインポートします。
3.  DMを使用して、MySQL/MariaDBからTiDBに増分データを複製します。

> <strong>ノート：</strong>
>
> Syncerツールはメンテナンスされなくなりました。 Syncerに関連するシナリオでは、代わりにDMのインクリメンタルタスクモードを使用することをお勧めします。

## OLAPクエリツール {#olap-query-tool}

TiDBは、OLAPクエリツールTiSparkを提供します。これにより、ネイティブSparkを使用しているかのようにTiDBテーブルをクエリできます。

### Sparkを使用してTiKVデータソースをクエリする {#query-tikv-data-source-using-spark}

[TiSpark](/tispark-overview.md)は、複雑なOLAPクエリに回答するためにTiKV上でApacheSparkを実行するために構築されたシンレイヤーです。 Sparkプラットフォームと分散TiKVクラスターの両方を活用し、TiDBにシームレスに接着し、ワンストップのハイブリッドトランザクションおよび分析処理（HTAP）ソリューションを提供します。
