---
title: Azure AD v2.0 のスコープ、アクセス許可、および同意 | Microsoft Docs
description: スコープ、アクセス許可、同意など、Azure AD v2.0 エンドポイントでの承認の説明。
services: active-directory
documentationcenter: ''
author: dstrockis
manager: mbaldwin
editor: ''

ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/30/2016
ms.author: dastrock

---
# <a name="scopes,-permissions,-&-consent-in-the-v2.0-endpoint"></a>v2.0 エンドポイントのスコープ、アクセス許可、および同意
Azure AD と統合されるアプリでは、アプリがどのようにデータにアクセスできるかをユーザーが制御できるようにする、特定の承認モデルが使用されます。  この承認モデルの v2.0 実装が更新され、アプリが Azure AD とやり取りする方法が変わりました。  このトピックでは、スコープ、アクセス許可、同意など、この承認モデルの基本的な概念について説明します。

> [!NOTE]
> Azure Active Directory のシナリオおよび機能のすべてが v2.0 エンドポイントでサポートされているわけではありません。  v2.0 エンドポイントを使用する必要があるかどうかを判断するには、 [v2.0 の制限事項](active-directory-v2-limitations.md)に関するページをお読みください。
> 
> 

## <a name="scopes-&-permissions"></a>スコープとアクセス許可
Azure AD は、[OAuth 2.0](active-directory-v2-protocols.md) 承認プロトコルを実装しています。このプロトコルによって、ユーザーに代わってサード パーティのアプリが、Web でホストされるリソースにアクセスできるようになります。  Azure AD と統合される、Web でホストされるすべてのリソースは、リソース識別子、つまり**アプリ ID URI** を持ちます。  Microsoft の Web でホストされるリソースには、次のようなものがあります。

* Office 365 統合メール API: `https://outlook.office.com`
* Azure AD Graph API: `https://graph.windows.net`
* Microsoft Graph: `https://graph.microsoft.com`

Azure AD と統合されたサード パーティのリソースも同様です。  これらのリソースのいずれでも、機能をより小さいまとまりに分割するために使用できるアクセス許可のセットを定義できます。  たとえば、Microsoft Graph では次のようなアクセス許可が定義されています。

* ユーザーの予定表の読み取り
* ユーザーの予定表への書き込み
* ユーザーとしてのメールの送信
* [その他](https://graph.microsoft.io)

これらのアクセス許可を定義することで、リソースはデータを外部にどのように公開するかをきめ細かく制御できます。  サード パーティのアプリは、エンド ユーザーにこれらのアクセス許可を要求します。アプリがユーザーに代わって動作するには、エンド ユーザーがアクセス許可を承認する必要があります。  リソースの機能を細かいアクセス許可セットにまとめることによって、サード パーティ アプリは、機能を実行するために必要な特定のアクセス許可のみを要求するように構築できます。  また、アプリが正確にはどのようにデータを使用するかをエンド ユーザーに知らせ、アプリが悪意のある動作をしないことを確信できるようにすることもできます。

Azure AD と OAuth では、これらのアクセス許可は **スコープ**と呼ばれます。  **oAuth2Permissions**と呼ばれる場合もあります。  スコープは、Azure AD では文字列値として表されます。  Microsoft Graph の例では、各アクセス許可のスコープ値は次のとおりです:

* ユーザーの予定表の読み取り: `Calendar.Read`
* ユーザーの予定表への書き込み: `Mail.ReadWrite`
* ユーザーとしてのメールの送信: `Mail.Send`

アプリは、以下に示すように、v2.0 エンドポイントへの要求でスコープを指定することによって、これらのアクセス許可を要求できます。

## <a name="openid-connect-scopes"></a>OpenID Connect のスコープ
OpenID Connect の v2.0 実装には、特定のリソースには適用されない、適切に定義されたスコープ `openid`、`email`、`profile`、`offline_access` があります。

#### <a name="openid"></a>OpenId
アプリは、[OpenID Connect](active-directory-v2-protocols.md#openid-connect-sign-in-flow) を使用してサインインを行う場合、`openid` スコープを要求する必要があります。  `openid` スコープは、職場アカウントの同意画面では "サインイン" アクセス許可として表示され、個人用 Microsoft アカウントの同意画面では "プロフィールの表示と、Microsoft アカウントを使うアプリとサービスへの接続" アクセス許可として表示されます。  このアクセス許可により、`sub` 要求の形式でユーザーの一意の識別子をアプリが受け取ることができるようになります。  アプリがユーザー情報エンドポイントにアクセスすることも可能にします。  `openid` スコープは、v2.0 トークン エンドポイントで id_tokens を取得するためにも使用できます。このトークンは、アプリ内の異なるコンポーネント間の HTTP 呼び出しをセキュリティで保護するために使用できます。

#### <a name="email"></a>電子メール
`email` スコープは `openid` やその他のスコープに含めることができます。  これにより、アプリがユーザーのプライマリ電子メール アドレスに `email` 要求の形式でアクセスできるようにします。  電子メール アドレスがユーザー アカウントと関連付けられている場合のみ (常にではないが)、 `email` 要求はトークンに含まれます。  `email` スコープを使用する場合、トークンに `email` 要求が存在しないケースにも対応できるよう、アプリを準備する必要があります。

#### <a name="profile"></a>プロファイル
`profile` スコープは `openid` やその他のスコープに含めることができます。  これで、アプリはユーザーのさまざまな情報にアクセスできるようになります。  これらは、ユーザーの名前、姓、推奨ユーザー名、オブジェクト ID などですが、これに限定されるものではありません。  id_tokens で使用できる特定のユーザーに対するプロファイル要求の完全な一覧については、「[v2.0 トークンのリファレンス](active-directory-v2-tokens.md)」を参照してください。

#### <a name="offline_access"></a>Offline_Access
[`offline_access` スコープ](http://openid.net/specs/openid-connect-core-1_0.html#OfflineAccess) を使用すると、アプリはユーザーの代わりに、長期間にわたってリソースにアクセスできます。  このスコープは、職場アカウントの同意画面には、"いつでもデータにアクセス" アクセス許可として表示されます。  個人用 Microsoft アカウントの同意画面には、"いつでも情報にアクセス" アクセス許可として表示されます。  ユーザーが `offline_access` スコープを承認すると、アプリは v2.0 トークン エンドポイントから更新トークンを取得できるようになります。  更新トークンは有効期間が長いので、古いアクセス トークンの有効期限が切れたときに、アプリは新しいアクセス トークンを取得できます。

アプリが `offline_access` スコープを要求しない場合、refresh_tokens を受け取ることはありません。  つまり、[OAuth 2.0 承認コード フロー](active-directory-v2-protocols.md#oauth2-authorization-code-flow)の authorization_code を使用すると、`/token` エンドポイントから access_token だけが取得されます。  その access_token は、短時間 (通常は 1 時間) 有効ですが、最終的には期限が切れます。  その時点で、アプリはユーザーを `/authorize` エンドポイントにリダイレクトして、新しい authorization_code を取得する必要があります。  このリダイレクト中に、アプリの種類によっては、ユーザーが資格情報を再入力したり、アクセス許可に再同意したりする必要がある場合もあります。

更新トークンの取得方法と使用方法の詳細については、「 [v2.0 プロトコルのリファレンス](active-directory-v2-protocols.md)」を参照してください。

## <a name="requesting-individual-user-consent"></a>個々のユーザーの同意を要求する
[OpenID Connect または OAuth 2.0](active-directory-v2-protocols.md) 承認要求では、アプリは `scope` クエリ パラメーターを使用して、必要なアクセス許可を要求できます。  たとえば、ユーザーがアプリにサインインするときに、アプリは次のような要求を送信します (読みやすくするために、改行を入れています)。

```
GET https://login.microsoftonline.com/common/oauth2/v2.0/authorize?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&response_type=code
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
&response_mode=query
&scope=
https%3A%2F%2Fgraph.microsoft.com%2Fcalendar.read%20
https%3A%2F%2Fgraph.microsoft.com%2Fmail.send
&state=12345
```

`scope` パラメーターは、アプリが要求しているスコープの、空白文字で区切られた一覧です。  個々のスコープは、リソースの識別子 (アプリ ID URI) にスコープ値を追加することによって示されます。  上の要求では、ユーザーの予定表を読み取り、ユーザーとしてメールを送信するためのアクセス許可をアプリが必要としていることが示されています。

ユーザーが資格情報を入力すると、v2.0 エンドポイントが **ユーザーの同意**の一致するレコードを確認します。  要求されたアクセス許可にユーザーがまだ同意したことがない場合、v2.0 エンドポイントは要求されたアクセス許可を付与するようにユーザーに求めます。  

![職場アカウントの同意のスクリーン ショット](../media/active-directory-v2-flows/work_account_consent.png)

ユーザーがアクセス許可を承認すると、同意が記録され、ユーザーが以降のサインインで再度同意しなくても済むようになります。

## <a name="requesting-consent-for-an-entire-tenant"></a>テナント全体の同意を要求する
組織がアプリケーションのライセンスまたはサブスクリプションを購入する場合、従業員に対して完全にプロビジョニングします。  このプロセスの一環として、会社の管理者は、従業員の代理としてアプリケーションへの同意を付与できます。  テナント全体への同意を付与することにより、その組織の従業員にはアプリケーションへの同意画面は表示されません。

テナントのユーザー全員に同意を要求するには、次で説明するように、アプリケーションで **管理者の同意エンドポイント**を使用できます。

## <a name="admin-restricted-scopes"></a>管理者によって制限されるスコープ
Microsoft のエコシステムにおける高い権限は、 **管理者によって制限されている**とマークされます。  このようなスコープの例は次のとおりです:

* 組織のディレクトリ データの読み込み: `Directory.Read`
* 組織のディレクトリ データへの書き込み: `Directory.ReadWrite`
* 組織のディレクトリでのセキュリティ グループの読み取り: `Groups.Read.All`

コンシューマー ユーザーがこのようなデータへのアプリケーション アクセスを許可しているとき、組織のユーザーは同一の会社の機密データへのアクセスが制限されます。  アプリケーションが組織のユーザーにアクセス許可を要求する場合は、そのユーザーにはアプリのアクセス許可に同意する権限がないという内容のエラーメッセージが表示されます。

アプリケーションが組織の管理者に制限されるスコープにアクセスする必要がある場合は、次のように **管理者の同意エンドポイント**を使用して、会社の管理者から直接要求する必要があります。

管理者の同意エンドポイントを介して管理者がこれらのアクセス許可を付与すると、前述の通り、テナントのユーザー全員に対して同意が付与されます。

## <a name="using-the-admin-consent-endpoint"></a>管理者の同意エンドポイントを使用する
これから説明する手順では、管理者に制限されているスコープを含む特定のテナントのユーザー全員のアクセス許可をアプリケーションで収集できます。  次の手順を実装するコード サンプルについては [管理者に制限されているスコープのサンプル](https://github.com/Azure-Samples/active-directory-dotnet-admin-restricted-scopes-v2)を参照してください。

#### <a name="request-the-permissions-in-the-app-registration-portal"></a>アプリケーション登録ポータルでアクセス許可を要求する
* [apps.dev.microsoft.com](https://apps.dev.microsoft.com) に移動して使用しているアプリケーションを選択するか、または、アプリを作成していない場合は[アプリを作成](active-directory-v2-app-registration.md)します。
* **Microsoft Graph のアクセス許可** セクションを検索し、アプリに必要なアクセス許可を追加します。
* アプリの登録を **保存** してください

#### <a name="recommended:-sign-the-user-into-your-app"></a>アプリで、ユーザーのサインインを行うことを推奨します
通常、管理者の同意エンドポイントを使用するアプリケーションを構築する場合は、アプリ側に管理者がアプリのアクセス許可を承認するページ/表示が必要となります。  このページは、アプリケーションのサインアップ フロー、アプリの設定、専用の "接続" フローに関する内容を含みます。  多くの場合、職場または学校の Microsoft アカウントでユーザーがサインインした後にのみ、”接続" ビューが表示されます。

ユーザーがアプリケーションにサインインすると、必要なアクセス許可の承認を依頼する前に、管理者が所属する組織を識別できます。  必須ではありませんが、組織のユーザーに向けたより直観的なエクスペリエンスの作成に役立ちます。  ユーザーのサインインを行うには、 [v2.0 プロトコル チュートリアル](active-directory-v2-protocols.md)に従ってください。

#### <a name="request-the-permissions-from-a-directory-admin"></a>ディレクトリ管理者にアクセス許可を要求する
会社の管理者にアクセス許可を要求する準備ができたら、ユーザーを v2.0 **管理者の同意エンドポイント**にリダイレクトできます。

```
// Line breaks for legibility only

GET https://login.microsoftonline.com/{tenant}/adminconsent?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&state=12345
&redirect_uri=http://localhost/myapp/permissions
```

```
// Pro Tip: Try pasting the below request in a browser!
```

```
https://login.microsoftonline.com/common/adminconsent?client_id=6731de76-14a6-49ae-97bc-6eba6914391e&state=12345&redirect_uri=http://localhost/myapp/permissions
```

| パラメーター |  | Description |
| --- | --- | --- |
| テナント |必須 |アクセス許可を要求するディレクトリ テナント。  Guid またはフレンドリ名の形式で指定できます。 |
| client_id |必須 |登録ポータル ([apps.dev.microsoft.com](https://apps.dev.microsoft.com)) によってアプリに割り当てられたアプリケーション ID。 |
| redirect_uri |必須 |処理するアプリのレスポンスの送信先となる redirect_uri。  ポータルで登録した redirect_uri のいずれかと完全に一致させる必要があります。 |
| state |推奨 |要求に含まれ、かつトークンの応答として返される値。  任意の文字列を指定することができます。  この状態は、認証要求の前にアプリ内でユーザーの状態 (表示中のページやビューなど) に関する情報をエンコードする目的に使用されます。 |

現時点で、Azure AD では、テナント管理者のみがサインインして、要求を完了することができます。  管理者はユーザーが登録ポータルで要求したすべてのアクセス許可への承認を求められます。 

##### <a name="successful-response"></a>成功応答
管理者がアプリケーション アクセス許可を承認すると、成功した応答は次のようになります:

```
GET http://localhost/myapp/permissions?tenant=a8990e1f-ff32-408a-9f8e-78d3b9139b95&state=state=12345&admin_consent=True
```

| パラメーター | Description |
| --- | --- | --- |
| テナント |アプリケーションが要求したアクセス許可を Guid 形式で付与するディレクトリ テナント。 |
| state |要求に含まれ、かつトークンの応答として返される値。  任意の文字列を指定することができます。  この状態は、認証要求の前にアプリ内でユーザーの状態 (表示中のページやビューなど) に関する情報をエンコードする目的に使用されます。 |
| admin_consent |`True` に設定されます。 |

##### <a name="error-response"></a>エラー応答
管理者がアプリケーション アクセス許可を承認しない場合、失敗した応答は次のようになります:

```
GET http://localhost/myapp/permissions?error=permission_denied&error_description=The+admin+canceled+the+request
```

| パラメーター | Description |
| --- | --- | --- |
| error |発生したエラーの種類を分類したりエラーに対処したりする際に使用するエラー コード文字列。 |
| error_description |エラーの根本的な原因を開発者が特定しやすいように記述した具体的なエラー メッセージ。 |

管理者の同意エンドポイントから正常な応答を受信した場合、アプリは要求したアクセス許可を獲得しています。  次で説明するように、目的のリソースのトークンの要求に移行できます。

## <a name="using-permissions"></a>アクセス許可の使用
ユーザーがアプリのアクセス許可に同意したら、アプリは一定範囲でリソースにアクセスするためのアプリのアクセス許可を表すアクセス トークンを取得できます。  特定のアクセス トークンは、1 つのリソースだけで使用できますが、その内部には、そのリソースに関してアプリに付与されたすべてのアクセス許可がエンコードされます。  アクセス トークンを取得するために、アプリは v2.0 トークン エンドポイントへの次のような要求を作成できます。

```
POST common/oauth2/v2.0/token HTTP/1.1
Host: https://login.microsoftonline.com
Content-Type: application/json

{
    "grant_type": "authorization_code",
    "client_id": "6731de76-14a6-49ae-97bc-6eba6914391e",
    "scope": "https://outlook.office.com/mail.read https://outlook.office.com/mail.send",
    "code": "AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq..."
    "redirect_uri": "https://localhost/myapp",
    "client_secret": "zc53fwe80980293klaj9823"  // NOTE: Only required for web apps
}
```

結果として得られるアクセス トークンは、リソースへの HTTP 要求で使用され、アプリが特定のタスクを実行するための適切なアクセス許可を持っていることをリソースに示します。  

OAuth 2.0 プロトコルとアクセス トークンの取得方法の詳細については、 [v2.0 エンドポイント プロトコルのリファレンス](active-directory-v2-protocols.md)を参照してください。

<!--HONumber=Oct16_HO2-->


