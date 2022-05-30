---
title: TiDB Dashboard Instance Profiling - Continuous Profiling
summary: Learn how to collect performance data from TiDB, TiKV and PD continuously to reduce MTTR.
---

# TiDBダッシュボードインスタンスプロファイリング-継続的なプロファイリング {#tidb-dashboard-instance-profiling-continuous-profiling}

> <strong>ノート：</strong>
>
> この機能は、データベースの専門家向けに設計されています。専門家以外のユーザーの場合は、PingCAPテクニカルサポートの指導の下でこの機能を使用することをお勧めします。

継続的プロファイリングにより、各TiDB、TiKV、およびPDインスタンスからパフォーマンスデータを<strong>継続的</strong>に収集できます。収集されたパフォーマンスデータは、FlameGraphまたはDAGとして視覚化できます。

これらのパフォーマンスデータを使用して、専門家はインスタンスのCPUやメモリなどのリソース消費の詳細を分析し、CPUオーバーヘッドの高さ、メモリ使用量の高さ、プロセスのストールなど、高度なパフォーマンスの問題をいつでも特定できます。問題を再現できない場合でも、専門家はその時点で収集された過去のパフォーマンスデータを表示することで問題を深く掘り下げることができます。このようにして、MTTRを効果的に減らすことができます。

## 手動プロファイリングと比較する {#compare-with-manual-profiling}

連続プロファイリングは、 [手動プロファイリング](/dashboard/dashboard-profiling.md)の拡張機能です。これらは両方とも、インスタンスごとにさまざまな種類のパフォーマンスデータを収集および分析するために使用できます。それらの違いは次のとおりです。

-   手動プロファイリングは、プロファイリングを開始した時点で短時間（たとえば、30秒）のパフォーマンスデータのみを収集しますが、連続プロファイリングは、有効になっている場合は継続的にデータを収集します。
-   手動プロファイリングは現在発生している問題の分析にのみ使用できますが、継続的プロファイリングは現在の問題と過去の問題の両方の分析に使用できます。
-   手動プロファイリングでは特定のインスタンスの特定のパフォーマンスデータを収集できますが、連続プロファイリングではすべてのインスタンスのすべてのパフォーマンスデータを収集できます。
-   連続プロファイリングはより多くのパフォーマンスデータを保存するため、より多くのディスクスペースを使用します。
-   連続プロファイリングは、プロファイリングが頻繁に実行される場合の安定性への影響のため、現在TiFlashからパフォーマンスデータを収集しません。

## サポートされているパフォーマンスデータ {#supported-performance-data}

安定性の理由で収集されないTiFlashCPUデータを除いて、 [手動プロファイリング](/dashboard/dashboard-profiling.md#supported-performance-data)のすべてのパフォーマンスデータが収集されます。

-   CPU：TiDB、TiKV、およびPDインスタンスの各内部機能のCPUオーバーヘッド

    > TiKVインスタンスのCPUオーバーヘッドは、現在ARMアーキテクチャではサポートされていません。

-   ヒープ：TiDBおよびPDインスタンスの各内部関数のメモリ消費量

-   ミューテックス：TiDBおよびPDインスタンスでのミューテックス競合状態

-   Goroutine：TiDBおよびPDインスタンス上のすべてのgoroutineの実行状態と呼び出しスタック

## ページにアクセスする {#access-the-page}

次のいずれかの方法を使用して、[ContinuousProfiling]ページにアクセスできます。

-   TiDBダッシュボードにログインした後、左側のナビゲーションバーで[<strong>高度なデバッグ</strong>]&gt;[<strong>インスタンスのプロファイリング</strong>]&gt;[<strong>継続的なプロファイリング</strong>]をクリックします。

    ![Access page](/media/dashboard/dashboard-conprof-access.png)

-   ブラウザで[http://127.0.0.1:2379/dashboard/#/continuous_profiling](http://127.0.0.1:2379/dashboard/#/continuous_profiling)にアクセスします。 `127.0.0.1:2379`を実際のPDインスタンスのアドレスとポートに置き換えます。

## 継続的なプロファイリングを有効にする {#enable-continuous-profiling}

> <strong>ノート：</strong>
>
> 継続的プロファイリングを使用するには、クラスターを最新バージョンのTiUP（v1.9.0以降）またはTiDBオペレーター（v1.3.0以降）でデプロイまたはアップグレードする必要があります。以前のバージョンのTiUPまたはTiDBOperatorを使用してクラスターをアップグレードした場合は、手順について[よくある質問](/dashboard/dashboard-faq.md#a-required-component-ngmonitoring-is-not-started-error-is-shown)を参照してください。

継続的プロファイリングはデフォルトでは有効になっていません。これを有効にすると、Webページを常にアクティブに保つことなく、パフォーマンスデータをバックグラウンドで継続的に収集できます。収集されたデータは一定期間保持でき、期限切れのデータは自動的にクリアされます。

この機能を有効にするには：

1.  [継続的なプロファイリングページ](#access-the-page)にアクセスします。
2.  [<strong>設定を開く]</strong>をクリックします。右側の<strong>[設定]</strong>領域で、[<strong>機能</strong>を有効にする]をオンに切り替え、必要に応じて<strong>保持期間</strong>のデフォルト値を変更します。
3.  [<strong>保存]</strong>をクリックします。

![Enable feature](/media/dashboard/dashboard-conprof-start.png)

## 現在のパフォーマンスデータを表示する {#view-current-performance-data}

手動プロファイリングは、継続的プロファイリングが有効になっているクラスターでは開始できません。現時点でのパフォーマンスデータを表示するには、最新のプロファイリング結果をクリックするだけです。

## 過去のパフォーマンスデータを表示する {#view-historical-performance-data}

リストページでは、この機能を有効にしてから収集されたすべてのパフォーマンスデータを確認できます。

![History results](/media/dashboard/dashboard-conprof-history.png)

## パフォーマンスデータをダウンロードする {#download-performance-data}

プロファイリング結果ページで、右上隅にある[<strong>プロファイリング結果のダウンロード</strong>]をクリックして、すべてのプロファイリング結果をダウンロードできます。

![Download profiling result](/media/dashboard/dashboard-conprof-download.png)

テーブル内の個々のインスタンスをクリックして、そのプロファイリング結果を表示することもできます。または、...にカーソルを合わせて生データをダウンロードすることもできます。

![View profiling result](/media/dashboard/dashboard-conprof-single.png)

## 継続的なプロファイリングを無効にする {#disable-continuous-profiling}

1.  [継続的なプロファイリングページ](#access-the-page)にアクセスします。
2.  右上隅にある歯車のアイコンをクリックして、設定ページを開きます。 [<strong>機能</strong>を有効にする]をオフに切り替えます。
3.  [<strong>保存]</strong>をクリックします。
4.  ポップアップダイアログボックスで、[<strong>無効</strong>にする]をクリックします。

![Disable feature](/media/dashboard/dashboard-conprof-stop.png)

## よくある質問 {#frequently-asked-questions}

<strong>1.連続プロファイリングを有効にできず、UIに「必要なコンポーネントNgMonitoringが開始されていません」と表示されます</strong>。

[TiDBダッシュボードFAQ](/dashboard/dashboard-faq.md#a-required-component-ngmonitoring-is-not-started-error-is-shown)を参照してください。

<strong>2.継続的プロファイリングを有効にした後、パフォーマンスは影響を受けますか？</strong>

ベンチマークによると、この機能を有効にした場合の平均パフォーマンスへの影響は1％未満です。

<strong>3.この機能のステータスは何ですか？</strong>

現在、一般的に利用可能な（GA）機能であり、実稼働環境で使用できます。
