---
title: TiDB 6.0.0 Release Notes
---

# TiDB 6.0.0 リリースノート {#tidb-6-0-0-release-notes}

リリース日:2022 年 4 月 7 日

TiDB バージョン:6.0.0-DMR

6.0.0-DMR の主な新機能または改善点は次のとおりです。

- SQL の配置ルールをサポートして、データ配置をより柔軟に管理できます。
- カーネルレベルでデータとインデックスの整合性チェックを追加し、システムの安定性と堅牢性を高め、リソースのオーバーヘッドを非常に少なくします。
- Top SQL、非専門家向けのセルフサービスのデータベースパフォーマンス監視および診断機能を提供します。
- クラスターパフォーマンスデータを常に収集する継続的プロファイリングをサポートし、技術専門家の MTTR を削減します。
- ホットスポットの小さなテーブルをメモリにキャッシュすると、アクセスパフォーマンスが大幅に向上し、スループットが向上し、アクセスレイテンシーが短縮されます。
- インメモリペシミスティックロックを最適化します。悲観的ロックに起因するパフォーマンスのボトルネックの下で、悲観的ロックのメモリ最適化により、レイテンシーを効果的に 10％削減し、QPS を 10％増加させることができます。
- プリペアドステートメントを拡張して実行プランを共有することで、CPU リソースの消費を減らし、SQL の実行効率を向上させます。
- より多くの式のプッシュダウンとエラスティックスレッドプールの一般可用性 (GA) をサポートすることにより、MPP エンジンのコンピューティングパフォーマンスを向上させます。
- DM WebUI を追加して、多数の移行タスクの管理を容易にします。
- 大規模なクラスターでデータを複製するときの tiCDC の安定性と効率を向上させます。ticDC は 100,000 テーブルの同時複製をサポートするようになりました。
- TikV ノードの再起動後にリーダーのバランシングを加速し、再起動後のビジネスリカバリの速度を向上させます。
- 統計の自動更新のキャンセルをサポートします。これにより、リソースの競合が軽減され、SQL パフォーマンスへの影響が制限されます。
- TiDB クラスターの自動診断サービス PingCap Clinic (テクニカルプレビュー版) を提供します。
- エンタープライズレベルのデータベース管理プラットフォームである TiDB Enterprise Manager を提供します。

また、TiDB の HTAP ソリューションのコアコンポーネントとして、TiFlash <sup> TM </sup> は、このリリースで正式にオープンソースです。詳細については、[tiFlash リポジトリ](https://github.com/pingcap/tiflash) を参照してください。

## リリース戦略の変更 {#release-strategy-changes}

TiDB v6.0.0 から、TiDB は 2 つのタイプのリリースを提供します。

- 長期サポートリリース

  長期サポート (LTS) のリリースは、約 6 か月ごとにリリースされます。LTS リリースでは、新しい機能と改善が導入され、リリースライフサイクル内でパッチリリースが受け入れられます。たとえば、v6.1.0 は LTS リリースになります。

- 開発マイルストーンリリース

  開発マイルストーンリリース (DMR) は、約 2 か月ごとにリリースされます。DMR は新しい機能と改善を導入しますが、パッチリリースは受け付けません。オンプレミスのユーザーが本番環境で DMR を使用することは推奨されません。たとえば、v6.0.0-DMR は DMR です。

TiDB v6.0.0 は DMR で、そのバージョンは 6.0.0-DMR です。

## 新機能 {#new-features}

### SQL {#sql}

- データの SQL ベースの配置ルール

  TiDB はスケーラビリティに優れた分散データベースです。通常、データは複数のサーバーまたは複数のデータセンターに展開されます。したがって、データスケジューリング管理は TiDB の最も重要な基本機能の 1 つです。ほとんどの場合、ユーザーはデータのスケジュールと管理の方法を気にする必要はありません。しかし、アプリケーションの複雑さが増すにつれて、分離とアクセスレイテンシーに起因するデプロイメントの変更は、TiDB にとって新たな課題となっています。v6.0.0 以降、TiDB は SQL インタフェースに基づくデータスケジューリングおよび管理機能を公式に提供します。レプリカ数、ロールタイプ、あらゆるデータの配置場所などのディメンションで、柔軟なスケジュール設定と管理をサポートします。また、TiDB は、マルチサービス共有クラスターと Cross-AZ 配備におけるデータ配置のより柔軟な管理もサポートします。

  [ユーザードキュメント](/placement-rules-in-sql.md)

- データベースによる TiFlash レプリカの構築をサポートします。データベース内のすべてのテーブルに TiFlash レプリカを追加するには、単一の SQL 文を使用するだけで、運用コストとメンテナンスコストを大幅に節約できます。

  [ユーザードキュメント](/tiflash/use-tiflash.md#create-tiflash-replicas-for-databases)

### トランザクション {#transaction}

- カーネルレベルでのデータインデックスの整合性チェックを追加する

  トランザクション実行時のデータインデックスの整合性チェックを追加します。これにより、リソースのオーバーヘッドが非常に少なくなり、システムの安定性と堅牢性が向上します。`tidb_enable_mutation_checker` 変数と `tidb_txn_assertion_level` 変数を使用して、チェックの動作を制御できます。デフォルト構成では、ほとんどのシナリオで QPS ドロップは 2% 以内に制御されます。整合性チェックのエラーの説明については、[ユーザードキュメント](/troubleshoot-data-inconsistency-errors.md) を参照してください。

### 可観測性 {#observability}

- 上位 SQL: 専門家以外のためのパフォーマンス診断

  Top SQL は、DBA およびアプリケーション開発者向けの TiDB ダッシュボードのセルフサービス型データベースパフォーマンス監視および診断機能で、TiDB v6.0 で一般提供されています。

  専門家向けの既存の診断機能とは異なり、Top SQL は非専門家向けに設計されています。相関関係を見つけたり、Raft Snapshot、RockSDB、MVCC、TSO などの TiDB 内部メカニズムを理解したりするために、何千もの監視チャートをトラバースする必要はありません。Top SQL を使用してデータベースの負荷をすばやく分析し、アプリのパフォーマンスを向上させるには、基本的なデータベース知識（インデックス、ロック競合、実行計画など）のみが必要です。

  Top SQL はデフォルトでは有効になっていません。有効にすると、Top SQL は各 TikV または TiDB ノードのリアルタイム CPU 負荷を提供します。したがって、高い CPU 負荷を消費する SQL 文を一目で見つけて、データベースのホットスポットや急激な負荷の増加などの問題をすばやく分析できます。たとえば、Top SQL を使用して、単一の TikV ノードの 90% の CPU を消費する異常なクエリを特定して診断できます。

  [ユーザードキュメント](/dashboard/top-sql.md)

- 継続的なプロファイリングをサポート

  TiDB ダッシュボードには、TiDB v6.0 で一般提供されている継続的プロファイリング機能が導入されています。継続的プロファイリングはデフォルトでは有効になっていません。有効にすると、個々の TiDB、TikV、および PD インスタンスのパフォーマンスデータが常に収集されますが、オーバーヘッドはごくわずかです。履歴パフォーマンスデータを使用して、技術専門家は、問題の再現が困難な場合でも、メモリ消費量の増加などの問題の根本原因を突き止めて特定できます。このようにして、平均回復時間（MTTR）を短縮できます。

  [ユーザードキュメント](/dashboard/continuous-profiling.md)

### パフォーマンス {#performance}

- ホットスポットの小さなテーブルをキャッシュする

  ホットスポットの小さなテーブルにアクセスするシナリオのユーザーアプリケーションでは、TiDB はホットスポットテーブルをメモリに明示的にキャッシュすることをサポートしています。これにより、アクセスパフォーマンスが大幅に向上し、スループットが向上し、アクセスレイテンシーが短縮されます。このソリューションにより、サードパーティのキャッシュミドルウェアの導入を効果的に回避し、アーキテクチャの複雑さを軽減し、運用と保守のコストを削減できます。このソリューションは、構成テーブルや為替レートテーブルなど、小さなテーブルが頻繁にアクセスされるがほとんど更新されないシナリオに適しています。

  [ユーザードキュメント](/cached-tables.md), PLACEHOLDER-2-PLACEHOLDER-E}

- インメモリペシミスティックロック

  TiDB v6.0.0 以降、インメモリペシミスティックロックはデフォルトで有効になっています。この機能を有効にすると、悲観的なトランザクションロックがメモリで管理されます。これにより、悲観的ロックの持続とロック情報の Raft レプリケーションが回避され、悲観的なトランザクションロックを管理するオーバーヘッドが大幅に削減されます。悲観的ロックに起因するパフォーマンスのボトルネックの下で、悲観的ロックのメモリ最適化により、レイテンシーを効果的に 10％削減し、QPS を 10％増加させることができます。

  [ユーザードキュメント](/pessimistic-transaction.md#in-memory-pessimistic-lock), PLACEHOLDER-2-PLACEHOLDER-E}

- 読み取りコミット分離レベルで TSO を取得するための最適化

  クエリレイテンシを減らすために、読み取りと書き込みの競合がまれな場合、TiDB は `tidb_rc_read_check_ts` システム変数を [コミットされた分離レベルの読み取り](/transaction-isolation-levels.md#read-committed-isolation-level) に追加して、不要な TSO を少なくします。この変数はデフォルトでは無効になっています。変数が有効な場合、この最適化により、読み取りと書き込みの競合がないシナリオでレイテンシを短縮するために TSO が重複することを回避します。ただし、読み取りと書き込みの競合が頻繁に発生するシナリオでは、この変数を有効にするとパフォーマンスが低下する可能性があります。

  [ユーザードキュメント](/transaction-isolation-levels.md#read-committed-isolation-level), PLACEHOLDER-2-PLACEHOLDER-E}

- 準備されたステートメントを強化して実行計画を共有

  SQL 実行プランを再利用すると、SQL 文の解析時間を効果的に短縮し、CPU リソースの消費を減らし、SQL 実行効率を向上させることができます。SQL チューニングの重要な方法の 1 つは、SQL 実行プランを効果的に再利用することです。TiDB は、プリペアドステートメントを使用した実行計画の共有をサポートしました。ただし、準備されたステートメントが閉じられると、TiDB は対応するプランキャッシュを自動的にクリアします。その後、TiDB は繰り返される SQL 文を不必要に解析し、実行効率に影響を与える可能性があります。v6.0.0 以降、TiDB は `tidb_ignore_prepared_cache_close_stmt` パラメーター (デフォルトでは無効) を介して `COM_STMT_CLOSE` コマンドを無視するかどうかの制御をサポートしています。パラメータを有効にすると、TiDB はプリペアドステートメントを閉じるコマンドを無視し、実行プランをキャッシュに保持し、実行プランの再利用率を向上させます。

  [ユーザードキュメント](/sql-prepared-plan-cache.md#ignore-the-com_stmt_close-command-and-the-deallocate-prepare-statement), PLACEHOLDER-2-PLACEHOLDER-E}

- クエリのプッシュダウンを改善

  コンピューティングとストレージを分離するネイティブアーキテクチャにより、TiDB は演算子をプッシュダウンして無効なデータを除外することをサポートします。これにより、TiDB と TikV 間のデータ転送が大幅に削減され、クエリ効率が向上します。v6.0.0 では、TiDB はより多くの式と `BIT` データ型を TikV にプッシュダウンすることをサポートし、式とデータ型を計算する際のクエリ効率を向上させます。

  [ユーザードキュメント](/functions-and-operators/expressions-pushed-down.md#add-to-the-blocklist), PLACEHOLDER-2-PLACEHOLDER-E}

- TiFlash MPP エンジンでパーティションテーブルの動的プルーニングモードをサポート (実験的)

  このモードでは、TiDB は TiFlash の MPP エンジンを使用してパーティションテーブル上のデータを読み取り、計算できます。これにより、パーティションテーブルのクエリパフォーマンスが大幅に向上します。

  [ユーザードキュメント](/tiflash/use-tiflash.md#access-partitioned-tables-in-the-mpp-mode)

- MPP エンジンのコンピューティングパフォーマンスを向上させる

  - より多くの機能とオペレーターを MPP エンジンに押し下げるのをサポート

    - 論理関数:`IS`, PLACEHOLDER-3-PLACEHOLDER-E}
    - 文字列関数:`REGEXP()`, PLACEHOLDER-3-PLACEHOLDER-E}
    - 数学関数:`GREATEST(int/real)`, PLACEHOLDER-3-PLACEHOLDER-E}
    - 日付関数:`DAYOFNAME()`、PLACEHOLDER-3-PLACEHOLDER-E}、PLACEHOLDER-7-PLACEHOLDER-E}、PLACEHOLDER-9-PLACEHOLDER-E}、PLACEHOLDER-11-PLACEHOLDER-11-PLACEHOLDER-E}
    - 演算子:反左外部半結合、左外部半結合

    [ユーザードキュメント](/tiflash/use-tiflash.md#supported-push-down-calculations)

  - エラスティックスレッドプール (デフォルトで有効) は GA になります。この機能は CPU 使用率の向上を目的としています。

    [ユーザードキュメント](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file)

### 安定性 {#stability}

- 実行計画のベースライン取得を強化する

  テーブル名、頻度、ユーザー名などのディメンションを含むブロックリストを追加することで、実行計画のベースラインキャプチャの使いやすさを向上させます。キャッシングバインディングのメモリ管理を最適化する新しいアルゴリズムを導入します。ベースラインキャプチャを有効にすると、システムはほとんどの OLTP クエリのバインディングを自動的に作成します。バインドされたステートメントの実行プランは固定されており、実行プランの変更によるパフォーマンスの問題を回避できます。ベースラインキャプチャは、メジャーバージョンのアップグレードやクラスターの移行などのシナリオに適用でき、実行計画の回帰によって引き起こされるパフォーマンスの問題を軽減するのに役立ちます。

  [ユーザードキュメント](/sql-plan-management.md#baseline-capturing), PLACEHOLDER-2-PLACEHOLDER-E}

- TikV クォータリミッターをサポート (実験的)

  TikV でデプロイされたマシンのリソースが限られていて、フォアグラウンドに過剰な量の要求がかかっている場合、バックグラウンド CPU リソースがフォアグラウンドで占有され、TikV のパフォーマンスが不安定になります。TiDB v6.0.0 では、クォータ関連の構成項目を使用して、CPU や読み取り/書き込み帯域幅など、フォアグラウンドで使用されるリソースを制限できます。これにより、長期にわたる負荷の高いワークロード下でのクラスターの安定性が大幅に向上します。

  [ユーザードキュメント](/tikv-configuration-file.md#quota), PLACEHOLDER-2-PLACEHOLDER-E}

- TiFlash の zstd 圧縮アルゴリズムをサポート

  TiFlash には、`profiles.default.dt_compression_method` と `profiles.default.dt_compression_level` という 2 つのパラメーターが導入されています。これにより、ユーザーはパフォーマンスと容量のバランスに基づいて最適な圧縮アルゴリズムを選択できます。

  [ユーザードキュメント](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file)

- デフォルトですべての I/O チェック (チェックサム) を有効にする

  この機能は、v5.4.0 で実験的に導入されました。ユーザーのビジネスに明らかな影響を与えることなく、データの正確性とセキュリティを強化します。

  警告:新しいバージョンのデータ形式は、v5.4.0 より前のバージョンにダウングレードできません。このようなダウングレード中は、TiFlash レプリカを削除し、ダウングレード後にデータをレプリケートする必要があります。あるいは、[dttool 移行](/tiflash/tiflash-command-line-flags.md#dttool-migrate) を参照してダウングレードを実行することもできます。

  [ユーザードキュメント](/tiflash/use-tiflash.md#use-data-validation)

- スレッド使用率を向上させる

  TiFlash は非同期 gRPC と min-TSO スケジューリングメカニズムを導入しています。このようなメカニズムにより、スレッドのより効率的な使用が保証され、過剰なスレッドによるシステムクラッシュが回避されます。

  [ユーザードキュメント](/tiflash/monitor-tiflash.md#coprocessor)

### データ移行 {#data-migration}

#### TiDB データ移行 (DM) {#tidb-data-migration-dm}

- WebUI を追加 (実験的)

  WebUI を使用すると、多数の移行タスクを簡単に管理できます。WebUI では、次のことができます。

  - ダッシュボードで移行タスクを表示する
  - 移行タスクを管理する
  - 上流設定を構成する
  - 複製ステータスの照会
  - マスターとワーカーの情報を表示する

  WebUI はまだ実験的で、まだ開発中です。したがって、試用版のみに推奨されます。既知の問題は、WebUI と dmctl を使用して同じタスクを操作すると問題が発生する可能性があることです。この問題は後のバージョンで解決される予定です。

  [ユーザードキュメント](/dm/dm-webui-guide.md)

- エラー処理メカニズムを追加する

  移行タスクを中断する問題に対処するために、さらに多くのコマンドが導入されています。たとえば、次のようになります。

  - スキーマエラーが発生した場合は、スキーマファイルを個別に編集する代わりに、`binlog-schema update` コマンドの `--from-source/--from-target` パラメーターを使用してスキーマファイルを更新できます。
  - バイナリログの位置を指定して、DDL ステートメントを挿入、置換、スキップ、または元に戻すことができます。

  [ユーザードキュメント](/dm/dm-manage-schema.md)

- Amazon S3 への完全なデータストレージをサポート

  DM がすべてのデータ移行タスクまたは完全なデータ移行タスクを実行する場合、アップストリームからの完全なデータを格納するのに十分なハードディスク容量が必要です。EBS と比較して、Amazon S3 には低コストでほぼ無限のストレージがあります。現在、DM は Amazon S3 をダンプディレクトリとして設定することをサポートしています。つまり、すべてのデータ移行タスクまたは完全なデータ移行タスクを実行するときに、S3 を使用して完全なデータを保存できます。

  [ユーザードキュメント](/dm/task-configuration-file-full.md#task-configuration-file-template-advanced)

- 指定された時間からの移行タスクの開始をサポート

  新しいパラメーター `--start-time` が移行タスクに追加されました。時間は「2021-10-21 00:01:00」または「2021-10-21T 00:01:00」の形式で定義できます。

  この機能は、シャード mysql インスタンスから増分データを移行およびマージするシナリオで特に便利です。特に、増分移行タスクでは、ソースごとにバイナリログの開始点を設定する必要はありません。代わりに、`safe-mode` の `--start-time` パラメーターを使用すると、増分移行タスクをすばやく作成できます。

  [ユーザードキュメント](/dm/dm-create-task.md#flags-description)

#### TiDB ライトニング {#tidb-lightning}

- 許容されるエラーの最大数の構成をサポート

  設定項目 `lightning.max-error` を追加しました。デフォルト値は 0 です。値が 0 より大きい場合、max-error 機能が有効になります。エンコード中に行でエラーが発生した場合、この行を含むレコードがターゲット TiDB の `lightning_task_info.type_error_v1` に追加され、この行は無視されます。エラーのある行がしきい値を超えると、TiDB Lightning はただちに終了します。

  `lightning.max-error` 構成と一致する `lightning.task-info-schema-name` 構成項目は、データ保存エラーを報告するデータベースの名前を記録します。

  この機能はすべてのタイプのエラーをカバーするわけではありません。たとえば、構文エラーは適用されません。

  [ユーザードキュメント](/tidb-lightning/tidb-lightning-error-resolution.md#type-error)

### TiDB データ共有サブスクリプション {#tidb-data-share-subscription}

- 100,000 テーブルの同時複製をサポート

  データ処理フローを最適化することにより、tiCDC は各テーブルの増分データ処理のリソース消費を削減し、大規模なクラスターでデータを複製する際のレプリケーションの安定性と効率を大幅に向上させます。内部テストの結果は、ticDC が 100,000 個のテーブルを同時に複製することを安定してサポートできることを示しています。

### 導入とメンテナンス {#deployment-and-maintenance}

- 新しい照合ルールを既定で有効にする

  v4.0 以降、TiDB は、大文字と小文字を区別しない、アクセントを区別しない、およびパディングルールで MySQL と同じように動作する新しい照合ルールをサポートしました。新しい照合ルールは、デフォルトでは無効になっていた `new_collations_enabled_on_first_bootstrap` パラメーターによって制御されます。v6.0 以降、TiDB はデフォルトで新しい照合ルールを有効にします。この設定は TiDB クラスターの初期化時にのみ有効になることに注意してください。

  [ユーザードキュメント](/tidb-configuration-file.md#new_collations_enabled_on_first_bootstrap)

- TikV ノードの再起動後にリーダーのバランシングを加速

  TikV ノードの再起動後、不均等に分散しているリーダーは、負荷分散のために再配分する必要があります。大規模クラスターでは、リーダーのバランシング時間はリージョンの数と正の相関関係があります。たとえば、100K リージョンのリーダーバランシングには 20〜30 分かかることがあり、不均一な負荷によるパフォーマンスの問題や安定性のリスクが発生しやすくなります。TiDB v6.0.0 は、バランシングの同時実行を制御するパラメーターを提供し、デフォルト値を元の 4 倍に拡大します。これにより、リーダーのリバランス時間が大幅に短縮され、TikV ノードの再起動後のビジネスリカバリが加速されます。

  [ユーザードキュメント](/pd-control.md#scheduler-config-balance-leader-scheduler), PLACEHOLDER-2-PLACEHOLDER-E}

- 統計の自動更新のキャンセルをサポートする

  統計は、SQL のパフォーマンスに影響を与える最も重要な基本データの 1 つです。統計の完全性と適時性を保証するために、TiDB はバックグラウンドでオブジェクト統計を定期的に自動的に更新します。ただし、統計情報の自動更新によってリソースの競合が発生し、SQL のパフォーマンスに影響を与える可能性があります。この問題に対処するには、v6.0 以降の統計情報の自動更新を手動でキャンセルできます。

  [ユーザードキュメント](/statistics.md#automatic-update)

- PingCap Clinic 診断サービス (テクニカルプレビュー版)

  PingCap Clinic は TiDB クラスターの診断サービスです。このサービスは、クラスターの問題をリモートでトラブルシューティングするのに役立ち、ローカルでクラスターのステータスをすばやくチェックします。PingCap Clinic を使用すると、ライフサイクル全体にわたって TiDB クラスターの安定した運用を確保し、潜在的な問題を予測し、問題の可能性を減らし、クラスターの問題の迅速なトラブルシューティングを行うことができます。

  クラスタの問題をトラブルシューティングするためにリモートアシスタンスについて PingCap テクニカルサポートに連絡する場合、PingCap Clinic サービスを使用して診断データを収集してアップロードすることで、トラブルシューティングの効率を向上させることができます。

  [ユーザードキュメント](/clinic/clinic-introduction.md)

- エンタープライズレベルのデータベース管理プラットフォーム、TiDB Enterprise Manager

  TiDB Enterprise Manager (TiEM) は、TiDB データベースをベースにしたエンタープライズレベルのデータベース管理プラットフォームで、オンプレミス環境またはパブリッククラウド環境でユーザーが TiDB クラスターを管理できるようにすることを目的としています。

  TiEM は、TiDB クラスタのライフサイクル全体を視覚的に管理できるだけでなく、パラメータ管理、バージョンアップグレード、クラスタクローン、アクティブ/スタンバイクラスタスイッチング、データのインポートとエクスポート、データレプリケーション、データのバックアップと復元などのワンストップサービスも提供します。TiEM は、TiDB 上の DevOps の効率を向上させ、企業の DevOps コストを削減することができます。

  現在、アイテムは [TiDB エンタープライズ](https://en.pingcap.com/tidb-enterprise/) エディションでのみ提供されています。アイテムを入手するには、[TiDB エンタープライズ](https://en.pingcap.com/tidb-enterprise/) ページからお問い合わせください。

- 監視コンポーネントの設定のカスタマイズをサポート

  TiUp を使用して TiDB クラスターをデプロイすると、TiUp は Prometheus、Grafana、Alertmanager などのモニタリングコンポーネントを自動的にデプロイし、スケールアウト後に新しいノードを監視スコープに自動的に追加します。`topology.yaml` ファイルに設定項目を追加することで、監視コンポーネントの設定をカスタマイズできます。

  [ユーザードキュメント](/tiup/customized-montior-in-tiup-environment.md)

## 互換性の変更 {#compatibility-changes}

> **注意:**
>
> 以前の TiDB バージョンから v6.0.0 にアップグレードする場合、すべての中間バージョンの互換性変更ノートを知りたい場合は、対応するバージョンの [リリースノート](/releases/release-notes.md) を確認してください。

### システム変数 {#system-variables}

| Variable name                                                                                                                                                                             | Change type | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :---------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `placement_checks`                                                                                                                                                                        | Deleted     | Controls whether the DDL statement validates the placement rules specified by [SQL の配置ルール](/placement-rules-in-sql.md). Replaced by `tidb_placement_mode`.                                                                                                                                                                                                                                                                                                                                                            |
| `tidb_enable_alter_placement`                                                                                                                                                             | Deleted     | Controls whether to enable [SQL の配置ルール](/placement-rules-in-sql.md).                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `tidb_mem_quota_hashjoin`<br/>`tidb_mem_quota_indexlookupjoin`<br/>`tidb_mem_quota_indexlookupreader` <br/>`tidb_mem_quota_mergejoin`<br/>`tidb_mem_quota_sort`<br/>`tidb_mem_quota_topn` | Deleted     | Since v5.0, these variables have been replaced by `tidb_mem_quota_query` and removed from the [システム変数](/system-variables.md) document. To ensure compatibility, these variables were kept in source code. Since TiDB 6.0.0, these variables are removed from the code, too.                                                                                                                                                                                                                                           |
| [`tidb_enable_mutation_checker`](/system-variables.md#tidb_enable_mutation_checker-new-in-v600)                                                                                           | Newly added | Controls whether to enable the mutation checker. The default value is `ON`.                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| [`tidb_ignore_prepared_cache_close_stmt`](/system-variables.md#tidb_ignore_prepared_cache_close_stmt-new-in-v600)                                                                         | Newly added | Controls whether to ignore the command that closes Prepared Statement. The default value is `OFF`.                                                                                                                                                                                                                                                                                                                                                                                                                          |
| [`tidb_mem_quota_binding_cache`](/system-variables.md#tidb_mem_quota_binding_cache-new-in-v600)                                                                                           | Newly added | Sets the memory usage threshold for the cache holding `binding`. The default value is `67108864` (64 MiB).                                                                                                                                                                                                                                                                                                                                                                                                                  |
| [`tidb_placement_mode`](/system-variables.md#tidb_placement_mode-new-in-v600)                                                                                                             | Newly added | Controls whether DDL statements ignore the placement rules specified by [SQL の配置ルール](/placement-rules-in-sql.md). The default value is `strict`, which means that DDL statements do not ignore placement rules.                                                                                                                                                                                                                                                                                                       |
| [`tidb_rc_read_check_ts`](/system-variables.md#tidb_rc_read_check_ts-new-in-v600)                                                                                                         | Newly added | <ul><li> Optimizes read statement latency within a transaction. If read/write conflicts are more severe, turning this variable on will add additional overhead and latency, causing regressions in performance. The default value is `off`.</li><li> This variable is not yet compatible with [レプリカ-読み取り](/system-variables.md#tidb_replica_read-new-in-v40). If a read request has `tidb_rc_read_check_ts` on, it might not be able to use replica-read. Do not turn on both variables at the same time.</li></ul> |
| [`tidb_sysdate_is_now`](/system-variables.md#tidb_sysdate_is_now-new-in-v600)                                                                                                             | Newly added | Controls whether the `SYSDATE` function can be replaced by the `NOW` function. This configuration item has the same effect as the MySQL option [`sysdate-is-now`](https://dev.mysql.com/doc/refman/8.0/en/server-options.html#option_mysqld_sysdate-is-now). The default value is `OFF`.                                                                                                                                                                                                                                    |
| [`tidb_table_cache_lease`](/system-variables.md#tidb_table_cache_lease-new-in-v600)                                                                                                       | Newly added | Controls the lease time of [テーブルキャッシュ](/cached-tables.md), in seconds. The default value is `3`.                                                                                                                                                                                                                                                                                                                                                                                                                   |
| [`tidb_top_sql_max_meta_count`](/system-variables.md#tidb_top_sql_max_meta_count-new-in-v600)                                                                                             | Newly added | Controls the maximum number of SQL statement types collected by [上位 SQL](/dashboard/top-sql.md) per minute. The default value is `5000`.                                                                                                                                                                                                                                                                                                                                                                                  |
| [`tidb_top_sql_max_time_series_count`](/system-variables.md#tidb_top_sql_max_time_series_count-new-in-v600)                                                                               | Newly added | Controls how many SQL statements that contribute the most to the load (that is, top N) can be recorded by [上位 SQL](/dashboard/top-sql.md) per minute. The default value is `100`.                                                                                                                                                                                                                                                                                                                                         |
| [`tidb_txn_assertion_level`](/system-variables.md#tidb_txn_assertion_level-new-in-v600)                                                                                                   | Newly added | Controls the assertion level. The assertion is a consistency check between data and indexes, which checks whether a key being written exists in the transaction commit process. By default, the check enables most of the check items, with almost no impact on performance.                                                                                                                                                                                                                                                |

### 設定ファイルパラメータ {#configuration-file-parameters}

| Configuration file | Configuration                                                                                                                                                                                                    | Change type | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| :----------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :---------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| TiDB               | `stmt-summary.enable` <br/> `stmt-summary.enable-internal-query` <br/> `stmt-summary.history-size` <br/> `stmt-summary.max-sql-length` <br/> `stmt-summary.max-stmt-count` <br/> `stmt-summary.refresh-interval` | Deleted     | Configuration related to the [ステートメントサマリーテーブル](/statement-summary-tables.md). All these configuration items are removed. You need to use SQL variables to control the statement summary tables.                                                                                                                                                                                                                                                                                                |
| TiDB               | [`new_collations_enabled_on_first_bootstrap`](/tidb-configuration-file.md#new_collations_enabled_on_first_bootstrap)                                                                                             | Modified    | Controls whether to enable support for the new collation. Since v6.0, the default value is changed from `false` to `true`. This configuration item only takes effect when the cluster is initialized for the first time. After the first bootstrap, you cannot enable or disable the new collation framework using this configuration item.                                                                                                                                                                   |
| TiKV               | [`backup.num-threads`](/tikv-configuration-file.md#num-threads-1)                                                                                                                                                | Modified    | The value range is modified to `[1, CPU]`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| TiKV               | [`raftstore.apply-max-batch-size`](/tikv-configuration-file.md#apply-max-batch-size)                                                                                                                             | Modified    | The maximum value is changed to `10240`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| TiKV               | [`raftstore.raft-max-size-per-msg`](/tikv-configuration-file.md#raft-max-size-per-msg)                                                                                                                           | Modified    | <ul><li>The minimum value is changed from `0` to larger than `0`.</li><li>The maximum value is set to `3GB`.</li><li>The unit is changed from `MB` to `KB\|MB\|GB`.</li></ul>                                                                                                                                                                                                                                                                                                                                 |
| TiKV               | [`raftstore.store-max-batch-size`](/tikv-configuration-file.md#store-max-batch-size)                                                                                                                             | Modified    | The maximum value is set to `10240`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| TiKV               | [`readpool.unified.max-thread-count`](/tikv-configuration-file.md#max-thread-count)                                                                                                                              | Modified    | The adjustable range is changed to `[min-thread-count, MAX(4, CPU)]`.                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| TiKV               | [`rocksdb.enable-pipelined-write`](/tikv-configuration-file.md#enable-pipelined-write)                                                                                                                           | Modified    | The default value is changed from `true` to `false`. When this configuration is enabled, the previous Pipelined Write is used. When this configuration is disabled, the new Pipelined Commit mechanism is used.                                                                                                                                                                                                                                                                                               |
| TiKV               | [`rocksdb.max-background-flushes`](/tikv-configuration-file.md#max-background-flushes)                                                                                                                           | Modified    | <ul><li>When the number of CPU cores is 10, the default value is `3`.</li><li>When the number of CPU cores is 8, the default value is `2`.</li></ul>                                                                                                                                                                                                                                                                                                                                                          |
| TiKV               | [`rocksdb.max-background-jobs`](/tikv-configuration-file.md#max-background-jobs)                                                                                                                                 | Modified    | <ul><li>When the number of CPU cores is 10, the default value is `9`.</li><li>When the number of CPU cores is 8, the default value is `7`.</li></ul>                                                                                                                                                                                                                                                                                                                                                          |
| TiFlash            | [`profiles.default.dt_enable_logical_split`](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file)                                                                                                   | Modified    | Determines whether the segment of DeltaTree Storage Engine uses logical split. The default value is changed from `true` to `false`.                                                                                                                                                                                                                                                                                                                                                                           |
| TiFlash            | [`profiles.default.enable_elastic_threadpool`](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file)                                                                                                 | Modified    | Controls whether to enable the elastic thread pool. The default value is changed from `false` to `true`.                                                                                                                                                                                                                                                                                                                                                                                                      |
| TiFlash            | [`storage.format_version`](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file)                                                                                                                     | Modified    | Controls the data validation feature of TiFlash. The default value is changed from `2` to `3`.<br/>When `format_version` is set to `3`, consistency check is performed on the read operations for all TiFlash data to avoid incorrect read due to hardware failure.<br/>Note that the new format version cannot be downgraded in place to versions earlier than v5.4.                                                                                                                                         |
| TiDB               | [`pessimistic-txn.pessimistic-auto-commit`](/tidb-configuration-file.md#pessimistic-auto-commit-new-in-v600)                                                                                                     | Newly added | Determines the transaction mode that the auto-commit transaction uses when the pessimistic transaction mode is globally enabled (`tidb_txn_mode='pessimistic'`).                                                                                                                                                                                                                                                                                                                                              |
| TiKV               | [`pessimistic-txn.in-memory`](/tikv-configuration-file.md#in-memory-new-in-v600)                                                                                                                                 | Newly added | Controls whether to enable the in-memory pessimistic lock. With this feature enabled, pessimistic transactions store pessimistic locks in TiKV memory as much as possible, instead of writing pessimistic locks to disks or replicating to other replicas. This improves the performance of pessimistic transactions; however, there is a low probability that a pessimistic lock will be lost, which might cause the pessimistic transaction to fail to commit. The default value is `true`.                 |
| TiKV               | [`quota`](/tikv-configuration-file.md#quota)                                                                                                                                                                     | Newly added | Add configuration items related to Quota Limiter, which limit the resources occupied by frontend requests. Quota Limiter is an experimental feature and is disabled by default. New quota-related configuration items are `foreground-cpu-time`, `foreground-write-bandwidth`, `foreground-read-bandwidth`, and `max-delay-duration`.                                                                                                                                                                         |
| TiFlash            | [`profiles.default.dt_compression_method`](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file)                                                                                                     | Newly added | Specifies the compression algorithm for TiFlash. The optional values are `LZ4`, `zstd` and `LZ4HC`, all case insensitive. The default value is `LZ4`.                                                                                                                                                                                                                                                                                                                                                         |
| TiFlash            | [`profiles.default.dt_compression_level`](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file)                                                                                                      | Newly added | Specifies the compression level of TiFlash. The default value is `1`.                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| DM                 | [`loaders.<name>.import-mode`](/dm/task-configuration-file-full.md#task-configuration-file-template-advanced)                                                                                                    | Newly added | The import mode during the full import phase. Since v6.0, DM uses TiDB Lightning’s TiDB-backend mode to import data during the full import phase; the previous Loader component is no longer used. This is an internal replacement and has no obvious impact on daily operations.<br/>The default value is set to `sql`, which means using `tidb-backend` mode. In some rare cases, `tidb-backend` might not be fully compatible. You can fall back to Loader mode by configuring this parameter to `loader`. |
| DM                 | [`loaders.<name>.on-duplicate`](/dm/task-configuration-file-full.md#task-configuration-file-template-advanced)                                                                                                   | Newly added | Specifies the methods to resolve conflicts during the full import phase. The default value is `replace`, which means using the new data to replace the existing data.                                                                                                                                                                                                                                                                                                                                         |
| TiCDC              | [`dial-timeout`](/ticdc/manage-ticdc.md#configure-sink-uri-with-kafka)                                                                                                                                           | Newly added | The timeout in establishing a connection with the downstream Kafka. The default value is `10s`.                                                                                                                                                                                                                                                                                                                                                                                                               |
| TiCDC              | [`read-timeout`](/ticdc/manage-ticdc.md#configure-sink-uri-with-kafka)                                                                                                                                           | Newly added | The timeout in getting a response returned by the downstream Kafka. The default value is `10s`.                                                                                                                                                                                                                                                                                                                                                                                                               |
| TiCDC              | [`write-timeout`](/ticdc/manage-ticdc.md#configure-sink-uri-with-kafka)                                                                                                                                          | Newly added | The timeout in sending a request to the downstream Kafka. The default value is `10s`.                                                                                                                                                                                                                                                                                                                                                                                                                         |

### その他 {#others}

- データ配置ポリシーには、次の互換性変更があります。
  - バインディングはサポートされていません。直接配置オプションは構文から削除されました。
  - `CREATE PLACEMENT POLICY` および `ALTER PLACEMENT POLICY` ステートメントは、`VOTERS` および `VOTER_CONSTRAINTS` 配置オプションをサポートしなくなりました。
  - TiDB エコシステムツール (TiDB Binlog、ticDC、および BR) は、配置ルールと互換性があります。配置オプションは TiDB Binlog の特別なコメントに移動されました。
  - `information_schema.placement_rules` システムテーブルの名前が `information_schema.placement_policies` に変更されました。このテーブルには、配置ポリシーに関する情報のみが表示されます。
  - `placement_checks` システム変数は `tidb_placement_mode` に置き換えられます。
  - TiFlash レプリカを持つテーブルに、配置ルールを持つパーティションを追加することは禁止されています。
  - `INFORMATION_SCHEMA` テーブルから `TIDB_DIRECT_PLACEMENT` 列を削除します。
- SQL プラン管理 (SPM) バインディングの `status` の値が変更されました。
  - `using` を削除します。
  - `using` (利用可能) を追加して `using` を置き換えます。
  - `disabled` を追加します (利用不可)。
- DM が OpenAPI インターフェースを変更
  - 内部メカニズムの変更により、タスク管理に関連するインターフェイスは、以前の実験バージョンと互換性がありません。適応するには、新しい [DM OpenAPI ドキュメント](/dm/dm-open-api.md) を参照する必要があります。
- DM は、完全インポートフェーズで競合を解決する方法を変更します
  - `loader.<name>.on-duplicate` パラメーターが追加されます。デフォルト値は `replace` です。これは、新しいデータを使用して既存のデータを置き換えることを意味します。以前の動作を維持したい場合は、値を `error` に設定できます。このパラメータは、完全インポートフェーズ中の動作のみを制御します。
- DM を使用するには、`dmctl` の対応するバージョンを使用する必要があります
  - 内部メカニズムの変更により、DM を v6.0.0 にアップグレードした後、`dmctl` も v6.0.0 にアップグレードする必要があります。
- v5.4 (v5.4 のみ) では、TiDB で一部の noop システム変数に不正な値が許可されています。v6.0.0 以降、TiDB はシステム変数に誤った値を設定することを禁止しています。[#31538](https://github.com/pingcap/tidb/issues/31538)

## 改良 {#improvements}

- TiDB

  - `FLASHBACK` または `RECOVER` ステートメント [#31668](https://github.com/pingcap/tidb/issues/31668) を使用してテーブルをリストアした後、テーブルの配置ルール設定を自動的にクリアします。
  - パフォーマンス概要ダッシュボードを追加して、一般的なクリティカルパスのコアパフォーマンスメトリクスを表示し、TiDB でのメトリクス分析をより簡単にします [#31676](https://github.com/pingcap/tidb/issues/31676)
  - `REPLACE` ステートメントで `REPLACE` キーワードの使用をサポート [#24515](https://github.com/pingcap/tidb/issues/24515)
  - 範囲パーティションテーブル [#26739](https://github.com/pingcap/tidb/issues/26739) の組み込み式 [#26739](https://github.com/pingcap/tidb/issues/26739) のパーティションプルーニングをサポート
  - MPP 集約クエリで潜在的に冗長な Exchange 操作を排除することにより、クエリの効率を向上 [#31762](https://github.com/pingcap/tidb/issues/31762)
  - `TRUNCATE PARTITION` と `DROP PARTITION` ステートメント [#31681](https://github.com/pingcap/tidb/issues/31681) でパーティション名の重複を許可することで、MySQL との互換性を高めます
  - `CREATE_TIME` ステートメント [#23494](https://github.com/pingcap/tidb/issues/23494) ステートメントの結果に `CREATE_TIME` 情報を表示することをサポート
  - 新しい組み込み関数 `CHARSET()` [#3931](https://github.com/pingcap/tidb/issues/3931) をサポート
  - ユーザー名によるベースラインキャプチャブロックリストのフィルタリングをサポート [#32558](https://github.com/pingcap/tidb/issues/32558)
  - ベースラインキャプチャブロックリストでのワイルドカードの使用をサポート [#32714](https://github.com/pingcap/tidb/issues/32714)
  - `ADMIN SHOW DDL JOBS` および `SHOW TABLE STATUS` ステートメントの結果を最適化するには、現在の `time_zone` に従って時間を表示します。
  - `DAYNAME()` と `MONTHNAME()` 関数を tiFlash [#32594](https://github.com/pingcap/tidb/issues/32594) にプッシュダウン
  - `REGEXP` 関数を tiFlash [#32637](https://github.com/pingcap/tidb/issues/32637) に押し下げる
  - `DAYOFMONTH()` と `LAST_DAY()` 関数を tiFlash [#33012](https://github.com/pingcap/tidb/issues/33012) にプッシュダウン
  - `DAYOFWEEK()` と `DAYOFYEAR()` 関数を tiFlash [#33130](https://github.com/pingcap/tidb/issues/33130) にプッシュダウン
  - `IS_TRUE`、`IS_FALSE`、および `IS_TRUE_WITH_NULL` 関数を tiFlash [#33047](https://github.com/pingcap/tidb/issues/33047) にプッシュダウン
  - `GREATEST` と `LEAST` 関数を tiFlash [#32787](https://github.com/pingcap/tidb/issues/32787) にプッシュダウン
  - `UnionScan` 演算子 [#32631](https://github.com/pingcap/tidb/issues/32631) の実行の追跡をサポート
  - `_tidb_rowid` 列 [#31543](https://github.com/pingcap/tidb/issues/31543) を読み取るクエリで PointGet プランを使用するサポート
  - `EXPLAIN` ステートメントの出力で、名前を小文字に変換せずに元のパーティション名を表示することをサポート [#32719](https://github.com/pingcap/tidb/issues/32719)
  - IN 条件および文字列型カラム [#32626](https://github.com/pingcap/tidb/issues/32626) の RANGE COLUMNS パーティショニングのパーティションプルーニングを有効
  - システム変数が NULL に設定されている場合にエラーメッセージを返す [#32850](https://github.com/pingcap/tidb/issues/32850)
  - 非 MPP モードからブロードキャスト参加を削除 [#31465](https://github.com/pingcap/tidb/issues/31465)
  - 動的プルーニングモードでの分割テーブルでの MPP プランの実行をサポート [#32347](https://github.com/pingcap/tidb/issues/32347)
  - 共通テーブル式 (CTE) の述語のプッシュダウンをサポート [#28163](https://github.com/pingcap/tidb/issues/28163)
  - `Statement Summary` と `Capture Plan Baselines` の構成を簡略化し、グローバルベースでのみ利用可能にする [#30557](https://github.com/pingcap/tidb/issues/30557)
  - gopsutil を v3.21.12 に更新し、macOS 12 でバイナリをビルドするときに報告されるアラームに対処する PLACEHOLDER-1-PLACEHOLDER-E}

- TikV

  - 多くのキー範囲を持つバッチの Raftstore のサンプリング精度を向上させる [#12327](https://github.com/tikv/tikv/issues/12327)
  - `debug/pprof/profile` に正しい「コンテンツタイプ」を追加して、プロファイルをより簡単に識別できるようにします [#11521](https://github.com/tikv/tikv/issues/11521)
  - Raftstore がハートビートを持っているとき、または読み取り要求を処理するときに、リーダーのリース時間を無限に更新します。これにより、レイテンシジッターの軽減に役立ちます [#11579](https://github.com/tikv/tikv/issues/11579)
  - リーダーを切り替えるときはコストが最も低いストアを選択してください。パフォーマンスの安定性が向上します [#10602](https://github.com/tikv/tikv/issues/10602)
  - Raftstore [#11320](https://github.com/tikv/tikv/issues/11320) をブロックすることによって生じるパフォーマンスのジッターを減らすために、Raftlogs を非同期にフェッチする
  - ベクトル計算で `QUARTER` 関数をサポートする [#5751](https://github.com/tikv/tikv/issues/5751)
  - `BIT` データ型を TikV [#30738](https://github.com/pingcap/tidb/issues/30738) にプッシュダウンするサポート
  - `MOD` 関数と `SYSDATE` 関数を TikV [#11916](https://github.com/tikv/tikv/issues/11916) にプッシュダウンすることをサポートします
  - ロックの解決ステップ [#11993](https://github.com/tikv/tikv/issues/11993) を必要とするリージョンの数を減らすことで、ticDC の回復時間を短縮します。
  - `raftstore.raft-max-inflight-msgs` [#11865](https://github.com/tikv/tikv/issues/11865) を動的に変更
  - `EXTRA_PHYSICAL_TABLE_ID_COL_ID` をサポートして動的プルーニングモードを有効にする [#11888](https://github.com/tikv/tikv/issues/11888)
  - バケットでの計算をサポート [#11759](https://github.com/tikv/tikv/issues/11759)
  - rawKV API V2 のキーを `user-key` + `memcomparable-padding` + `timestamp` [#11965](https://github.com/tikv/tikv/issues/11965) としてエンコードします
  - rawKV API V2 の値を `user-value` + `ttl` + \{B-PLACEHOLDER-5-PLACEHOLDER としてエンコードし、`delete` を `ValueMeta` でエンコードします。PLACEHOLDER-11-PLACEHOLDER-E}
  - `raftstore.raft-max-size-per-msg` [#12017](https://github.com/tikv/tikv/issues/12017) を動的に変更
  - Grafana [#12014](https://github.com/tikv/tikv/issues/12014) でのマルチ K8 の監視をサポート
  - リーダーシップを CDC オブザーバーに移してレイテンシのジッターを減らす [#12111](https://github.com/tikv/tikv/issues/12111)
  - `raftstore.apply_max_batch_size` と `raftstore.store_max_batch_size` [#11982](https://github.com/tikv/tikv/issues/11982) の動的
  - rawkV V2 は `raw_get` または `raw_scan` リクエスト [#11965](https://github.com/tikv/tikv/issues/11965) を受け取ると、最新バージョンを返します
  - rcCheckts 整合性読み取りをサポート [#12097](https://github.com/tikv/tikv/issues/12097)
  - `storage.scheduler-worker-pool-size` (スケジューラプールのスレッド数) の動的な変更をサポート [#12067](https://github.com/tikv/tikv/issues/12067)
  - グローバルフォアグラウンドフローコントローラーを使用して CPU と帯域幅の使用を制御し、tikV [#11855](https://github.com/tikv/tikv/issues/11855) のパフォーマンスの安定性を向上させます。
  - `readpool.unified.max-thread-count` (UnifyReadPOOL のスレッド数) の動的な変更をサポート [#11781](https://github.com/tikv/tikv/issues/11781)
  - TikV 内部パイプラインを使用して rocksDB パイプラインを置き換え、`rocksdb.enable-multibatch-write` パラメーターを非推奨 [#12059](https://github.com/tikv/tikv/issues/12059)

- PD

  - リーダーを退去させるときに、移送する最速のオブジェクトを自動的に選択するようサポートします。これにより、立ち退きプロセスのスピードアップに役立ちます [#4229](https://github.com/tikv/pd/issues/4229)
  - リージョンが利用できなくなった場合に、2 つのレプリカのいかだグループから投票者を削除することを禁止する [#4564](https://github.com/tikv/pd/issues/4564)
  - バランスリーダーのスケジューリングをスピードアップする [#4652](https://github.com/tikv/pd/issues/4652)

- tiFlash

  - TiFlash ファイルの論理分割を禁止します (デフォルト値の `profiles.default.dt_enable_logical_split` を `false` に調整します。詳細については [ユーザードキュメント](/tiflash/tiflash-configuration.md#tiflash-configuration-parameters) を参照してください）、TiFlash 列指向ストレージのスペース使用効率を向上させ、TiFlash に同期するテーブルのスペース占有が TiKV でのテーブルのスペース占有とほぼ同じになるようにします。
  - 以前のクラスタ管理モジュールを TiDB に統合することにより、TiFlash のクラスタ管理とレプリカレプリケーションメカニズムを最適化します。これにより、小さなテーブルのレプリカ作成が高速化されます [#29924](https://github.com/pingcap/tidb/issues/29924)

- ツール

  - バックアップと復元 (BR)

    - バックアップデータの復元速度を向上させます。シミュレーションテストでは、BR が 16TB のデータを 15 ノード（各ノードに 16 個の CPU コアがある）の TikV クラスタにリストアすると、スループットは 2.66 GiB/秒に達します。[#27036](https://github.com/pingcap/tidb/issues/27036)

    - 配置ルールのインポートとエクスポートをサポートします。`--with-tidb-placement-mode` パラメーターを追加して、データをインポートするときに配置ルールを無視するかどうかを制御します。[#32290](https://github.com/pingcap/tidb/issues/32290)

  - ticDC

    - `Lag analyze` パネルを Grafana に追加する [#4891](https://github.com/pingcap/tiflow/issues/4891)
    - 配置ルールをサポート [#4846](https://github.com/pingcap/tiflow/issues/4846)
    - HTTP API 処理の同期 [#1710](https://github.com/pingcap/tiflow/issues/1710)
    - チェンジフィードを再開するためのエクスポネンシャルバックオフメカニズムを追加 [#3329](https://github.com/pingcap/tiflow/issues/3329)
    - MySQL シンクのデフォルトの独立性レベルを読み取りコミットに設定して、MySQL のデッドロックを減らします [#3589](https://github.com/pingcap/tiflow/issues/3589)
    - 作成時にチェンジフィードパラメータを検証し、エラーメッセージを絞り込む [#1716](https://github.com/pingcap/tiflow/issues/1716) [#1718](https://github.com/pingcap/tiflow/issues/1718) PLACEHOLDER-5-PLACEHOLDER-E} [#4472](https://github.com/pingcap/tiflow/issues/4472)
    - Kafka プロデューサーの設定パラメーターを公開して、ticDC [#4385](https://github.com/pingcap/tiflow/issues/4385) で設定可能にする

  - TiDB データ移行 (DM)

    - 上流のテーブルスキーマに一貫性がなく、オプティミスティックモードの場合のタスクの開始をサポート [#3629](https://github.com/pingcap/tiflow/issues/3629) [#3708](https://github.com/pingcap/tiflow/issues/3708) PLACEHOLDER-5-PLACEHOLDER-E}
    - `stopped` 状態のタスクの作成をサポート [#4484](https://github.com/pingcap/tiflow/issues/4484)
    - 内部ファイルを書き込むために `/tmp` ではなく DM-Worker の作業ディレクトリを使用して Syncer をサポートし、タスクが停止した後にディレクトリをクリーニングする [#4107](https://github.com/pingcap/tiflow/issues/4107)
    - 事前チェックが改善されました。一部の重要なチェックはスキップされなくなりました。[#3608](https://github.com/pingcap/tiflow/issues/3608)

  - TiDB ライトニング

    - 再試行可能なエラータイプをさらに追加 [#31376](https://github.com/pingcap/tidb/issues/31376)
    - base64 形式のパスワード文字列 [#31194](https://github.com/pingcap/tidb/issues/31194) をサポートする
    - エラーコードとエラー出力を標準化 [#32239](https://github.com/pingcap/tidb/issues/32239)

## バグ修正 {#bug-fixes}

- TiDB

  - `SCHEDULE = majority_in_primary` と `PrimaryRegion` と `Regions` が同じ値である場合、TiDB が配置ルールを持つテーブルを作成できないバグを修正 [#31271](https://github.com/pingcap/tidb/issues/31271)
  - インデックスルックアップ結合 [#30468](https://github.com/pingcap/tidb/issues/30468) を使用してクエリを実行するときの `invalid transaction` エラーを修正
  - `show grants` が 2 つ以上の権限が付与された場合に誤った結果を返すバグを修正 [#30855](https://github.com/pingcap/tidb/issues/30855)
  - `INSERT INTO t1 SET timestamp_col = DEFAULT` が `CURRENT_TIMESTAMP` [#29926](https://github.com/pingcap/tidb/issues/29926) にデフォルト設定されているフィールドのタイムスタンプをゼロタイムスタンプに設定するバグを修正
  - 文字列型 [#31721](https://github.com/pingcap/tidb/issues/31721) の最大値と最小の非ヌル値のエンコードを避けて、結果の読み取りで報告されたエラーを修正しました
  - データがエスケープ文字 [#31589](https://github.com/pingcap/tidb/issues/31589) で壊れている場合にロードデータパニックを修正
  - 照合順序を持つ `greatest` または `least` 関数が間違った結果になる問題を修正しました [#31789](https://github.com/pingcap/tidb/issues/31789)
  - date_add 関数と date_sub 関数が不正なデータ型を返すことがあるバグを修正 [#31809](https://github.com/pingcap/tidb/issues/31809)
  - INSERT ステートメント [#31735](https://github.com/pingcap/tidb/issues/31735) を使用して仮想的に生成されたカラムにデータを挿入するときに発生する可能性があったパニックを修正
  - 作成したリストパーティション [#31784](https://github.com/pingcap/tidb/issues/31784) に重複する列が存在してもエラーが報告されない不具合を修正
  - `select for update union select` が不適切なスナップショットを使用している場合に返される間違った結果を修正 [#31530](https://github.com/pingcap/tidb/issues/31530)
  - 復元操作が完了した後にリージョンが不均等に分散される可能性があるという潜在的な問題を修正 [#31034](https://github.com/pingcap/tidb/issues/31034)
  - `json` タイプ [#31541](https://github.com/pingcap/tidb/issues/31541) で強制性が間違っているバグを修正
  - この型が組み込み関数 [#31320](https://github.com/pingcap/tidb/issues/31320) を使用して処理されるときに、`json` 型の誤った照合順序を修正しました
  - TiFlash レプリカの数が 0 に設定されている場合、PD ルールが削除されないバグを修正 [#32190](https://github.com/pingcap/tidb/issues/32190)
  - `alter column set default` がテーブルスキーマを誤って更新する問題を修正 [#31074](https://github.com/pingcap/tidb/issues/31074)
  - TiDB の `date_format` が MySQL と互換性のない方法で処理する問題を修正しました [#32232](https://github.com/pingcap/tidb/issues/32232)
  - join [#31629](https://github.com/pingcap/tidb/issues/31629) を使用して分割テーブルを更新するとエラーが発生する可能性があるバグを修正
  - 列挙型の値 [#32428](https://github.com/pingcap/tidb/issues/32428) に対する Nulleq 関数の間違った範囲の計算結果を修正
  - `upper()` と `lower()` 関数で発生する可能性があったパニックを修正 PLACEHOLDER-5-PLACEHOLDER-
  - 他のタイプのカラムをタイムスタンプタイプのカラムに変更するときに発生したタイムゾーンの問題を修正 [#29585](https://github.com/pingcap/tidb/issues/29585)
  - ChunkRPC \{B-PLACEHOLDER-1-PLACEHOLDER [#30880](https://github.com/pingcap/tidb/issues/30880) を使用してデータをエクスポートするときの TiDB OOM を修正
  - 動的パーティションプルーニングモードでサブ SELECT LIMIT が期待どおりに動作しないバグを修正 [#32516](https://github.com/pingcap/tidb/issues/32516)
  - `INFORMATION_SCHEMA.COLUMNS` テーブル [#32655](https://github.com/pingcap/tidb/issues/32655) のビットデフォルト値の間違った形式または一貫性のない形式を修正しました
  - サーバーの再起動後にパーティションテーブルの一覧表示でパーティションテーブルのプルーニングが機能しないことがあるバグを修正 [#32416](https://github.com/pingcap/tidb/issues/32416)
  - `add column` が `SET timestamp` を実行した後、間違ったデフォルトタイムスタンプを使用する可能性があるバグを修正 [#31968](https://github.com/pingcap/tidb/issues/31968)
  - MySQL 5.5 または 5.6 クライアントから TiDB パスワードなしアカウントに接続できないバグを修正 [#32334](https://github.com/pingcap/tidb/issues/32334)
  - トランザクション [#29851](https://github.com/pingcap/tidb/issues/29851) でダイナミックモードでパーティションテーブルを読み取るときに間違った結果を修正
  - TiDB が TiFlash に重複したタスクをディスパッチする可能性があるバグを修正 PLACEHOLDER-1-PLACEHOLDER-E
  - `timdiff` 関数の入力にミリ秒が含まれている場合に返される間違った結果を修正 [#31680](https://github.com/pingcap/tidb/issues/31680)
  - 明示的にパーティションを読み込み、indexJoin プランを使用するときに間違った結果を修正 [#32007](https://github.com/pingcap/tidb/issues/32007)
  - カラムタイプを同時に変更するとカラム名の変更に失敗するバグを修正 [#31075](https://github.com/pingcap/tidb/issues/31075)
  - TiFlash プランの純コストを計算する式が TikV プランと一致しないバグを修正 [#30103](https://github.com/pingcap/tidb/issues/30103)
  - `KILL TIDB` がアイドル状態の接続ですぐに有効にならないバグを修正 [#24031](https://github.com/pingcap/tidb/issues/24031)
  - 生成されたカラム [#33038](https://github.com/pingcap/tidb/issues/33038) を含むテーブルをクエリするときに発生する可能性があった間違った結果を修正
  - `left join` [#31321](https://github.com/pingcap/tidb/issues/31321) を使用して複数のテーブルのデータを削除した場合の誤った結果を修正
  - `SUBTIME` 関数がオーバーフローの場合に間違った結果を返すバグを修正 [#31868](https://github.com/pingcap/tidb/issues/31868)
  - 集計クエリに `having` 条件 [#33166](https://github.com/pingcap/tidb/issues/33166) 条件が含まれている場合、`selection` 演算子を押し下げることができない不具合を修正
  - クエリがエラーを報告すると CTE がブロックされることがあるバグを修正 [#31302](https://github.com/pingcap/tidb/issues/31302)
  - 非厳密モードでテーブルを作成するときに varbinary または varchar カラムの長さが長すぎると、エラーが発生する可能性があるバグを修正 [#30328](https://github.com/pingcap/tidb/issues/30328)
  - フォロワーが指定されていないときに `information_schema.placement_policies` のフォロワー数が間違っていたのを修正 [#31702](https://github.com/pingcap/tidb/issues/31702)
  - インデックスの作成時に TiDB でカラムプレフィックス長を 0 に指定できる問題を修正 [#31972](https://github.com/pingcap/tidb/issues/31972)
  - TiDB がスペースで終わるパーティション名を許可する問題を修正 [#31535](https://github.com/pingcap/tidb/issues/31535)
  - `RENAME TABLE` ステートメントのエラーメッセージを修正してください [#29893](https://github.com/pingcap/tidb/issues/29893)

- TikV

  - ピアのステータスが `Applying` [#11746](https://github.com/tikv/tikv/issues/11746) のときにスナップショットファイルを削除することによって引き起こされるパニックの問題を修正しました
  - フロー制御が有効で、`level0_slowdown_trigger` が明示的に設定されている場合の QPS ドロップの問題を修正しました [#11424](https://github.com/tikv/tikv/issues/11424)
  - ピアを破棄するとレイテンシーが長くなることがある問題を修正 [#10210](https://github.com/tikv/tikv/issues/10210)
  - GC ワーカーがビジー状態の場合、TikV が一定範囲のデータを削除できない（内部コマンド `unsafe_destroy_range` が実行される）バグを修正 [#11903](https://github.com/tikv/tikv/issues/11903)
  - `StoreMeta` のデータがいくつかのコーナーケースで誤って削除されたときに TikV がパニックになるバグを修正 [#11852](https://github.com/tikv/tikv/issues/11852)
  - ARM プラットフォームでプロファイリングを実行すると TikV がパニックになるバグを修正 [#10658](https://github.com/tikv/tikv/issues/10658)
  - TikV が 2 年以上稼働しているとパニックになるバグを修正 [#11940](https://github.com/tikv/tikv/issues/11940)
  - SSE 命令セット [#12034](https://github.com/tikv/tikv/issues/12034) が欠落しているために発生する ARM64 アーキテクチャでのコンパイルの問題を修正しました。
  - 初期化されていないレプリカを削除すると、古いレプリカが再作成される可能性があるという問題を修正 [#10533](https://github.com/tikv/tikv/issues/10533)
  - 古いメッセージが原因で TikV がパニックになるバグを修正 [#12023](https://github.com/tikv/tikv/issues/12023)
  - TSSet 変換で未定義の動作 (UB) が発生する可能性がある問題を修正 [#12070](https://github.com/tikv/tikv/issues/12070)
  - レプリカの読み取りが線形化可能性に違反する可能性があるバグを修正 [#12109](https://github.com/tikv/tikv/issues/12109)
  - TikV が Ubuntu 18.04 でプロファイリングを実行するときに発生する潜在的なパニックの問題を修正しました [#9765](https://github.com/tikv/tikv/issues/9765)
  - tikv-ctl が間違った文字列一致のために間違った結果を返す問題を修正しました [#12329](https://github.com/tikv/tikv/issues/12329)
  - メモリメトリクスのオーバーフローによる断続的なパケット損失とメモリ不足 (OOM) の問題を修正しました [#12160](https://github.com/tikv/tikv/issues/12160)
  - TikV を終了するときに TikV パニックを誤って報告する潜在的な問題を修正 [#12231](https://github.com/tikv/tikv/issues/12231)

- PD

  - PD が共同コンセンサスの無意味なステップでオペレータを生成する問題を修正しました [#4362](https://github.com/tikv/pd/issues/4362)
  - PD クライアントを閉じるときに TSO 取り消しプロセスが停止することがあるバグを修正 [#4549](https://github.com/tikv/pd/issues/4549)
  - リージョンスキャッタラースケジューリングがいくつかのピアを失った問題を修正しました [#4565](https://github.com/tikv/pd/issues/4565)
  - `dr-autosync` の `Duration` フィールドを動的に設定できない問題を修正しました [#4651](https://github.com/tikv/pd/issues/4651)

- tiFlash

  - メモリ制限が有効になっているときに発生する TiFlash パニックの問題を修正 [#3902](https://github.com/pingcap/tiflash/issues/3902)
  - 期限切れのデータがゆっくりリサイクルされる問題を修正しました [#4146](https://github.com/pingcap/tiflash/issues/4146)
  - `Snapshot` が複数の DDL 操作と同時に適用される場合の TiFlash パニックの潜在的な問題を修正 [#4072](https://github.com/pingcap/tiflash/issues/4072)
  - 重い読み取りワークロードで列を追加した後の潜在的なクエリエラーを修正 [#3967](https://github.com/pingcap/tiflash/issues/3967)
  - 負の引数を持つ `SQRT` 関数が `Null` [#3598](https://github.com/pingcap/tiflash/issues/3598) の代わりに `NaN` を返す問題を修正しました
  - `INT` を `DECIMAL` にキャストすると、オーバーフローが発生する可能性がある問題を修正しました [#3920](https://github.com/pingcap/tiflash/issues/3920)
  - `IN` の結果が複数値の式で正しくない問題を修正しました [#4016](https://github.com/pingcap/tiflash/issues/4016)
  - 日付形式で `'\n'` が無効な区切り文字として識別される問題を修正しました [#4036](https://github.com/pingcap/tiflash/issues/4036)
  - 同時実行性が高いシナリオでは、学習者の読み取りプロセスに時間がかかりすぎるという問題を修正 [#3555](https://github.com/pingcap/tiflash/issues/3555)
  - `DATETIME` を `DECIMAL` [#4151](https://github.com/pingcap/tiflash/issues/4151) にキャストするときに発生する間違った結果を修正しました
  - クエリがキャンセルされたときに発生するメモリリークの問題を修正 [#4098](https://github.com/pingcap/tiflash/issues/4098)
  - エラスティックスレッドプールを有効にするとメモリリークが発生する可能性があるバグを修正 [#4098](https://github.com/pingcap/tiflash/issues/4098)
  - ローカルトンネルが有効になっていると、MPP クエリをキャンセルするとタスクが永久にハングすることがあるバグを修正 [#4229](https://github.com/pingcap/tiflash/issues/4229)
  - HashJoin ビルドサイドの失敗により MPP クエリが永久にハングするバグを修正 [#4195](https://github.com/pingcap/tiflash/issues/4195)
  - MPP タスクがスレッドを永久にリークする可能性があるバグを修正 [#4238](https://github.com/pingcap/tiflash/issues/4238)

- ツール

  - バックアップと復元 (BR)

    - 復元操作で回復不能なエラーが発生すると BR がスタックするバグを修正 [#33200](https://github.com/pingcap/tidb/issues/33200)
    - バックアップの再試行中に暗号化情報が失われたときに復元操作が失敗するバグを修正 [#32423](https://github.com/pingcap/tidb/issues/32423)

  - ticDC

    - `batch-replace-enable` が無効になっている場合、MySQL シンクが重複した `replace` SQL ステートメントを生成するバグを修正 [#4501](https://github.com/pingcap/tiflow/issues/4501)
    - PD リーダーが殺されると tiCDC ノードが異常終了するバグを修正 [#4248](https://github.com/pingcap/tiflow/issues/4248)
    - 一部の MySQL バージョンでエラー `Unknown system variable 'transaction_isolation'` を修正 [#4504](https://github.com/pingcap/tiflow/issues/4504)
    - `Canal-JSON` が `string` [#4635](https://github.com/pingcap/tiflow/issues/4635) を誤って処理した場合に発生する可能性のある ticDC パニック問題を修正しました。
    - 場合によってはシーケンスが誤って複製されるバグを修正 [#4563](https://github.com/pingcap/tiflow/issues/4552)
    - `Canal-JSON` は nil [#4736](https://github.com/pingcap/tiflow/issues/4736) をサポートしていないために発生する可能性のある ticDC パニックの問題を修正しました
    - タイプ `Enum/Set` と `TinyText/MediumText/Text/LongText` の avro コーデックの間違ったデータマッピングを修正しました [#4454](https://github.com/pingcap/tiflow/issues/4454)
    - アブロが `NOT NULL` 列をヌル可能なフィールドに変換するバグを修正 [#4818](https://github.com/pingcap/tiflow/issues/4818)
    - ticDC が終了できない問題を修正 [#4699](https://github.com/pingcap/tiflow/issues/4699)

  - TiDB データ移行 (DM)

    - ステータス [#4281](https://github.com/pingcap/tiflow/issues/4281) を照会するときにのみ同期メトリクスが更新される問題を修正しました
    - セーフモードでの更新ステートメントの実行エラーが DM-Worker パニックを引き起こす可能性がある問題を修正 [#4317](https://github.com/pingcap/tiflow/issues/4317)
    - 長い varchars がエラーを報告するバグを修正 `Column length too big` [#4637](https://github.com/pingcap/tiflow/issues/4637)
    - 複数の DM ワーカーが同じアップストリームからデータを書き込むことによって引き起こされる競合の問題を修正しました [#3737](https://github.com/pingcap/tiflow/issues/3737)
    - ログに何百もの「チェックポイントに変更がない、同期フラッシュチェックポイントをスキップする」と印刷され、レプリケーションが非常に遅くなる問題を修正 [#4619](https://github.com/pingcap/tiflow/issues/4619)
    - 悲観的モード [#5002](https://github.com/pingcap/tiflow/issues/5002) でシャードをマージし、アップストリームから増分データをレプリケートするときの DML 損失の問題を修正しました

  - TiDB ライトニング

    - 一部のインポートタスクにソースファイルが含まれていない場合、TiDB Lightning がメタデータスキーマを削除しないことがある不具合を修正 [#28144](https://github.com/pingcap/tidb/issues/28144)
    - ソースファイルとターゲットクラスタのテーブル名が異なる場合に発生するパニックを修正 [#31771](https://github.com/pingcap/tidb/issues/31771)
    - チェックサムエラー「GC ライフタイムがトランザクション期間より短い」を修正しました [#32733](https://github.com/pingcap/tidb/issues/32733)
    - 空のテーブルのチェックに失敗すると TiDB Lightning がスタックする問題を修正しました [#31797](https://github.com/pingcap/tidb/issues/31797)

  - 餃子

    - `dumpling --sql $query` [#30532](https://github.com/pingcap/tidb/issues/30532) を実行すると、表示されている進行状況が正確でない問題を修正しました
    - Amazon S3 が圧縮データのサイズを正しく計算できない問題を修正しました [#30534](https://github.com/pingcap/tidb/issues/30534)

  - TiDB ブログ

    - 大規模なアップストリームの書き込みトランザクションが Kafka [#1136](https://github.com/pingcap/tidb-binlog/issues/1136) にレプリケートされると、TiDB Binlog がスキップされることがある問題を修正
