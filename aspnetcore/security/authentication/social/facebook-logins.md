---
title: "ASP.NET Core で Facebook 外部ログインのセットアップ"
author: rick-anderson
description: "ASP.NET Core で Facebook 外部ログインのセットアップ"
keywords: ASP.NET Core
ms.author: riande
manager: wpickett
ms.date: 8/1/2017
ms.topic: article
ms.assetid: 8c65179b-688c-4af1-8f5e-1862920cda95
ms.technology: aspnet
ms.prod: asp.net-core
uid: security/authentication/facebook-logins
ms.openlocfilehash: 9554d66712f93df6d2c50503b60162757986e707
ms.sourcegitcommit: 74e22e08e3b08cb576e5184d16f4af5656c13c0c
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/25/2017
---
# <a name="configuring-facebook-authentication"></a><span data-ttu-id="55f1f-104">Facebook 認証を構成します。</span><span class="sxs-lookup"><span data-stu-id="55f1f-104">Configuring Facebook authentication</span></span>

<a name=security-authentication-facebook-logins></a>

<span data-ttu-id="55f1f-105">によって[Valeriy Novytskyy](https://github.com/01binary)と[Rick Anderson](https://twitter.com/RickAndMSFT)</span><span class="sxs-lookup"><span data-stu-id="55f1f-105">By [Valeriy Novytskyy](https://github.com/01binary) and [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

<span data-ttu-id="55f1f-106">このチュートリアルで作成されたサンプルの ASP.NET Core 2.0 プロジェクトを使用して自分の Facebook アカウントでサインインするユーザーを有効にする方法を示します、[前のページ](index.md)です。</span><span class="sxs-lookup"><span data-stu-id="55f1f-106">This tutorial shows you how to enable your users to sign in with their Facebook account using a sample ASP.NET Core 2.0 project created on the [previous page](index.md).</span></span> <span data-ttu-id="55f1f-107">まず、次の Facebook アプリケーションの ID を作成することで、[公式手順](https://developers.facebook.com/docs/apps/register)です。</span><span class="sxs-lookup"><span data-stu-id="55f1f-107">We start by creating a Facebook App ID by following the [official steps](https://developers.facebook.com/docs/apps/register).</span></span>

## <a name="create-the-app-in-facebook"></a><span data-ttu-id="55f1f-108">Facebook でのアプリを作成します。</span><span class="sxs-lookup"><span data-stu-id="55f1f-108">Create the app in Facebook</span></span>

*  <span data-ttu-id="55f1f-109">移動し、[開発者のための Facebook](https://developers.facebook.com/apps)ページし、サインインします。</span><span class="sxs-lookup"><span data-stu-id="55f1f-109">Navigate to the [Facebook for Developers](https://developers.facebook.com/apps) page and sign in.</span></span> <span data-ttu-id="55f1f-110">既に Facebook アカウントを持っていない場合は使用して、 **Facebook にサインアップする**作成するのにはログイン ページにリンクします。</span><span class="sxs-lookup"><span data-stu-id="55f1f-110">If you don't already have a Facebook account, use the **Sign up for Facebook** link on the login page to create one.</span></span>

* <span data-ttu-id="55f1f-111">タップして、**のアプリの作成**新しいアプリ ID を作成する右上隅のボタン</span><span class="sxs-lookup"><span data-stu-id="55f1f-111">Tap the **Create App** button in the upper right corner to create a new App ID.</span></span>

   ![Microsoft Edge で Facebook 開発者ポータルを開く](index/_static/FBMyApps.png)

* <span data-ttu-id="55f1f-113">フォームに入力し、タップ、**アプリ ID の作成**ボタンをクリックします。</span><span class="sxs-lookup"><span data-stu-id="55f1f-113">Fill out the form and tap the **Create App ID** button.</span></span>

   ![アプリ ID を新しいフォームを作成します。](index/_static/FBNewAppId.png)

* <span data-ttu-id="55f1f-115">表示**製品を選択**プロンプト をクリック**設定**上、 **Facebook ログイン**カードです。</span><span class="sxs-lookup"><span data-stu-id="55f1f-115">When presented with **Select a product** prompt, Click **Set Up** on the **Facebook Login** card.</span></span>

   ![製品のセットアップ ページ](index/_static/FBProductSetup.png)

* <span data-ttu-id="55f1f-117">**クイック スタート**とウィザードが起動**プラットフォームを選択して**最初のページとして。</span><span class="sxs-lookup"><span data-stu-id="55f1f-117">The **Quickstart** wizard will launch with **Choose a Platform** as the first page.</span></span> <span data-ttu-id="55f1f-118">クリックして、ここでは、ウィザードをバイパス、**設定**左側のメニュー内のリンク。</span><span class="sxs-lookup"><span data-stu-id="55f1f-118">Bypass the wizard for now by clicking the **Settings** link in the menu on the left:</span></span>

   ![Skip のクイック スタート](index/_static/FBSkipQuickStart.png)

* <span data-ttu-id="55f1f-120">表示され、**クライアント OAuth 設定**ページ。</span><span class="sxs-lookup"><span data-stu-id="55f1f-120">You are presented with the **Client OAuth Settings** page:</span></span>

![クライアントの OAuth 設定 ページ](index/_static/FBOAuthSetup.png)

* <span data-ttu-id="55f1f-122">開発 URI を入力と*/signin-facebook*に追加されます、**有効な OAuth リダイレクト Uri**フィールド (例: `https://localhost:44320/signin-facebook`)。</span><span class="sxs-lookup"><span data-stu-id="55f1f-122">Enter your development URI with */signin-facebook* appended into the **Valid OAuth Redirect URIs** field (for example: `https://localhost:44320/signin-facebook`).</span></span> <span data-ttu-id="55f1f-123">このチュートリアルで後で構成されている Facebook 認証はで、要求を自動的に処理*/signin-facebook* OAuth フローを実装するルート。</span><span class="sxs-lookup"><span data-stu-id="55f1f-123">The Facebook authentication configured later in this tutorial will automatically handle requests at */signin-facebook* route to implement the OAuth flow.</span></span>

* <span data-ttu-id="55f1f-124">をクリックして**変更を保存**です。</span><span class="sxs-lookup"><span data-stu-id="55f1f-124">Click **Save Changes**.</span></span>

* <span data-ttu-id="55f1f-125">クリックして、**ダッシュ ボード**左側のナビゲーション リンク。</span><span class="sxs-lookup"><span data-stu-id="55f1f-125">Click the **Dashboard** link in the left navigation.</span></span> 

    <span data-ttu-id="55f1f-126">このページで、書き留めて、`App ID`と`App Secret`です。</span><span class="sxs-lookup"><span data-stu-id="55f1f-126">On this page, make a note of your `App ID` and your `App Secret`.</span></span> <span data-ttu-id="55f1f-127">次のセクションでは、ASP.NET Core アプリケーションの両方を追加します。</span><span class="sxs-lookup"><span data-stu-id="55f1f-127">You will add both into your ASP.NET Core application in the next section:</span></span>

   ![Facebook 開発者向けダッシュ ボード](index/_static/FBDashboard.png)

* <span data-ttu-id="55f1f-129">サイトを展開するときに再表示する必要があります、 **Facebook ログイン**セットアップ ページと、新しいパブリック URI を登録します。</span><span class="sxs-lookup"><span data-stu-id="55f1f-129">When deploying the site you need to revisit the **Facebook Login** setup page and register a new public URI.</span></span>

## <a name="store-facebook-app-id-and-app-secret"></a><span data-ttu-id="55f1f-130">Facebook アプリケーションの ID とアプリのシークレットを格納します。</span><span class="sxs-lookup"><span data-stu-id="55f1f-130">Store Facebook App ID and App Secret</span></span>

<span data-ttu-id="55f1f-131">Facebook などの機密設定をリンク`App ID`と`App Secret`、アプリケーションを使用して構成する、[シークレット Manager](xref:security/app-secrets)です。</span><span class="sxs-lookup"><span data-stu-id="55f1f-131">Link sensitive settings like Facebook `App ID` and `App Secret` to your application configuration using the [Secret Manager](xref:security/app-secrets).</span></span> <span data-ttu-id="55f1f-132">このチュートリアルの目的で、名前トークン`Authentication:Facebook:AppId`と`Authentication:Facebook:AppSecret`です。</span><span class="sxs-lookup"><span data-stu-id="55f1f-132">For the purposes of this tutorial, name the tokens `Authentication:Facebook:AppId` and `Authentication:Facebook:AppSecret`.</span></span>

## <a name="configure-facebook-authentication"></a><span data-ttu-id="55f1f-133">Facebook 認証を構成します。</span><span class="sxs-lookup"><span data-stu-id="55f1f-133">Configure Facebook Authentication</span></span>

<span data-ttu-id="55f1f-134">このチュートリアルで使用されるプロジェクト テンプレートにより[Microsoft.AspNetCore.Authentication.Facebook](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Facebook)パッケージが既にインストールされています。</span><span class="sxs-lookup"><span data-stu-id="55f1f-134">The project template used in this tutorial ensures that [Microsoft.AspNetCore.Authentication.Facebook](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Facebook) package is already installed.</span></span>

* <span data-ttu-id="55f1f-135">Visual Studio 2017 でこのパッケージをインストールするには、クリックし、プロジェクトを右クリックし**NuGet パッケージの管理**です。</span><span class="sxs-lookup"><span data-stu-id="55f1f-135">To install this package with Visual Studio 2017, right-click on the project and select **Manage NuGet Packages**.</span></span>
* <span data-ttu-id="55f1f-136">.NET Core cli をインストールするには、プロジェクト ディレクトリに、次を実行します。</span><span class="sxs-lookup"><span data-stu-id="55f1f-136">To install with .NET Core CLI, execute the following in your project directory:</span></span>

   `dotnet add package Microsoft.AspNetCore.Authentication.Facebook`

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="55f1f-137">ASP.NET 2.x のコア</span><span class="sxs-lookup"><span data-stu-id="55f1f-137">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

<span data-ttu-id="55f1f-138">Facebook のサービスを追加、`ConfigureServices`メソッドで、 *Startup.cs*ファイル。</span><span class="sxs-lookup"><span data-stu-id="55f1f-138">Add the Facebook service in the `ConfigureServices` method in the *Startup.cs* file:</span></span>

```csharp
services.AddAuthentication().AddFacebook(facebookOptions =>
{
    facebookOptions.AppId = Configuration["Authentication:Facebook:AppId"];
    facebookOptions.AppSecret = Configuration["Authentication:Facebook:AppSecret"];
});
```

<span data-ttu-id="55f1f-139">`AddAuthentication`メソッドのみ呼び出されます一度に複数の認証プロバイダーを追加する場合にします。</span><span class="sxs-lookup"><span data-stu-id="55f1f-139">The `AddAuthentication` method should only be called once when adding multiple authentication providers.</span></span> <span data-ttu-id="55f1f-140">後続の呼び出しのいずれかの以前に構成をオーバーライドする可能性のある[AuthenticationOptions](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.builder.authenticationoptions)プロパティです。</span><span class="sxs-lookup"><span data-stu-id="55f1f-140">Subsequent calls to it have the potential of overriding any previously configured [AuthenticationOptions](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.builder.authenticationoptions) properties.</span></span>

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="55f1f-141">ASP.NET Core 1.x</span><span class="sxs-lookup"><span data-stu-id="55f1f-141">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

<span data-ttu-id="55f1f-142">Facebook ミドルウェア内の追加、`Configure`メソッド*Startup.cs*ファイル。</span><span class="sxs-lookup"><span data-stu-id="55f1f-142">Add the Facebook middleware in the `Configure` method in *Startup.cs* file:</span></span>

```csharp
app.UseFacebookAuthentication(new FacebookOptions()
{
    AppId = Configuration["Authentication:Facebook:AppId"],
    AppSecret = Configuration["Authentication:Facebook:AppSecret"]
});
```

---

<span data-ttu-id="55f1f-143">参照してください、 [FacebookOptions](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.builder.facebookoptions) Facebook 認証でサポートされる構成オプションの詳細についての API リファレンスです。</span><span class="sxs-lookup"><span data-stu-id="55f1f-143">See the [FacebookOptions](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.builder.facebookoptions) API reference for more information on configuration options supported by Facebook authentication.</span></span> <span data-ttu-id="55f1f-144">構成オプションを使用できます。</span><span class="sxs-lookup"><span data-stu-id="55f1f-144">Configuration options can be used to:</span></span>

* <span data-ttu-id="55f1f-145">ユーザーに関するさまざまな情報を要求します。</span><span class="sxs-lookup"><span data-stu-id="55f1f-145">Request different information about the user.</span></span>
* <span data-ttu-id="55f1f-146">ログイン エクスペリエンスをカスタマイズするクエリ文字列引数を追加します。</span><span class="sxs-lookup"><span data-stu-id="55f1f-146">Add query string arguments to customize the login experience.</span></span>

## <a name="sign-in-with-facebook"></a><span data-ttu-id="55f1f-147">Facebook でサインイン</span><span class="sxs-lookup"><span data-stu-id="55f1f-147">Sign in with Facebook</span></span>

<span data-ttu-id="55f1f-148">アプリケーションを実行し、をクリックして**ログイン**です。</span><span class="sxs-lookup"><span data-stu-id="55f1f-148">Run your application and click **Log in**.</span></span> <span data-ttu-id="55f1f-149">Facebook でサインインするオプションが表示されます。</span><span class="sxs-lookup"><span data-stu-id="55f1f-149">You see an option to sign in with Facebook.</span></span>

![Web アプリケーション: ユーザーが認証されません](index/_static/DoneFacebook.png)

<span data-ttu-id="55f1f-151">クリックすると**Facebook**認証に Facebook にリダイレクトされます。</span><span class="sxs-lookup"><span data-stu-id="55f1f-151">When you click on **Facebook**, you are redirected to Facebook for authentication:</span></span>

![Facebook の認証 ページ](index/_static/FBLogin.png)

<span data-ttu-id="55f1f-153">Facebook の認証は、既定でパブリック プロファイルと電子メール アドレスを要求します。</span><span class="sxs-lookup"><span data-stu-id="55f1f-153">Facebook authentication requests public profile and email address by default:</span></span>

![Facebook の認証 ページ](index/_static/FBLoginDone.png)

<span data-ttu-id="55f1f-155">Facebook 資格情報を入力すると、電子メールを設定することができますのサイトにリダイレクトされます。</span><span class="sxs-lookup"><span data-stu-id="55f1f-155">Once you enter your Facebook credentials you are redirected back to your site where you can set your email.</span></span>

<span data-ttu-id="55f1f-156">これで、Facebook の資格情報を使用してをログインしています。</span><span class="sxs-lookup"><span data-stu-id="55f1f-156">You are now logged in using your Facebook credentials:</span></span>

![Web アプリケーション: 認証されたユーザー](index/_static/Done.png)

## <a name="troubleshooting"></a><span data-ttu-id="55f1f-158">トラブルシューティング</span><span class="sxs-lookup"><span data-stu-id="55f1f-158">Troubleshooting</span></span>

* <span data-ttu-id="55f1f-159">**ASP.NET Core 2.x のみ:**場合の Id が呼び出すことによって構成されていない`services.AddIdentity`で`ConfigureServices`、認証を試みるが*ArgumentException: 'SignInScheme' オプションを指定する必要があります*です。</span><span class="sxs-lookup"><span data-stu-id="55f1f-159">**ASP.NET Core 2.x only:** If Identity is not configured by calling `services.AddIdentity` in `ConfigureServices`, attempting to authenticate will result in *ArgumentException: The 'SignInScheme' option must be provided*.</span></span> <span data-ttu-id="55f1f-160">このチュートリアルで使用されるプロジェクト テンプレートは、これが行われるようにします。</span><span class="sxs-lookup"><span data-stu-id="55f1f-160">The project template used in this tutorial ensures that this is done.</span></span>
* <span data-ttu-id="55f1f-161">取得する場合は、初期の移行を適用することで、サイト データベースが作成されていない、*要求の処理中にデータベース操作が失敗しました*エラーです。</span><span class="sxs-lookup"><span data-stu-id="55f1f-161">If the site database has not been created by applying the initial migration, you get *A database operation failed while processing the request* error.</span></span> <span data-ttu-id="55f1f-162">タップ**適用移行**データベースを作成し、エラーを越えて続行を更新します。</span><span class="sxs-lookup"><span data-stu-id="55f1f-162">Tap **Apply Migrations** to create the database and refresh to continue past the error.</span></span>

## <a name="next-steps"></a><span data-ttu-id="55f1f-163">次のステップ</span><span class="sxs-lookup"><span data-stu-id="55f1f-163">Next steps</span></span>

* <span data-ttu-id="55f1f-164">この記事では、facebook の認証方法を示しました。</span><span class="sxs-lookup"><span data-stu-id="55f1f-164">This article showed how you can authenticate with Facebook.</span></span> <span data-ttu-id="55f1f-165">記載されているその他のプロバイダーでの認証に同様のアプローチを行うことができる、[前のページ](index.md)です。</span><span class="sxs-lookup"><span data-stu-id="55f1f-165">You can follow a similar approach to authenticate with other providers listed on the [previous page](index.md).</span></span>

* <span data-ttu-id="55f1f-166">リセットする必要があります、web サイトを Azure web アプリに発行した後、 `AppSecret` Facebook 開発者ポータルにします。</span><span class="sxs-lookup"><span data-stu-id="55f1f-166">Once you publish your web site to Azure web app, you should reset the `AppSecret` in the Facebook developer portal.</span></span>

* <span data-ttu-id="55f1f-167">設定、`Authentication:Facebook:AppId`と`Authentication:Facebook:AppSecret`として Azure ポータルでのアプリケーション設定。</span><span class="sxs-lookup"><span data-stu-id="55f1f-167">Set the `Authentication:Facebook:AppId` and `Authentication:Facebook:AppSecret` as application settings in the Azure portal.</span></span> <span data-ttu-id="55f1f-168">構成システムは、環境変数からキーを読み取れませんを設定します。</span><span class="sxs-lookup"><span data-stu-id="55f1f-168">The configuration system is set up to read keys from environment variables.</span></span>