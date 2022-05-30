---
title: TiDB Introduction
summary: Learn about the NewSQL database TiDB that supports HTAP workloads.
aliases: ['/docs/dev/']
---

# TiDBの紹介 {#tidb-introduction}

[TiDB](https://github.com/pingcap/tidb) （/&#39;taɪdiːbi：/、「Ti」はTitaniumの略）は、Hybrid Transactional and Analytical Processing（HTAP）ワークロードをサポートするオープンソースの分散型NewSQLデータベースです。 MySQLと互換性があり、水平方向のスケーラビリティ、強力な一貫性、および高可用性を備えています。 TiDBは、オンプレミスまたはクラウド内にデプロイできます。

クラウド向けに設計されたTiDBは、クラウドプラットフォームに柔軟なスケーラビリティ、信頼性、セキュリティを提供します。ユーザーは、変化するワークロードの要件を満たすためにTiDBを柔軟にスケーリングできます。 [TiDBオペレーター](https://docs.pingcap.com/tidb-in-kubernetes/v1.1/tidb-operator-overview)は、KubernetesでのTiDBの管理を支援し、運用タスクを自動化します。これにより、管理対象のKubernetesを提供するクラウドへのTiDBのデプロイが容易になります。フルマネージドのTiDBサービスである[TiDBクラウド](https://pingcap.com/tidb-cloud/)は、 [クラウド内のTiDB](https://docs.pingcap.com/tidbcloud/)のフルパワーをアンロックするための最も簡単で、最も経済的で、最も回復力のある方法であり、数回クリックするだけでTiDBクラスターを展開および実行できます。

<NavColumns>
<NavColumn>
<ColumnTitle>About TiDB</ColumnTitle>

-   [TiDBの紹介](/overview.md)
-   [基本的な機能](/basic-features.md)
-   [TiDB6.0リリースノート](/releases/release-6.0.0-dmr.md)
-   [TiDBリリースタイムライン](/releases/release-timeline.md)
-   [MySQLとの互換性](/mysql-compatibility.md)
-   [使用制限](/tidb-limitations.md)
-   [TiDB採用者](/adopters.md)

</NavColumn>

<NavColumn>
<ColumnTitle>Quick Start</ColumnTitle>

-   [TiDBのクイックスタート](/quick-start-with-tidb.md)
-   [HTAPのクイックスタート](/quick-start-with-htap.md)
-   [TiDBでSQLを探索する](/basic-sql-operations.md)
-   [HTAPを探索する](/explore-htap.md)

</NavColumn>

<NavColumn>
<ColumnTitle>Deploy and Use</ColumnTitle>

-   [ハードウェアとソフトウェアの要件](/hardware-and-software-requirements.md)
-   [環境と構成を確認する](/check-before-deployment.md)
-   [TiUPを使用してTiDBクラスターをデプロイする](/production-deployment-using-tiup.md)
-   [分析処理にTiFlashを使用する](/tiflash/tiflash-overview.md)
-   [KubernetesにTiDBをデプロイする](https://docs.pingcap.com/tidb-in-kubernetes/stable)

</NavColumn>

<NavColumn>
<ColumnTitle>Migrate Data</ColumnTitle>

-   [移行の概要](/migration-overview.md)
-   [CSVファイルからTiDBへのデータの移行](/migrate-from-csv-files-to-tidb.md)
-   [SQLファイルからTiDBへのデータの移行](/migrate-from-sql-files-to-tidb.md)
-   [AmazonAuroraからTiDBへのデータの移行](/migrate-aurora-to-tidb.md)

</NavColumn>

<NavColumn>
<ColumnTitle>Maintain</ColumnTitle>

-   [TiUPを使用してTiDBをアップグレードする](/upgrade-tidb-using-tiup.md)
-   [TiUPを使用してTiDBをスケーリングする](/scale-tidb-using-tiup.md)
-   [データのバックアップと復元](/br/backup-and-restore-tool.md)
-   [TiCDCの展開と管理](/ticdc/manage-ticdc.md)
-   [TiUPを使用してTiDBを維持する](/maintain-tidb-using-tiup.md)
-   [TiFlashを維持する](/tiflash/maintain-tiflash.md)

</NavColumn>

<NavColumn>
<ColumnTitle>Monitor and Alert</ColumnTitle>

-   [監視フレームワーク](/tidb-monitoring-framework.md)
-   [モニタリングAPI](/tidb-monitoring-api.md)
-   [監視サービスの展開](/deploy-monitoring-services.md)
-   [Grafanaスナップショットのエクスポート](/exporting-grafana-snapshots.md)
-   [アラートルールとソリューション](/alert-rules.md)
-   [TiFlashアラートルールとソリューション](/tiflash/tiflash-alert-rules.md)

</NavColumn>

<NavColumn>
<ColumnTitle>Troubleshoot</ColumnTitle>

-   [TiDBトラブルシューティングマップ](/tidb-troubleshooting-map.md)
-   [遅いクエリを特定する](/identify-slow-queries.md)
-   [遅いクエリを分析する](/analyze-slow-queries.md)
-   [SQL診断](/information-schema/information-schema-sql-diagnostics.md)
-   [ホットスポットの問題のトラブルシューティング](/troubleshoot-hot-spot-issues.md)
-   [TiDBクラスターのトラブルシューティング](/troubleshoot-tidb-cluster.md)
-   [TiCDCのトラブルシューティング](/ticdc/troubleshoot-ticdc.md)
-   [TiFlashのトラブルシューティング](/tiflash/troubleshoot-tiflash.md)

</NavColumn>

<NavColumn>
<ColumnTitle>Reference</ColumnTitle>

-   [TiDBアーキテクチャ](/tidb-architecture.md)
-   [主要な監視指標](/grafana-overview-dashboard.md)
-   [TLSを有効にする](/enable-tls-between-clients-and-servers.md)
-   [特権管理](/privilege-management.md)
-   [ロールベースのアクセス制御](/role-based-access-control.md)
-   [証明書ベースの認証](/certificate-authentication.md)

</NavColumn>

<NavColumn>
<ColumnTitle>FAQs</ColumnTitle>

-   [製品に関するよくある質問](/faq/tidb-faq.md)
-   [高可用性に関するFAQ](/faq/high-availability-faq.md)
-   [SQLのFAQ](/faq/sql-faq.md)
-   [FAQの展開と保守](/faq/deploy-and-maintain-faq.md)
-   [アップグレードおよびアップグレード後のFAQ](/faq/upgrade-faq.md)
-   [移行に関するよくある質問](/faq/migration-tidb-faq.md)

</NavColumn>
</NavColumns>
