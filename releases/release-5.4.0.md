---
title: TiDB 5.4 Release Notes
---

# TiDB5.4 リリースノート {#tidb-5-4-release-notes}

発売日：2022 年 2 月 15 日

TiDB バージョン：5.4.0

v5.4 では、主な新機能または改善点は次のとおりです。

- GBK 文字セットをサポートする
- 複数の列のインデックスのフィルタリング結果をマージするデータへのアクセスにインデックスマージを使用することをサポートします
- セッション変数を使用した古いデータの読み取りをサポート
- 統計を収集するための構成の永続化をサポート
- TiKV のログストレージエンジンとしての RaftEngine の使用のサポート（実験的）
- クラスターへのバックアップの影響を最適化する
- バックアップストレージとしての AzureBlob ストレージの使用のサポート
- TiFlash と MPP エンジンの安定性とパフォーマンスを継続的に改善します
- TiDB Lightning にスイッチを追加して、データを含む既存のテーブルへのインポートを許可するかどうかを決定します
- 連続プロファイリング機能の最適化（実験的）
- TiSpark はユーザーの識別と認証をサポートします

## 互換性の変更 {#compatibility-changes}

> <strong>ノート：</strong>
>
> 以前の TiDB バージョンから v5.4.0 にアップグレードするときに、すべての中間バージョンの互換性変更に関する注意事項を知りたい場合は、対応するバージョンの[リリースノート](/releases/release-notes.md)を確認できます。

### システム変数 {#system-variables}

| 変数名                                                                                              | タイプを変更する     | 説明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| :-------------------------------------------------------------------------------------------------- | :------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`tidb_enable_column_tracking`](/system-variables.md#tidb_enable_column_tracking-new-in-v540)       | 新しく追加されました | TiDB が`PREDICATE COLUMNS`を収集できるようにするかどうかを制御します。デフォルト値は`OFF`です。                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| [`tidb_enable_paging`](/system-variables.md#tidb_enable_paging-new-in-v540)                         | 新しく追加されました | `IndexLookUp`のオペレーターでコプロセッサー要求を送信するためにページングの方法を使用するかどうかを制御します。デフォルト値は`OFF`です。<br/> `IndexLookup`と`Limit`を使用し、 `Limit`を`IndexScan`にプッシュダウンできない読み取りクエリの場合、読み取りクエリの待機時間が長くなり、TiKV の`unified read pool`の CPU 使用率が高くなる可能性があります。このような場合、 `Limit`演算子は少数のデータセットしか必要としないため、 `tidb_enable_paging`を`ON`に設定すると、TiDB が処理するデータが少なくなり、クエリの待機時間とリソース消費が削減されます。 |
| [`tidb_enable_top_sql`](/system-variables.md#tidb_enable_top_sql-new-in-v540)                       | 新しく追加されました | トップ SQL 機能を有効にするかどうかを制御します。デフォルト値は`OFF`です。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| [`tidb_persist_analyze_options`](/system-variables.md#tidb_persist_analyze_options-new-in-v540)     | 新しく追加されました | [ANALYZE 構成の永続性](/statistics.md#persist-analyze-configurations)機能を有効にするかどうかを制御します。デフォルト値は`ON`です。                                                                                                                                                                                                                                                                                                                                                                                                                        |
| [`tidb_read_staleness`](/system-variables.md#tidb_read_staleness-new-in-v540)                       | 新しく追加されました | 現在のセッションで読み取ることができる履歴データの範囲を制御します。デフォルト値は`0`です。                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| [`tidb_regard_null_as_point`](/system-variables.md#tidb_regard_null_as_point-new-in-v540)           | 新しく追加されました | オプティマイザが、インデックスアクセスのプレフィックス条件として null 等価を含むクエリ条件を使用できるかどうかを制御します。                                                                                                                                                                                                                                                                                                                                                                                                                               |
| [`tidb_stats_load_sync_wait`](/system-variables.md#tidb_stats_load_sync_wait-new-in-v540)           | 新しく追加されました | 統計の同期ロード機能を有効にするかどうかを制御します。デフォルト値`0`は、機能が無効になっていて、統計が非同期にロードされることを意味します。この機能が有効になっている場合、この変数は、SQL 最適化がタイムアウトする前に統計の同期ロードを待機できる最大時間を制御します。                                                                                                                                                                                                                                                                                |
| [`tidb_stats_load_pseudo_timeout`](/system-variables.md#tidb_stats_load_pseudo_timeout-new-in-v540) | 新しく追加されました | SQL が失敗するか（ `OFF` ）、疑似統計の使用にフォールバックするか（ `ON` ）、統計の同期ロードがタイムアウトに達するタイミングを制御します。デフォルト値は`OFF`です。                                                                                                                                                                                                                                                                                                                                                                                       |
| [`tidb_backoff_lock_fast`](/system-variables.md#tidb_backoff_lock_fast)                             | 変更                 | デフォルト値は`100`から`10`に変更されます。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| [`tidb_enable_index_merge`](/system-variables.md#tidb_enable_index_merge-new-in-v40)                | 変更                 | デフォルト値は`OFF`から`ON`に変更されます。<br/><ul><li> TiDB クラスターを v4.0.0 より前のバージョンから v5.4.0 以降にアップグレードする場合、この変数はデフォルトで`OFF`です。</li><li> TiDB クラスターを v4.0.0 以降から v5.4.0 以降にアップグレードする場合、この変数はアップグレード前と同じままです。</li><li> v5.4.0 以降の新しく作成された TiDB クラスターの場合、この変数はデフォルトで`ON`です。</li></ul>                                                                                                                                        |
| [`tidb_store_limit`](/system-variables.md#tidb_store_limit-new-in-v304-and-v40)                     | 変更                 | v5.4.0 より前では、この変数はインスタンスレベルおよびグローバルに構成できました。 v5.4.0 以降、この変数はグローバル構成のみをサポートします。                                                                                                                                                                                                                                                                                                                                                                                                              |

### 構成ファイルのパラメーター {#configuration-file-parameters}

| 構成ファイル          | 構成                                                                                                            | タイプを変更する     | 説明                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| :-------------------- | :-------------------------------------------------------------------------------------------------------------- | :------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| TiDB                  | [`stats-load-concurrency`](/tidb-configuration-file.md#stats-load-concurrency-new-in-v540)                      | 新しく追加されました | TiDB 同期ロード統計機能が同時に処理できる列の最大数を制御します。デフォルト値は`5`です。                                                                                                                                                                                                                                                                                                                                                                              |
| TiDB                  | [`stats-load-queue-size`](/tidb-configuration-file.md#stats-load-queue-size-new-in-v540)                        | 新しく追加されました | TiDB 同期ロード統計機能がキャッシュできる列要求の最大数を制御します。デフォルト値は`1000`です。                                                                                                                                                                                                                                                                                                                                                                       |
| TiKV                  | [`snap-generator-pool-size`](/tikv-configuration-file.md#snap-generator-pool-size-new-in-v540)                  | 新しく追加されました | `snap-generator`スレッドプールのサイズ。デフォルト値は`2`です。                                                                                                                                                                                                                                                                                                                                                                                                       |
| TiKV                  | `log.file.max-size` `log.file.max-days` `log.file.max-backups`                                                  | 新しく追加されました | 詳細については、 [TiKV 構成ファイル`log.file`](/tikv-configuration-file.md#logfile-new-in-v540)を参照してください。                                                                                                                                                                                                                                                                                                                                                   |
| TiKV                  | `raft-engine`                                                                                                   | 新しく追加されました | `enable` `recovery-mode` `target-file-size` `bytes-per-sync` `recovery-read-block-size` `purge-threshold` `recovery-read-block-size` `batch-compression-threshold` `recovery-threads` `dir`詳細については、 [TiKV 構成ファイル-raft `raft-engine`](/tikv-configuration-file.md#raft-engine)を参照してください。                                                                                                                                                       |
| TiKV                  | [`backup.enable-auto-tune`](/tikv-configuration-file.md#enable-auto-tune-new-in-v540)                           | 新しく追加されました | v5.3.0 では、デフォルト値は`false`です。 v5.4.0 以降、デフォルト値は`true`に変更されています。このパラメーターは、クラスターリソースの使用率が高い場合に、バックアップタスクで使用されるリソースを制限して、クラスターへの影響を減らすかどうかを制御します。デフォルトの構成では、バックアップタスクの速度が低下する可能性があります。                                                                                                                                |
| TiKV                  | `log-level` `log-format` `log-file` `log-rotation-size`                                                         | 変更                 | TiKV ログパラメータの名前は、 `log.enable-timestamp` `log.level` `log.format`置き換えられ`log.file.filename` 。古いパラメーターのみを設定し、それらの値がデフォルト以外の値に設定されている場合、古いパラメーターは新しいパラメーターとの互換性を維持します。古いパラメータと新しいパラメータの両方が設定されている場合、新しいパラメータが有効になります。詳細については、 [TiKV 構成ファイル-ログ](/tikv-configuration-file.md#log-new-in-v540)を参照してください。 |
| TiKV                  | `log-rotation-timespan`                                                                                         | 削除                 | ログローテーション間のタイムスパン。このタイムスパンが経過すると、ログファイルがローテーションされます。つまり、現在のログファイルのファイル名にタイムスタンプが追加され、新しいログファイルが作成されます。                                                                                                                                                                                                                                                          |
| TiKV                  | `allow-remove-leader`                                                                                           | 削除                 | メインスイッチの削除を許可するかどうかを決定します。                                                                                                                                                                                                                                                                                                                                                                                                                  |
| TiKV                  | `raft-msg-flush-interval`                                                                                       | 削除                 | Raft メッセージがバッチで送信される間隔を決定します。 Raft メッセージは、この構成アイテムで指定された間隔ごとにバッチで送信されます。                                                                                                                                                                                                                                                                                                                                 |
| PD                    | [`log.level`](/pd-configuration-file.md#level)                                                                  | 変更                 | デフォルト値は「INFO」から「info」に変更され、大文字と小文字が区別されないことが保証されています。                                                                                                                                                                                                                                                                                                                                                                    |
| TiFlash               | [`profile.default.enable_elastic_threadpool`](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file) | 新しく追加されました | エラスティックスレッドプール機能を有効にするか無効にするかを決定します。この構成アイテムを有効にすると、同時実行性の高いシナリオで TiFlashCPU 使用率を大幅に向上させることができます。デフォルト値は`false`です。                                                                                                                                                                                                                                                     |
| TiFlash               | [`storage.format_version`](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file)                    | 新しく追加されました | DTFile のバージョンを指定します。デフォルト値は`2`で、その下にハッシュがデータファイルに埋め込まれます。値を`3`に設定することもできます。 `3`の場合、データファイルにはメタデータとトークンデータのチェックサムが含まれ、複数のハッシュアルゴリズムがサポートされます。                                                                                                                                                                                               |
| TiFlash               | [`logger.count`](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file)                              | 変更                 | デフォルト値は`10`に変更されます。                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| TiFlash               | [`status.metrics_port`](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file)                       | 変更                 | デフォルト値は`8234`に変更されます。                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| TiFlash               | [`raftstore.apply-pool-size`](/tiflash/tiflash-configuration.md#configure-the-tiflash-learnertoml-file)         | 新しく追加されました | Raft データをストレージにフラッシュするプール内のスレッドの許容数。デフォルト値は`4`です。                                                                                                                                                                                                                                                                                                                                                                            |
| TiFlash               | [`raftstore.store-pool-size`](/tiflash/tiflash-configuration.md#configure-the-tiflash-learnertoml-file)         | 新しく追加されました | Raft を処理するスレッドの許容数。これは Raftstore スレッドプールのサイズです。デフォルト値は`4`です。                                                                                                                                                                                                                                                                                                                                                                 |
| TiDB データ移行（DM） | [`collation_compatible`](/dm/task-configuration-file-full.md#task-configuration-file-template-advanced)         | 新しく追加されました | `CREATE`の SQL ステートメントでデフォルトの照合を同期するモード。値のオプションは「loose」（デフォルト）と「strict」です。                                                                                                                                                                                                                                                                                                                                            |
| TiCDC                 | `max-message-bytes`                                                                                             | 変更                 | Kafka シンクのデフォルト値`max-message-bytes`を`104857601` （10MB）に変更します                                                                                                                                                                                                                                                                                                                                                                                       |
| TiCDC                 | `partition-num`                                                                                                 | 変更                 | KafkaSink のデフォルト値`partition-num`を`4`から`3`に変更します。これにより、TiCDC は Kafaka パーティションにメッセージをより均等に送信します。                                                                                                                                                                                                                                                                                                                       |
| TiDB Lightning        | `meta-schema-name`                                                                                              | 変更                 | ターゲット TiDB のメタデータのスキーマ名を指定します。 v5.4.0 以降、このスキーマは[並列インポート](/tidb-lightning/tidb-lightning-distributed-import.md)を有効にした場合にのみ作成されます（対応するパラメーターは`tikv-importer.incremental-import = true`です）。                                                                                                                                                                                                   |
| TiDB Lightning        | `task-info-schema-name`                                                                                         | 新しく追加されました | TiDBLightning が競合を検出したときに重複データが保存されるデータベースの名前を指定します。デフォルトでは、値は「lightning_task_info」です。このパラメーターは、「重複解決」機能を有効にしている場合にのみ指定してください。                                                                                                                                                                                                                                           |
| TiDB Lightning        | `incremental-import`                                                                                            | 新しく追加されました | データがすでに存在するテーブルへのデータのインポートを許可するかどうかを決定します。デフォルト値は`false`です。                                                                                                                                                                                                                                                                                                                                                       |

### その他 {#others}

- TiDB と PD の間にインターフェースが追加されます。 `information_schema.TIDB_HOT_REGIONS_HISTORY`システムテーブルを使用する場合、TiDB は対応するバージョンの PD を使用する必要があります。
- TiDB サーバー、PD サーバー、および TiKV サーバーは、ログに関連するパラメーターに統一された命名方法を使用して、ログ名、出力形式、およびローテーションと有効期限のルールを管理し始めます。詳細については、 [TiKV 構成ファイル-ログ](/tikv-configuration-file.md#log-new-in-v540)を参照してください。
- v5.4.0 以降、プランキャッシュを介してキャッシュされた実行プランの SQL バインディングを作成すると、バインディングは、対応するクエリに対してすでにキャッシュされているプランを無効にします。新しいバインディングは、v5.4.0 より前にキャッシュされた実行プランには影響しません。
- v5.3 以前のバージョンでは、 [TiDB データ移行（DM）](https://docs.pingcap.com/tidb-data-migration/v5.3/)のドキュメントは TiDB ドキュメントから独立しています。 v5.4 以降、DM ドキュメントは同じバージョンの TiDB ドキュメントに統合されています。 DM ドキュメントサイトにアクセスしなくても、 [DM ドキュメント](/dm/dm-overview.md)を直接読むことができます。
- cdclog とともにポイントインタイムリカバリ（PITR）の実験的機能を削除します。 v5.4.0 以降、cdclog ベースの PITR および cdclog はサポートされなくなりました。

## 新機能 {#new-features}

### SQL {#sql}

- <strong>TiDB は、v5.4.0 以降の GBK 文字セットをサポートしています</strong>

  `latin1`より前では、 `binary`は`ascii` 、および`utf8`の文字セットをサポートしてい`utf8mb4` 。

  中国語ユーザーをより適切にサポートするために、TiDB は v5.4.0 以降の GBK 文字セットをサポートしています。 TiDB クラスターを初めて初期化するときに TiDB 構成ファイルで[`new_collations_enabled_on_first_bootstrap`](/tidb-configuration-file.md#new_collations_enabled_on_first_bootstrap)オプションを有効にした後、TiDBGBK 文字セットは`gbk_bin`と`gbk_chinese_ci`の両方の照合をサポートします。

  GBK 文字セットを使用する場合は、互換性の制限に注意する必要があります。詳細については、 [文字セットと照合-GBK](/character-set-gbk.md)を参照してください。

### 安全 {#security}

- <strong>TiSpark はユーザー認証と承認をサポートします</strong>

  TiSpark 2.5.0 以降、TiSpark は、データベースユーザー認証とデータベースまたはテーブルレベルでの読み取り/書き込み認証の両方をサポートしています。この機能を有効にすると、データを取得するための描画など、ビジネスが不正なバッチタスクを実行するのを防ぐことができます。これにより、オンラインクラスターの安定性とデータセキュリティが向上します。

  この機能はデフォルトで無効になっています。有効になっている場合、TiSpark を介して操作するユーザーに必要な権限がない場合、ユーザーは TiSpark から例外を受け取ります。

  [ユーザードキュメント](/tispark-overview.md#security)

- <strong>TiUP は、root ユーザーの初期パスワードの生成をサポートしています</strong>

  クラスターを開始するためのコマンドに`--init`パラメーターが導入されています。このパラメーターを使用すると、TiUP を使用してデプロイされた TiDB クラスターで、TiUP はデータベース root ユーザーの初期の強力なパスワードを生成します。これにより、パスワードが空の root ユーザーを使用する際のセキュリティリスクが回避され、データベースのセキュリティが確保されます。

  [ユーザードキュメント](/production-deployment-using-tiup.md#step-7-start-a-tidb-cluster)

### パフォーマンス {#performance}

- <strong>カラム型ストレージエンジン TiFlash とコンピューティングエンジン MPP の安定性とパフォーマンスを継続的に改善します</strong>

  - MPP エンジンへのより多くの機能の融合をサポートします。

    - 文字列`RPAD()` `STRCMP()` `LPAD()`
    - `SUBDATE()` `QUARTER()` `DATE_SUB()` `ADDDATE()` `DATE_ADD()`

  - リソース使用率を改善するためのエラスティックスレッドプール機能の導入（実験的）

  - TiKV からデータを複製するときに、行ベースのストレージ形式から列ベースのストレージ形式にデータを変換する効率を向上させます。これにより、データ複製の全体的なパフォーマンスが 50％向上します。

  - 一部の構成アイテムのデフォルト値を調整することにより、TiFlash のパフォーマンスと安定性を向上させます。 HTAP ハイブリッドロードでは、単一のテーブルに対する単純なクエリのパフォーマンスが最大 20％向上します。

  ユーザー[tiflash.toml ファイルを構成します](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file) ： [サポートされているプッシュダウン計算](/tiflash/use-tiflash.md#supported-push-down-calculations)

- <strong>セッション変数を介して指定された時間範囲内の履歴データを読み取ります</strong>

  TiDB は、Raft コンセンサスアルゴリズムに基づくマルチレプリカ分散データベースです。同時実行性とスループットが高いアプリケーションシナリオに直面した場合、TiDB は、フォロワーレプリカを介して読み取りパフォーマンスをスケールアウトし、読み取り要求と書き込み要求を分離できます。

  さまざまなアプリケーションシナリオに対して、TiDB は、強一貫性のある読み取りと弱一貫性のある履歴の読み取りという 2 つのフォロワー読み取りモードを提供します。強一貫性のある読み取りモードは、リアルタイムデータを必要とするアプリケーションシナリオに適しています。ただし、このモードでは、リーダーとフォロワー間のデータレプリケーションのレイテンシーとスループットの低下により、特に地理的に分散された展開の場合、読み取りリクエストのレイテンシーが高くなる可能性があります。

  リアルタイムデータに対する要件がそれほど厳しくないアプリケーションシナリオでは、履歴読み取りモードをお勧めします。このモードは、待ち時間を短縮し、スループットを向上させることができます。 TiDB は現在、次の方法による履歴データの読み取りをサポートしています。SQL ステートメントを使用して、過去のある時点からデータを読み取るか、過去の時点に基づいて読み取り専用トランザクションを開始します。どちらの方法も、特定の時点または指定された時間範囲内の履歴データの読み取りをサポートしています。詳しくは[`AS OF TIMESTAMP`句を使用して履歴データを読み取る](/as-of-timestamp.md)をご覧ください。

  v5.4.0 以降、TiDB は、セッション変数を介して指定された時間範囲内の履歴データの読み取りをサポートすることにより、履歴読み取りモードの使いやすさを向上させます。このモードは、準リアルタイムのシナリオで低遅延、高スループットの読み取り要求を処理します。変数は次のように設定できます。

  ```sql
  set @@tidb_replica_read=leader_and_follower
  set @@tidb_read_staleness="-5"
  ```

  この設定により、TiDB は最も近いリーダーまたはフォロワーノードを選択し、5 秒以内に最新の履歴データを読み取ることができます。

  [ユーザードキュメント](/tidb-read-staleness.md)

- <strong>インデックスマージの GA</strong>

  インデックスマージは、SQL 最適化の実験的な機能として TiDBv4.0 に導入されました。このメソッドは、クエリで複数列のデータのスキャンが必要な場合に、条件フィルタリングを大幅に高速化します。例として次のクエリを取り上げます。 `WHERE`ステートメントで、 `OR`で接続されたフィルタリング条件の列<em>key1</em>と<em>key2</em>にそれぞれのインデックスがある場合、インデックスマージ機能は、それぞれのインデックスを同時にフィルタリングし、クエリ結果をマージして、マージされた結果を返します。

  ```sql
  SELECT * FROM table WHERE key1 <= 100 OR key2 = 200;
  ```

  TiDB v4.0 より前では、テーブルに対するクエリは、一度に 1 つのインデックスのみを使用してフィルタリングすることをサポートしていました。データの複数の列をクエリする場合は、インデックスマージを有効にして、個々の列のインデックスを使用することにより、正確なクエリ結果を短時間で取得できます。インデックスマージは、不要な全表スキャンを回避し、多数の複合インデックスを確立する必要がありません。

  v5.4.0 では、インデックスマージは GA になります。ただし、次の制限に注意する必要があります。

  - インデックスマージは、選言標準形（ <sub>X1⋁X2⋁</sub> <sub>…</sub> <sub>Xn</sub> ）のみをサポートします。つまり、この機能は、 `WHERE`句のフィルタリング条件が`OR`で接続されている場合にのみ機能します。

  - v5.4.0 以降の新しくデプロイされた TiDB クラスターの場合、この機能はデフォルトで有効になっています。以前のバージョンからアップグレードされた v5.4.0 以降の TiDB クラスターの場合、この機能はアップグレード前の設定を継承し、必要に応じて設定を変更できます（v4.0 より前の TiDB クラスターの場合、この機能は存在せず、デフォルトで無効になっています） 。

  [ユーザードキュメント](/explain-index-merge.md)

- <strong>いかだエンジンのサポート（実験的）</strong>

  TiKV のログストレージエンジンとして[いかだエンジン](https://github.com/tikv/raft-engine)を使用することをサポートします。 RocksDB と比較して、RaftEngine は TiKVI / O 書き込みトラフィックを最大 40％、CPU 使用率を 10％削減し、特定の負荷の下でフォアグラウンドスループットを約 5％向上させ、テールレイテンシーを 20％削減します。さらに、Raft Engine はログのリサイクルの効率を改善し、極端な条件でのログの蓄積の問題を修正します。

  Raft Engine はまだ実験的な機能であり、デフォルトでは無効になっています。 v5.4.0 の RaftEngine のデータ形式は、以前のバージョンと互換性がないことに注意してください。クラスタをアップグレードまたはダウングレードする前に、すべての TiKV ノードで RaftEngine が無効になっていることを確認する必要があります。 RaftEngine は v5.4.0 以降のバージョンでのみ使用することをお勧めします。

  [ユーザードキュメント](/tikv-configuration-file.md#raft-engine)

- <strong><code>PREDICATE COLUMNS</code>に関する統計の収集をサポート（実験的）</strong>

  ほとんどの場合、SQL ステートメントを実行するとき、オプティマイザーは一部の列（ `WHERE` 、および`JOIN` `GROUP BY`の列など）の統計のみを使用し`ORDER BY` 。これらの使用される列は`PREDICATE COLUMNS`と呼ばれます。

  v5.4.0 以降、 [`tidb_enable_column_tracking`](/system-variables.md#tidb_enable_column_tracking-new-in-v540)システム変数の値を`ON`に設定して、TiDB が`PREDICATE COLUMNS`を収集できるようにすることができます。

  設定後、TiDB は 100\* [`stats-lease`](/tidb-configuration-file.md#stats-lease)ごとに`PREDICATE COLUMNS`の情報を`mysql.column_stats_usage`のシステムテーブルに書き込みます。ビジネスのクエリパターンが安定している場合は、 `ANALYZE TABLE TableName PREDICATE COLUMNS`構文を使用して、 `PREDICATE COLUMNS`列のみの統計を収集できます。これにより、統計収集のオーバーヘッドを大幅に削減できます。

  [ユーザードキュメント](/statistics.md#collect-statistics-on-some-columns)

- <strong>統計の同期ロードをサポート（実験的）</strong>

  v5.4.0 以降、TiDB は同期ロード統計機能を導入しています。この機能はデフォルトで無効になっています。この機能を有効にした後、TiDB は、SQL ステートメントの実行時に、大きなサイズの統計（ヒストグラム、TopN、Count-Min Sketch 統計など）をメモリに同期的にロードできます。これにより、SQL 最適化の統計の完全性が向上します。

  [ユーザードキュメント](/statistics.md#load-statistics)

### 安定 {#stability}

- <strong>ANALYZE 構成の永続化をサポート</strong>

  統計は、オプティマイザーが実行計画を生成するときに参照する基本情報の 1 つのタイプです。統計の正確さは、生成された実行計画が妥当であるかどうかに直接影響します。統計の正確性を確保するために、テーブル、パーティション、およびインデックスごとに異なるコレクション構成を設定する必要がある場合があります。

  v5.4.0 以降、TiDB はいくつかの`ANALYZE`構成の永続化をサポートしています。この機能を使用すると、既存の構成を将来の統計収集に簡単に再利用できます。

  `ANALYZE`構成永続化機能はデフォルトで有効になっています（システム変数`tidb_analyze_version`は`2`で、 [`tidb_persist_analyze_options`](/system-variables.md#tidb_persist_analyze_options-new-in-v540)はデフォルトで`ON`です）。この機能を使用して、ステートメントを手動で実行するときに、 `ANALYZE`ステートメントで指定された永続性構成を記録できます。記録されると、次に TiDB が統計を自動的に更新するか、これらの構成を指定せずに手動で統計を収集すると、TiDB は記録された構成に従って統計を収集します。

  [ユーザードキュメント](/statistics.md#persist-analyze-configurations)

### 高可用性とディザスタリカバリ {#high-availability-and-disaster-recovery}

- <strong>クラスターへのバックアップタスクの影響を減らす</strong>

  Backup＆Restore（BR）は、自動調整機能を導入します（デフォルトで有効になっています）。この機能は、クラスターリソースの使用状況を監視し、バックアップタスクで使用されるスレッドの数を調整することで、クラスターへのバックアップタスクの影響を減らすことができます。場合によっては、バックアップ用のクラスターハードウェアリソースを増やして自動調整機能を有効にすると、クラスターへのバックアップタスクの影響を 10％以下に制限できます。

  [ユーザードキュメント](/br/br-auto-tune.md)

- <strong>バックアップのターゲットストレージとして AzureBlobStorage をサポートする</strong>

  Backup＆Restore（BR）は、リモートバックアップストレージとして AzureBlobStorage をサポートします。 TiDB を AzureCloud にデプロイすると、クラスターデータを AzureBlobStorage サービスにバックアップできるようになります。

  [ユーザードキュメント](/br/backup-and-restore-azblob.md)

### データ移行 {#data-migration}

- <strong>TiDB Lightning には、データを含むテーブルへのデータのインポートを許可するかどうかを決定するための新機能が導入されています。</strong>

  TiDB Lightning では、構成アイテム`incremental-import`が導入されています。データを含むテーブルへのデータのインポートを許可するかどうかを決定します。デフォルト値は`false`です。並列インポートモードを使用する場合は、構成を`true`に設定する必要があります。

  [ユーザードキュメント](/tidb-lightning/tidb-lightning-configuration.md#tidb-lightning-task)

- <strong>TiDB Lightning は、並列インポート用のメタ情報を格納するスキーマ名を導入します</strong>

  TiDB Lightning は、 `meta-schema-name`の構成アイテムを導入します。並列インポートモードでは、このパラメーターは、ターゲットクラスター内の各 TiDBLightning インスタンスのメタ情報を格納するスキーマ名を指定します。デフォルトでは、値は「lightning_metadata」です。このパラメーターに設定される値は、同じ並列インポートに参加する各 TiDBLightning インスタンスで同じである必要があります。そうしないと、インポートされたデータの正確性を保証できません。

  [ユーザードキュメント](/tidb-lightning/tidb-lightning-configuration.md#tidb-lightning-task)

- <strong>TiDBLightning は重複した解像度を導入します</strong>

  ローカルバックエンドモードでは、TiDB Lightning は、データのインポートが完了する前に重複データを出力し、その重複データをデータベースから削除します。インポートの完了後に重複データを解決し、アプリケーションルールに従って挿入する適切なデータを選択できます。後続の増分データ移行フェーズで検出された重複データによって引き起こされるデータの不整合を回避するために、重複データに基づいてアップストリームデータソースをクリーンアップすることをお勧めします。

  [ユーザードキュメント](/tidb-lightning/tidb-lightning-error-resolution.md)

- <strong>TiDB データ移行（DM）でのリレーログの使用を最適化する</strong>

  - `source`の構成で`enable-relay`のスイッチを回復します。

  - `start-relay`および`stop-relay`コマンドを使用して、リレーログの動的な有効化と無効化をサポートします。

  - リレーログのステータスを`source`にバインドします。 `source`は、DM ワーカーに移行された後も、有効または無効の元のステータスを維持します。

  - リレーログのストレージパスを DM-worker 構成ファイルに移動します。

  [ユーザードキュメント](/dm/relay-log.md)

- <strong>DM での<a href="/character-set-and-collation.md">照合</a>の処理を最適化する</strong>

  `collation_compatible`の構成アイテムを追加します。値のオプションは`loose` （デフォルト）と`strict`です：

  - アプリケーションに照合に関する厳密な要件がなく、クエリ結果の照合がアップストリームとダウンストリームで異なる可能性がある場合は、デフォルトの`loose`モードを使用してエラーの報告を回避できます。
  - アプリケーションに照合に関する厳しい要件があり、照合がアップストリームとダウンストリームの間で一貫している必要がある場合は、 `strict`モードを使用できます。ただし、ダウンストリームがアップストリームのデフォルトの照合をサポートしていない場合、データレプリケーションはエラーを報告する可能性があります。

  [ユーザードキュメント](/dm/task-configuration-file-full.md#task-configuration-file-template-advanced)

- <strong>DM の<code>transfer source</code>を最適化して、レプリケーションタスクのスムーズな実行をサポートします</strong>

  DM ワーカーノードの負荷が不均衡な場合、 `transfer source`コマンドを使用して、 `source`の構成を別の負荷に手動で転送できます。最適化後、 `transfer source`コマンドは手動操作を簡素化します。 DM は他の操作を内部で完了するため、関連するすべてのタスクを一時停止するのではなく、ソースをスムーズに転送できます。

- <strong>DM OpenAPI が一般提供になります（GA）</strong>

  DM は、データソースの追加やタスクの管理など、API を介した日常の管理をサポートします。 v5.4.0 では、DMOpenAPI が GA になります。

  [ユーザードキュメント](/dm/dm-open-api.md)

### 診断効率 {#diagnostic-efficiency}

- <strong>トップ SQL（実験的機能）</strong>

  新しい実験的な機能である TopSQL（デフォルトでは無効）が導入され、ソースを消費するクエリを簡単に見つけることができます。

  [ユーザードキュメント](/dashboard/top-sql.md)

### TiDB データ共有サブスクリプション {#tidb-data-share-subscription}

- <strong>クラスターに対する TiCDC の影響を最適化する</strong>

  TiCDC を使用すると、TiDB クラスターのパフォーマンスへの影響が大幅に減少します。テスト環境では、TiDB に対する TiCDC のパフォーマンスへの影響を 5％未満に減らすことができます。

### 展開とメンテナンス {#deployment-and-maintenance}

- <strong>継続的なプロファイリングの強化（実験的）</strong>

  - サポートされるその他のコンポーネント：TiDB、PD、および TiKV に加えて、TiDBv5.4.0 は TiFlash の CPU プロファイリングもサポートします。

  - より多くの形式のプロファイリング表示：フレームチャートでの CPU プロファイリングと Goroutine の結果の表示をサポートします。

  - サポートされるより多くのデプロイメント環境：継続的プロファイリングは、TiDB オペレーターを使用してデプロイされたクラスターにも使用できます。

  継続的プロファイリングはデフォルトで無効になっており、TiDB ダッシュボードで有効にできます。

  継続的プロファイリングは、v1.9.0 以降の TiUP または v1.3.0 以降の TiDB オペレーターを使用してデプロイまたはアップグレードされたクラスターに適用できます。

  [ユーザードキュメント](/dashboard/continuous-profiling.md)

## 改善 {#improvements}

- TiDB

  - 新しいシステム変数[`tidb_enable_paging`](/system-variables.md#tidb_enable_paging-new-in-v540)を追加して、ページングを使用してコプロセッサー要求を送信するかどうかを決定します。この機能を有効にすると、処理するデータの量を減らし、レイテンシとリソース消費を減らすことができます[＃30578](https://github.com/pingcap/tidb/issues/30578)
  - キャッシュされたクエリプラン[＃30370](https://github.com/pingcap/tidb/pull/30370)をクリアするための`ADMIN {SESSION | INSTANCE | GLOBAL} PLAN_CACHE`構文をサポートする

- TiKV

  - コプロセッサーは、ストリームのような方法でリクエストを処理するためのページング API をサポートします[＃11448](https://github.com/tikv/tikv/issues/11448)
  - 読み取り操作がセカンダリロックが解決されるのを待つ必要がないように`read-through-lock`をサポートします[＃11402](https://github.com/tikv/tikv/issues/11402)
  - ディスクスペースの枯渇によるパニックを回避するために、ディスク保護メカニズムを追加します[＃10537](https://github.com/tikv/tikv/issues/10537)
  - ログのアーカイブとローテーションのサポート[＃11651](https://github.com/tikv/tikv/issues/11651)
  - Raft クライアントによるシステムコールを減らし、CPU 効率を向上させます[＃11309](https://github.com/tikv/tikv/issues/11309)
  - コプロセッサーは、部分文字列を TiKV1 にプッシュダウンすることをサポートし[＃11495](https://github.com/tikv/tikv/issues/11495)
  - 読み取りコミット分離レベル[＃11485](https://github.com/tikv/tikv/issues/11485)で読み取りロックをスキップすることにより、スキャンパフォーマンスを向上させます。
  - バックアップ操作で使用されるデフォルトのスレッドプールサイズを減らし、ストレスが高い場合はスレッドプールの使用を制限します[＃11000](https://github.com/tikv/tikv/issues/11000)
  - アプライスレッドプールとストアスレッドプールのサイズの動的調整をサポート[＃11159](https://github.com/tikv/tikv/issues/11159)
  - `snap-generator`スレッドプール[＃11247](https://github.com/tikv/tikv/issues/11247)のサイズの構成をサポートします。
  - 読み取りと書き込みが頻繁に行われるファイルが多数ある場合に発生するグローバルロック競合の問題を最適化する[＃250](https://github.com/tikv/rocksdb/pull/250)

- PD

  - デフォルトで履歴ホットスポット情報を記録する[＃25281](https://github.com/pingcap/tidb/issues/25281)
  - HTTP コンポーネントの署名を追加して、要求元を識別します[＃4490](https://github.com/tikv/pd/issues/4490)
  - TiDB ダッシュボードを v2021.12.31 に更新します[＃4257](https://github.com/tikv/pd/issues/4257)

- TiFlash

  - ローカルオペレーターのコミュニケーションを最適化する
  - スレッドの頻繁な作成または破棄を回避するために、gRPC の非一時的なスレッド数を増やします

- ツール

  - バックアップと復元（BR）

    - BR が暗号化されたバックアップを実行するときにキーの有効性チェックを追加する[＃29794](https://github.com/pingcap/tidb/issues/29794)

  - TiCDC

    - 「EventFeed 再試行率制限」ログの数を減らす[＃4006](https://github.com/pingcap/tiflow/issues/4006)
    - 多くのテーブルを複製するときの複製待ち時間を短縮する[＃3900](https://github.com/pingcap/tiflow/issues/3900)
    - TiKV ストアがダウンしたときに KV クライアントが回復する時間を短縮する[＃3191](https://github.com/pingcap/tiflow/issues/3191)

  - TiDB データ移行（DM）

    - リレー有効時の CPU 使用率を下げる[＃2214](https://github.com/pingcap/dm/issues/2214)

  - TiDB Lightning

    - デフォルトで楽観的なトランザクションを使用してデータを書き込み、TiDB バックエンドモード[＃30953](https://github.com/pingcap/tidb/pull/30953)のパフォーマンスを向上させます

  - 団子

    - Dumpling がデータベースバージョン[＃29500](https://github.com/pingcap/tidb/pull/29500)をチェックするときの互換性を改善します
    - `CREATE DATABASE`と[＃3420](https://github.com/pingcap/tiflow/issues/3420)をダンプするときにデフォルトの照合を追加し`CREATE TABLE`

## バグの修正 {#bug-fixes}

- TiDB

  - クラスターを v4.x から[＃25422](https://github.com/pingcap/tidb/issues/25422)にアップグレードするときに発生する`tidb_analyze_version`の値の変更の問題を修正します。
  - サブクエリで異なる照合を使用するときに発生する間違った結果の問題を修正します[＃30748](https://github.com/pingcap/tidb/issues/30748)
  - TiDB の`concat(ifnull(time(3))`の結果が MySQL3 の結果と異なる問題を修正し[＃29498](https://github.com/pingcap/tidb/issues/29498)
  - 楽観的トランザクションモード[＃30410](https://github.com/pingcap/tidb/issues/30410)での潜在的なデータインデックスの不整合の問題を修正します
  - [＃30200](https://github.com/pingcap/tidb/issues/30200)を TiKV1 にプッシュダウンできない場合に、IndexMerge のクエリ実行プランが間違っている問題を修正します。
  - 列タイプを同時に変更すると、スキーマとデータの間に不整合が生じる問題を修正します[＃31048](https://github.com/pingcap/tidb/issues/31048)
  - サブクエリ[＃30913](https://github.com/pingcap/tidb/issues/30913)がある場合に IndexMerge クエリの結果が間違っている問題を修正します
  - クライアントで FetchSize の設定が大きすぎる場合に発生するパニックの問題を修正します[＃30896](https://github.com/pingcap/tidb/issues/30896)
  - LEFTJOIN が誤って INNERJOIN1 に変換される可能性がある問題を修正し[＃20510](https://github.com/pingcap/tidb/issues/20510)
  - `CASE-WHEN`式と照合を一緒に使用するとパニックが発生する可能性がある問題を修正します[＃30245](https://github.com/pingcap/tidb/issues/30245)
  - `IN`の値にバイナリ定数[＃31261](https://github.com/pingcap/tidb/issues/31261)が含まれている場合に発生する誤ったクエリ結果の問題を修正します
  - CTE にサブクエリ[＃31255](https://github.com/pingcap/tidb/issues/31255)がある場合に発生する誤ったクエリ結果の問題を修正します
  - `INSERT ... SELECT ... ON DUPLICATE KEY UPDATE`ステートメントを実行するとパニックになる問題を修正します[＃28078](https://github.com/pingcap/tidb/issues/28078)
  - INDEXHASHJOIN が`send on closed channel`エラー[＃31129](https://github.com/pingcap/tidb/issues/31129)を返す問題を修正します

- TiKV

  - MVCC 削除レコードが GC1 によってクリアされない問題を修正し[＃11217](https://github.com/tikv/tikv/issues/11217)
  - 悲観的なトランザクションモードで事前書き込み要求を再試行すると、まれにデータの不整合のリスクが発生する可能性があるという問題を修正します[＃11187](https://github.com/tikv/tikv/issues/11187)
  - GC スキャンがメモリオーバーフローを引き起こす問題を修正します[＃11410](https://github.com/tikv/tikv/issues/11410)
  - ディスク容量がいっぱいになると、RocksDB のフラッシュまたは圧縮によってパニックが発生する問題を修正します[＃11224](https://github.com/tikv/tikv/issues/11224)

- PD

  - リージョン統計が[＃4295](https://github.com/tikv/pd/issues/4295)の影響を受けない問題を修正し`flow-round-by-digit`
  - ターゲットストアがダウンしているためにスケジューリングオペレーターが迅速に失敗できない問題を修正します[＃3353](https://github.com/tikv/pd/issues/3353)
  - オフラインストアのリージョンをマージできない問題を修正します[＃4119](https://github.com/tikv/pd/issues/4119)
  - コールドホットスポットデータをホットスポット統計から削除できない問題を修正します[＃4390](https://github.com/tikv/pd/issues/4390)

- TiFlash

  - MPP クエリが停止したときに TiFlash がパニックになる可能性がある問題を修正します
  - `where <string>`句を使用したクエリが間違った結果を返す問題を修正します
  - 整数主キーの列タイプをより広い範囲に設定するときに発生する可能性があるデータの不整合の潜在的な問題を修正します
  - 入力時刻が 1970-01-0100:00:01UTC より前の場合、 `unix_timestamp`の動作が TiDB または MySQL の動作と矛盾する問題を修正します。
  - 再起動後に TiFlash が`EstablishMPPConnection`エラーを返す可能性がある問題を修正します
  - `CastStringAsDecimal`の動作が TiFlash と TiDB/TiKV で一貫していない問題を修正します
  - クエリ結果に`DB::Exception: Encode type of coprocessor response is not CHBlock`のエラーが返される問題を修正します
  - `castStringAsReal`の動作が TiFlash と TiDB/TiKV で一貫していない問題を修正します
  - TiFlash の`date_add_string_xxx`関数の返される結果が MySQL の結果と矛盾する問題を修正します

- ツール

  - バックアップと復元（BR）

    - 復元操作の終了後にリージョンの分布が不均一になる可能性があるという潜在的な問題を修正します[＃30425](https://github.com/pingcap/tidb/issues/30425)
    - `minio`がバックアップストレージとして使用されている場合、エンドポイントで`'/'`を指定できない問題を修正します[＃30104](https://github.com/pingcap/tidb/issues/30104)
    - システムテーブルを同時にバックアップするとテーブル名の更新に失敗するため、システムテーブルを復元できない問題を修正します[＃29710](https://github.com/pingcap/tidb/issues/29710)

  - TiCDC

    - `min.insync.replicas`が[＃3994](https://github.com/pingcap/tiflow/issues/3994)より小さい場合、レプリケーションを実行できない問題を修正し`replication-factor` 。
    - `cached region`モニタリングメトリックが負の[＃4300](https://github.com/pingcap/tiflow/issues/4300)である問題を修正します
    - `mq sink write row`に監視データがないという問題を修正します[＃3431](https://github.com/pingcap/tiflow/issues/3431)
    - [＃3810](https://github.com/pingcap/tiflow/issues/3810)の互換性の問題を修正し`sql mode`
    - レプリケーションタスクが削除されたときに発生する可能性のあるパニックの問題を修正する[＃3128](https://github.com/pingcap/tiflow/issues/3128)
    - デフォルトの列値[＃3929](https://github.com/pingcap/tiflow/issues/3929)を出力するときに発生するパニックとデータの不整合の問題を修正します
    - デフォルト値を複製できない問題を修正します[＃3793](https://github.com/pingcap/tiflow/issues/3793)
    - デッドロックによってレプリケーションタスクがスタックするという潜在的な問題を修正します[＃4055](https://github.com/pingcap/tiflow/issues/4055)
    - ディスクが完全に書き込まれたときにログが出力されない問題を修正します[＃3362](https://github.com/pingcap/tiflow/issues/3362)
    - DDL ステートメントの特別なコメントによってレプリケーションタスクが停止する問題を修正します[＃3755](https://github.com/pingcap/tiflow/issues/3755)
    - RHEL リリース[＃3584](https://github.com/pingcap/tiflow/issues/3584)のタイムゾーンの問題が原因でサービスを開始できない問題を修正します
    - 不正確なチェックポイント[＃3545](https://github.com/pingcap/tiflow/issues/3545)によって引き起こされる潜在的なデータ損失の問題を修正します
    - コンテナ環境での OOM の問題を修正する[＃1798](https://github.com/pingcap/tiflow/issues/1798)
    - `config.Metadata.Timeout`の誤った構成によって引き起こされるレプリケーション停止の問題を修正し[＃3352](https://github.com/pingcap/tiflow/issues/3352) 。

  - TiDB データ移行（DM）

    - `CREATE VIEW`ステートメントがデータレプリケーションを中断する問題を修正します[＃4173](https://github.com/pingcap/tiflow/issues/4173)
    - DDL ステートメントがスキップされた後にスキーマをリセットする必要がある問題を修正します[＃4177](https://github.com/pingcap/tiflow/issues/4177)
    - DDL ステートメントがスキップされた後にテーブルチェックポイントが時間内に更新されない問題を修正します[＃4184](https://github.com/pingcap/tiflow/issues/4184)
    - TiDB バージョンと Parser バージョン[＃4298](https://github.com/pingcap/tiflow/issues/4298)の互換性の問題を修正します
    - シンカーメトリックがステータス[＃4281](https://github.com/pingcap/tiflow/issues/4281)を照会するときにのみ更新される問題を修正します

  - TiDB Lightning

    - TiDBLightning に`mysql.tidb`テーブル[＃31088](https://github.com/pingcap/tidb/issues/31088)にアクセスする権限がない場合に発生する誤ったインポート結果の問題を修正します。
    - TiDBLightning の再起動時に一部のチェックがスキップされる問題を修正します[＃30772](https://github.com/pingcap/tidb/issues/30772)
    - S3 パスが存在しない場合に TiDBLighting がエラーを報告できない問題を修正します[＃30674](https://github.com/pingcap/tidb/pull/30674)

  - TiDB Binlog

    - Drainer が`CREATE PLACEMENT POLICY`ステートメント[＃1118](https://github.com/pingcap/tidb-binlog/issues/1118)と互換性がないために失敗する問題を修正します
