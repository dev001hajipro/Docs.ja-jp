---
title: ASP.NET および ASP.NET Core とアプリ間で cookie を共有します。
author: rick-anderson
description: ASP.NET 認証 cookie を共有する方法を説明 4.x および ASP.NET Core アプリケーションです。
manager: wpickett
ms.author: riande
ms.custom: mvc
ms.date: 01/19/2017
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: security/cookie-sharing
ms.openlocfilehash: 2c0f5de4ecedb796e85c08fc50d9697947a75a3f
ms.sourcegitcommit: f8852267f463b62d7f975e56bea9aa3f68fbbdeb
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/06/2018
---
# <a name="share-cookies-among-apps-with-aspnet-and-aspnet-core"></a>ASP.NET および ASP.NET Core とアプリ間で cookie を共有します。

によって[Rick Anderson](https://twitter.com/RickAndMSFT)と[Luke Latham](https://github.com/guardrex)

Web サイトは多くの場合、連携して、個々 の web アプリで構成されます。 シングル サインオン (SSO) エクスペリエンスを提供するには、サイト内で web アプリは認証クッキーを共有する必要があります。 このシナリオをサポートするには、データ保護スタックは、共有 Katana cookie 認証および ASP.NET Core cookie 認証チケットを許可します。

[サンプル コードを表示またはダウンロード](https://github.com/aspnet/Docs/tree/master/aspnetcore/security/cookie-sharing/sample/)します ([ダウンロード方法](xref:tutorials/index#how-to-download-a-sample))。

このサンプルは、cookie の cookie 認証を使用する 3 つのアプリ間で共有を示しています。

* ASP.NET Core 2.0 Razor ページのアプリを使用せず[ASP.NET Core Id](xref:security/authentication/identity)
* ASP.NET Core Id を持つ ASP.NET Core 2.0 MVC アプリ
* ASP.NET Identity Framework 4.6.1 の ASP.NET MVC アプリ

次の例。

* 認証 cookie の名前がの共通の値に設定されている`.AspNet.SharedCookie`です。
* `AuthenticationType`に設定されている`Identity.Application`明示的にまたは既定のいずれか。
* 一般的なアプリ名を使用してデータ保護キーを共有するデータ保護システムを有効に (`SharedCookieApp`)。
* `Identity.Application` 認証スキームとして使用されます。 どのような式を使用して、一貫して使用する必要があります*内外*共有 cookie アプリか、または明示的に設定することで、既定のスキームとして。 ためのアプリ間で一貫したスキームを使用する必要があります、cookie を暗号化および暗号化スキームが使用されます。
* 一般的に使われる[データ保護キー](xref:security/data-protection/implementation/key-management)記憶域の場所を使用します。 サンプル アプリで使用するという名前のフォルダー*キーリング*データ保護キーを保持するために、ソリューションのルートに位置します。
* アプリでは、ASP.NET Core、 [PersistKeysToFileSystem](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions.persistkeystofilesystem)キー記憶域の場所を設定するために使用します。 [SetApplicationName](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions.setapplicationname)一般的なアプリの共有名を構成するために使用します。
* Cookie 認証ミドルウェアを .NET Framework アプリでの実装を使用して[DataProtectionProvider](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionprovider)です。 `DataProtectionProvider` 暗号化と認証 cookie のペイロード データの暗号化解除用のデータ保護サービスを提供します。 `DataProtectionProvider`インスタンスは、アプリの他の部分で使用するデータ保護システムから分離します。
  * [DataProtectionProvider.Create(System.IO.DirectoryInfo, Action\<IDataProtectionBuilder>)](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionprovider.create?view=aspnetcore-2.0#Microsoft_AspNetCore_DataProtection_DataProtectionProvider_Create_System_IO_DirectoryInfo_System_Action_Microsoft_AspNetCore_DataProtection_IDataProtectionBuilder__) accepts a [DirectoryInfo](/dotnet/api/system.io.directoryinfo) to specify the location for data protection key storage. サンプル アプリのパスを提供する、*キーリング*フォルダー`DirectoryInfo`です。 [DataProtectionBuilderExtensions.SetApplicationName](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions.setapplicationname?view=aspnetcore-2.0#Microsoft_AspNetCore_DataProtection_DataProtectionBuilderExtensions_SetApplicationName_Microsoft_AspNetCore_DataProtection_IDataProtectionBuilder_System_String_)共通のアプリ名を設定します。
  * [DataProtectionProvider](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionprovider)が必要です、 [Microsoft.AspNetCore.DataProtection.Extensions](https://www.nuget.org/packages/Microsoft.AspNetCore.DataProtection.Extensions/) NuGet パッケージです。 このパッケージの ASP.NET Core 2.0 と以降のアプリの入手、参照、 [Microsoft.AspNetCore.All](xref:fundamentals/metapackage) metapackage です。 .NET Framework を対象とする場合にパッケージ参照の追加`Microsoft.AspNetCore.DataProtection.Extensions`です。

## <a name="share-authentication-cookies-among-aspnet-core-apps"></a>ASP.NET Core アプリケーション間での認証クッキーを共有します。

ASP.NET Core の Id を使用する: 場合

#### <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x/)
`ConfigureServices`メソッドを使用して、 [ConfigureApplicationCookie](/dotnet/api/microsoft.extensions.dependencyinjection.identityservicecollectionextensions.configureapplicationcookie) cookie のデータ保護サービスをセットアップする拡張メソッド。

[!code-csharp[](cookie-sharing/sample/CookieAuthWithIdentity.Core/Startup.cs?name=snippet1)]

データ保護キーとアプリ名は、アプリ間で共有する必要があります。 アプリでは、サンプル、`GetKeyRingDirInfo`に共通のキー記憶域の場所を返します、 [PersistKeysToFileSystem](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions.persistkeystofilesystem)メソッドです。 使用して[SetApplicationName](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions.setapplicationname)一般的なアプリの共有名を構成する (`SharedCookieApp`サンプル)。 詳細については、次を参照してください。[データ保護を構成する](xref:security/data-protection/configuration/overview)です。

参照してください、 *CookieAuthWithIdentity.Core*プロジェクトで、[のサンプル コード](https://github.com/aspnet/Docs/tree/master/aspnetcore/security/cookie-sharing/sample/)([をダウンロードする方法](xref:tutorials/index#how-to-download-a-sample))。

#### <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x/)
`Configure`メソッドを使用して、 [CookieAuthenticationOptions](/dotnet/api/microsoft.aspnetcore.builder.cookieauthenticationoptions)を設定します。

* Cookie のデータ保護サービスです。
* `AuthenticationScheme`合わせて ASP.NET 4.x です。

```csharp
app.AddIdentity<ApplicationUser, IdentityRole>(options =>
{
    options.Cookies.ApplicationCookie.AuthenticationScheme = 
        "ApplicationCookie";

    var protectionProvider = 
        DataProtectionProvider.Create(
            new DirectoryInfo(@"PATH_TO_KEY_RING_FOLDER"));

    options.Cookies.ApplicationCookie.DataProtectionProvider = 
        protectionProvider;

    options.Cookies.ApplicationCookie.TicketDataFormat = 
        new TicketDataFormat(protectionProvider.CreateProtector(
            "Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationMiddleware", 
            "Cookies", 
            "v2"));
});
```

* * *
ときに、cookie を直接使用するには。

#### <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x/)
[!code-csharp[](cookie-sharing/sample/CookieAuth.Core/Startup.cs?name=snippet1)]

データ保護キーとアプリ名は、アプリ間で共有する必要があります。 アプリでは、サンプル、`GetKeyRingDirInfo`に共通のキー記憶域の場所を返します、 [PersistKeysToFileSystem](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions.persistkeystofilesystem)メソッドです。 使用して[SetApplicationName](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions.setapplicationname)一般的なアプリの共有名を構成する (`SharedCookieApp`サンプル)。 詳細については、次を参照してください。[データ保護を構成する](xref:security/data-protection/configuration/overview)です。 

参照してください、 *CookieAuth.Core*プロジェクトで、[のサンプル コード](https://github.com/aspnet/Docs/tree/master/aspnetcore/security/cookie-sharing/sample/)([をダウンロードする方法](xref:tutorials/index#how-to-download-a-sample))。

#### <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x/)
```csharp
app.UseCookieAuthentication(new CookieAuthenticationOptions
{
    DataProtectionProvider = 
        DataProtectionProvider.Create(
            new DirectoryInfo(@"PATH_TO_KEY_RING_FOLDER"))
});
```

* * *
## <a name="encrypting-data-protection-keys-at-rest"></a>静止したデータ保護キーの暗号化

運用環境の配置、構成、 `DataProtectionProvider` DPAPI または X509Certificate で残りの部分でキーを暗号化します。 参照してください[キー時の暗号化](xref:security/data-protection/implementation/key-encryption-at-rest)詳細についてはします。

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)

```csharp
services.AddDataProtection()
    .ProtectKeysWithCertificate("thumbprint");
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

```csharp
app.UseCookieAuthentication(new CookieAuthenticationOptions
{
    DataProtectionProvider = DataProtectionProvider.Create(
        new DirectoryInfo(@"PATH_TO_KEY_RING"),
        configure =>
        {
            configure.ProtectKeysWithCertificate("thumbprint");
        })
});
```

---

## <a name="sharing-authentication-cookies-between-aspnet-4x-and-aspnet-core-apps"></a>ASP.NET との間の認証クッキーを共有 4.x および ASP.NET Core アプリケーション

ASP.NET Core cookie 認証ミドルウェアと互換性がある認証 cookie を生成するを Katana cookie 認証ミドルウェアを使用する ASP.NET 4.x アプリを構成することができます。 これにより、サイト全体で滑らかな SSO エクスペリエンスを提供しつつ、段階的な部分、大規模なサイトの個々 のアプリケーションをアップグレードできます。

> [!TIP]
> アプリは、Katana cookie 認証ミドルウェアを使用しているときに呼び出す`UseCookieAuthentication`プロジェクトの*Startup.Auth.cs*ファイル。 ASP.NET 4.x web アプリのプロジェクトでは、Visual Studio 2013 で作成され、後で、既定では、Katana cookie 認証ミドルウェアを使用しています。

> [!NOTE]
> ASP.NET 4.x アプリが .NET Framework 4.5.1 を対象する必要がありますまたはそれ以降。 それ以外の場合、必要な NuGet パッケージは、インストールに失敗します。

ASP.NET 4.x 用アプリと ASP.NET Core の間での認証クッキーを共有するには、前述のように ASP.NET Core アプリケーションを構成するし、以下の手順に従って、ASP.NET 4.x アプリを構成します。

1. パッケージをインストール[Microsoft.Owin.Security.Interop](https://www.nuget.org/packages/Microsoft.Owin.Security.Interop/)各 ASP.NET 4.x アプリにします。

2. *Startup.Auth.cs*への呼び出しを見つける`UseCookieAuthentication`し、次のように変更します。 ASP.NET Core cookie 認証ミドルウェアで使用される名前と一致する cookie の名前を変更します。 インスタンスを提供する`DataProtectionProvider`一般的なデータ保護キー記憶域の場所を初期化します。 アプリ名が、cookie を共有しているすべてのアプリで使用される共通のアプリ名 に設定されていることを確認してください`SharedCookieApp`サンプル アプリでします。

#### <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x/)
[!code-csharp[](cookie-sharing/sample/CookieAuthWithIdentity.NETFramework/CookieAuthWithIdentity.NETFramework/App_Start/Startup.Auth.cs?name=snippet1)]

参照してください、 *CookieAuthWithIdentity.NETFramework*プロジェクトで、[のサンプル コード](https://github.com/aspnet/Docs/tree/master/aspnetcore/security/cookie-sharing/sample/)([をダウンロードする方法](xref:tutorials/index#how-to-download-a-sample))。

ユーザー id を生成するには、認証の種類で定義された型と一致しなければならない`AuthenticationType`で設定された`UseCookieAuthentication`です。

*Models/IdentityModels.cs*:

[!code-csharp[](cookie-sharing/sample/CookieAuthWithIdentity.NETFramework/CookieAuthWithIdentity.NETFramework/Models/IdentityModels.cs?name=snippet1)]

#### <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x/)
設定、`CookieManager`相互運用`ChunkingCookieManager`チャンク形式は互換性のあるようにします。

```csharp
app.UseCookieAuthentication(new CookieAuthenticationOptions
{
    AuthenticationType = DefaultAuthenticationTypes.ApplicationCookie,
    CookieName = ".AspNetCore.Cookies",
    // CookieName = ".AspNetCore.ApplicationCookie", (if using ASP.NET Identity)
    // CookiePath = "...", (if necessary)
    // ...
    TicketDataFormat = new AspNetTicketDataFormat(
        new DataProtectorShim(
            DataProtectionProvider.Create(
                new DirectoryInfo(@"PATH_TO_KEY_RING_FOLDER"))
            .CreateProtector(
                "Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationMiddleware",
                "Cookies", 
                "v2"))),
    CookieManager = new ChunkingCookieManager()
});
```

* * *
## <a name="use-a-common-user-database"></a>一般的なユーザー データベースを使用します。

各アプリケーションの id システムは、同じユーザー データベースを指すことを確認します。 それ以外の場合、id システムでは、そのデータベース内の情報に対する認証クッキーの情報と一致しようとしたときに実行時にエラーが生成されます。
