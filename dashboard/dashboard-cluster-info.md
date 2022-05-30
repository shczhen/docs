---
title: TiDB Dashboard Cluster Information Page
summary: View the running status of TiDB, TiKV, PD, TiFlash components in the entire cluster and the running status of the host on which these components are located.
aliases: ['/docs/dev/dashboard/dashboard-cluster-info/']
---

# TiDBダッシュボードクラスター情報ページ {#tidb-dashboard-cluster-information-page}

クラスター情報ページでは、クラスター全体のTiDB、TiKV、PD、TiFlashコンポーネントの実行状況と、これらのコンポーネントが配置されているホストの実行状況を確認できます。

## ページにアクセスする {#access-the-page}

次の2つの方法のいずれかを使用して、クラスタ情報ページにアクセスできます。

-   TiDBダッシュボードにログインした後、左側のナビゲーションメニューで[<strong>クラスター情報</strong>]をクリックします。

    ![Access cluster information page](/media/dashboard/dashboard-cluster-info-access.png)

-   ブラウザで[http://127.0.0.1:2379/dashboard/#/cluster_info/instance](http://127.0.0.1:2379/dashboard/#/cluster_info/instance)にアクセスします。 `127.0.0.1:2379`を実際のPDインスタンスのアドレスとポートに置き換えます。

## インスタンスリスト {#instance-list}

[<strong>インスタンス</strong>]をクリックして、インスタンスのリストを表示します。

![Instance list](/media/dashboard/dashboard-cluster-info-instances.png)

このインスタンスリストには、クラスター内のTiDB、TiKV、PD、およびTiFlashコンポーネントのすべてのインスタンスの概要情報が表示されます。

このリストには、次の情報が含まれています。

-   アドレス：インスタンスアドレス。
-   ステータス：インスタンスの実行ステータス。
-   アップタイム：インスタンスの開始時間。
-   バージョン：インスタンスのバージョン番号。
-   デプロイメントディレクトリ：インスタンスバイナリファイルが配置されているディレクトリ。
-   Git Hash：インスタンスのバイナリファイルに対応するGitHash値。

インスタンスの実行ステータスは次のとおりです。

-   上：インスタンスは正常に実行されています。
-   ダウンまたは到達不能：インスタンスが開始されていないか、対応するホストにネットワークの問題が存在します。
-   トゥームストーン：インスタンスのデータは完全に移行され、スケールインは完了しています。このステータスは、TiKVまたはTiFlashインスタンスにのみ存在します。
-   離脱：インスタンス上のデータが移行され、スケールインが進行中です。このステータスは、TiKVまたはTiFlashインスタンスにのみ存在します。
-   不明：インスタンスの実行状態は不明です。

> <strong>ノート：</strong>
>
> テーブルの一部の列は、インスタンスが起動している場合にのみ表示できます。

## ホストリスト {#host-list}

[<strong>ホスト</strong>]をクリックして、ホストのリストを表示します。

![Host list](/media/dashboard/dashboard-cluster-info-hosts.png)

このホストリストには、クラスター内のTiDB、TiKV、PD、およびTiFlashコンポーネントのすべてのインスタンスに対応するホストの実行ステータスが表示されます。

このリストには、次の情報が含まれています。

-   アドレス：ホストIPアドレス。
-   CPU：ホストCPUの論理コアの数。
-   CPU使用率：現在の1秒間のユーザーモードおよびカーネルモードのCPU使用率。
-   メモリ：ホストの物理メモリの合計サイズ。
-   メモリ使用量：ホストの現在のメモリ使用量。
-   ディスク：インスタンスが実行されているホスト上のディスクのファイルシステムと、このディスクのマウントパス。
-   ディスク使用量：インスタンスが実行されているホスト上のディスクのスペース使用量。

> <strong>ノート：</strong>
>
> ホストリスト情報は各インスタンスプロセスによって提供されるため、ホスト上のすべてのインスタンスがダウンしている場合、ホスト情報は表示されません。
