---
title: TiDB 6.0.0 Release Notes
---

# TiDB6.0.0 リリースノート {#tidb-6-0-0-release-notes}

発売日：2022 年 4 月 7 日

TiDB バージョン：6.0.0-DMR

6.0.0-DMR では、主な新機能または改善点は次のとおりです。

- SQL で配置ルールをサポートして、データ配置のより柔軟な管理を提供します。
- カーネルレベルでデータとインデックスの間に整合性チェックを追加します。これにより、システムの安定性と堅牢性が向上し、リソースのオーバーヘッドが非常に低くなります。
- 非専門家向けのセルフサービスのデータベースパフォーマンス監視および診断機能である TopSQL を提供します。
- クラスターのパフォーマンスデータを常に収集する継続的なプロファイリングをサポートし、技術専門家の MTTR を削減します。
- ホットスポットの小さなテーブルをメモリにキャッシュします。これにより、アクセスパフォーマンスが大幅に向上し、スループットが向上し、アクセスの待ち時間が短縮されます。
- インメモリの悲観的ロックを最適化します。ペシミスティックロックによって引き起こされるパフォーマンスのボトルネックの下で、ペシミスティックロックのメモリ最適化により、レイテンシを 10％削減し、QPS を 10％向上させることができます。
- プリペアドステートメントを拡張して実行プランを共有します。これにより、CPU リソースの消費が減り、SQL の実行効率が向上します。
- より多くの式のプッシュダウンとエラスティックスレッドプールの一般的な可用性（GA）をサポートすることにより、MPP エンジンのコンピューティングパフォーマンスを向上させます。
- DM WebUI を追加して、多数の移行タスクの管理を容易にします。
- 大規模なクラスターでデータを複製する際の TiCDC の安定性と効率を向上させます。 TiCDC は、100,000 テーブルの同時複製をサポートするようになりました。
- TiKV ノードを再起動した後、リーダーのバランシングを加速します。これにより、再起動後のビジネス回復の速度が向上します。
- 統計の自動更新のキャンセルをサポートします。これにより、リソースの競合が減少し、SQL パフォーマンスへの影響が制限されます。
- TiDB クラスターの自動診断サービスである PingCAP クリニックを提供します（テクニカルプレビューバージョン）。
- エンタープライズレベルのデータベース管理プラットフォームである TiDBEnterpriseManager を提供します。

また、TiDB の HTAP ソリューションのコアコンポーネントとして、TiFlashTM はこのリリースで正式にオープンソースになっています。詳細については、[TiFlash リポジトリ](https://github.com/pingcap/tiflash)を参照してください。

## リリース戦略の変更 {#release-strategy-changes}

TiDB v6.0.0 以降、TiDB は次の 2 種類のリリースを提供します。

- 長期サポートリリース

  ロングタームサポート（LTS）リリースは、約 6 か月ごとにリリースされます。 LTS リリースでは、新機能と改善点が導入され、リリースライフサイクル内でパッチリリースが受け入れられます。たとえば、v6.1.0 は LTS リリースになります。

- 開発マイルストーンリリース

  開発マイルストーンリリース（DMR）は、約 2 か月ごとにリリースされます。 DMR は新機能と改善点を導入しますが、パッチリリースは受け入れません。オンプレミスユーザーが実稼働環境で DMR を使用することはお勧めしません。たとえば、v6.0.0-DMR は DMR です。

TiDB v6.0.0 は DMR であり、そのバージョンは 6.0.0-DMR です。

## 新機能 {#new-features}

### SQL {#sql}

- データの SQL ベースの配置ルール

  TiDB は、優れたスケーラビリティを備えた分散データベースです。通常、データは複数のサーバーまたは複数のデータセンターに展開されます。したがって、データスケジューリング管理は、TiDB の最も重要な基本機能の 1 つです。ほとんどの場合、ユーザーはデータのスケジュールと管理の方法を気にする必要はありません。ただし、アプリケーションの複雑さが増すにつれて、分離とアクセス遅延によって引き起こされる展開の変更は、TiDB にとって新たな課題になっています。 v6.0.0 以降、TiDB は SQL インターフェイスに基づくデータのスケジューリングおよび管理機能を公式に提供します。レプリカ数、役割タイプ、データの配置場所などのディメンションでの柔軟なスケジューリングと管理をサポートします。 TiDB は、マルチサービス共有クラスターおよびクロス AZ 展開でのデータ配置のより柔軟な管理もサポートします。

  [ユーザードキュメント](/placement-rules-in-sql.md)

- データベースによる TiFlash レプリカの構築をサポートします。データベース内のすべてのテーブルに TiFlash レプリカを追加するには、単一の SQL ステートメントを使用するだけで済み、運用と保守のコストを大幅に節約できます。

  [ユーザードキュメント](/tiflash/use-tiflash.md#create-tiflash-replicas-for-databases)

### 取引 {#transaction}

- カーネルレベルでのデータインデックスの整合性のチェックを追加します

  トランザクションの実行時にデータインデックスの整合性のチェックを追加します。これにより、システムの安定性と堅牢性が向上し、リソースのオーバーヘッドが非常に低くなります。 `tidb_enable_mutation_checker`変数と`tidb_txn_assertion_level`変数を使用して、チェック動作を制御できます。デフォルト構成では、ほとんどのシナリオで QPS ドロップは 2％以内に制御されます。整合性チェックのエラーの説明については、[ユーザードキュメント](/troubleshoot-data-inconsistency-errors.md)を参照してください。

### 可観測性 {#observability}

- トップ SQL：専門家以外のパフォーマンス診断

  トップ SQL は、DBA およびアプリ開発者向けの TiDB ダッシュボードのセルフサービスデータベースパフォーマンス監視および診断機能であり、現在 TiDBv6.0 で一般的に利用可能です。

  専門家向けの既存の診断機能とは異なり、Top SQL は非専門家向けに設計されています。相関関係を見つけたり、Raft スナップショット、RocksDB、MVCC、TSO などの TiDB 内部メカニズムを理解したりするために、何千もの監視チャートをトラバースする必要はありません。トップ SQL を使用してデータベースの負荷をすばやく分析し、アプリのパフォーマンスを向上させるには、基本的なデータベースの知識（インデックス、ロックの競合、実行計画など）のみが必要です。

  トップ SQL はデフォルトでは有効になっていません。有効にすると、TopSQL は各 TiKV または TiDB ノードのリアルタイム CPU 負荷を提供します。したがって、CPU 負荷の高い SQL ステートメントを一目見ただけで、データベースのホットスポットや突然の負荷の増加などの問題をすばやく分析できます。たとえば、Top SQL を使用して、単一の TiKV ノードの 90％の CPU を消費する異常なクエリを特定して診断できます。

  [ユーザードキュメント](/dashboard/top-sql.md)

- 継続的なプロファイリングをサポートする

  TiDB ダッシュボードには、継続的プロファイリング機能が導入されています。この機能は、TiDBv6.0 で一般的に利用可能になりました。継続的なプロファイリングはデフォルトでは有効になっていません。有効にすると、個々の TiDB、TiKV、および PD インスタンスのパフォーマンスデータが常に収集され、オーバーヘッドはごくわずかです。履歴パフォーマンスデータを使用すると、技術専門家は、問題の再現が困難な場合でも、メモリ消費量が多いなどの問題の根本原因をさかのぼって特定できます。このようにして、平均修復時間（MTTR）を短縮できます。

  [ユーザードキュメント](/dashboard/continuous-profiling.md)

### パフォーマンス {#performance}

- ホットスポットの小さなテーブルをキャッシュする

  ホットスポットの小さなテーブルにアクセスするシナリオのユーザーアプリケーションの場合、TiDB はホットスポットテーブルをメモリに明示的にキャッシュすることをサポートします。これにより、アクセスパフォーマンスが大幅に向上し、スループットが向上し、アクセス遅延が減少します。このソリューションは、サードパーティのキャッシュミドルウェアの導入を効果的に回避し、アーキテクチャの複雑さを軽減し、運用と保守のコストを削減できます。このソリューションは、構成テーブルや為替レートテーブルなど、小さなテーブルが頻繁にアクセスされるが更新されることはめったにないシナリオに適しています。

  [ユーザードキュメント](/cached-tables.md)、[＃25293](https://github.com/pingcap/tidb/issues/25293)

- インメモリ悲観的ロック

  TiDB v6.0.0 以降、メモリ内のペシミスティックロックはデフォルトで有効になっています。この機能を有効にすると、悲観的なトランザクションロックがメモリで管理されます。これにより、ペシミスティックロックの永続化とロック情報の Raft レプリケーションが回避され、ペシミスティックトランザクションロックの管理のオーバーヘッドが大幅に削減されます。ペシミスティックロックによって引き起こされるパフォーマンスのボトルネックの下で、ペシミスティックロックのメモリ最適化により、レイテンシを 10％削減し、QPS を 10％向上させることができます。

  [ユーザードキュメント](/pessimistic-transaction.md#in-memory-pessimistic-lock)、[＃11452](https://github.com/tikv/tikv/issues/11452)

- 読み取りコミット分離レベルで TSO を取得するための最適化

  クエリの待ち時間を短縮するために、読み取りと書き込みの競合がまれな場合、TiDB は`tidb_rc_read_check_ts`システム変数を[コミットされた分離レベルを読み取る](/transaction-isolation-levels.md#read-committed-isolation-level)に追加して不要性を減らします TSO。この変数はデフォルトで無効になっています。変数が有効になっている場合、この最適化により TSO の重複が回避され、読み取りと書き込みの競合がないシナリオでの遅延が減少します。ただし、読み取りと書き込みの競合が頻繁に発生するシナリオでは、この変数を有効にするとパフォーマンスが低下する可能性があります。

  [ユーザードキュメント](/transaction-isolation-levels.md#read-committed-isolation-level)、[＃33159](https://github.com/pingcap/tidb/issues/33159)

- 実行計画を共有するために準備されたステートメントを強化する

  SQL 実行プランを再利用すると、SQL ステートメントの解析時間を効果的に短縮し、CPU リソースの消費を減らし、SQL の実行効率を向上させることができます。 SQL チューニングの重要な方法の 1 つは、SQL 実行プランを効果的に再利用することです。 TiDB は、プリペアドステートメントとの実行計画の共有をサポートしています。ただし、プリペアドステートメントが閉じられると、TiDB は対応するプランキャッシュを自動的にクリアします。その後、TiDB は繰り返される SQL ステートメントを不必要に解析し、実行効率に影響を与える可能性があります。 v6.0.0 以降、TiDB は、`tidb_ignore_prepared_cache_close_stmt`パラメーター（デフォルトでは無効）を使用して、`COM_STMT_CLOSE`コマンドを無視するかどうかの制御をサポートしています。パラメータが有効になっている場合、TiDB はプリペアドステートメントを閉じるコマンドを無視し、実行プランをキャッシュに保持して、実行プランの再利用率を向上させます。

  [ユーザードキュメント](/sql-prepared-plan-cache.md#ignore-the-com_stmt_close-command-and-the-deallocate-prepare-statement)、[＃31056](https://github.com/pingcap/tidb/issues/31056)

- クエリプッシュダウンを改善する

  コンピューティングをストレージから分離するネイティブアーキテクチャにより、TiDB は演算子をプッシュダウンすることで無効なデータを除外することをサポートします。これにより、TiDB と TiKV 間のデータ転送が大幅に削減され、クエリの効率が向上します。 v6.0.0 では、TiDB はより多くの式と`BIT`データ型を TiKV にプッシュダウンすることをサポートし、式とデータ型を計算する際のクエリ効率を向上させます。

  [ユーザードキュメント](/functions-and-operators/expressions-pushed-down.md#add-to-the-blocklist)、[＃30738](https://github.com/pingcap/tidb/issues/30738)

- TiFlash MPP エンジンのパーティションテーブルの動的プルーニングモードをサポート（実験的）

  このモードでは、TiDB は TiFlash の MPP エンジンを使用してパーティションテーブルのデータを読み取り、計算できます。これにより、パーティションテーブルのクエリパフォーマンスが大幅に向上します。

  [ユーザードキュメント](/tiflash/use-tiflash.md#access-partitioned-tables-in-the-mpp-mode)

- MPP エンジンのコンピューティングパフォーマンスを向上させる

  - MPP エンジンへのより多くの機能とオペレーターのプッシュダウンをサポート

    - 論理関数：`IS`、`IS NOT`
    - 文字列関数：`REGEXP()`、`NOT REGEXP()`
    - 数学関数：`GREATEST(int/real)`、`LEAST(int/real)`
    - 日付関数：`DAYOFNAME()`、`DAYOFMONTH()`、`DAYOFWEEK()`、PLACEHOLDER -7-PLACEHOLDER、`LAST_DAY()`、`MONTHNAME()`
    - オペレーター：アンチレフトアウターセミジョイン、レフトアウターセミジョイン

    [ユーザードキュメント](/tiflash/use-tiflash.md#supported-push-down-calculations)

  - エラスティックスレッドプール（デフォルトで有効）は GA になります。この機能は、CPU 使用率を向上させることを目的としています。

    [ユーザードキュメント](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file)

### 安定 {#stability}

- 実行計画のベースラインキャプチャを強化する

  テーブル名、頻度、ユーザー名などのディメンションを持つブロックリストを追加することにより、実行計画のベースラインキャプチャの使いやすさを向上させます。バインディングをキャッシュするためのメモリ管理を最適化するための新しいアルゴリズムを導入します。ベースラインキャプチャを有効にすると、システムはほとんどの OLTP クエリのバインディングを自動的に作成します。バインドされたステートメントの実行計画が修正され、実行計画の変更によるパフォーマンスの問題が回避されます。ベースラインキャプチャは、メジャーバージョンのアップグレードやクラスターの移行などのシナリオに適用でき、実行計画のリグレッションによって引き起こされるパフォーマンスの問題を軽減するのに役立ちます。

  [ユーザードキュメント](/sql-plan-management.md#baseline-capturing)、[＃32466](https://github.com/pingcap/tidb/issues/32466)

- TiKV クォータリミッターをサポート（実験的）

  TiKV を使用してデプロイされたマシンのリソースが限られており、フォアグラウンドに大量のリクエストが発生している場合、バックグラウンド CPU リソースがフォアグラウンドによって占有され、TiKV のパフォーマンスが不安定になります。 TiDB v6.0.0 では、クォータ関連の構成アイテムを使用して、CPU や読み取り/書き込み帯域幅など、フォアグラウンドで使用されるリソースを制限できます。これにより、長期間の重いワークロードでのクラスターの安定性が大幅に向上します。

  [ユーザードキュメント](/tikv-configuration-file.md#quota)、[＃12131](https://github.com/tikv/tikv/issues/12131)

- TiFlash で zstd 圧縮アルゴリズムをサポートする

  TiFlash には、`profiles.default.dt_compression_method`と`profiles.default.dt_compression_level`の 2 つのパラメーターが導入されており、ユーザーはパフォーマンスと容量のバランスに基づいて最適な圧縮アルゴリズムを選択できます。

  [ユーザードキュメント](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file)

- デフォルトですべての I/O チェック（チェックサム）を有効にする

  この機能は、実験として v5.4.0 で導入されました。ユーザーのビジネスに明らかな影響を与えることなく、データの正確性とセキュリティを強化します。

  警告：新しいバージョンのデータ形式は、v5.4.0 より前のバージョンにダウングレードすることはできません。このようなダウングレード中は、TiFlash レプリカを削除し、ダウングレード後にデータを複製する必要があります。または、[dttool の移行](/tiflash/tiflash-command-line-flags.md#dttool-migrate)を参照してダウングレードを実行することもできます。

  [ユーザードキュメント](/tiflash/use-tiflash.md#use-data-validation)

- スレッドの使用率を向上させる

  TiFlash は、非同期 gRPC および Min-TSO スケジューリングメカニズムを導入しています。このようなメカニズムは、スレッドのより効率的な使用を保証し、過剰なスレッドによって引き起こされるシステムクラッシュを回避します。

  [ユーザードキュメント](/tiflash/monitor-tiflash.md#coprocessor)

### データ移行 {#data-migration}

#### TiDB データ移行（DM） {#tidb-data-migration-dm}

- WebUI の追加（実験的）

  WebUI を使用すると、多数の移行タスクを簡単に管理できます。 WebUI では、次のことができます。

  - ダッシュボードで移行タスクを表示する
  - 移行タスクを管理する
  - アップストリーム設定を構成する
  - レプリケーションステータスのクエリ
  - マスターとワーカーの情報を表示する

  WebUI はまだ実験段階であり、開発中です。したがって、試用のみをお勧めします。既知の問題は、WebUI と dmctl を使用して同じタスクを操作すると問題が発生する可能性があることです。この問題は、以降のバージョンで解決される予定です。

  [ユーザードキュメント](/dm/dm-webui-guide.md)

- エラー処理メカニズムを追加する

  移行タスクを中断する問題に対処するために、より多くのコマンドが導入されています。例えば：

  - スキーマエラーが発生した場合は、編集する代わりに、`binlog-schema update`コマンドの`--from-source/--from-target`パラメータを使用してスキーマファイルを更新できます。スキーマファイルを個別に。
  - binlog の位置を指定して、DDL ステートメントを挿入、置換、スキップ、または元に戻すことができます。

  [ユーザードキュメント](/dm/dm-manage-schema.md)

- AmazonS3 への完全なデータストレージをサポートする

  DM がすべてまたは完全なデータ移行タスクを実行する場合、アップストリームからの完全なデータを格納するために十分なハードディスク容量が必要です。 EBS と比較して、AmazonS3 は低コストでほぼ無限のストレージを備えています。現在、DM は AmazonS3 をダンプディレクトリとして設定することをサポートしています。つまり、すべてまたは完全なデータ移行タスクを実行するときに、S3 を使用して完全なデータを保存できます。

  [ユーザードキュメント](/dm/task-configuration-file-full.md#task-configuration-file-template-advanced)

- 指定された時間からの移行タスクの開始をサポート

  新しいパラメータ`--start-time`が移行タスクに追加されました。時刻は「2021-10-2100：01：00」または「2021-10-21T00：01：00」の形式で定義できます。

  この機能は、シャード mysql インスタンスからインクリメンタルデータを移行およびマージするシナリオで特に役立ちます。具体的には、増分移行タスクで各ソースに binlog 開始点を設定する必要はありません。代わりに、`safe-mode`の`--start-time`パラメーターを使用して、増分移行タスクをすばやく作成できます。

  [ユーザードキュメント](/dm/dm-create-task.md#flags-description)

#### TiDB Lightning {#tidb-lightning}

- 許容可能なエラーの最大数の構成をサポート

  構成アイテム`lightning.max-error`を追加しました。デフォルト値は 0 です。値が 0 より大きい場合、最大エラー機能が有効になります。エンコード中に行でエラーが発生した場合、この行を含むレコードがターゲット TiDB の`lightning_task_info.type_error_v1`に追加され、この行は無視されます。エラーのある行がしきい値を超えると、TiDBLightning はすぐに終了します。

  `lightning.max-error`構成と一致して、`lightning.task-info-schema-name`構成アイテムは、データ保存エラーを報告するデータベースの名前を記録します。

  この機能は、すべてのタイプのエラーを網羅しているわけではありません。たとえば、構文エラーは適用されません。

  [ユーザードキュメント](/tidb-lightning/tidb-lightning-error-resolution.md#type-error)

### TiDB データ共有サブスクリプション {#tidb-data-share-subscription}

- 100,000 テーブルの同時複製をサポート

  TiCDC は、データ処理フローを最適化することにより、各テーブルの増分データを処理するためのリソース消費を削減します。これにより、大規模なクラスターでデータを複製する際の複製の安定性と効率が大幅に向上します。内部テストの結果は、TiCDC が 100,000 テーブルの同時複製を安定してサポートできることを示しています。

### 展開とメンテナンス {#deployment-and-maintenance}

- デフォルトで新しい照合ルールを有効にする

  v4.0 以降、TiDB は、大文字と小文字を区別しない、アクセントを区別しない、およびパディングルールで MySQL と同じように動作する新しい照合ルールをサポートしています。新しい照合ルールは、デフォルトで無効になっている`new_collations_enabled_on_first_bootstrap`パラメーターによって制御されます。 v6.0 以降、TiDB はデフォルトで新しい照合ルールを有効にします。この構成は、TiDB クラスターの初期化時にのみ有効になることに注意してください。

  [ユーザードキュメント](/tidb-configuration-file.md#new_collations_enabled_on_first_bootstrap)

- TiKV ノードを再起動した後、リーダーのバランシングを加速します

  TiKV ノードの再起動後、負荷分散のために、不均一に分散したリーダーを再分散する必要があります。大規模なクラスターでは、リーダーのバランシング時間はリージョンの数と正の相関があります。たとえば、100K リージョンのリーダーバランシングには 20〜30 分かかる場合があります。これは、不均一な負荷によるパフォーマンスの問題や安定性のリスクが発生しやすい傾向があります。 TiDB v6.0.0 は、バランシングの同時実行性を制御するパラメータを提供し、デフォルト値を元の値の 4 倍に拡大します。これにより、リーダーのリバランシング時間が大幅に短縮され、TiKV ノードの再起動後のビジネス回復が加速されます。

  [ユーザードキュメント](/pd-control.md#scheduler-config-balance-leader-scheduler)、[＃4610](https://github.com/tikv/pd/issues/4610)

- 統計の自動更新のキャンセルをサポート

  統計は、SQL のパフォーマンスに影響を与える最も重要な基本データの 1 つです。統計の完全性と適時性を確保するために、TiDB はバックグラウンドでオブジェクト統計を定期的に自動的に更新します。ただし、統計の自動更新によりリソースの競合が発生し、SQL のパフォーマンスに影響を与える可能性があります。この問題に対処するために、v6.0 以降の統計の自動更新を手動でキャンセルできます。

  [ユーザードキュメント](/statistics.md#automatic-update)

- PingCAP クリニック診断サービス（テクニカルプレビュー版）

  PingCAP クリニックは TiDB クラスターの診断サービスです。このサービスは、クラスターの問題をリモートでトラブルシューティングするのに役立ち、クラスターのステータスをローカルですばやく確認できます。 PingCAP Clinic を使用すると、TiDB クラスターのライフサイクル全体での安定した動作を保証し、潜在的な問題を予測し、問題の可能性を減らし、クラスターの問題を迅速にトラブルシューティングできます。

  クラスターの問題をトラブルシューティングするためのリモートアシスタンスについて PingCAP テクニカルサポートに連絡する場合、PingCAP クリニックサービスを使用して診断データを収集およびアップロードできるため、トラブルシューティングの効率が向上します。

  [ユーザードキュメント](/clinic/clinic-introduction.md)

- エンタープライズレベルのデータベース管理プラットフォーム、TiDB Enterprise Manager

  TiDB Enterprise Manager（TiEM）は、TiDB データベースに基づくエンタープライズレベルのデータベース管理プラットフォームであり、ユーザーがオンプレミスまたはパブリッククラウド環境で TiDB クラスターを管理できるようにすることを目的としています。

  TiEM は、TiDB クラスターの完全なライフサイクルビジュアル管理を提供するだけでなく、パラメーター管理、バージョンアップグレード、クラスタークローン、アクティブスタンバイクラスタースイッチング、データのインポートとエクスポート、データレプリケーション、およびデータのバックアップと復元のサービスなどのワンストップサービスも提供します。 TiEM は、TiDB 上の DevOps の効率を改善し、企業の DevOps コストを削減できます。

  現在、TiEM は[TiDB エンタープライズ](https://en.pingcap.com/tidb-enterprise/)エディションでのみ提供されています。 TiEM を入手するには、[TiDB エンタープライズ](https://en.pingcap.com/tidb-enterprise/)ページからお問い合わせください。

- 監視コンポーネントの構成のカスタマイズをサポート

  TiUP を使用して TiDB クラスターをデプロイすると、TiUP は Prometheus、Grafana、Alertmanager などのモニタリングコンポーネントを自動的にデプロイし、スケールアウト後に新しいノードをモニタリングスコープに自動的に追加します。 `topology.yaml`ファイルに構成アイテムを追加することにより、監視コンポーネントの構成をカスタマイズできます。

  [ユーザードキュメント](/tiup/customized-montior-in-tiup-environment.md)

## 互換性の変更 {#compatibility-changes}

> <strong>ノート：</strong>
>
> 以前の TiDB バージョンから v6.0.0 にアップグレードするときに、すべての中間バージョンの互換性の変更に関する注意事項を知りたい場合は、対応するバージョンの[リリースノート](/releases/release-notes.md)を確認できます。

### システム変数 {#system-variables}

| 変数名                                                                                                                                                                   | タイプを変更する     | 説明                                                                                                                                                                                                                                                                                                 |
| :----------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `placement_checks`                                                                                                                                                       | 削除                 | DDL ステートメントが[SQL の配置ルール](/placement-rules-in-sql.md)で指定された配置ルールを検証するかどうかを制御します。 `tidb_placement_mode`に置き換えられました。                                                                                                                                 |
| `tidb_enable_alter_placement`                                                                                                                                            | 削除                 | [SQL の配置ルール](/placement-rules-in-sql.md)を有効にするかどうかを制御します。                                                                                                                                                                                                                     |
| `tidb_mem_quota_hashjoin` `tidb_mem_quota_indexlookupjoin` `tidb_mem_quota_indexlookupreader` PLACEHOLDER-7-PLACEHOLDER-E }} `tidb_mem_quota_sort` `tidb_mem_quota_topn` | 削除                 | v5.0 以降、これらの変数は`tidb_mem_quota_query`に置き換えられ、[システム変数](/system-variables.md)ドキュメントから削除されました。互換性を確保するために、これらの変数はソースコードに保持されていました。 TiDB 6.0.0 以降、これらの変数もコードから削除されています。                              |
| [<code>tidb_enable_mutation_checker</code>](/system-variables.md#tidb_enable_mutation_checker-new-in-v600)                                                               | 新しく追加されました | ミューテーションチェッカーを有効にするかどうかを制御します。デフォルト値は`ON`です。                                                                                                                                                                                                                 |
| [<code>tidb_ignore_prepared_cache_close_stmt</code>](/system-variables.md#tidb_ignore_prepared_cache_close_stmt-new-in-v600)                                             | 新しく追加されました | プリペアドステートメントを閉じるコマンドを無視するかどうかを制御します。デフォルト値は`OFF`です。                                                                                                                                                                                                    |
| [<code>tidb_mem_quota_binding_cache</code>](/system-variables.md#tidb_mem_quota_binding_cache-new-in-v600)                                                               | 新しく追加されました | `binding`を保持するキャッシュのメモリ使用量のしきい値を設定します。デフォルト値は`67108864`（64 MiB）です。                                                                                                                                                                                          |
| [<code>tidb_placement_mode</code>](/system-variables.md#tidb_placement_mode-new-in-v600)                                                                                 | 新しく追加されました | DDL ステートメントが[SQL の配置ルール](/placement-rules-in-sql.md)で指定された配置ルールを無視するかどうかを制御します。デフォルト値は`strict`です。これは、DDL ステートメントが配置ルールを無視しないことを意味します。                                                                             |
| [<code>tidb_rc_read_check_ts</code>](/system-variables.md#tidb_rc_read_check_ts-new-in-v600)                                                                             | 新しく追加されました |                                                                                                                                                                                                                                                                                                      |
| [<code>tidb_sysdate_is_now</code>](/system-variables.md#tidb_sysdate_is_now-new-in-v600)                                                                                 | 新しく追加されました | `SYSDATE`関数を`NOW`関数に置き換えることができるかどうかを制御します。この構成アイテムは、MySQL オプション[<code>sysdate-is-now</code>](https://dev.mysql.com/doc/refman/8.0/en/server-options.html#option_mysqld_sysdate-is-now)と同じ効果があります。デフォルト値は`OFF`です。                     |
| [<code>tidb_table_cache_lease</code>](/system-variables.md#tidb_table_cache_lease-new-in-v600)                                                                           | 新しく追加されました | [テーブルキャッシュ](/cached-tables.md)のリース時間を秒単位で制御します。デフォルト値は`3`です。                                                                                                                                                                                                     |
| [<code>tidb_top_sql_max_meta_count</code>](/system-variables.md#tidb_top_sql_max_meta_count-new-in-v600)                                                                 | 新しく追加されました | 1 分あたり[トップ SQL](/dashboard/top-sql.md)によって収集される SQL ステートメントタイプの最大数を制御します。デフォルト値は`5000`です。                                                                                                                                                             |
| [<code>tidb_top_sql_max_time_series_count</code>](/system-variables.md#tidb_top_sql_max_time_series_count-new-in-v600)                                                   | 新しく追加されました | 負荷に最も寄与する SQL ステートメント（つまり、上位 N）の数を 1 分あたり[トップ SQL](/dashboard/top-sql.md)で記録できるかどうかを制御します。デフォルト値は`100`です。                                                                                                                               |
| [<code>tidb_txn_assertion_level</code>](/system-variables.md#tidb_txn_assertion_level-new-in-v600)                                                                       | 新しく追加されました | アサーションレベルを制御します。アサーションは、データとインデックス間の整合性チェックであり、書き込まれているキーがトランザクションコミットプロセスに存在するかどうかをチェックします。デフォルトでは、チェックによりほとんどのチェック項目が有効になり、パフォーマンスにほとんど影響を与えません。 |

### 構成ファイルのパラメーター {#configuration-file-parameters}

| 構成ファイル | 構成                                                                                                                                                                                 | タイプを変更する     | 説明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| :----------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| TiDB         | `stmt-summary.enable` `stmt-summary.enable-internal-query` `stmt-summary.history-size` PLACEHOLDER-12-PLACEHOLDER-E }} `stmt-summary.max-stmt-count` `stmt-summary.refresh-interval` | 削除                 | [ステートメント要約表](/statement-summary-tables.md)に関連する構成。これらの構成アイテムはすべて削除されます。ステートメントサマリーテーブルを制御するには、SQL 変数を使用する必要があります。                                                                                                                                                                                                                                                                                                                                               |
| TiDB         | [<code>new_collations_enabled_on_first_bootstrap</code>](/tidb-configuration-file.md#new_collations_enabled_on_first_bootstrap)                                                      | 変更                 | 新しい照合のサポートを有効にするかどうかを制御します。 v6.0 以降、デフォルト値は`false`から`true`に変更されています。この構成項目は、クラスターが初めて初期化されたときにのみ有効になります。最初のブートストラップの後、この構成アイテムを使用して新しい照合フレームワークを有効または無効にすることはできません。                                                                                                                                                                                                                          |
| TiKV         | [<code>backup.num-threads</code>](/tikv-configuration-file.md#num-threads-1)                                                                                                         | 変更                 | 値の範囲が`[1, CPU]`に変更されます。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| TiKV         | [<code>raftstore.apply-max-batch-size</code>](/tikv-configuration-file.md#apply-max-batch-size)                                                                                      | 変更                 | 最大値が`10240`に変更されます。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| TiKV         | [<code>raftstore.raft-max-size-per-msg</code>](/tikv-configuration-file.md#raft-max-size-per-msg)                                                                                    | 変更                 |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| TiKV         | [<code>raftstore.store-max-batch-size</code>](/tikv-configuration-file.md#store-max-batch-size)                                                                                      | 変更                 | 最大値は`10240`に設定されます。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| TiKV         | [<code>readpool.unified.max-thread-count</code>](/tikv-configuration-file.md#max-thread-count)                                                                                       | 変更                 | 調整可能範囲が`[min-thread-count, MAX(4, CPU)]`に変更されました。                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| TiKV         | [<code>rocksdb.enable-pipelined-write</code>](/tikv-configuration-file.md#enable-pipelined-write)                                                                                    | 変更                 | デフォルト値は`true`から`false`に変更されます。この構成を有効にすると、以前のパイプライン書き込みが使用されます。この構成を無効にすると、新しいパイプラインコミットメカニズムが使用されます。                                                                                                                                                                                                                                                                                                                                                |
| TiKV         | [<code>rocksdb.max-background-flushes</code>](/tikv-configuration-file.md#max-background-flushes)                                                                                    | 変更                 |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| TiKV         | [<code>rocksdb.max-background-jobs</code>](/tikv-configuration-file.md#max-background-jobs)                                                                                          | 変更                 |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| TiFlash      | [<code>profiles.default.dt_enable_logical_split</code>](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file)                                                            | 変更                 | DeltaTreeStorageEngine のセグメントが論理分割を使用するかどうかを決定します。デフォルト値は`true`から`false`に変更されます。                                                                                                                                                                                                                                                                                                                                                                                                                 |
| TiFlash      | [<code>profiles.default.enable_elastic_threadpool</code>](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file)                                                          | 変更                 | エラスティックスレッドプールを有効にするかどうかを制御します。デフォルト値は`false`から`true`に変更されます。                                                                                                                                                                                                                                                                                                                                                                                                                                |
| TiFlash      | [<code>storage.format_version</code>](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file)                                                                              | 変更                 | TiFlash のデータ検証機能を制御します。デフォルト値は`2`から`3`に変更されます。`format_version`が設定されている場合`3`に、ハードウェア障害による誤った読み取りを回避するために、すべての TiFlash データの読み取り操作で整合性チェックが実行されます。新しいフォーマットバージョンを以前のバージョンにダウングレードできないことに注意してください。 v5.4 より。                                                                                                                                                                               |
| TiDB         | [<code>pessimistic-txn.pessimistic-auto-commit</code>](/tidb-configuration-file.md#pessimistic-auto-commit-new-in-v600)                                                              | 新しく追加されました | 悲観的トランザクションモードがグローバルに有効になっている場合に自動コミットトランザクションが使用するトランザクションモードを決定します（`tidb_txn_mode='pessimistic'`）。                                                                                                                                                                                                                                                                                                                                                                  |
| TiKV         | [<code>pessimistic-txn.in-memory</code>](/tikv-configuration-file.md#in-memory-new-in-v600)                                                                                          | 新しく追加されました | メモリ内のペシミスティックロックを有効にするかどうかを制御します。この機能を有効にすると、ペシミスティックトランザクションは、ペシミスティックロックをディスクに書き込んだり、他のレプリカに複製したりする代わりに、ペシミスティックロックを可能な限り TiKV メモリに保存します。これにより、悲観的なトランザクションのパフォーマンスが向上します。ただし、ペシミスティックロックが失われる可能性は低く、ペシミスティックトランザクションのコミットに失敗する可能性があります。デフォルト値は`true`です。                                     |
| TiKV         | [<code>quota</code>](/tikv-configuration-file.md#quota)                                                                                                                              | 新しく追加されました | フロントエンドリクエストが占めるリソースを制限するクォータリミッターに関連する構成アイテムを追加します。クォータリミッターは実験的な機能であり、デフォルトでは無効になっています。新しいクォータ関連の構成アイテムは、`foreground-cpu-time`、`foreground-write-bandwidth`、`foreground-read-bandwidth`、および`max-delay-duration`。                                                                                                                                                                                                         |
| TiFlash      | [<code>profiles.default.dt_compression_method</code>](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file)                                                              | 新しく追加されました | TiFlash の圧縮アルゴリズムを指定します。オプションの値は、`LZ4`、`zstd`、および`LZ4HC`で、すべて大文字と小文字が区別されません。デフォルト値は`LZ4`です。                                                                                                                                                                                                                                                                                                                                                                                    |
| TiFlash      | [<code>profiles.default.dt_compression_level</code>](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file)                                                               | 新しく追加されました | TiFlash の圧縮レベルを指定します。デフォルト値は`1`です。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| DM           | [<code>loaders.&#x3C;name>.import-mode</code>](/dm/task-configuration-file-full.md#task-configuration-file-template-advanced)                                                        | 新しく追加されました | フルインポートフェーズ中のインポートモード。 v6.0 以降、DM は TiDB Lightning の TiDB バックエンドモードを使用して、完全なインポートフェーズ中にデータをインポートします。以前のローダーコンポーネントは使用されなくなりました。これは内部交換であり、日常業務に明らかな影響はありません。デフォルト値は`sql`に設定されています。これは、`tidb-backend`を使用することを意味します。モード。まれに、`tidb-backend`が完全に互換性がない場合があります。このパラメーターを`loader`に構成することにより、ローダーモードにフォールバックできます。 |
| DM           | [<code>loaders.&#x3C;name>.on-duplicate</code>](/dm/task-configuration-file-full.md#task-configuration-file-template-advanced)                                                       | 新しく追加されました | フルインポートフェーズ中に競合を解決する方法を指定します。デフォルト値は`replace`です。これは、新しいデータを使用して既存のデータを置き換えることを意味します。                                                                                                                                                                                                                                                                                                                                                                              |
| TiCDC        | [<code>dial-timeout</code>](/ticdc/manage-ticdc.md#configure-sink-uri-with-kafka)                                                                                                    | 新しく追加されました | ダウンストリーム Kafka との接続を確立する際のタイムアウト。デフォルト値は`10s`です。                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| TiCDC        | [<code>read-timeout</code>](/ticdc/manage-ticdc.md#configure-sink-uri-with-kafka)                                                                                                    | 新しく追加されました | ダウンストリームの Kafka から返される応答を取得する際のタイムアウト。デフォルト値は`10s`です。                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| TiCDC        | [<code>write-timeout</code>](/ticdc/manage-ticdc.md#configure-sink-uri-with-kafka)                                                                                                   | 新しく追加されました | ダウンストリーム Kafka にリクエストを送信する際のタイムアウト。デフォルト値は`10s`です。                                                                                                                                                                                                                                                                                                                                                                                                                                                     |

### その他 {#others}

- データ配置ポリシーには、次の互換性の変更があります。
  - バインディングはサポートされていません。直接配置オプションは構文から削除されます。
  - `CREATE PLACEMENT POLICY`および`ALTER PLACEMENT POLICY`ステートメントは、`VOTERS`および{{B -PLACEHOLDER-7-PLACEHOLDER 配置オプション。
  - TiDB エコシステムツール（TiDB Binlog、TiCDC、および BR）は、配置ルールと互換性があります。配置オプションは、TiDBBinlog の特別なコメントに移動されます。
  - `information_schema.placement_rules`システムテーブルの名前が`information_schema.placement_policies`に変更されました。このテーブルには、配置ポリシーに関する情報のみが表示されるようになりました。
  - `placement_checks`システム変数は`tidb_placement_mode`に置き換えられます。
  - TiFlash レプリカを持つテーブルに配置ルールのあるパーティションを追加することは禁止されています。
  - `INFORMATION_SCHEMA`列を`INFORMATION_SCHEMA`テーブルから削除します。
- SQL 計画管理（SPM）バインディングの`status`値が変更されました。
  - `using`を削除します。
  - `enabled`（使用可能）を追加して、`using`を置き換えます。
  - `disabled`（使用不可）を追加します。
- DM は OpenAPI インターフェースを変更します
  - 内部メカニズムの変更により、タスク管理に関連するインターフェースは以前の実験バージョンと互換性がありません。適応については、新しい[DMOpenAPI ドキュメント](/dm/dm-open-api.md)を参照する必要があります。
- DM は、完全なインポートフェーズ中に競合を解決するためにメソッドを変更します
  - `loader.<name>.on-duplicate`パラメータが追加されました。デフォルト値は`replace`です。これは、新しいデータを使用して既存のデータを置き換えることを意味します。以前の動作を維持したい場合は、値を`error`に設定できます。このパラメーターは、完全なインポートフェーズ中の動作のみを制御します。
- DM を使用するには、対応するバージョンの`dmctl`を使用する必要があります
  - 内部メカニズムの変更により、DM を v6.0.0 にアップグレードした後、`dmctl`も v6.0.0 にアップグレードする必要があります。
- v5.4（v5.4 のみ）では、TiDB は一部の noop システム変数に誤った値を許可します。 v6.0.0 以降、TiDB はシステム変数に誤った値を設定することを許可していません。 [＃31538](https://github.com/pingcap/tidb/issues/31538)

## 改善 {#improvements}

- TiDB

  - `FLASHBACK`または`RECOVER`ステートメント[＃31668](https://github.com/pingcap/tidb/issues/31668)
  - パフォーマンス概要ダッシュボードを追加して、一般的なクリティカルパスのコアパフォーマンスメトリックを表示し、TiDB でのメトリック分析を容易にします[＃31676](https://github.com/pingcap/tidb/issues/31676)
  - `LOAD DATA LOCAL INFILE`ステートメント[＃24515](https://github.com/pingcap/tidb/issues/24515)での`REPLACE`キーワードの使用のサポート
  - Range パーティションテーブル[＃26739](https://github.com/pingcap/tidb/issues/26739)の組み込み`IN`式のパーティションプルーニングをサポートします。
  - MPP 集計クエリで冗長になる可能性のある Exchange 操作を排除することにより、クエリの効率を向上させます[＃31762](https://github.com/pingcap/tidb/issues/31762)
  - `TRUNCATE PARTITION`および`DROP PARTITION`ステートメント PLACEHOLDER-5-PLACEHOLDER-E }}
  - `ADMIN SHOW DDL JOBS`ステートメント[＃23494](https://github.com/pingcap/tidb/issues/23494)の結果に`CREATE_TIME`情報を表示することをサポート
  - 新しい組み込み関数をサポートする`CHARSET()`[＃3931](https://github.com/pingcap/tidb/issues/3931)
  - ユーザー名[＃32558](https://github.com/pingcap/tidb/issues/32558)によるベースラインキャプチャブロックリストのフィルタリングをサポート
  - ベースラインキャプチャブロックリストでのワイルドカードの使用のサポート[＃32714](https://github.com/pingcap/tidb/issues/32714)
  - 現在の PLACEHOLDER-5-PLACEHOLDER）に従って時間を表示することにより、`ADMIN SHOW DDL JOBS`および`SHOW TABLE STATUS`ステートメントの結果を最適化します。 [＃26642](https://github.com/pingcap/tidb/issues/26642)
  - `DAYNAME()`および`MONTHNAME()`関数を TiFlash[＃32594](https://github.com/pingcap/tidb/issues/32594)にプッシュダウンすることをサポートします
  - `REGEXP`関数を TiFlash[＃32637](https://github.com/pingcap/tidb/issues/32637)にプッシュダウンすることをサポートします
  - `DAYOFMONTH()`および`LAST_DAY()`関数を TiFlash[＃33012](https://github.com/pingcap/tidb/issues/33012)にプッシュダウンすることをサポートします
  - `DAYOFWEEK()`および`DAYOFYEAR()`関数を TiFlash[＃33130](https://github.com/pingcap/tidb/issues/33130)にプッシュダウンすることをサポートします
  - `IS_TRUE`、`IS_FALSE`、および`IS_TRUE_WITH_NULL`関数の TiFlash へのプッシュダウンをサポート{ {B-PLACEHOLDER-7-PLACEHOLDER
  - `GREATEST`および`LEAST`関数を TiFlash[＃32787](https://github.com/pingcap/tidb/issues/32787)にプッシュダウンすることをサポートします
  - `UnionScan`演算子[＃32631](https://github.com/pingcap/tidb/issues/32631)の実行の追跡をサポートします
  - `_tidb_rowid`列[＃31543](https://github.com/pingcap/tidb/issues/31543)を読み取るクエリでの PointGet プランの使用をサポートします
  - 名前を小文字の[＃32719](https://github.com/pingcap/tidb/issues/32719)に変換せずに、`EXPLAIN`ステートメントの出力に元のパーティション名を表示することをサポートします。
  - IN 条件および文字列タイプの列での RANGECOLUMNS パーティショニングのパーティションプルーニングを有効にします[＃32626](https://github.com/pingcap/tidb/issues/32626)
  - システム変数が NULL に設定されている場合にエラーメッセージを返します[＃32850](https://github.com/pingcap/tidb/issues/32850)
  - 非 MPP モードからブロードキャスト参加を削除します[＃31465](https://github.com/pingcap/tidb/issues/31465)
  - 動的プルーニングモードでのパーティションテーブルでの MPP プランの実行をサポート[＃32347](https://github.com/pingcap/tidb/issues/32347)
  - Common Table Expression（CTE）の述語のプッシュダウンをサポート[＃28163](https://github.com/pingcap/tidb/issues/28163)
  - `Statement Summary`と`Capture Plan Baselines`の構成を簡略化して、グローバルベースでのみ利用できるようにします PLACEHOLDER-5-PLACEHOLDER-E }}
  - gopsutil を v3.21.12 に更新して、macOS12 でバイナリをビルドするときに報告されるアラームに対処します[＃31607](https://github.com/pingcap/tidb/issues/31607)

- TiKV

  - 多くのキー範囲を持つバッチの Raftstore のサンプリング精度を向上させます[＃12327](https://github.com/tikv/tikv/issues/12327)
  - `debug/pprof/profile`に正しい「Content-Type」を追加して、プロファイルをより簡単に識別できるようにします[＃11521](https://github.com/tikv/tikv/issues/11521)
  - Raftstore にハートビートがある場合、または読み取り要求を処理する場合に、リーダーのリース時間を無限に更新します。これにより、遅延ジッターが減少します[＃11579](https://github.com/tikv/tikv/issues/11579)
  - リーダーを切り替えるときにコストが最も低いストアを選択します。これにより、パフォーマンスの安定性が向上します[＃10602](https://github.com/tikv/tikv/issues/10602)
  - Raft ログを非同期的にフェッチして、Raftstore のブロックによって引き起こされるパフォーマンスジッターを低減します[＃11320](https://github.com/tikv/tikv/issues/11320)
  - ベクトル計算で`QUARTER`関数をサポート[＃5751](https://github.com/tikv/tikv/issues/5751)
  - `BIT`データ型を TiKV[＃30738](https://github.com/pingcap/tidb/issues/30738)にプッシュダウンすることをサポートします
  - `MOD`関数と`SYSDATE`関数を TiKV[＃11916](https://github.com/tikv/tikv/issues/11916)にプッシュダウンすることをサポートします。
  - ロックの解決ステップ[＃11993](https://github.com/tikv/tikv/issues/11993)を必要とするリージョンの数を減らすことにより、TiCDC の回復時間を短縮します。
  - `raftstore.raft-max-inflight-msgs`[＃11865](https://github.com/tikv/tikv/issues/11865)の動的変更をサポート
  - `EXTRA_PHYSICAL_TABLE_ID_COL_ID`をサポートして、動的プルーニングモードを有効にします[＃11888](https://github.com/tikv/tikv/issues/11888)
  - バケットでの計算をサポート[＃11759](https://github.com/tikv/tikv/issues/11759)
  - RawKVAPIV2 のキーを`user-key`+`memcomparable-padding` + `timestamp`{ {B-PLACEHOLDER-7-PLACEHOLDER
  - RawKVAPIV2 の値を`user-value`+`ttl`+`ValueMeta`としてエンコードします。 `delete`を`ValueMeta`[＃11965](https://github.com/tikv/tikv/issues/11965)にエンコードします
  - `raftstore.raft-max-size-per-msg`[＃12017](https://github.com/tikv/tikv/issues/12017)の動的変更をサポート
  - Grafana でのマルチ k8 のモニタリングをサポート[＃12014](https://github.com/tikv/tikv/issues/12014)
  - リーダーシップを CDC オブザーバーに移して、レイテンシージッターを減らします[＃12111](https://github.com/tikv/tikv/issues/12111)
  - `raftstore.apply_max_batch_size`および`raftstore.store_max_batch_size`[＃11982](https://github.com/tikv/tikv/issues/11982)の動的変更をサポート
  - RawKV V2 は、`raw_get`または`raw_scan`リクエスト[＃11965](https://github.com/tikv/tikv/issues/11965)を受信すると、最新バージョンを返します。
  - RCCheckTS 整合性読み取りをサポート[＃12097](https://github.com/tikv/tikv/issues/12097)
  - `storage.scheduler-worker-pool-size`（スケジューラプールのスレッド数）[＃12067](https://github.com/tikv/tikv/issues/12067)の動的変更をサポート
  - グローバルフォアグラウンドフローコントローラーを使用して CPU と帯域幅の使用を制御し、TiKV のパフォーマンスの安定性を向上させます[＃11855](https://github.com/tikv/tikv/issues/11855)
  - `readpool.unified.max-thread-count`（UnifyReadPool のスレッド数）[＃11781](https://github.com/tikv/tikv/issues/11781)の動的変更をサポート
  - TiKV 内部パイプラインを使用して RocksDB パイプラインを置き換え、`rocksdb.enable-multibatch-write`パラメーター[＃12059](https://github.com/tikv/tikv/issues/12059)を廃止します

- PD

  - リーダーを追い出すときに転送する最速のオブジェクトを自動的に選択することをサポートします。これにより、立ち退きプロセスがスピードアップします[＃4229](https://github.com/tikv/pd/issues/4229)
  - リージョンが使用できなくなった場合に備えて、2 つのレプリカの Raft グループから投票者を削除することを禁止します[＃4564](https://github.com/tikv/pd/issues/4564)
  - バランスリーダーのスケジューリングをスピードアップします[＃4652](https://github.com/tikv/pd/issues/4652)

- TiFlash

  - TiFlash ファイルの論理分割を禁止します（`profiles.default.dt_enable_logical_split`のデフォルト値を`false`に調整します。[ユーザードキュメント](/tiflash/tiflash-configuration.md#tiflash-configuration-parameters)の詳細）および TiFlash 列型ストレージのスペース使用効率を改善して、TiFlash に同期されたテーブルのスペース占有が TiKV のテーブルのスペース占有と同様になるようにします。
  - 以前のクラスター管理モジュールを TiDB に統合することにより、TiFlash のクラスター管理とレプリカ複製メカニズムを最適化します。これにより、小さなテーブルのレプリカ作成が高速化されます[＃29924](https://github.com/pingcap/tidb/issues/29924)

- ツール

  - バックアップと復元（BR）

    - バックアップデータの復元速度を向上させます。 BR が 16TB のデータを 15 ノード（各ノードに 16 個の CPU コアがある）の TiKV クラスターに復元するシミュレーションテストでは、スループットは 2.66 GiB/s に達します。 [＃27036](https://github.com/pingcap/tidb/issues/27036)

    - 配置ルールのインポートとエクスポートをサポートします。 `--with-tidb-placement-mode`パラメータを追加して、データをインポートするときに配置ルールを無視するかどうかを制御します。 [＃32290](https://github.com/pingcap/tidb/issues/32290)

  - TiCDC

    - Grafana[＃4891](https://github.com/pingcap/tiflow/issues/4891)に`Lag analyze`パネルを追加します
    - 配置ルールをサポートする[＃4846](https://github.com/pingcap/tiflow/issues/4846)
    - HTTPAPI 処理を同期します[＃1710](https://github.com/pingcap/tiflow/issues/1710)
    - チェンジフィードを再開するための指数バックオフメカニズムを追加します[＃3329](https://github.com/pingcap/tiflow/issues/3329)
    - MySQL シンクのデフォルトの分離レベルを読み取りコミットに設定して、MySQL のデッドロックを減らします[＃3589](https://github.com/pingcap/tiflow/issues/3589)
    - 作成時にチェンジフィードパラメータを検証し、エラーメッセージを調整します[＃1716](https://github.com/pingcap/tiflow/issues/1716) [＃1718](https://github.com/pingcap/tiflow/issues/1718) [＃1719](https://github.com/pingcap/tiflow/issues/1719) {{ B-PLACEHOLDER-7-PLACEHOLDER
    - Kafka プロデューサーの構成パラメーターを公開して、TiCDC で構成可能にします[＃4385](https://github.com/pingcap/tiflow/issues/4385)

  - TiDB データ移行（DM）

    - アップストリームテーブルスキーマに一貫性がなく、楽観的モードの場合のタスクの開始をサポート[＃3629](https://github.com/pingcap/tiflow/issues/3629) [＃3708](https://github.com/pingcap/tiflow/issues/3708) PLACEHOLDER-5-PLACEHOLDER- E}}
    - `stopped`状態[＃4484](https://github.com/pingcap/tiflow/issues/4484)でのタスクの作成をサポートします
    - `/tmp`ではなく DM-worker の作業ディレクトリを使用して内部ファイルを書き込み、タスクの停止後にディレクトリをクリーンアップすることで Syncer をサポートします PLACEHOLDER-3-PLACEHOLDER- E}}
    - 事前チェックが改善されました。一部の重要なチェックがスキップされなくなりました。 [＃3608](https://github.com/pingcap/tiflow/issues/3608)

  - TiDB Lightning

    - 再試行可能なエラータイプをさらに追加します[＃31376](https://github.com/pingcap/tidb/issues/31376)
    - base64 形式のパスワード文字列をサポートします[＃31194](https://github.com/pingcap/tidb/issues/31194)
    - エラーコードとエラー出力を標準化する[＃32239](https://github.com/pingcap/tidb/issues/32239)

## バグの修正 {#bug-fixes}

- TiDB

  - `SCHEDULE = majority_in_primary`、`PrimaryRegion`および PLACEHOLDER-5- PLACEHOLDER は同じ値です[＃31271](https://github.com/pingcap/tidb/issues/31271)
  - インデックスルックアップ結合[＃30468](https://github.com/pingcap/tidb/issues/30468)を使用してクエリを実行するときの`invalid transaction`エラーを修正しました
  - 2 つ以上の特権が付与されたときに`show grants`が誤った結果を返すバグを修正します[＃30855](https://github.com/pingcap/tidb/issues/30855)
  - `INSERT INTO t1 SET timestamp_col = DEFAULT`が、デフォルトで`CURRENT_TIMESTAMP` [＃29926](https://github.com/pingcap/tidb/issues/29926)
  - 文字列タイプ[＃31721](https://github.com/pingcap/tidb/issues/31721)の最大値と最小の非 null 値のエンコードを回避することにより、結果の読み取りで報告されたエラーを修正します
  - エスケープ文字[＃31589](https://github.com/pingcap/tidb/issues/31589)でデータが壊れた場合のデータのロードパニックを修正しました
  - 照合を伴う`greatest`または`least`関数が誤った結果 PLACEHOLDER-5-PLACEHOLDER-E }}
  - date_add 関数と date_sub 関数が誤ったデータ型を返す可能性があるバグを修正します[＃31809](https://github.com/pingcap/tidb/issues/31809)
  - 挿入ステートメント[＃31735](https://github.com/pingcap/tidb/issues/31735)を使用して、仮想的に生成された列にデータを挿入するときに発生する可能性があったパニックを修正します。
  - 作成されたリストパーティションに重複する列が存在する場合にエラーが報告されないバグを修正します[＃31784](https://github.com/pingcap/tidb/issues/31784)
  - `select for update union select`が誤ったスナップショット[＃31530](https://github.com/pingcap/tidb/issues/31530)を使用したときに返される誤った結果を修正します
  - 復元操作の終了後にリージョンが不均一に分散される可能性があるという潜在的な問題を修正します[＃31034](https://github.com/pingcap/tidb/issues/31034)
  - `json`タイプ[＃31541](https://github.com/pingcap/tidb/issues/31541)の COERCIBILITY が間違っているバグを修正します
  - このタイプが builtin-func[＃31320](https://github.com/pingcap/tidb/issues/31320)を使用して処理される場合の、`json`タイプの誤った照合を修正します。
  - TiFlash レプリカの数が 0 に設定されている場合に PD ルールが削除されないバグを修正します[＃32190](https://github.com/pingcap/tidb/issues/32190)
  - `alter column set default`がテーブルスキーマ[＃31074](https://github.com/pingcap/tidb/issues/31074)を誤って更新する問題を修正します
  - TiDB の`date_format`が MySQL と互換性のない方法で処理する問題を修正します[＃32232](https://github.com/pingcap/tidb/issues/32232)
  - join[＃31629](https://github.com/pingcap/tidb/issues/31629)を使用してパーティションテーブルを更新するときにエラーが発生する可能性があるバグを修正します
  - 列挙値[＃32428](https://github.com/pingcap/tidb/issues/32428)の Nulleq 関数の誤った範囲計算結果を修正しました
  - `upper()`および`lower()`関数[＃32488](https://github.com/pingcap/tidb/issues/32488)で発生する可能性のあるパニックを修正
  - 他のタイプの列をタイムスタンプタイプの列に変更するときに発生するタイムゾーンの問題を修正します[＃29585](https://github.com/pingcap/tidb/issues/29585)
  - ChunkRPC[＃31981](https://github.com/pingcap/tidb/issues/31981)[＃30880](https://github.com/pingcap/tidb/issues/30880)を使用してデータをエクスポートするときの TiDBOOM を修正しました
  - サブ SELECTLIMIT が動的パーティションプルーニングモードで期待どおりに機能しないバグを修正します[＃32516](https://github.com/pingcap/tidb/issues/32516)
  - `INFORMATION_SCHEMA.COLUMNS`テーブル[＃32655](https://github.com/pingcap/tidb/issues/32655)のビットデフォルト値の誤った形式または一貫性のない形式を修正
  - サーバーの再起動後にパーティションテーブルを一覧表示するためにパーティションテーブルのプルーニングが機能しない可能性があるバグを修正します[＃32416](https://github.com/pingcap/tidb/issues/32416)
  - `SET timestamp`が実行された後に`add column`が誤ったデフォルトのタイムスタンプを使用する可能性があるバグを修正します PLACEHOLDER-5-PLACEHOLDER-E }}
  - MySQL5.5 または 5.6 クライアントから TiDB パスワードなしアカウントに接続すると失敗する可能性があるバグを修正します[＃32334](https://github.com/pingcap/tidb/issues/32334)
  - トランザクション[＃29851](https://github.com/pingcap/tidb/issues/29851)で動的モードでパーティションテーブルを読み取るときの誤った結果を修正します
  - TiDB が重複タスクを TiFlash にディスパッチする可能性があるバグを修正します[＃32814](https://github.com/pingcap/tidb/issues/32814)
  - `timdiff`関数の入力にミリ秒の[＃31680](https://github.com/pingcap/tidb/issues/31680)が含まれている場合に返される誤った結果を修正します
  - パーティションを明示的に読み取り、IndexJoin プランを使用する場合の誤った結果を修正します[＃32007](https://github.com/pingcap/tidb/issues/32007)
  - 列タイプを同時に変更すると列の名前変更が失敗するバグを修正します[＃31075](https://github.com/pingcap/tidb/issues/31075)
  - TiFlash プランの正味コストを計算する式が TiKV プランと一致しないバグを修正します[＃30103](https://github.com/pingcap/tidb/issues/30103)
  - `KILL TIDB`がアイドル状態の接続ですぐに有効にならないバグを修正します[＃24031](https://github.com/pingcap/tidb/issues/24031)
  - 生成された列を含むテーブルをクエリするときに発生する可能性のある誤った結果を修正します[＃33038](https://github.com/pingcap/tidb/issues/33038)
  - `left join`[＃31321](https://github.com/pingcap/tidb/issues/31321)を使用して複数のテーブルのデータを削除した誤った結果を修正します
  - `SUBTIME`関数がオーバーフローの場合に間違った結果を返すバグを修正します[＃31868](https://github.com/pingcap/tidb/issues/31868)
  - 集計クエリに`having`条件 PLACEHOLDER-5 が含まれている場合、`selection`演算子をプッシュダウンできないバグを修正します。 -PLACEHOLDER
  - クエリがエラーを報告したときに CTE がブロックされる可能性があるバグを修正します[＃31302](https://github.com/pingcap/tidb/issues/31302)
  - 非厳密モードでテーブルを作成するときに varbinary または varchar 列の長さが長すぎると、エラーが発生する可能性があるバグを修正します[＃30328](https://github.com/pingcap/tidb/issues/30328)
  - フォロワーが指定されていない場合の`information_schema.placement_policies`の間違ったフォロワー数を修正する[＃31702](https://github.com/pingcap/tidb/issues/31702)
  - インデックスが作成されるときに TiDB が列プレフィックスの長さを 0 として指定できる問題を修正します[＃31972](https://github.com/pingcap/tidb/issues/31972)
  - TiDB がスペースで終わるパーティション名を許可する問題を修正します[＃31535](https://github.com/pingcap/tidb/issues/31535)
  - `RENAME TABLE`ステートメント[＃29893](https://github.com/pingcap/tidb/issues/29893)のエラーメッセージを修正します

- TiKV

  - ピアステータスが`Applying`[＃11746](https://github.com/tikv/tikv/issues/11746)のときにスナップショットファイルを削除することによって引き起こされるパニックの問題を修正します
  - フロー制御が有効で、`level0_slowdown_trigger`が明示的に設定されている場合の QPS ドロップの問題を修正します[＃11424](https://github.com/tikv/tikv/issues/11424)
  - ピアを破棄すると待ち時間が長くなる可能性があるという問題を修正します[＃10210](https://github.com/tikv/tikv/issues/10210)
  - GC ワーカーがビジー状態のときに TiKV がデータの範囲を削除できない（つまり、内部コマンド`unsafe_destroy_range`が実行される）バグを修正します PLACEHOLDER-3-PLACEHOLDER-E }}
  - 一部のコーナーケース[＃11852](https://github.com/tikv/tikv/issues/11852)で`StoreMeta`のデータが誤って削除されたときに TiKV がパニックになるバグを修正します。
  - ARM プラットフォームでプロファイリングを実行するときに TiKV がパニックになるバグを修正します[＃10658](https://github.com/tikv/tikv/issues/10658)
  - TiKV が 2 年以上実行されている場合にパニックになる可能性があるバグを修正します[＃11940](https://github.com/tikv/tikv/issues/11940)
  - SSE 命令セット[＃12034](https://github.com/tikv/tikv/issues/12034)が欠落しているために発生する ARM64 アーキテクチャのコンパイルの問題を修正します。
  - 初期化されていないレプリカを削除すると、古いレプリカが再作成される可能性がある問題を修正します[＃10533](https://github.com/tikv/tikv/issues/10533)
  - 古いメッセージが原因で TiKV がパニックになるバグを修正します[＃12023](https://github.com/tikv/tikv/issues/12023)
  - TsSet 変換で未定義動作（UB）が発生する可能性がある問題を修正します[＃12070](https://github.com/tikv/tikv/issues/12070)
  - レプリカの読み取りが線形化可能性に違反する可能性があるバグを修正します[＃12109](https://github.com/tikv/tikv/issues/12109)
  - TiKV が Ubuntu18.04 でプロファイリングを実行するときに発生する可能性のあるパニックの問題を修正します[＃9765](https://github.com/tikv/tikv/issues/9765)
  - 文字列の一致が正しくないために tikv-ctl が誤った結果を返す問題を修正します[＃12329](https://github.com/tikv/tikv/issues/12329)
  - メモリメトリックのオーバーフローによって引き起こされる断続的なパケット損失とメモリ不足（OOM）の問題を修正します[＃12160](https://github.com/tikv/tikv/issues/12160)
  - TiKV を終了するときに誤って TiKV パニックを報告するという潜在的な問題を修正します[＃12231](https://github.com/tikv/tikv/issues/12231)

- PD

  - PD がジョイントコンセンサスの無意味なステップで演算子を生成する問題を修正します[＃4362](https://github.com/tikv/pd/issues/4362)
  - PD クライアントを閉じるときに TSO 取り消しプロセスがスタックする可能性があるバグを修正します[＃4549](https://github.com/tikv/pd/issues/4549)
  - リージョンスキャッタースケジューリングが一部のピアを失った問題を修正します[＃4565](https://github.com/tikv/pd/issues/4565)
  - `dr-autosync`の`Duration`フィールドを動的に構成できない[＃4651](https://github.com/tikv/pd/issues/4651)の問題を修正します。

- TiFlash

  - メモリ制限が有効になっているときに発生する TiFlash パニックの問題を修正します[＃3902](https://github.com/pingcap/tiflash/issues/3902)
  - 期限切れのデータがゆっくりとリサイクルされる問題を修正します[＃4146](https://github.com/pingcap/tiflash/issues/4146)
  - `Snapshot`が複数の DDL 操作[＃4072](https://github.com/pingcap/tiflash/issues/4072)と同時に適用される場合の TiFlash パニックの潜在的な問題を修正します。
  - 読み取りワークロードが重い場合に列を追加した後の潜在的なクエリエラーを修正します[＃3967](https://github.com/pingcap/tiflash/issues/3967)
  - 負の引数を持つ`SQRT`関数が PLACEHOLDER-5-PLACEHOLDER-E ではなく`NaN`を返す問題を修正します}} [＃3598](https://github.com/pingcap/tiflash/issues/3598)
  - `INT`を`DECIMAL`にキャストするとオーバーフローが発生する可能性がある問題を修正します[＃3920](https://github.com/pingcap/tiflash/issues/3920)
  - `IN`の結果が複数値式[＃4016](https://github.com/pingcap/tiflash/issues/4016)で正しくない問題を修正します
  - 日付形式が`'\n'`を無効な区切り文字[＃4036](https://github.com/pingcap/tiflash/issues/4036)として識別する問題を修正します
  - 同時実行性の高いシナリオでは、学習者の読み取りプロセスに時間がかかりすぎるという問題を修正します[＃3555](https://github.com/pingcap/tiflash/issues/3555)
  - `DATETIME`を`DECIMAL`[＃4151](https://github.com/pingcap/tiflash/issues/4151)にキャストするときに発生する間違った結果を修正します
  - クエリがキャンセルされたときに発生するメモリリークの問題を修正します[＃4098](https://github.com/pingcap/tiflash/issues/4098)
  - エラスティックスレッドプールを有効にするとメモリリークが発生する可能性があるバグを修正します[＃4098](https://github.com/pingcap/tiflash/issues/4098)
  - ローカルトンネルが有効になっている場合、MPP クエリをキャンセルすると、タスクが永久にハングする可能性があるバグを修正します[＃4229](https://github.com/pingcap/tiflash/issues/4229)
  - HashJoin ビルド側の失敗により、MPP クエリが永久にハングする可能性があるバグを修正します[＃4195](https://github.com/pingcap/tiflash/issues/4195)
  - MPP タスクがスレッドを永久にリークする可能性があるバグを修正します[＃4238](https://github.com/pingcap/tiflash/issues/4238)

- ツール

  - バックアップと復元（BR）

    - 復元操作で回復不能なエラーが発生したときに BR がスタックするバグを修正します[＃33200](https://github.com/pingcap/tidb/issues/33200)
    - バックアップの再試行中に暗号化情報が失われた場合に復元操作が失敗するバグを修正します[＃32423](https://github.com/pingcap/tidb/issues/32423)

  - TiCDC

    - `batch-replace-enable`が無効になっている場合に MySQL シンクが重複した`replace`SQL ステートメントを生成するバグを修正します PLACEHOLDER-5-PLACEHOLDER-E }}
    - PD リーダーが強制終了されたときに TiCDC ノードが異常終了するバグを修正します[＃4248](https://github.com/pingcap/tiflow/issues/4248)
    - 一部の MySQL バージョン[＃4504](https://github.com/pingcap/tiflow/issues/4504)のエラー`Unknown system variable 'transaction_isolation'`を修正します
    - `Canal-JSON`が`string`[＃4635](https://github.com/pingcap/tiflow/issues/4635)を誤って処理した場合に発生する可能性がある TiCDC パニックの問題を修正します
    - シーケンスが誤って複製される場合があるバグを修正します[＃4563](https://github.com/pingcap/tiflow/issues/4552)
    - `Canal-JSON`が nil[＃4736](https://github.com/pingcap/tiflow/issues/4736)をサポートしていないために発生する可能性がある TiCDC パニックの問題を修正します
    - タイプ`Enum/Set`および`TinyText/MediumText/Text/LongText`[＃4454](https://github.com/pingcap/tiflow/issues/4454)の avro コーデックの誤ったデータマッピングを修正します
    - Avro が`NOT NULL`列を null 許容フィールド[＃4818](https://github.com/pingcap/tiflow/issues/4818)に変換するバグを修正します
    - TiCDC が[＃4699](https://github.com/pingcap/tiflow/issues/4699)を終了できない問題を修正します

  - TiDB データ移行（DM）

    - ステータス[＃4281](https://github.com/pingcap/tiflow/issues/4281)を照会するときにのみシンカーメトリックが更新される問題を修正します
    - セーフモードでの更新ステートメントの実行エラーが DM ワーカーのパニックを引き起こす可能性がある問題を修正します[＃4317](https://github.com/pingcap/tiflow/issues/4317)
    - longvarchars がエラーを報告するバグを修正します`Column length too big`[＃4637](https://github.com/pingcap/tiflow/issues/4637)
    - 複数の DM ワーカーが同じアップストリーム[＃3737](https://github.com/pingcap/tiflow/issues/3737)からデータを書き込むことによって引き起こされる競合の問題を修正します
    - 何百もの「チェックポイントに変更がない、同期フラッシュチェックポイントをスキップ」がログに出力され、レプリケーションが非常に遅いという問題を修正します[＃4619](https://github.com/pingcap/tiflow/issues/4619)
    - シャードをマージし、ペシミスティックモードでアップストリームからインクリメンタルデータを複製するときの DML 損失の問題を修正します[＃5002](https://github.com/pingcap/tiflow/issues/5002)

  - TiDB Lightning

    - 一部のインポートタスクにソースファイルが含まれていない場合に TiDBLightning がメタデータスキーマを削除しない可能性があるバグを修正します[＃28144](https://github.com/pingcap/tidb/issues/28144)
    - ソースファイルとターゲットクラスターのテーブル名が異なる場合に発生するパニックを修正します[＃31771](https://github.com/pingcap/tidb/issues/31771)
    - チェックサムエラー「GC の有効期間がトランザクション期間より短い」を修正しました[＃32733](https://github.com/pingcap/tidb/issues/32733)
    - 空のテーブルのチェックに失敗したときに TiDBLightning がスタックする問題を修正します[＃31797](https://github.com/pingcap/tidb/issues/31797)

  - 団子

    - `dumpling --sql $query` [＃30532](https://github.com/pingcap/tidb/issues/30532)を実行すると、表示される進行状況が正確でない問題を修正します。
    - AmazonS3 が圧縮データのサイズを正しく計算できない問題を修正します[＃30534](https://github.com/pingcap/tidb/issues/30534)

  - TiDB Binlog

    - 大規模なアップストリーム書き込みトランザクションが Kafka[＃1136](https://github.com/pingcap/tidb-binlog/issues/1136)に複製されるときに TiDBBinlog がスキップされる可能性がある問題を修正します。
