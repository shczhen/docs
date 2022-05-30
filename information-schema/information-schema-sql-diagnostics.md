---
title: SQL Diagnostics
summary: Understand SQL diagnostics in TiDB.
aliases: ['/docs/dev/system-tables/system-table-sql-diagnostics/','/docs/dev/reference/system-databases/sql-diagnosis/','/docs/dev/system-tables/system-table-sql-diagnosis/','/tidb/dev/system-table-sql-diagnostics/','/tidb/dev/check-cluster-status-using-sql-statements','/docs/dev/check-cluster-status-using-sql-statements/','/docs/dev/reference/performance/check-cluster-status-using-sql-statements/']
---

# SQL診断 {#sql-diagnostics}

> <strong>警告：</strong>
>
> SQL診断はまだ実験的な機能です。実稼働環境で使用することはお勧めし<strong>ません</strong>。

SQL診断は、TiDBv4.0で導入された機能です。この機能を使用すると、TiDBの問題をより効率的に見つけることができます。 TiDB v4.0より前では、さまざまなツールを使用してさまざまな情報を取得する必要がありました。

SQL診断システムには次の利点があります。

-   システム全体のすべてのコンポーネントからの情報を統合します。
-   これは、システムテーブルを介して上位層への一貫したインターフェイスを提供します。
-   監視の概要と自動診断を提供します。
-   クラスター情報の照会が簡単になります。

## 概要 {#overview}

SQL診断システムは、次の3つの主要部分で構成されています。

-   <strong>クラスター情報テーブル</strong>：SQL診断システムは、各インスタンスの個別の情報を取得するための統一された方法を提供するクラスター情報テーブルを導入します。このシステムは、クラスタートポロジ、ハードウェア情報、ソフトウェア情報、カーネルパラメーター、監視、システム情報、低速クエリ、ステートメント、およびクラスター全体のログをテーブルに完全に統合します。したがって、SQLステートメントを使用してこれらの情報を照会できます。

-   <strong>クラスター監視テーブル</strong>：SQL診断システムはクラスター監視テーブルを導入します。これらのテーブルはすべて`metrics_schema`にあり、SQLステートメントを使用して監視情報をクエリできます。 v4.0より前の視覚化された監視と比較すると、このSQLベースの方法を使用して、クラスター全体のすべての監視情報に対して相関クエリを実行し、さまざまな期間の結果を比較して、パフォーマンスのボトルネックをすばやく特定できます。 TiDBクラスターには多くの監視メトリックがあるため、SQL診断システムは監視要約テーブルも提供し、異常な監視項目をより簡単に見つけることができます。

<strong>自動診断</strong>：SQLステートメントを手動で実行して、クラスター情報テーブル、クラスター監視テーブル、および要約テーブルを照会して問題を見つけることができますが、自動診断を使用すると、一般的な問題をすばやく見つけることができます。 SQL診断システムは、既存のクラスター情報テーブルと監視テーブルに基づいて自動診断を実行し、関連する診断結果テーブルと診断要約テーブルを提供します。

## クラスター情報テーブル {#cluster-information-tables}

クラスタ情報テーブルは、クラスタ内のすべてのインスタンスとインスタンスの情報をまとめたものです。これらのテーブルを使用すると、1つのSQLステートメントのみを使用してすべてのクラスター情報を照会できます。以下は、クラスター情報テーブルのリストです。

-   クラスタトポロジテーブル[`information_schema.cluster_info`](/information-schema/information-schema-cluster-info.md)から、クラスタの現在のトポロジ情報、各インスタンスのバージョン、バージョンに対応するGitハッシュ、各インスタンスの開始時刻、および各インスタンスの実行時刻を取得できます。
-   クラスター構成テーブル[`information_schema.cluster_config`](/information-schema/information-schema-cluster-config.md)から、クラスター内のすべてのインスタンスの構成を取得できます。 4.0より前のバージョンの場合、これらの構成情報を取得するには、各インスタンスのHTTPAPIに1つずつアクセスする必要があります。
-   クラスタハードウェアテーブル[`information_schema.cluster_hardware`](/information-schema/information-schema-cluster-hardware.md)で、クラスタハードウェア情報をすばやくクエリできます。
-   クラスター負荷テーブル[`information_schema.cluster_load`](/information-schema/information-schema-cluster-load.md)では、クラスターのさまざまなインスタンスとハードウェアタイプの負荷情報を照会できます。
-   カーネルパラメータテーブル[`information_schema.cluster_systeminfo`](/information-schema/information-schema-cluster-systeminfo.md)で、クラスタ内のさまざまなインスタンスのカーネル構成情報をクエリできます。現在、TiDBはsysctl情報のクエリをサポートしています。
-   クラスターログテーブル[`information_schema.cluster_log`](/information-schema/information-schema-cluster-log.md)で、クラスターログを照会できます。クエリ条件を各インスタンスにプッシュダウンすることにより、クラスターのパフォーマンスに対するクエリの影響は、 `grep`コマンドの影響よりも少なくなります。

TiDB v4.0より前のシステムテーブルでは、現在のインスタンスのみを表示できます。 TiDB v4.0では、対応するクラスターテーブルが導入されており、単一のTiDBインスタンスでクラスター全体のグローバルビューを表示できます。これらのテーブルは現在`information_schema`にあり、クエリ方法は他の`information_schema`のシステムテーブルと同じです。

## クラスター監視テーブル {#cluster-monitoring-tables}

さまざまな期間のクラスター状態を動的に監視および比較するために、SQL診断システムはクラスター監視システムテーブルを導入しています。すべての監視テーブルは`metrics_schema`にあり、SQLステートメントを使用して監視情報を照会できます。この方法を使用すると、クラスター全体のすべての監視情報に対して相関クエリを実行し、さまざまな期間の結果を比較して、パフォーマンスのボトルネックをすばやく特定できます。

-   [`information_schema.metrics_tables`](/information-schema/information-schema-metrics-tables.md) ：現在多くのシステムテーブルが存在するため、 `information_schema.metrics_tables`のテーブルでこれらの監視テーブルのメタ情報を照会できます。

TiDBクラスターには多くの監視メトリックがあるため、TiDBはv4.0で次の監視要約テーブルを提供します。

-   監視の要約表[`information_schema.metrics_summary`](/information-schema/information-schema-metrics-summary.md)は、すべての監視データを要約して、各監視メトリックをより効率的にチェックできるようにします。
-   [`information_schema.metrics_summary_by_label`](/information-schema/information-schema-metrics-summary.md)は、すべての監視データも要約します。特に、このテーブルは、各監視メトリックの異なるラベルを使用して統計を集約します。

## 自動診断 {#automatic-diagnostics}

上記のクラスター情報テーブルとクラスター監視テーブルでは、SQLステートメントを手動で実行してクラスターのトラブルシューティングを行う必要があります。 TiDB v4.0は、自動診断をサポートしています。既存の基本情報テーブルに基づいた診断関連のシステムテーブルを使用して、診断が自動的に実行されるようにすることができます。自動診断に関連するシステムテーブルは次のとおりです。

-   診断結果表[`information_schema.inspection_result`](/information-schema/information-schema-inspection-result.md)は、システムの診断結果を示しています。診断は受動的にトリガーされます。 `select * from inspection_result`を実行すると、システムを診断するためのすべての診断ルールがトリガーされ、システムの障害またはリスクが結果に表示されます。
-   診断要約表[`information_schema.inspection_summary`](/information-schema/information-schema-inspection-summary.md)は、特定のリンクまたはモジュールの監視情報を要約したものです。モジュール全体またはリンクのコンテキストに基づいて、問題のトラブルシューティングと特定を行うことができます。
