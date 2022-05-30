---
title: Share TiDB Dashboard Sessions
summary: Learn how to share the current TiDB Dashboard session to other users.
---

# TiDBダッシュボードセッションを共有する {#share-tidb-dashboard-sessions}

TiDBダッシュボードの現在のセッションを他のユーザーと共有して、ユーザーがユーザーパスワードを入力せずにTiDBダッシュボードにアクセスして操作できるようにすることができます。

## 招待者のための手順 {#steps-for-the-inviter}

1.  TiDBダッシュボードにサインインします。

2.  左側のサイドバーのユーザー名をクリックして、構成ページにアクセスします。

3.  [<strong>現在のセッションを共有]</strong>をクリックします。

    ![Sample Step](/media/dashboard/dashboard-session-share-settings-1.png)

    > <strong>ノート：</strong>
    >
    > セキュリティ上の理由から、共有セッションを再度共有することはできません。

4.  ポップアップダイアログで共有設定を調整します。

    -   有効期限：共有セッションが有効になる期間。現在のセッションからサインアウトしても、共有セッションの有効時間には影響しません。

    -   読み取り専用特権として共有：共有セッションでは、読み取り操作のみが許可され、書き込み操作（構成の変更など）は許可されません。

5.  [<strong>認証コードの生成]を</strong>クリックします。

    ![Sample Step](/media/dashboard/dashboard-session-share-settings-2.png)

6.  生成された<strong>認証コード</strong>を、セッションを共有するユーザーに提供します。

    ![Sample Step](/media/dashboard/dashboard-session-share-settings-3.png)

    > <strong>警告：</strong>
    >
    > 承認コードは安全に保管し、信頼できない人には送信しないでください。それ以外の場合は、許可なくTiDBダッシュボードにアクセスして操作することができます。

## 招待者のための手順 {#steps-for-the-invitee}

1.  TiDBダッシュボードのサインインページで、[<strong>代替認証を使用</strong>]をクリックします。

    ![Sample Step](/media/dashboard/dashboard-session-share-signin-1.png)

2.  サインインに使用するには、[<strong>認証コード]</strong>をクリックします。

    ![Sample Step](/media/dashboard/dashboard-session-share-signin-2.png)

3.  招待者から受け取った認証コードを入力します。

4.  [<strong>サインイン]</strong>をクリックします。

    ![Sample Step](/media/dashboard/dashboard-session-share-signin-3.png)
