---
title: tiup cluster list
---

# tiupクラスターリスト {#tiup-cluster-list}

tiup-clusterは、同じ制御マシンを使用した複数のクラスターのデプロイをサポートします。 `tiup cluster list`コマンドは、このコントロールマシンを使用して現在ログインしているユーザーによって展開されたすべてのクラスターを出力します。

> <strong>ノート：</strong>
>
> デプロイされたクラスターデータはデフォルトで`~/.tiup/storage/cluster/clusters/`ディレクトリに保存されるため、同じコントロールマシン上で、現在ログインしているユーザーは他のユーザーによってデプロイされたクラスターを表示できません。

## 構文 {#syntax}

```shell
tiup cluster list [flags]
```

## オプション {#options}

### -h、-help {#h-help}

-   ヘルプ情報を出力します。
-   データ型： `BOOLEAN`
-   このオプションはデフォルトで無効になっており、デフォルト値は`false`です。このオプションを有効にするには、このオプションをコマンドに追加して、 `true`の値を渡すか、値を渡さないようにします。

## 出力 {#outputs}

次のフィールドを含むテーブルを出力します。

-   名前：クラスター名
-   ユーザー：デプロイメントユーザー
-   バージョン：クラスターバージョン
-   パス：コントロールマシン上のクラスター展開データのパス
-   PrivateKey：クラスターの接続に使用される秘密鍵のパス

[&lt;&lt;前のページに戻る-TiUPClusterコマンドリスト](/tiup/tiup-component-cluster.md#command-list)
