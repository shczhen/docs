---
title: Deploy Monitoring Services for the TiDB Cluster
summary: Learn how to deploy monitoring services for the TiDB cluster.
aliases: ['/docs/dev/deploy-monitoring-services/','/docs/dev/how-to/monitor/monitor-a-cluster/','/docs/dev/monitor-a-tidb-cluster/']
---

# TiDBクラスターの監視サービスを展開する {#deploy-monitoring-services-for-the-tidb-cluster}

このドキュメントは、TiDB監視およびアラートサービスを手動で展開するユーザーを対象としています。

TiUPを使用してTiDBクラスターを展開する場合、監視およびアラートサービスは自動的に展開され、手動で展開する必要はありません。

## PrometheusとGrafanaをデプロイする {#deploy-prometheus-and-grafana}

TiDBクラスタートポロジが次のとおりであると想定します。

| 名前    | ホストIP           | サービス                                    |
| :---- | :-------------- | :-------------------------------------- |
| Node1 | 192.168.199.113 | PD1、TiDB、node_export、Prometheus、Grafana |
| Node2 | 192.168.199.114 | PD2、node_export                         |
| Node3 | 192.168.199.115 | PD3、node_export                         |
| Node4 | 192.168.199.116 | TiKV1、node_export                       |
| Node5 | 192.168.199.117 | TiKV2、node_export                       |
| Node6 | 192.168.199.118 | TiKV3、node_export                       |

### ステップ1：バイナリパッケージをダウンロードする {#step-1-download-the-binary-package}

{{< copyable "" >}}

```bash
# Downloads the package.
wget https://download.pingcap.org/prometheus-2.27.1.linux-amd64.tar.gz
wget https://download.pingcap.org/node_exporter-0.17.0.linux-amd64.tar.gz
wget https://download.pingcap.org/grafana-6.1.6.linux-amd64.tar.gz
```

{{< copyable "" >}}

```bash
# Extracts the package.
tar -xzf prometheus-2.27.1.linux-amd64.tar.gz
tar -xzf node_exporter-0.17.0.linux-amd64.tar.gz
tar -xzf grafana-6.1.6.linux-amd64.tar.gz
```

### ステップ2：Node1、Node2、Node3、およびNode4で<code>node_exporter</code>を開始します {#step-2-start-code-node-exporter-code-on-node1-node2-node3-and-node4}

{{< copyable "" >}}

```bash
cd node_exporter-0.17.0.linux-amd64

# Starts the node_exporter service.
$ ./node_exporter --web.listen-address=":9100" \
    --log.level="info" &
```

### ステップ3：Node1でPrometheusを起動します {#step-3-start-prometheus-on-node1}

Prometheus構成ファイルを編集します。

{{< copyable "" >}}

```bash
cd prometheus-2.27.1.linux-amd64 &&
vi prometheus.yml
```

```ini
...

global:
  scrape_interval:     15s  # By default, scrape targets every 15 seconds.
  evaluation_interval: 15s  # By default, scrape targets every 15 seconds.
  # scrape_timeout is set to the global default value (10s).
  external_labels:
    cluster: 'test-cluster'
    monitor: "prometheus"

scrape_configs:
  - job_name: 'overwritten-nodes'
    honor_labels: true  # Do not overwrite job & instance labels.
    static_configs:
    - targets:
      - '192.168.199.113:9100'
      - '192.168.199.114:9100'
      - '192.168.199.115:9100'
      - '192.168.199.116:9100'
      - '192.168.199.117:9100'
      - '192.168.199.118:9100'

  - job_name: 'tidb'
    honor_labels: true  # Do not overwrite job & instance labels.
    static_configs:
    - targets:
      - '192.168.199.113:10080'

  - job_name: 'pd'
    honor_labels: true  # Do not overwrite job & instance labels.
    static_configs:
    - targets:
      - '192.168.199.113:2379'
      - '192.168.199.114:2379'
      - '192.168.199.115:2379'

  - job_name: 'tikv'
    honor_labels: true  # Do not overwrite job & instance labels.
    static_configs:
    - targets:
      - '192.168.199.116:20180'
      - '192.168.199.117:20180'
      - '192.168.199.118:20180'

...

```

Prometheusサービスを開始します。

```bash
$ ./prometheus \
    --config.file="./prometheus.yml" \
    --web.listen-address=":9090" \
    --web.external-url="http://192.168.199.113:9090/" \
    --web.enable-admin-api \
    --log.level="info" \
    --storage.tsdb.path="./data.metrics" \
    --storage.tsdb.retention="15d" &
```

### ステップ4：Node1でGrafanaを起動する {#step-4-start-grafana-on-node1}

Grafana構成ファイルを編集します。

{{< copyable "" >}}

```ini
cd grafana-6.1.6 &&
vi conf/grafana.ini

...

[paths]
data = ./data
logs = ./data/log
plugins = ./data/plugins
[server]
http_port = 3000
domain = 192.168.199.113
[database]
[session]
[analytics]
check_for_updates = true
[security]
admin_user = admin
admin_password = admin
[snapshots]
[users]
[auth.anonymous]
[auth.basic]
[auth.ldap]
[smtp]
[emails]
[log]
mode = file
[log.console]
[log.file]
level = info
format = text
[log.syslog]
[event_publisher]
[dashboards.json]
enabled = false
path = ./data/dashboards
[metrics]
[grafana_net]
url = https://grafana.net

...

```

Grafanaサービスを開始します。

{{< copyable "" >}}

```bash
./bin/grafana-server \
    --config="./conf/grafana.ini" &
```

## Grafanaを構成する {#configure-grafana}

このセクションでは、Grafanaを構成する方法について説明します。

### ステップ1：Prometheusデータソースを追加する {#step-1-add-a-prometheus-data-source}

1.  GrafanaWebインターフェースにログインします。

    -   デフォルトアドレス： [http：// localhost：3000](http://localhost:3000)

    -   デフォルトのアカウント：admin

    -   デフォルトのパスワード：admin

    > <strong>ノート：</strong>
    >
    > [<strong>パスワードの変更]</strong>ステップでは、[<strong>スキップ]</strong>を選択できます。

2.  Grafanaサイドバーメニューで、<strong>構成</strong>内の<strong>データソース</strong>をクリックします。

3.  [<strong>データソースの追加]を</strong>クリックします。

4.  データソース情報を指定します。

    -   データソースの<strong>名前</strong>を指定します。
    -   [<strong>タイプ]</strong>で、[<strong>プロメテウス]</strong>を選択します。
    -   <strong>URL</strong>には、Prometheusアドレスを指定します。
    -   必要に応じて他のフィールドを指定します。

5.  [<strong>追加]</strong>をクリックして、新しいデータソースを保存します。

### ステップ2：Grafanaダッシュボードをインポートする {#step-2-import-a-grafana-dashboard}

PDサーバー、TiKVサーバー、およびTiDBサーバーのGrafanaダッシュボードをインポートするには、それぞれ次の手順を実行します。

1.  Grafanaロゴをクリックして、サイドバーメニューを開きます。

2.  サイドバーメニューで、[<strong>ダッシュボード</strong>]-&gt; [<strong>インポート</strong>]をクリックして、[ダッシュボードの<strong>インポート</strong>]ウィンドウを開きます。

3.  [ <strong>.jsonファイルのアップロード]</strong>をクリックして、JSONファイルをアップロードします（ダウンロード[TiDBGrafana構成ファイル](https://github.com/pingcap/tidb-ansible/tree/master/scripts) ）。

    > <strong>ノート：</strong>
    >
    > `tidb.json` 、PD、および`tikv_trouble_shooting.json`ダッシュボードの場合、対応する`tidb_summary.json`ファイルは`tikv_summary.json` 、および`pd.json` `tikv_details.json` 。

4.  [<strong>ロード]</strong>をクリックします。

5.  Prometheusデータソースを選択します。

6.  [<strong>インポート]</strong>をクリックします。 Prometheusダッシュボードがインポートされます。

## コンポーネントメトリックを表示する {#view-component-metrics}

トップメニューの[<strong>新しいダッシュボード</strong>]をクリックして、表示するダッシュボードを選択します。

![view dashboard](/media/view-dashboard.png)

クラスタコンポーネントについて、次のメトリックを取得できます。

-   <strong>TiDBサーバー：</strong>

    -   レイテンシとスループットを監視するためのクエリ処理時間
    -   DDLプロセスの監視
    -   TiKVクライアント関連の監視
    -   PDクライアント関連の監視

-   <strong>PDサーバー：</strong>

    -   コマンドが実行される合計回数
    -   特定のコマンドが失敗した合計回数
    -   コマンドが成功する期間
    -   コマンドが失敗する期間
    -   コマンドが終了して結果を返す期間

-   <strong>TiKVサーバー：</strong>

    -   ガベージコレクション（GC）の監視
    -   TiKVコマンドが実行される合計回数
    -   スケジューラがコマンドを実行する期間
    -   ラフトプロポーザルコマンドの合計回数
    -   Raftがコマンドを実行する期間
    -   Raftコマンドが失敗した合計回数
    -   Raftが準備完了状態を処理する合計回数
