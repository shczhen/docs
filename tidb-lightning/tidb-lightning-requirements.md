---
title: Prerequisites for using TiDB Lightning
summary: Learn prerequisites for running TiDB Lightning.
---

# TiDBLightningを使用するための前提条件 {#prerequisites-for-using-tidb-lightning}

TiDB Lightningを使用する前に、環境が要件を満たしているかどうかを確認する必要があります。これにより、インポート中のエラーが減り、インポートが確実に成功します。

## ダウンストリーム特権要件 {#downstream-privilege-requirements}

インポートモードと有効な機能に基づいて、ダウンストリームデータベースユーザーに異なる権限を付与する必要があります。次の表に参照を示します。

<table border="1">
   <tr>
      <td></td>
      <td>Feature</td>
      <td>Scope</td>
      <td>Required privilege</td>
      <td>Remarks</td>
   </tr>
   <tr>
      <td rowspan="2">Mandatory</td>
      <td rowspan="2">Basic functions</td>
      <td>Target table</td>
      <td>CREATE, SELECT, INSERT, UPDATE, DELETE, DROP, ALTER</td>
      <td>DROP is required only when tidb-lightning-ctl runs the checkpoint-destroy-all command</td>
   </tr>
   <tr>
      <td>Target database</td>
      <td>CREATE</td>
      <td></td>
   </tr>
   <tr>
      <td rowspan="4">Mandatory</td>
      <td>tidb-backend</td>
      <td>information_schema.columns</td>
      <td>SELECT</td>
      <td></td>
   </tr>
   <tr>
      <td  rowspan="3">local-backend</td>
      <td>mysql.tidb</td>
      <td>SELECT</td>
      <td></td>
   </tr>
   <tr>
      <td>-</td>
      <td>SUPER</td>
      <td></td>
   </tr>
   <tr>
      <td>-</td>
      <td>RESTRICTED_VARIABLES_ADMIN,RESTRICTED_TABLES_ADMIN</td>
      <td>Required when the target TiDB enables SEM</td>
   </tr>
   <tr>
      <td>Recommended</td>
      <td>Conflict detection, max-error</td>
      <td>Schema configured for lightning.task-info-schema-name</td>
      <td>SELECT, INSERT, UPDATE, DELETE, CREATE, DROP</td>
      <td>If not required, the value must be set to ""</td>
   </tr>
   <tr>
      <td>Optional</td>
      <td>Parallel import</td>
      <td>Schema configured for lightning.meta-schema-name</td>
      <td>SELECT, INSERT, UPDATE, DELETE, CREATE, DROP</td>
      <td>If not required, the value must be set to ""</td>
   </tr>
   <tr>
      <td>Optional</td>
      <td>checkpoint.driver = “mysql”</td>
      <td>checkpoint.schema setting</td>
      <td>SELECT,INSERT,UPDATE,DELETE,CREATE,DROP</td>
      <td>Required when checkpoint information is stored in databases, instead of files</td>
   </tr>
</table>

## ダウンストリームストレージスペースの要件 {#downstream-storage-space-requirements}

ターゲットTiKVクラスターには、インポートされたデータを格納するのに十分なディスク容量が必要です。 [標準のハードウェア要件](/hardware-and-software-requirements.md)に加えて、ターゲットTiKVクラスターのストレージスペースは<strong>、データソースのサイズxレプリカの数x2</strong>よりも大きくする必要があります。たとえば、クラスターがデフォルトで3つのレプリカを使用する場合、ターゲットTiKVクラスターには、データソースのサイズの6倍を超えるストレージスペースが必要です。数式にはx2があります。理由は次のとおりです。

-   インデックスは余分なスペースを必要とする場合があります。
-   RocksDBにはスペース増幅効果があります。

MySQLからDumplingによってエクスポートされた正確なデータ量を計算することは困難です。ただし、次のSQLステートメントを使用してinformation_schema.tablesテーブルのdata-lengthフィールドを要約することにより、データ量を見積もることができます。

MiBですべてのスキーマのサイズを計算します。 ${schema_name}をスキーマ名に置き換えます。

```sql
select table_schema,sum(data_length)/1024/1024 as data_length,sum(index_length)/1024/1024 as index_length,sum(data_length+index_length)/1024/1024 as sum from information_schema.tables where table_schema = "${schema_name}" group by table_schema;
```

最大のテーブルのサイズをMiBで計算します。 ${schema_name}をスキーマ名に置き換えます。

{{< copyable "" >}}

```sql
select table_name,table_schema,sum(data_length)/1024/1024 as data_length,sum(index_length)/1024/1024 as index_length,sum(data_length+index_length)/1024/1024 as sum from information_schema.tables where table_schema = "${schema_name}" group by table_name,table_schema order by sum  desc limit 5;
```

## リソース要件 {#resource-requirements}

<strong>オペレーティングシステム</strong>：このドキュメントの例では、新しいCentOS7インスタンスを使用しています。仮想マシンは、ローカルホストまたはクラウドのいずれかにデプロイできます。 TiDB Lightningはデフォルトで必要なだけのCPUリソースを消費するため、専用サーバーにデプロイすることをお勧めします。これが不可能な場合は、他のTiDBコンポーネント（tikv-serverなど）と一緒に単一のサーバーにデプロイしてから、TiDBLightningからのCPU使用率を制限するように`region-concurrency`を構成できます。通常、サイズは論理CPUの75％に設定できます。

<strong>メモリとCPU</strong> ：

TiDB Lightningが消費するCPUとメモリは、バックエンドモードによって異なります。使用するバックエンドに基づいて最適なインポートパフォーマンスをサポートする環境でTiDBLightningを実行します。

-   ローカルバックエンド：TiDB lightningは、このモードで多くのCPUとメモリを消費します。 32コアを超えるCPUと64GiBを超えるメモリを割り当てることをお勧めします。

> <strong>注</strong>：
>
> インポートするデータが大きい場合、1回の並列インポートで約2GiBのメモリを消費する可能性があります。この場合、合計メモリ使用量は`region-concurrency` x2GiBになります。 `region-concurrency`は論理CPUの数と同じです。メモリサイズ（GiB）がCPUの2倍未満であるか、インポート中にOOMが発生する場合は、 `region-concurrency`を減らしてOOMをアドレス指定できます。

-   TiDBバックエンド：このモードでは、パフォーマンスのボトルネックはTiDBにあります。 TiDBLightningには4コアCPUと8GiBメモリを割り当てることをお勧めします。 TiDBクラスターがインポートで書き込みしきい値に達しない場合は、 `region-concurrency`を増やすことができます。
-   インポーターバックエンド：このモードでは、リソース消費量はローカルバックエンドの場合とほぼ同じです。インポーターバックエンドは推奨されません。特に要件がない場合は、ローカルバックエンドを使用することをお勧めします。

<strong>記憶域スペース</strong>： `sorted-kv-dir`構成項目は、ソートされたキー値ファイルの一時記憶域ディレクトリーを指定します。ディレクトリは空である必要があり、ストレージスペースはインポートするデータセットのサイズよりも大きい必要があります。インポートのパフォーマンスを向上させるには、 `data-source-dir`とは異なるディレクトリを使用し、そのディレクトリにフラッシュストレージと排他的I/Oを使用することをお勧めします。
