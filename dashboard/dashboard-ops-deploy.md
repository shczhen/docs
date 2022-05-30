---
title: Deploy TiDB Dashboard
summary: Learn how to deploy TiDB Dashboard.
aliases: ['/docs/dev/dashboard/dashboard-ops-deploy/']
---

# TiDBダッシュボードを展開する {#deploy-tidb-dashboard}

TiDBダッシュボードUIは、v4.0以降のバージョンのPDコンポーネントに組み込まれており、追加の展開は必要ありません。標準のTiDBクラスターをデプロイするだけで、TiDBダッシュボードが表示されます。

標準のTiDBクラスターを展開する方法については、次のドキュメントを参照してください。

-   [TiDBデータベースプラットフォームのクイックスタートガイド](/quick-start-with-tidb.md)
-   [TiDBを本番環境にデプロイする](/production-deployment-using-tiup.md)
-   [Kubernetes環境のデプロイ](https://docs.pingcap.com/tidb-in-kubernetes/stable/access-dashboard)

> <strong>ノート：</strong>
>
> v4.0より前のTiDBクラスターにTiDBダッシュボードをデプロイすることはできません。

## 複数のPDインスタンスを使用した展開 {#deployment-with-multiple-pd-instances}

複数のPDインスタンスがクラスターにデプロイされている場合、これらのインスタンスの1つだけがTiDBダッシュボードにサービスを提供します。

PDインスタンスが初めて実行されるとき、それらは自動的に相互にネゴシエートして、TiDBダッシュボードにサービスを提供する1つのインスタンスを選択します。 TiDBダッシュボードは他のPDインスタンスでは実行されません。 TiDBダッシュボードサービスは、PDインスタンスが再起動されたり、新しいPDインスタンスが参加したりしても、選択したPDインスタンスによって常に提供されます。ただし、TiDBダッシュボードを提供するPDインスタンスがクラスターから削除される（スケールインされる）と、再ネゴシエーションが発生します。交渉プロセスはユーザーの介入を必要としません。

TiDBダッシュボードを提供しないPDインスタンスにアクセスすると、ブラウザが自動的にリダイレクトされ、TiDBダッシュボードを提供するPDインスタンスにアクセスできるようになり、サービスに正常にアクセスできるようになります。このプロセスを下の画像に示します。

![Process Schematic](/media/dashboard/dashboard-ops-multiple-pd.png)

> <strong>ノート：</strong>
>
> TiDBダッシュボードを提供するPDインスタンスは、PDリーダーではない可能性があります。

### TiDBダッシュボードを実際に提供するPDインスタンスを確認します {#check-the-pd-instance-that-actually-serves-tidb-dashboard}

TiUPを使用して展開された実行中のクラスターの場合、 `tiup cluster display`コマンドを使用して、どのPDインスタンスがTiDBダッシュボードにサービスを提供しているかを確認できます。 `CLUSTER_NAME`をクラスター名に置き換えます。

{{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

```bash
tiup cluster display CLUSTER_NAME --dashboard
```

出力例は次のとおりです。

```bash
http://192.168.0.123:2379/dashboard/
```

> <strong>ノート：</strong>
>
> この機能は、 `tiup cluster`デプロイメントツールの新しいバージョン（v1.0.3以降）でのみ使用できます。
>
> <details>
> <summary>Upgrade TiUP Cluster</summary>
>
> {{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}
>
> ```bash
> tiup update --self
> tiup update cluster --force
> ```
>
> </details>

### 別のPDインスタンスに切り替えて、TiDBダッシュボードを提供します {#switch-to-another-pd-instance-to-serve-tidb-dashboard}

TiUPを使用してデプロイされた実行中のクラスターの場合、 `tiup ctl pd`コマンドを使用してTiDBダッシュボードを提供するPDインスタンスを変更するか、無効になっているときにTiDBダッシュボードを提供するPDインスタンスを再指定できます。

{{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

```bash
tiup ctl pd -u http://127.0.0.1:2379 config set dashboard-address http://9.9.9.9:2379
```

上記のコマンドでは：

-   `127.0.0.1:2379`を任意のPDインスタンスのIPとポートに置き換えます。
-   `9.9.9.9:2379`を、TiDBダッシュボードサービスを実行する新しいPDインスタンスのIPとポートに置き換えます。

`tiup cluster display`コマンドを使用して、変更が有効になっているかどうかを確認できます（ `CLUSTER_NAME`をクラスター名に置き換えます）。

{{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

```bash
tiup cluster display CLUSTER_NAME --dashboard
```

> <strong>警告：</strong>
>
> TiDBダッシュボードを実行するようにインスタンスを変更すると、以前のTiDBダッシュボードインスタンスに保存されていたローカルデータ（キーの視覚化履歴や検索履歴など）が失われます。

## TiDBダッシュボードを無効にする {#disable-tidb-dashboard}

TiUPを使用してデプロイされた実行中のクラスターの場合、 `tiup ctl pd`コマンドを使用してすべてのPDインスタンスでTiDBダッシュボードを無効にします（ `127.0.0.1:2379`を任意のPDインスタンスのIPとポートに置き換えます）。

{{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

```bash
tiup ctl pd -u http://127.0.0.1:2379 config set dashboard-address none
```

TiDBダッシュボードを無効にした後、どのPDインスタンスがTiDBダッシュボードサービスを提供するかを確認すると失敗します。

```
Error: TiDB Dashboard is disabled
```

ブラウザを介してPDインスタンスのTiDBダッシュボードアドレスにアクセスすることも失敗します。

```
Dashboard is not started.
```

## TiDBダッシュボードを再度有効にする {#re-enable-tidb-dashboard}

TiUPを使用してデプロイされた実行中のクラスターの場合、 `tiup ctl pd`コマンドを使用してPDにインスタンスを再ネゴシエートしてTiDBダッシュボードを実行するように要求します（ `127.0.0.1:2379`を任意のPDインスタンスのIPとポートに置き換えます）。

{{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

```bash
tiup ctl pd -u http://127.0.0.1:2379 config set dashboard-address auto
```

上記のコマンドを実行した後、 `tiup cluster display`コマンドを使用して、PDによって自動的にネゴシエートされたTiDBダッシュボードインスタンスアドレスを表示できます（ `CLUSTER_NAME`をクラスター名に置き換えます）。

{{&lt;コピー可能な&quot;shell-regular&quot;&gt;}}

```bash
tiup cluster display CLUSTER_NAME --dashboard
```

TiDBダッシュボードを提供するPDインスタンスを手動で指定することにより、TiDBダッシュボードを再度有効にすることもできます。 [別のPDインスタンスに切り替えて、TiDBダッシュボードを提供します](#switch-to-another-pd-instance-to-serve-tidb-dashboard)を参照してください。

> <strong>警告：</strong>
>
> 新しく有効にしたTiDBダッシュボードインスタンスが、TiDBダッシュボードを提供していた以前のインスタンスと異なる場合、以前のTiDBダッシュボードインスタンスに保存されていたローカルデータ（キーの視覚化履歴や検索履歴など）は失われます。

## 次は何ですか {#what-s-next}

-   TiDBダッシュボードUIにアクセスしてログインする方法については、 [TiDBダッシュボードにアクセスする](/dashboard/dashboard-access.md)を参照してください。

-   ファイアウォールの構成など、TiDBダッシュボードのセキュリティを強化する方法については、 [安全なTiDBダッシュボード](/dashboard/dashboard-ops-security.md)を参照してください。
