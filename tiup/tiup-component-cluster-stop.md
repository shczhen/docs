---
title: tiup cluster stop
---

# tiupクラスター停止 {#tiup-cluster-stop}

`tiup cluster stop`コマンドは、指定されたクラスターのすべてのサービスまたは一部のサービスを停止するために使用されます。

> <strong>ノート：</strong>
>
> クラスタのコアサービスが停止すると、クラスタはサービスを提供できなくなります。

## 構文 {#syntax}

```shell
tiup cluster stop <cluster-name> [flags]
```

`<cluster-name>`は、操作するクラスターの名前です。クラスタ名を忘れた場合は、 [`tiup cluster list`](/tiup/tiup-component-cluster-list.md)コマンドで確認できます。

## オプション {#options}

### -N、-node {#n-node}

-   停止するノードを指定します。このオプションの値は、ノードIDのコンマ区切りのリストです。 `tiup cluster display`コマンドによって返された[クラスターステータステーブル](/tiup/tiup-component-cluster-display.md)の最初の列からノードIDを取得できます。
-   データ型： `STRINGS`
-   このオプションがコマンドで指定されていない場合、コマンドはデフォルトですべてのノードを停止します。

> <strong>ノート：</strong>
>
> `-R, --role`オプションを同時に指定すると、 `-N, --node`と`-R, --role`の両方の仕様に一致するサービスノードのみが停止します。

### -R、-role {#r-role}

-   停止するノードの役割を指定します。このオプションの値は、ノードの役割のコンマ区切りのリストです。 `tiup cluster display`コマンドによって返される[クラスターステータステーブル](/tiup/tiup-component-cluster-display.md)の2番目の列からノードの役割を取得できます。
-   データ型： `STRINGS`
-   このオプションがコマンドで指定されていない場合、コマンドはデフォルトですべての役割を停止します。

> <strong>ノート：</strong>
>
> `-N, --node`オプションを同時に指定すると、 `-N, --node`と`-R, --role`の両方の仕様に一致するサービスノードのみが停止します。

### -h、-help {#h-help}

-   ヘルプ情報を出力します。
-   データ型： `BOOLEAN`
-   このオプションは、デフォルトで`false`の値で無効になっています。このオプションを有効にするには、このオプションをコマンドに追加し、 `true`の値を渡すか、値を渡さないようにします。

## 出力 {#output}

サービス停止のログ。

[&lt;&lt;前のページに戻る-TiUPClusterコマンドリスト](/tiup/tiup-component-cluster.md#command-list)
