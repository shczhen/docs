---
title: Configure SSO for TiDB Dashboard
summary: Learn how to enable SSO to sign into TiDB Dashboard.
---

# TiDBダッシュボードのSSOを構成する {#configure-sso-for-tidb-dashboard}

TiDBダッシュボードは、 [OIDC](https://openid.net/connect/)ベースのシングルサインオン（SSO）をサポートします。 TiDBダッシュボードのSSO機能を有効にすると、構成されたSSOサービスがサインイン認証に使用され、SQLユーザーパスワードを入力せずにTiDBダッシュボードにアクセスできるようになります。

## OIDCSSOを構成する {#configure-oidc-sso}

### SSOを有効にする {#enable-sso}

1.  TiDBダッシュボードにサインインします。

2.  左側のサイドバーのユーザー名をクリックして、構成ページにアクセスします。

3.  [<strong>シングルサインオン</strong>]セクションで、[ <strong>TiDBダッシュボードにサインインするときにSSOを使用できるようにする]を</strong>選択します。

4.  フォームの<strong>OIDCクライアントID</strong>と<strong>OIDCディスカバリーURL</strong>フィールドに入力します。

    通常、SSOサービスプロバイダーから2つのフィールドを取得できます。

    -   OIDCクライアントIDは、OIDCトークン発行者とも呼ばれます。
    -   OIDCディスカバリURLは、OIDCトークンオーディエンスとも呼ばれます。

5.  [<strong>偽装の承認]</strong>をクリックして、SQLパスワードを入力します。

    TiDBダッシュボードはこのSQLパスワードを保存し、SSOサインインが完了した後に通常のSQLサインインを偽装するために使用します。

    ![Sample Step](/media/dashboard/dashboard-session-sso-enable-1.png)

    > <strong>ノート：</strong>
    >
    > 入力したパスワードは暗号化されて保存されます。 SQLユーザーのパスワードが変更された後、SSOサインインは失敗します。この場合、パスワードを再入力してSSOを元に戻すことができます。

6.  [<strong>承認して保存]を</strong>クリックします。

    ![Sample Step](/media/dashboard/dashboard-session-sso-enable-2.png)

7.  [<strong>更新</strong>（更新）]をクリックして、構成を保存します。

    ![Sample Step](/media/dashboard/dashboard-session-sso-enable-3.png)

これで、TiDBダッシュボードでSSOサインインが有効になりました。

> <strong>ノート：</strong>
>
> セキュリティ上の理由から、一部のSSOサービスでは、信頼できるサインインおよびサインアウトURIなど、SSOサービスの追加構成が必要です。詳細については、SSOサービスのドキュメントを参照してください。

### SSOを無効にする {#disable-sso}

SSOを無効にすると、保存されているSQLパスワードが完全に消去されます。

1.  TiDBダッシュボードにサインインします。

2.  左側のサイドバーのユーザー名をクリックして、構成ページにアクセスします。

3.  [<strong>シングルサインオン</strong>]セクションで、 <strong>[TiDBダッシュボードにサインインするときにSSOを使用するには[有効にする]]</strong>の選択を解除します。

4.  [<strong>更新</strong>（更新）]をクリックして、構成を保存します。

    ![Sample Step](/media/dashboard/dashboard-session-sso-disable.png)

### パスワード変更後、パスワードを再入力してください {#re-enter-the-password-after-a-password-change}

SQLユーザーのパスワードが変更されると、SSOサインインは失敗します。この場合、SQLパスワードを再入力することにより、SSOサインインを元に戻すことができます。

1.  TiDBダッシュボードにサインインします。

2.  左側のサイドバーのユーザー名をクリックして、構成ページにアクセスします。

3.  [<strong>シングルサインオン</strong>]セクションで、[<strong>偽装の承認]</strong>をクリックし、更新されたSQLパスワードを入力します。

    ![Sample Step](/media/dashboard/dashboard-session-sso-reauthorize.png)

4.  [<strong>承認して保存]を</strong>クリックします。

## SSO経由でサインイン {#sign-in-via-sso}

TiDBダッシュボード用にSSOを構成したら、次の手順を実行してSSO経由でサインインできます。

1.  TiDBダッシュボードのサインインページで、[<strong>会社のアカウントからサインイン</strong>]をクリックします。

    ![Sample Step](/media/dashboard/dashboard-session-sso-signin.png)

2.  SSOサービスが構成されているシステムにサインインします。

3.  サインインを完了するために、TiDBダッシュボードにリダイレクトされます。

## 例1：TiDBダッシュボードのSSOサインインにOktaを使用する {#example-1-use-okta-for-tidb-dashboard-sso-sign-in}

[オクタ](https://www.okta.com/)はOIDCSSOIDサービスであり、TiDBダッシュボードのSSO機能と互換性があります。以下の手順は、OktaをTiDBダッシュボードSSOプロバイダーとして使用できるようにOktaおよびTiDBダッシュボードを構成する方法を示しています。

### ステップ1：Oktaを構成する {#step-1-configure-okta}

まず、SSOを統合するためのOktaアプリケーション統合を作成します。

1.  Okta管理サイトにアクセスします。

2.  左側のサイドバーから[<strong>アプリケーション</strong>]&gt;[<strong>アプリケーション</strong>]に移動します。

3.  [<strong>アプリ統合の作成]</strong>をクリックします。

    ![Sample Step](/media/dashboard/dashboard-session-sso-okta-1.png)

4.  ポップアップダイアログで、[OIDC]-[OpenID ConnectinSign <strong>-</strong> <strong>inmethod</strong> ]を選択します。

5.  <strong>アプリケーションタイプ</strong>で<strong>シングルページアプリケーション</strong>を選択します。

6.  [<strong>次へ</strong>]ボタンをクリックします。

    ![Sample Step](/media/dashboard/dashboard-session-sso-okta-2.png)

7.  <strong>サインインリダイレクトURI</strong>を次のように入力します。

    ```
    http://DASHBOARD_IP:PORT/dashboard/?sso_callback=1
    ```

    `DASHBOARD_IP:PORT`を、ブラウザでTiDBダッシュボードにアクセスするために使用する実際のドメイン（またはIPアドレス）とポートに置き換えます。

8.  <strong>サインアウトリダイレクトURI</strong>を次のように入力します。

    ```
    http://DASHBOARD_IP:PORT/dashboard/
    ```

    同様に、 `DASHBOARD_IP:PORT`を実際のドメイン（またはIPアドレス）とポートに置き換えます。

    ![Sample Step](/media/dashboard/dashboard-session-sso-okta-3.png)

9.  組織内のどのタイプのユーザーにSSOサインインを許可するかを[<strong>割り当て</strong>]フィールドで構成し、[<strong>保存</strong>]をクリックして構成を保存します。

    ![Sample Step](/media/dashboard/dashboard-session-sso-okta-4.png)

### ステップ2：OIDC情報を取得し、TiDBダッシュボードに入力します {#step-2-obtain-oidc-information-and-fill-in-tidb-dashboard}

1.  Oktaで作成したばかりのアプリケーション統合で、[<strong>サインオン</strong>]をクリックします。

    ![Sample Step 1](/media/dashboard/dashboard-session-sso-okta-info-1.png)

2.  <strong>OpenID ConnectIDToken</strong>セクションから<strong>Issuer</strong>フィールドと<strong>Audience</strong>フィールドの値をコピーします。

    ![Sample Step 2](/media/dashboard/dashboard-session-sso-okta-info-2.png)

3.  TiDBダッシュボード構成ページを開き、最後の手順で取得した<strong>発行者</strong>を<strong>OIDCクライアントID</strong>に入力し、 <strong>OIDCディスカバリーURL</strong>に<strong>オーディエンス</strong>を入力します。次に、認証を終了し、構成を保存します。例えば：

    ![Sample Step 3](/media/dashboard/dashboard-session-sso-okta-info-3.png)

これで、TiDBダッシュボードはサインインにOktaSSOを使用するように構成されました。

## 例2：TiDBダッシュボードのSSOサインインにAuth0を使用する {#example-2-use-auth0-for-tidb-dashboard-sso-sign-in}

Oktaと同様に、 [Auth0](https://auth0.com/)もOIDCSSOIDサービスを提供します。次の手順では、Auth0をTiDBダッシュボードSSOプロバイダーとして使用できるようにAuth0とTiDBダッシュボードを構成する方法について説明します。

### ステップ1：Auth0を構成する {#step-1-configure-auth0}

1.  Auth0管理サイトにアクセスします。

2.  左側のサイドバー[<strong>アプリケーション</strong>]&gt;[<strong>アプリケーション</strong>]に移動します。

3.  [<strong>アプリ統合の作成]</strong>をクリックします。

    ![Create Application](/media/dashboard/dashboard-session-sso-auth0-create-app.png)

    ポップアップダイアログで、「<strong>名前</strong>」（「TiDBダッシュボード」など）を入力します。<strong>アプリケーションタイプの</strong>選択で<strong>シングルページWebアプリケーション</strong>を選択します。 [<strong>作成]</strong>をクリックします。

4.  [<strong>設定]</strong>をクリックします。

    ![Settings](/media/dashboard/dashboard-session-sso-auth0-settings-1.png)

5.  <strong>許可されたコールバックURL</strong>を次のように入力します。

    ```
    http://DASHBOARD_IP:PORT/dashboard/?sso_callback=1
    ```

    `DASHBOARD_IP:PORT`を、ブラウザでTiDBダッシュボードにアクセスするために使用する実際のドメイン（またはIPアドレス）とポートに置き換えます。

6.  <strong>許可されたログアウトURL</strong>を次のように入力します。

    ```
    http://DASHBOARD_IP:PORT/dashboard/
    ```

    同様に、 `DASHBOARD_IP:PORT`を実際のドメイン（またはIPアドレス）とポートに置き換えます。

    ![Settings](/media/dashboard/dashboard-session-sso-auth0-settings-2.png)

7.  他の設定のデフォルト値を保持し、[<strong>変更を保存</strong>]をクリックします。

### ステップ2：OIDC情報を取得し、TiDBダッシュボードに入力します {#step-2-obtain-oidc-information-and-fill-in-tidb-dashboard}

1.  Auth0の<strong>[設定</strong>]タブの[<strong>基本情報</strong>]に、TiDBダッシュボードの<strong>OIDCクライアントID</strong>に<strong>クライアント</strong>IDを入力します。

2.  <strong>OIDC Discovery URL</strong>に、プレフィックスが`https://`でサフィックスが`/`の<strong>Domain</strong>フィールド値を入力します（例： `https://example.us.auth0.com/` ）。承認を完了し、構成を保存します。

    ![Settings](/media/dashboard/dashboard-session-sso-auth0-settings-3.png)

これで、TiDBダッシュボードはサインインにAuth0SSOを使用するように構成されました。

## 例3：TiDBダッシュボードのSSOサインインにCasdoorを使用する {#example-3-use-casdoor-for-tidb-dashboard-sso-sign-in}

[キャスドア](https://casdoor.org/)は、独自のホストに展開できるオープンソースのSSOプラットフォームです。 TiDBダッシュボードのSSO機能と互換性があります。次の手順では、CasdoorをTiDBダッシュボードSSOプロバイダーとして使用できるようにCasdoorおよびTiDBダッシュボードを構成する方法について説明します。

### ステップ1：Casdoorを構成する {#step-1-configure-casdoor}

1.  Casdoor管理サイトを展開してアクセスします。

2.  トップサイドバーの<strong>アプリケーション</strong>から移動します。

3.  [<strong>アプリケーション]-[追加]を</strong>クリックします。 ![Settings](/media/dashboard/dashboard-session-sso-casdoor-settings-1.png)

4.  <strong>名前</strong>と<strong>表示名</strong>を入力します（例： <strong>TiDBダッシュボード</strong>）。

5.  次のように<strong>リダイレクトURL</strong>を追加します。

    ```
    http://DASHBOARD_IP:PORT/dashboard/?sso_callback=1
    ```

    `DASHBOARD_IP:PORT`を、ブラウザでTiDBダッシュボードにアクセスするために使用する実際のドメイン（またはIPアドレス）とポートに置き換えます。

    ![Settings](/media/dashboard/dashboard-session-sso-casdoor-settings-2.png)

6.  他の設定のデフォルト値を保持し、[<strong>保存して終了</strong>]をクリックします。

7.  ページに表示されている<strong>クライアントID</strong>を保存します。

### ステップ2：OIDC情報を取得し、TiDBダッシュボードに入力します {#step-2-obtain-oidc-information-and-fill-in-tidb-dashboard}

1.  TiDBダッシュボードの<strong>OIDCクライアントID</strong>に、前の手順で保存した<strong>クライアントID</strong>を入力します。

2.  <strong>OIDC Discovery URL</strong>に、プレフィックスが`https://`でサフィックスが`/`の<strong>Domain</strong>フィールド値を入力します（例： `https://casdoor.example.com/` ）。承認を完了し、構成を保存します。

    ![Settings](/media/dashboard/dashboard-session-sso-casdoor-settings-3.png)

これで、TiDBダッシュボードはサインインにCasdoorSSOを使用するように構成されました。
