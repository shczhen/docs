---
title: TiDB Ecosystem Tools Overview
summary: Learn an overview of the TiDB ecosystem tools.
---

# TiDB エコシステムツールの概要 {#tidb-ecosystem-tools-overview}

TiDB は、完全なデータ移行、増分データ移行、バックアップと復元、データ複製など、さまざまなシナリオに対応する複数のデータ移行ツールを提供します。

このドキュメントでは、これらのツールのユーザーシナリオ、利点、および制限を紹介します。必要に応じて適切なツールを選択できます。

<!--The following diagram shows the user scenario of each migration tool.

!TiDB Migration Tools media/migration-tools.png-->

次の表は、ユーザーシナリオ、サポートされている移行ツールのアップストリームおよびダウンストリームを示しています。

| ツール名                                                                    | ユーザーシナリオ                                                                                                                                                                             | アップストリーム（またはインポートされたソースファイル）                                                                        | ダウンストリーム（または出力ファイル）          | 利点                                                                                                                                                                                                                                                                                             | 制限                                                                                                                                                                                                                                                                                                 |
| :-------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------ | :---------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [TiDB データ移行（DM）](/dm/dm-overview.md)                                 | MySQL 互換データベースから TiDB へのデータ移行                                                                                                                                               | MySQL、MariaDB、Aurora、MySQL                                                                                                   | TiDB                                            | <li>完全なデータ移行と増分レプリケーションをサポートする便利で統合されたデータ移行タスク管理ツール</li><li>フィルタリングテーブルと操作をサポートする</li><li>シャードのマージと移行をサポートする</li>                                                                                          | データのインポート速度は、TiDB Lighting の TiDB バックエンドとほぼ同じであり、TiDBLighting のローカルバックエンドよりもはるかに低速です。したがって、DM を使用して、1TiB 未満のサイズの完全なデータを移行することをお勧めします。                                                                    |
| [団子](/dumpling-overview.md)                                               | MySQL または TiDB からの完全なデータエクスポート                                                                                                                                             | MySQL、TiDB                                                                                                                     | SQL、CSV                                        | <li>データをより簡単にフィルタリングできるテーブルフィルター機能をサポートする</li><li>AmazonS3 へのデータのエクスポートをサポート</li>                                                                                                                                                          | <li>エクスポートしたデータを TiDB 以外のデータベースに復元する場合は、Dumpling を使用することをお勧めします。</li><li>エクスポートしたデータを別の TiDB クラスターに復元する場合は、バックアップと復元（BR）を使用することをお勧めします。</li>                                                      |
| [TiDB Lightning](/tidb-lightning/tidb-lightning-overview.md)                | TiDB への完全なデータインポート                                                                                                                                                              | <li>餃子からエクスポートされたファイル</li><li>CSV ファイル</li><li>ローカルディスクまたは AmazonS3 から読み取られたデータ</li> | TiDB                                            | <li>大量のデータの迅速なインポートと TiDB クラスター内の特定のテーブルの迅速な初期化をサポート</li><li>インポートの進行状況を保存するチェックポイントをサポートし、 `tidb-lightning`が再起動後に中断したところからインポートを続行できるようにします</li><li>データフィルタリングをサポート</li> | <li>ローカルバックエンドがデータのインポートに使用されている場合、インポートプロセス中、TiDB クラスターはサービスを提供できません。</li><li> TiDB サービスに影響を与えたくない場合は、TiDBLightningTiDB バックエンドに従ってデータのインポートを実行します。</li>                                    |
| [バックアップと復元（BR）](/br/backup-and-restore-tool.md)                  | 巨大なデータサイズの TiDB クラスターのバックアップと復元                                                                                                                                     | TiDB                                                                                                                            | SST、backup.meta ファイル、backup.lock ファイル | <li>別の TiDB クラスターにデータを復元するのに適しています</li><li>災害復旧のための外部ストレージへのデータのバックアップをサポート</li>                                                                                                                                                         | <li>BR が TiCDC または Drainer のアップストリームクラスターにデータを復元する場合、復元されたデータを TiCDC または Drainer によってダウンストリームに複製することはできません。</li><li> BR は、同じ`new_collations_enabled_on_first_bootstrap`値を持つクラスター間の操作のみをサポートします。</li> |
| [TiCDC](/ticdc/ticdc-overview.md)                                           | このツールは、TiKV 変更ログをプルすることによって実装されます。データをアップストリーム TSO との整合性のある状態に復元し、他のシステムがデータ変更をサブスクライブするのをサポートできます。 | TiDB                                                                                                                            | TiDB、MySQL、Apache Pulsar、Kafka、Confluent    | TiCDC オープンプロトコルを提供する                                                                                                                                                                                                                                                               | TiCDC は、少なくとも 1 つの有効なインデックスを持つテーブルのみを複製します。次のシナリオはサポートされていません。<ul><li> RawKV のみを使用する TiKV クラスター。</li><li> TiDB の DDL 操作`CREATE SEQUENCE`および`SEQUENCE`関数。</li></ul>                                                        |
| [TiDB Binlog](/tidb-binlog/tidb-binlog-overview.md)                         | 1 つの TiDB クラスターを別の TiDB クラスターのセカンダリクラスターとして使用するなど、TiDB クラスター間の増分レプリケーション                                                                | TiDB                                                                                                                            | TiDB、MySQL、Kafka、増分バックアップファイル    | リアルタイムのバックアップと復元をサポートします。災害復旧のために復元する TiDB クラスターデータをバックアップします                                                                                                                                                                             | 一部の TiDB バージョンと互換性がありません                                                                                                                                                                                                                                                           |
| [sync-diff-inspector](/sync-diff-inspector/sync-diff-inspector-overview.md) | データベースに保存されているデータを MySQL プロトコルと比較する                                                                                                                              | TiDB、MySQL                                                                                                                     | TiDB、MySQL                                     | 少量のデータに一貫性がないシナリオでデータを修復するために使用できます                                                                                                                                                                                                                           | <li>MySQL と TiDB 間のデータ移行では、オンラインチェックはサポートされていません。</li><li> JSON、BIT、BINARY、BLOB およびその他のタイプのデータはサポートされていません。</li>                                                                                                                      |

## TiUP を使用してツールをインストールする {#install-tools-using-tiup}

TiDB v4.0 以降、TiUP は、TiDB エコシステム内のさまざまなクラスターコンポーネントの管理を支援するパッケージマネージャーとして機能します。これで、1 つのコマンドを使用して任意のクラスターコンポーネントを管理できます。

### 手順 1.TiUP をインストールします {#step-1-install-tiup}

{{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

```shell
curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
```

グローバル環境変数を再宣言します。

{{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

```shell
source ~/.bash_profile
```

### ステップ 2.コンポーネントをインストールします {#step-2-install-components}

次のコマンドを使用して、使用可能なすべてのコンポーネントを表示できます。

{{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

```shell
tiup list
```

コマンド出力には、使用可能なすべてのコンポーネントが一覧表示されます。

```bash
Available components:
Name            Owner    Description
----            -----    -----------
bench           pingcap  Benchmark database with different workloads
br              pingcap  TiDB/TiKV cluster backup restore tool
cdc             pingcap  CDC is a change data capture tool for TiDB
client          pingcap  Client to connect playground
cluster         pingcap  Deploy a TiDB cluster for production
ctl             pingcap  TiDB controller suite
dm              pingcap  Data Migration Platform manager
dmctl           pingcap  dmctl component of Data Migration Platform
errdoc          pingcap  Document about TiDB errors
pd-recover      pingcap  PD Recover is a disaster recovery tool of PD, used to recover the PD cluster which cannot start or provide services normally
playground      pingcap  Bootstrap a local TiDB cluster for fun
tidb            pingcap  TiDB is an open source distributed HTAP database compatible with the MySQL protocol
tidb-lightning  pingcap  TiDB Lightning is a tool used for fast full import of large amounts of data into a TiDB cluster
tiup            pingcap  TiUP is a command-line component management tool that can help to download and install TiDB platform components to the local system
```

インストールするコンポーネントを選択します。

{{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

```shell
tiup install dumpling tidb-lightning
```

> <strong>ノート：</strong>
>
> 特定のバージョンのコンポーネントをインストールするには、 `tiup install <component>[:version]`コマンドを使用します。

### 手順 3.TiUP とそのコンポーネントを更新する（オプション） {#step-3-update-tiup-and-its-components-optional}

新しいバージョンのリリースログと互換性に関する注意事項を確認することをお勧めします。

{{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

```shell
tiup update --self && tiup update dm
```

## も参照してください {#see-also}

- [TiUP をオフラインでデプロイする](/production-deployment-using-tiup.md#method-2-deploy-tiup-offline)
- [ツールをバイナリでダウンロードしてインストールする](/download-ecosystem-tools.md)
