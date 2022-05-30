---
title: Upgrade TiDB Using TiUP
summary: Learn how to upgrade TiDB using TiUP.
aliases: ['/docs/dev/upgrade-tidb-using-tiup/','/docs/dev/how-to/upgrade/using-tiup/','/tidb/dev/upgrade-tidb-using-tiup-offline','/docs/dev/upgrade-tidb-using-tiup-offline/']
---

# TiUPを使用してTiDBをアップグレードする {#upgrade-tidb-using-tiup}

このドキュメントは、次のアップグレードパスを対象としています。

-   TiDB4.0バージョンからTiDB6.0にアップグレードします。
-   TiDB5.0-5.4バージョンからTiDB6.0にアップグレードします。

> <strong>警告：</strong>
>
> -   TiFlashを5.3より前のバージョンから5.3以降のバージョンにオンラインでアップグレードすることはできません。代わりに、最初に初期バージョンのすべてのTiFlashインスタンスを停止してから、クラスターをオフラインでアップグレードする必要があります。他のコンポーネント（TiDBやTiKVなど）がオンラインアップグレードをサポートしていない場合は、 [オンラインアップグレード](#online-upgrade)の警告の指示に従ってください。
> -   DDLステートメントがクラスターで実行されているときは<strong>TiDB</strong>クラスターをアップグレードしないでください（通常、 `ADD INDEX`などの時間のかかるDDLステートメントや列タイプの変更の場合）。
> -   アップグレードする前に、 [`ADMIN SHOW DDL`](/sql-statements/sql-statement-admin-show-ddl.md)コマンドを使用して、TiDBクラスターに進行中のDDLジョブがあるかどうかを確認することをお勧めします。クラスタにDDLジョブがある場合、クラスタをアップグレードするには、DDLの実行が終了するまで待つか、 [`ADMIN CANCEL DDL`](/sql-statements/sql-statement-admin-cancel-ddl.md)コマンドを使用してDDLジョブをキャンセルしてからクラスタをアップグレードします。
> -   さらに、クラスターのアップグレード中は、DDLステートメントを実行し<strong>ない</strong>でください。そうしないと、未定義動作の問題が発生する可能性があります。

> <strong>ノート：</strong>
>
> アップグレードするクラスターがv3.1以前のバージョン（v3.0またはv2.1）の場合、v6.0またはそのパッチバージョンへの直接アップグレードはサポートされていません。最初にクラスターをv4.0にアップグレードし、次にv6.0にアップグレードする必要があります。

## アップグレードの警告 {#upgrade-caveat}

-   TiDBは現在、バージョンのダウングレードまたはアップグレード後の以前のバージョンへのロールバックをサポートしていません。
-   TiDB Ansibleを使用して管理されるv4.0クラスターの場合、 [TiUP（v4.0）を使用してTiDBをアップグレードする](https://docs.pingcap.com/tidb/v4.0/upgrade-tidb-using-tiup#import-tidb-ansible-and-the-inventoryini-configuration-to-tiup)に従って新しい管理を行うために、クラスターをTiUP（ `tiup cluster` ）にインポートする必要があります。次に、このドキュメントに従って、クラスターをv6.0またはそのパッチバージョンにアップグレードできます。
-   3.0より前のバージョンを6.0に更新するには：
    1.  [TiDB Ansible](https://docs.pingcap.com/tidb/v3.0/upgrade-tidb-using-ansible)を使用してこのバージョンを3.0に更新します。
    2.  TiUP（ `tiup cluster` ）を使用して、TiDBAnsible構成をインポートします。
    3.  [TiUP（v4.0）を使用してTiDBをアップグレードする](https://docs.pingcap.com/tidb/v4.0/upgrade-tidb-using-tiup#import-tidb-ansible-and-the-inventoryini-configuration-to-tiup)に従って3.0バージョンを4.0に更新します。
    4.  このドキュメントに従って、クラスターをv6.0にアップグレードします。
-   TiDB Binlog、TiCDC、TiFlash、およびその他のコンポーネントのバージョンのアップグレードをサポートします。
-   異なるバージョンの互換性の変更の詳細については、各バージョンの[リリースノート](/releases/release-notes.md)を参照してください。対応するリリースノートの「互換性の変更」セクションに従って、クラスタ構成を変更します。
-   v5.3より前のバージョンからv5.3以降のバージョンにアップグレードするクラスターの場合、デフォルトでデプロイされているPrometheusはv2.8.1からv2.27.1にアップグレードされます。 Prometheus v2.27.1は、より多くの機能を提供し、セキュリティの問題を修正します。 v2.8.1と比較して、v2.27.1のアラート時間の表現が変更されています。詳細については、 [プロメテウスコミット](https://github.com/prometheus/prometheus/commit/7646cbca328278585be15fa615e22f2a50b47d06)を参照してください。

## 準備 {#preparations}

このセクションでは、TiUPおよびTiUPクラスターコンポーネントのアップグレードを含む、TiDBクラスターをアップグレードする前に必要な準備作業を紹介します。

### ステップ1：TiUPまたはTiUPオフラインミラーをアップグレードする {#step-1-upgrade-tiup-or-tiup-offline-mirror}

TiDBクラスターをアップグレードする前に、まずTiUPまたはTiUPミラーをアップグレードする必要があります。

#### TiUPとTiUPクラスターをアップグレードする {#upgrade-tiup-and-tiup-cluster}

> <strong>ノート：</strong>
>
> アップグレードするクラスターの制御マシンが`https://tiup-mirrors.pingcap.com`にアクセスできない場合は、このセクションをスキップして[TiUPオフラインミラーをアップグレードする](#upgrade-tiup-offline-mirror)を参照してください。

1.  TiUPバージョンをアップグレードします。 TiUPのバージョンは`1.9.3`以降にすることをお勧めします。

    {{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

    ```shell
    tiup update --self
    tiup --version
    ```

2.  TiUPクラスターのバージョンをアップグレードします。 TiUPクラスターのバージョンは`1.9.3`以降にすることをお勧めします。

    {{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

    ```shell
    tiup update cluster
    tiup cluster --version
    ```

#### TiUPオフラインミラーをアップグレードする {#upgrade-tiup-offline-mirror}

> <strong>ノート：</strong>
>
> アップグレードするクラスターがオフライン方式を使用せずにデプロイされた場合は、このステップをスキップしてください。

[TiUPを使用してTiDBクラスターをデプロイする-TiUPをオフラインでデプロイする](/production-deployment-using-tiup.md#method-2-deploy-tiup-offline)を参照して、新しいバージョンのTiUPミラーをダウンロードし、制御マシンにアップロードします。 `local_install.sh`を実行した後、TiUPは上書きアップグレードを完了します。

{{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

```shell
tar xzvf tidb-community-server-${version}-linux-amd64.tar.gz
sh tidb-community-server-${version}-linux-amd64/local_install.sh
source /home/tidb/.bash_profile
```

上書きアップグレード後、次のコマンドを実行してTiUPクラスターコンポーネントをアップグレードします。

{{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

```shell
tiup update cluster
```

これで、オフラインミラーが正常にアップグレードされました。上書き後のTiUP動作中にエラーが発生した場合は、 `manifest`が更新されていない可能性があります。 TiUPを再度実行する前に`rm -rf ~/.tiup/manifests/*`を試すことができます。

### ステップ2：TiUPトポロジー構成ファイルを編集する {#step-2-edit-tiup-topology-configuration-file}

> <strong>ノート：</strong>
>
> 次のいずれかの状況が当てはまる場合は、この手順をスキップしてください。
>
> -   元のクラスターの構成パラメーターは変更していません。または、 `tiup cluster`を使用して構成パラメーターを変更しましたが、それ以上の変更は必要ありません。
> -   アップグレード後、変更されていない構成アイテムにv6.0のデフォルトのパラメーター値を使用する必要があります。

1.  `vi`編集モードに入り、トポロジファイルを編集します。

    {{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

    ```shell
    tiup cluster edit-config <cluster-name>
    ```

2.  [トポロジー](https://github.com/pingcap/tiup/blob/master/embed/examples/cluster/topology.example.yaml)の構成テンプレートの形式を参照し、トポロジーファイルの`server_configs`のセクションに変更するパラメーターを入力します。

3.  変更後、 <kbd>：</kbd> + <kbd>w</kbd> + <kbd>q</kbd>と入力して変更を保存し、編集モードを終了します。 <kbd>Y</kbd>を入力して、変更を確認します。

> <strong>ノート：</strong>
>
> クラスタをv6.0にアップグレードする前に、v4.0で変更したパラメータがv6.0で互換性があることを確認してください。詳細については、 [TiKV構成ファイル](/tikv-configuration-file.md)を参照してください。
>
> 次の3つのTiKVパラメータは、TiDBv5.0では廃止されています。元のクラスターで以下のパラメーターが構成されている場合は、これらのパラメーターを`edit-config`から削除する必要があります。
>
> -   pessimistic-txn.enabled
> -   server.request-batch-enable-cross-command
> -   server.request-batch-wait-duration

### ステップ3：現在のクラスターのヘルスステータスを確認する {#step-3-check-the-health-status-of-the-current-cluster}

アップグレード中の未定義の動作やその他の問題を回避するために、アップグレードの前に現在のクラスターのリージョンのヘルスステータスを確認することをお勧めします。これを行うには、 `check`サブコマンドを使用できます。

{{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

```shell
tiup cluster check <cluster-name> --cluster
```

コマンド実行後、「リージョンステータス」チェック結果が出力されます。

-   結果が「すべてのリージョンは正常です」の場合、現在のクラスター内のすべてのリージョンは正常であり、アップグレードを続行できます。
-   結果が「リージョンが完全に正常ではない：mミスピア、n保留中ピア」の場合、「他の操作の前に異常なリージョンを修正してください」。プロンプト、現在のクラスター内の一部のリージョンが異常です。チェック結果が「すべてのリージョンが正常」になるまで、異常のトラブルシューティングを行う必要があります。その後、アップグレードを続行できます。

## TiDBクラスターをアップグレードする {#upgrade-the-tidb-cluster}

このセクションでは、TiDBクラスターをアップグレードし、アップグレード後にバージョンを確認する方法について説明します。

### TiDBクラスターを指定されたバージョンにアップグレードします {#upgrade-the-tidb-cluster-to-a-specified-version}

クラスターは、オンラインアップグレードとオフラインアップグレードの2つの方法のいずれかでアップグレードできます。

デフォルトでは、TiUPクラスターはオンライン方式を使用してTiDBクラスターをアップグレードします。つまり、TiDBクラスターはアップグレードプロセス中に引き続きサービスを提供できます。オンライン方式では、アップグレードして再起動する前に、各ノードでリーダーが1つずつ移行されます。したがって、大規模なクラスターの場合、アップグレード操作全体を完了するのに長い時間がかかります。

アプリケーションに、メンテナンスのためにデータベースを停止するためのメンテナンスウィンドウがある場合は、オフラインアップグレード方法を使用して、アップグレード操作をすばやく実行できます。

#### オンラインアップグレード {#online-upgrade}

{{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

```shell
tiup cluster upgrade <cluster-name> <version>
```

たとえば、クラスターをv6.0.0にアップグレードする場合は、次のようにします。

{{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

```shell
tiup cluster upgrade <cluster-name> v6.0.0
```

> <strong>ノート：</strong>
>
> -   オンラインアップグレードでは、すべてのコンポーネントが1つずつアップグレードされます。 TiKVのアップグレード中、TiKVインスタンスのすべてのリーダーは、インスタンスを停止する前に削除されます。デフォルトのタイムアウト時間は5分（300秒）です。このタイムアウト時間の後、インスタンスは直接停止します。
>
> -   `--force`パラメーターを使用すると、リーダーを削除せずにクラスターをすぐにアップグレードできます。ただし、アップグレード中に発生したエラーは無視されます。つまり、アップグレードの失敗は通知されません。したがって、 `--force`パラメータは注意して使用してください。
>
> -   安定したパフォーマンスを維持するには、インスタンスを停止する前に、TiKVインスタンスのすべてのリーダーが削除されていることを確認してください。 `--transfer-timeout`をより大きな値（たとえば、 `--transfer-timeout 3600` （単位：秒））に設定できます。
>
> <!---->
>
> -   TiDBクラスターを5.3より前のバージョンから5.3以降のバージョンにアップグレードする場合、TiFlashをオンラインでアップグレードすることはできません。代わりに、最初にすべてのTiFlashインスタンスを停止してから、クラスターをオフラインでアップグレードする必要があります。次に、クラスターをリロードして、他のコンポーネントが中断することなくオンラインでアップグレードされるようにします。

#### オフラインアップグレード {#offline-upgrade}

1.  オフラインアップグレードの前に、まずクラスター全体を停止する必要があります。

    {{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

    ```shell
    tiup cluster stop <cluster-name>
    ```

2.  オフラインアップグレードを実行するには、 `upgrade`コマンドと`--offline`オプションを使用します。

    {{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

    ```shell
    tiup cluster upgrade <cluster-name> <version> --offline
    ```

3.  アップグレード後、クラスターは自動的に再起動されません。再起動するには、 `start`コマンドを使用する必要があります。

    {{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

    ```shell
    tiup cluster start <cluster-name>
    ```

### クラスターのバージョンを確認する {#verify-the-cluster-version}

`display`コマンドを実行して、最新のクラスターバージョン`TiDB Version`を表示します。

{{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

```shell
tiup cluster display <cluster-name>
```

```
Cluster type:       tidb
Cluster name:       <cluster-name>
Cluster version:    v6.0.0
```

> <strong>ノート：</strong>
>
> デフォルトでは、TiUPとTiDBは使用法の詳細をPingCAPと共有して、製品を改善する方法を理解するのに役立ちます。共有される内容と共有を無効にする方法の詳細については、 [テレメトリー](/telemetry.md)を参照してください。

## よくある質問 {#faq}

このセクションでは、TiUPを使用してTiDBクラスターを更新するときに発生する一般的な問題について説明します。

### エラーが発生してアップグレードが中断された場合、このエラーを修正した後にアップグレードを再開するにはどうすればよいですか？ {#if-an-error-occurs-and-the-upgrade-is-interrupted-how-to-resume-the-upgrade-after-fixing-this-error}

`tiup cluster upgrade`コマンドを再実行して、アップグレードを再開します。アップグレード操作は、以前にアップグレードされたノードを再起動します。アップグレードされたノードを再起動したくない場合は、 `replay`サブコマンドを使用して操作を再試行します。

1.  `tiup cluster audit`を実行して、操作レコードを確認します。

    {{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

    ```shell
    tiup cluster audit
    ```

    失敗したアップグレード操作レコードを見つけて、この操作レコードのIDを保持します。 IDは、次のステップの`<audit-id>`の値です。

2.  `tiup cluster replay <audit-id>`を実行して、対応する操作を再試行します。

    {{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

    ```shell
    tiup cluster replay <audit-id>
    ```

### アップグレードの間、エビクトリーダーはあまりにも長い間待っていました。クイックアップグレードのためにこのステップをスキップするにはどうすればよいですか？ {#the-evict-leader-has-waited-too-long-during-the-upgrade-how-to-skip-this-step-for-a-quick-upgrade}

`--force`を指定できます。その後、PDリーダーの転送とTiKVリーダーの削除のプロセスは、アップグレード中にスキップされます。クラスターを直接再起動してバージョンを更新します。これは、オンラインで実行されるクラスターに大きな影響を与えます。コマンドは次のとおりです。

{{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

```shell
tiup cluster upgrade <cluster-name> <version> --force
```

### TiDBクラスターをアップグレードした後にpd-ctlなどのツールのバージョンを更新するにはどうすればよいですか？ {#how-to-update-the-version-of-tools-such-as-pd-ctl-after-upgrading-the-tidb-cluster}

TiUPを使用してツールのバージョンをアップグレードし、対応するバージョンの`ctl`つのコンポーネントをインストールできます。

{{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

```shell
tiup install ctl:v6.0.0
```

## TiDB6.0の互換性の変更 {#tidb-6-0-compatibility-changes}

-   互換性の変更については、TiDB6.0リリースノートを参照してください。
-   TiDB Binlogを使用してクラスターにローリング更新を適用する場合は、新しいクラスター化インデックステーブルの作成を避けてください。
