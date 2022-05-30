---
title: Use TiDB Dashboard behind a Reverse Proxy
aliases: ['/docs/dev/dashboard/dashboard-ops-reverse-proxy/']
---

# リバースプロキシの背後でTiDBダッシュボードを使用する {#use-tidb-dashboard-behind-a-reverse-proxy}

リバースプロキシを使用して、TiDBダッシュボードサービスを内部ネットワークから外部に安全に公開できます。

## 手順 {#procedures}

### ステップ1：実際のTiDBダッシュボードアドレスを取得する {#step-1-get-the-actual-tidb-dashboard-address}

複数のPDインスタンスがクラスターにデプロイされている場合、実際にTiDBダッシュボードを実行するのは1つのPDインスタンスのみです。したがって、リバースプロキシのアップストリームが正しいアドレスを指していることを確認する必要があります。このメカニズムの詳細については、 [複数のPDインスタンスを使用した展開](/dashboard/dashboard-ops-deploy.md#deployment-with-multiple-pd-instances)を参照してください。

展開にTiUPツールを使用する場合は、次のコマンドを実行して、実際のTiDBダッシュボードアドレスを取得します（ `CLUSTER_NAME`をクラスター名に置き換えます）。

{{< copyable "" >}}

```shell
tiup cluster display CLUSTER_NAME --dashboard
```

出力は実際のTiDBダッシュボードアドレスです。サンプルは次のとおりです。

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
> {{< copyable "" >}}
>
> ```bash
> tiup update --self
> tiup update cluster --force
> ```
>
> </details>

### 手順2：リバースプロキシを構成する {#step-2-configure-the-reverse-proxy}

<details>
<summary> <strong>Use HAProxy</strong> </summary>

リバースプロキシとして[HAProxy](https://www.haproxy.org/)を使用する場合は、次の手順を実行します。

1.  たとえば、 `8033`ポートでTiDBダッシュボードにリバースプロキシを使用します。 HAProxy構成ファイルに、次の構成を追加します。

    {{&lt;コピー可能&quot;&quot;&gt;}}

    ```haproxy
    frontend tidb_dashboard_front
      bind *:8033
      use_backend tidb_dashboard_back if { path /dashboard } or { path_beg /dashboard/ }

    backend tidb_dashboard_back
      mode http
      server tidb_dashboard 192.168.0.123:2379
    ```

    `192.168.0.123:2379`を[ステップ1](#step-1-get-the-actual-tidb-dashboard-address)で取得したTiDBダッシュボードの実際のアドレスのIPとポートに置き換えます。

    > <strong>警告：</strong>
    >
    > <strong>このパスのサービスのみ</strong>がリバースプロキシの背後にあることを保証するために、 `use_backend`ディレクティブの`if`の部分を保持する必要があります。そうしないと、セキュリティリスクが発生する可能性があります。 [安全なTiDBダッシュボード](/dashboard/dashboard-ops-security.md)を参照してください。

2.  設定を有効にするためにHAProxyを再起動します。

3.  リバースプロキシが有効かどうかをテストします。HAProxyが配置されているマシンの`8033`ポート（ `http://example.com:8033/dashboard/`など）の`/dashboard/`アドレスにアクセスして、TiDBダッシュボードにアクセスします。

</details>

<details>
<summary> <strong>Use NGINX</strong> </summary>

リバースプロキシとして[NGINX](https://nginx.org/)を使用する場合は、次の手順を実行します。

1.  たとえば、 `8033`ポートでTiDBダッシュボードにリバースプロキシを使用します。 NGINX構成ファイルに、次の構成を追加します。

    {{&lt;コピー可能&quot;&quot;&gt;}}

    ```nginx
    server {
        listen 8033;
        location /dashboard/ {
        proxy_pass http://192.168.0.123:2379/dashboard/;
        }
    }
    ```

    `http://192.168.0.123:2379/dashboard/`を[ステップ1](#step-1-get-the-actual-tidb-dashboard-address)で取得したTiDBダッシュボードの実際のアドレスに置き換えます。

    > <strong>警告：</strong>
    >
    > このパスの下のサービスのみが逆プロキシされるようにするには、 `proxy_pass`ディレクティブの`/dashboard/`パスを保持する必要があります。そうしないと、セキュリティリスクが発生します。 [安全なTiDBダッシュボード](/dashboard/dashboard-ops-security.md)を参照してください。

2.  構成を有効にするためにNGINXをリロードします。

    {{< copyable "" >}}

    ```shell
    sudo nginx -s reload
    ```

3.  リバースプロキシが有効かどうかをテストします。NGINXが配置されているマシンの`8033`ポート（ `http://example.com:8033/dashboard/`など）の`/dashboard/`アドレスにアクセスして、TiDBダッシュボードにアクセスします。

</details>

## パスプレフィックスをカスタマイズする {#customize-path-prefix}

TiDBダッシュボードは、デフォルトで`http://example.com:8033/dashboard/`などの`/dashboard/`のパスでサービスを提供します。これは、リバースプロキシの場合にも当てはまります。 TiDBダッシュボードサービスに`http://example.com:8033/foo/`や`http://example.com:8033/`などのデフォルト以外のパスを提供するようにリバースプロキシを構成するには、次の手順を実行します。

### ステップ1：PD構成を変更して、TiDBダッシュボードサービスのパスプレフィックスを指定します {#step-1-modify-pd-configuration-to-specify-the-path-prefix-of-tidb-dashboard-service}

PD構成の`[dashboard]`のカテゴリーの`public-path-prefix`の構成項目を変更して、TiDBダッシュボードサービスのパスプレフィックスを指定します。この項目を変更した後、PDインスタンスを再起動して、変更を有効にします。

たとえば、クラスターがTiUPを使用してデプロイされ、サービスを`http://example.com:8033/foo/`で実行する場合は、次の構成を指定できます。

{{&lt;コピー可能&quot;&quot;&gt;}}

```yaml
server_configs:
  pd:
    dashboard.public-path-prefix: /foo
```

<details>
<summary> <strong>Modify configuration when deploying a new cluster using TiUP</strong> </summary>

新しいクラスターをデプロイする場合は、上記の構成を`topology.yaml` TiUPトポロジファイルに追加して、クラスターをデプロイできます。具体的な手順については、 [TiUP導入ドキュメント](/production-deployment-using-tiup.md#step-3-initialize-cluster-topology-file)を参照してください。

</details>

<details>

<summary> <strong>Modify configuration of a deployed cluster using TiUP</strong> </summary>

デプロイされたクラスターの場合：

1.  クラスターの構成ファイルを編集モードで開きます（ `CLUSTER_NAME`をクラスター名に置き換えます）。

    {{< copyable "" >}}

    ```shell
    tiup cluster edit-config CLUSTER_NAME
    ```

2.  `server_configs`の`pd`構成で構成アイテムを変更または追加します。 `server_configs`が存在しない場合は、トップレベルに追加します。

    {{&lt;コピー可能&quot;&quot;&gt;}}

    ```yaml
    monitored:
      ...
    server_configs:
      tidb: ...
      tikv: ...
      pd:
        dashboard.public-path-prefix: /foo
      ...
    ```

    変更後の構成ファイルは、次のファイルのようになります。

    {{&lt;コピー可能&quot;&quot;&gt;}}

    ```yaml
    server_configs:
      pd:
        dashboard.public-path-prefix: /foo
      global:
        user: tidb
        ...
    ```

    または

    {{&lt;コピー可能&quot;&quot;&gt;}}

    ```yaml
    monitored:
      ...
    server_configs:
      tidb: ...
      tikv: ...
      pd:
        dashboard.public-path-prefix: /foo
    ```

3.  変更した構成を有効にするために、すべてのPDインスタンスに対してローリングリスタートを実行します（ `CLUSTER_NAME`をクラスター名に置き換えます）。

    {{< copyable "" >}}

    ```shell
    tiup cluster reload CLUSTER_NAME -R pd
    ```

詳細については、 [一般的なTiUP操作-構成を変更します](/maintain-tidb-using-tiup.md#modify-the-configuration)を参照してください。

</details>

TiDBダッシュボードサービスをルートパス（ `http://example.com:8033/`など）で実行する場合は、次の構成を使用します。

{{&lt;コピー可能&quot;&quot;&gt;}}

```yaml
server_configs:
  pd:
    dashboard.public-path-prefix: /
```

> <strong>警告：</strong>
>
> 変更およびカスタマイズされたパスプレフィックスが有効になると、TiDBダッシュボードに直接アクセスできなくなります。パスプレフィックスと一致するリバースプロキシを介してのみTiDBダッシュボードにアクセスできます。

### 手順2：リバースプロキシ構成を変更する {#step-2-modify-the-reverse-proxy-configuration}

<details>
<summary> <strong>Use HAProxy</strong> </summary>

`http://example.com:8033/foo/`を例にとると、対応するHAProxy構成は次のとおりです。

{{&lt;コピー可能&quot;&quot;&gt;}}

```haproxy
frontend tidb_dashboard_front
  bind *:8033
  use_backend tidb_dashboard_back if { path /foo } or { path_beg /foo/ }

backend tidb_dashboard_back
  mode http
  http-request set-path %[path,regsub(^/foo/?,/dashboard/)]
  server tidb_dashboard 192.168.0.123:2379
```

`192.168.0.123:2379`を[ステップ1](#step-1-get-the-actual-tidb-dashboard-address)で取得したTiDBダッシュボードの実際のアドレスのIPとポートに置き換えます。

> <strong>警告：</strong>
>
> <strong>このパスのサービスのみ</strong>がリバースプロキシの背後にあることを保証するために、 `use_backend`ディレクティブの`if`の部分を保持する必要があります。そうしないと、セキュリティリスクが発生する可能性があります。 [安全なTiDBダッシュボード](/dashboard/dashboard-ops-security.md)を参照してください。

TiDBダッシュボードサービスをルートパス（ `http://example.com:8033/`など）で実行する場合は、次の構成を使用します。

```haproxy
frontend tidb_dashboard_front
  bind *:8033
  use_backend tidb_dashboard_back
backend tidb_dashboard_back
  mode http
  http-request set-path /dashboard%[path]
  server tidb_dashboard 192.168.0.123:2379
```

構成を変更し、変更した構成を有効にするためにHAProxyを再起動します。

</details>

<details>
<summary> <strong>Use NGINX</strong> </summary>

`http://example.com:8033/foo/`を例にとると、対応するNGINX構成は次のとおりです。

{{&lt;コピー可能&quot;&quot;&gt;}}

```nginx
server {
  listen 8033;
  location /foo/ {
    proxy_pass http://192.168.0.123:2379/dashboard/;
  }
}
```

`http://192.168.0.123:2379/dashboard/`を[ステップ1](#step-1-get-the-actual-tidb-dashboard-address)で取得したTiDBダッシュボードの実際のアドレスに置き換えます。

> <strong>警告：</strong>
>
> `proxy_pass`ディレクティブの`/dashboard/`パスを保持して<strong>、このパスのサービスのみ</strong>がリバースプロキシの背後にあることを確認する必要があります。そうしないと、セキュリティリスクが発生する可能性があります。 [安全なTiDBダッシュボード](/dashboard/dashboard-ops-security.md)を参照してください。

TiDBダッシュボードサービスをルートパス（ `http://example.com:8033/`など）で実行する場合は、次の構成を使用します。

{{&lt;コピー可能&quot;&quot;&gt;}}

```nginx
server {
  listen 8033;
  location / {
    proxy_pass http://192.168.0.123:2379/dashboard/;
  }
}
```

構成を変更し、変更した構成を有効にするためにNGINXを再起動します。

{{< copyable "" >}}

```shell
sudo nginx -s reload
```

</details>

## 次は何ですか {#what-s-next}

ファイアウォールの構成など、TiDBダッシュボードのセキュリティを強化する方法については、 [安全なTiDBダッシュボード](/dashboard/dashboard-ops-security.md)を参照してください。
