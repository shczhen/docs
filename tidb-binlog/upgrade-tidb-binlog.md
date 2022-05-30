---
title: Upgrade TiDB Binlog
summary: Learn how to upgrade TiDB Binlog to the latest cluster version.
aliases: ['/docs/dev/tidb-binlog/upgrade-tidb-binlog/','/docs/dev/reference/tidb-binlog/upgrade/','/docs/dev/how-to/upgrade/tidb-binlog/']
---

# TiDBBinlogをアップグレードする {#upgrade-tidb-binlog}

このドキュメントでは、手動でデプロイされたTiDBBinlogを最新の[集まる](/tidb-binlog/tidb-binlog-overview.md)バージョンにアップグレードする方法を紹介します。 TiDB Binlogを以前の互換性のないバージョン（Kafka /ローカルバージョン）から最新バージョンにアップグレードする方法に関するセクションもあります。

## 手動でデプロイされたTiDBBinlogをアップグレードする {#upgrade-tidb-binlog-deployed-manually}

TiDB Binlogを手動でデプロイする場合は、このセクションの手順に従ってください。

### アップグレードポンプ {#upgrade-pump}

まず、クラスター内の各Pumpインスタンスを1つずつアップグレードします。これにより、TiDBからbinlogを受信できるPumpインスタンスがクラスター内に常に存在するようになります。手順は次のとおりです。

1.  元のファイルを新しいバージョンの`pump`に置き換えます。
2.  ポンププロセスを再開します。

### ドレイナーをアップグレードする {#upgrade-drainer}

次に、Drainerコンポーネントをアップグレードします。

1.  元のファイルを新しいバージョンの`drainer`に置き換えます。
2.  ドレイナープロセスを再開します。

## TiDBBinlogをKafka/Localバージョンからクラスターバージョンにアップグレードします {#upgrade-tidb-binlog-from-kafka-local-version-to-the-cluster-version}

新しいTiDBバージョン（v2.0.8-binlog、v2.1.0-rc.5以降）は、KafkaバージョンまたはローカルバージョンのTiDBBinlogと互換性がありません。 TiDBを新しいバージョンのいずれかにアップグレードする場合は、クラスターバージョンのTiDBBinlogを使用する必要があります。アップグレードする前にKafkaまたはローカルバージョンのTiDBBinlogを使用する場合は、TiDBBinlogをクラスターバージョンにアップグレードする必要があります。

次の表に、TiDBBinlogバージョンとTiDBバージョンの対応する関係を示します。

| TiDBBinlogバージョン | TiDBバージョン                       | ノート                                                            |
| --------------- | ------------------------------- | -------------------------------------------------------------- |
| ローカル            | TiDB1.0以前                       |                                                                |
| カフカ             | TiDB 1.0〜TiDB 2.1 RC5           | TiDB 1.0は、ローカルバージョンとKafkaバージョンの両方のTiDBBinlogをサポートします。          |
| 集まる             | TiDB v2.0.8-binlog、TiDB2.1RC5以降 | TiDB v2.0.8-binlogは、TiDBBinlogのクラスターバージョンをサポートする特別な2.0バージョンです。 |

### アップグレードプロセス {#upgrade-process}

> <strong>ノート：</strong>
>
> 完全なデータのインポートが許容される場合は、古いバージョンを破棄して、 [TiDBBinlogクラスターの展開](/tidb-binlog/deploy-tidb-binlog.md)の後にTiDBBinlogをデプロイできます。

元のチェックポイントからレプリケーションを再開する場合は、次の手順を実行してTiDBBinlogをアップグレードします。

1.  新しいバージョンのPumpをデプロイします。

2.  TiDBクラスターサービスを停止します。

3.  TiDBと構成をアップグレードし、binlogデータを新しいPumpクラスターに書き込みます。

4.  TiDBクラスターをサービスに再接続します。

5.  古いバージョンのDrainerが、古いバージョンのPumpのデータをダウンストリームに完全に複製していることを確認してください。

    Drainerの`status`のインターフェースを照会し、次のようにコマンドを実行します。

    {{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

    ```bash
    curl 'http://172.16.10.49:8249/status'
    ```

    ```
    {"PumpPos":{"172.16.10.49:8250":{"offset":32686}},"Synced": true ,"DepositWindow":{"Upper":398907800202772481,"Lower":398907799455662081}}
    ```

    戻り値`Synced`がTrueの場合、Drainerが古いバージョンのPumpのデータをダウンストリームに完全に複製したことを意味します。

6.  Drainerの新しいバージョンを起動します。

7.  古いバージョンのポンプとドレイナー、および依存するKafkaとZooKeeperを閉じます。
