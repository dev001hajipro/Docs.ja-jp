---
title: "Azure Key Vault の構成プロバイダー"
author: guardrex
description: "Azure キー資格情報コンテナーの構成プロバイダーを使用して、実行時に読み込まれる名前と値のペアを使用してアプリケーションを構成する方法を説明します。"
keywords: "ASP.NET Core、構成では、Azure Key Vault"
ms.author: riande
manager: wpickett
ms.date: 08/09/2017
ms.topic: article
ms.assetid: 0292bdae-b3ed-4637-bd67-19b9bb8b65cb
ms.prod: asp.net-core
uid: security/key-vault-configuration
ms.openlocfilehash: 72b6098b2a71957da338ef36beff4808201773f4
ms.sourcegitcommit: 9cdbfd0d670d70b9c354216aabee260c52dad5ee
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/12/2017
---
# <a name="azure-key-vault-configuration-provider"></a><span data-ttu-id="79c2b-104">Azure Key Vault の構成プロバイダー</span><span class="sxs-lookup"><span data-stu-id="79c2b-104">Azure Key Vault configuration provider</span></span>

<span data-ttu-id="79c2b-105">によって[Luke Latham](https://github.com/GuardRex)と[Andrew スタントン-看護師](https://github.com/anurse)</span><span class="sxs-lookup"><span data-stu-id="79c2b-105">By [Luke Latham](https://github.com/GuardRex) and [Andrew Stanton-Nurse](https://github.com/anurse)</span></span>

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="79c2b-106">ASP.NET Core 2.x</span><span class="sxs-lookup"><span data-stu-id="79c2b-106">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

<span data-ttu-id="79c2b-107">表示または 2.x のサンプル コードをダウンロードします。</span><span class="sxs-lookup"><span data-stu-id="79c2b-107">View or download sample code for 2.x:</span></span>

* <span data-ttu-id="79c2b-108">[基本的なサンプル](https://github.com/aspnet/Docs/tree/master/aspnetcore/security/key-vault-configuration/samples/basic-sample/2.x)-をアプリに秘密の値を読み取ります。</span><span class="sxs-lookup"><span data-stu-id="79c2b-108">[Basic sample](https://github.com/aspnet/Docs/tree/master/aspnetcore/security/key-vault-configuration/samples/basic-sample/2.x) - Reads secret values into an app.</span></span>
* <span data-ttu-id="79c2b-109">[キー名のプレフィックス サンプル](https://github.com/aspnet/Docs/tree/master/aspnetcore/security/key-vault-configuration/samples/key-name-prefix-sample/2.x)- 読み取り秘密の値を別の各アプリのバージョンのシークレットの値セットを読み込むことができるアプリのバージョンを表すキー名のプレフィックスを使用します。</span><span class="sxs-lookup"><span data-stu-id="79c2b-109">[Key name prefix sample](https://github.com/aspnet/Docs/tree/master/aspnetcore/security/key-vault-configuration/samples/key-name-prefix-sample/2.x) - Reads secret values using a key name prefix that represents the version of an app, which allows you to load a different set of secret values for each app version.</span></span>

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="79c2b-110">ASP.NET Core 1.x</span><span class="sxs-lookup"><span data-stu-id="79c2b-110">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

<span data-ttu-id="79c2b-111">表示または 1.x のサンプル コードをダウンロードします。</span><span class="sxs-lookup"><span data-stu-id="79c2b-111">View or download sample code for 1.x:</span></span>

* <span data-ttu-id="79c2b-112">[基本的なサンプル](https://github.com/aspnet/Docs/tree/master/aspnetcore/security/key-vault-configuration/samples/basic-sample/1.x)-をアプリに秘密の値を読み取ります。</span><span class="sxs-lookup"><span data-stu-id="79c2b-112">[Basic sample](https://github.com/aspnet/Docs/tree/master/aspnetcore/security/key-vault-configuration/samples/basic-sample/1.x) - Reads secret values into an app.</span></span>
* <span data-ttu-id="79c2b-113">[キー名のプレフィックス サンプル](https://github.com/aspnet/Docs/tree/master/aspnetcore/security/key-vault-configuration/samples/key-name-prefix-sample/1.x)- 読み取り秘密の値を別の各アプリのバージョンのシークレットの値セットを読み込むことができるアプリのバージョンを表すキー名のプレフィックスを使用します。</span><span class="sxs-lookup"><span data-stu-id="79c2b-113">[Key name prefix sample](https://github.com/aspnet/Docs/tree/master/aspnetcore/security/key-vault-configuration/samples/key-name-prefix-sample/1.x) - Reads secret values using a key name prefix that represents the version of an app, which allows you to load a different set of secret values for each app version.</span></span> 

---

<span data-ttu-id="79c2b-114">このドキュメントの使用方法を説明します、 [Microsoft Azure Key Vault](https://azure.microsoft.com/services/key-vault/)構成プロバイダーを Azure Key Vault シークレットからアプリケーションの構成値を読み込めません。</span><span class="sxs-lookup"><span data-stu-id="79c2b-114">This document explains how to use the [Microsoft Azure Key Vault](https://azure.microsoft.com/services/key-vault/) configuration provider to load application configuration values from Azure Key Vault secrets.</span></span> <span data-ttu-id="79c2b-115">Azure Key Vault は、クラウド ベース サービス暗号化キーやアプリやサービスによって使用されるシークレットを保護するのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="79c2b-115">Azure Key Vault is a cloud-based service that helps you safeguard cryptographic keys and secrets used by apps and services.</span></span> <span data-ttu-id="79c2b-116">一般的なシナリオは、機密性の高い構成データにアクセスを制御して、FIPS 140-2 の要件を満たすレベル 2 検証ハードウェア セキュリティ モジュール (HSM) の構成データを格納する場合。</span><span class="sxs-lookup"><span data-stu-id="79c2b-116">Common scenarios include controlling access to sensitive configuration data and meeting the requirement for FIPS 140-2 Level 2 validated Hardware Security Modules (HSM's) when storing configuration data.</span></span> <span data-ttu-id="79c2b-117">この機能は、以降の ASP.NET Core 1.1 を対象とするアプリケーションで使用可能です。</span><span class="sxs-lookup"><span data-stu-id="79c2b-117">This feature is available for applications that target ASP.NET Core 1.1 or higher.</span></span>

## <a name="package"></a><span data-ttu-id="79c2b-118">Package</span><span class="sxs-lookup"><span data-stu-id="79c2b-118">Package</span></span>
<span data-ttu-id="79c2b-119">使用するには、プロバイダーへの参照を追加、 [Microsoft.Extensions.Configuration.AzureKeyVault](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.AzureKeyVault/)パッケージです。</span><span class="sxs-lookup"><span data-stu-id="79c2b-119">To use the provider, add a reference to the [Microsoft.Extensions.Configuration.AzureKeyVault](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.AzureKeyVault/) package.</span></span>

## <a name="application-configuration"></a><span data-ttu-id="79c2b-120">アプリケーションの構成</span><span class="sxs-lookup"><span data-stu-id="79c2b-120">Application configuration</span></span>
<span data-ttu-id="79c2b-121">使用してプロバイダーを調べることができます、[アプリのサンプル](https://github.com/aspnet/Docs/tree/master/aspnetcore/security/key-vault-configuration/samples)です。</span><span class="sxs-lookup"><span data-stu-id="79c2b-121">You can explore the provider with the [sample apps](https://github.com/aspnet/Docs/tree/master/aspnetcore/security/key-vault-configuration/samples).</span></span> <span data-ttu-id="79c2b-122">Key vault を確立し、資格情報コンテナー内の機密情報を作成すると、サンプル アプリでは、安全にシークレットの値の構成に読み込んで web ページに表示します。</span><span class="sxs-lookup"><span data-stu-id="79c2b-122">Once you establish a key vault and create secrets in the vault, the sample apps securely load the secret values into their configurations and display them in webpages.</span></span>

<span data-ttu-id="79c2b-123">プロバイダーを追加、`ConfigurationBuilder`で、`AddAzureKeyVault`拡張機能です。</span><span class="sxs-lookup"><span data-stu-id="79c2b-123">The provider is added to the `ConfigurationBuilder` with the `AddAzureKeyVault` extension.</span></span> <span data-ttu-id="79c2b-124">アプリでは、サンプル、拡張機能はから読み込まれた 3 つの構成値を使用して、*される appsettings.json*ファイル。</span><span class="sxs-lookup"><span data-stu-id="79c2b-124">In the sample apps, the extension uses three configuration values loaded from the *appsettings.json* file.</span></span>

| <span data-ttu-id="79c2b-125">アプリケーション設定</span><span class="sxs-lookup"><span data-stu-id="79c2b-125">App Setting</span></span>    | <span data-ttu-id="79c2b-126">説明</span><span class="sxs-lookup"><span data-stu-id="79c2b-126">Description</span></span>                    | <span data-ttu-id="79c2b-127">例</span><span class="sxs-lookup"><span data-stu-id="79c2b-127">Example</span></span>                                      |
| -------------- | ------------------------------ | -------------------------------------------- |
| `Vault`        | <span data-ttu-id="79c2b-128">Azure Key Vault の名前</span><span class="sxs-lookup"><span data-stu-id="79c2b-128">Azure Key Vault name</span></span>           | <span data-ttu-id="79c2b-129">contosovault</span><span class="sxs-lookup"><span data-stu-id="79c2b-129">contosovault</span></span>                                 |
| `ClientId`     | <span data-ttu-id="79c2b-130">Azure Active Directory アプリ Id</span><span class="sxs-lookup"><span data-stu-id="79c2b-130">Azure Active Directory App Id</span></span>  | <span data-ttu-id="79c2b-131">627e911e-43cc-61d4-992e-12db9c81b413</span><span class="sxs-lookup"><span data-stu-id="79c2b-131">627e911e-43cc-61d4-992e-12db9c81b413</span></span>         |
| `ClientSecret` | <span data-ttu-id="79c2b-132">Azure Active Directory アプリ キー</span><span class="sxs-lookup"><span data-stu-id="79c2b-132">Azure Active Directory App Key</span></span> | <span data-ttu-id="79c2b-133">g58K3dtg59o1Pa + e59v2Tx829w6VxTB2yv9sv/101di =</span><span class="sxs-lookup"><span data-stu-id="79c2b-133">g58K3dtg59o1Pa+e59v2Tx829w6VxTB2yv9sv/101di=</span></span> |

<span data-ttu-id="79c2b-134">[!code-csharp[プログラム](key-vault-configuration/samples/basic-sample/2.x/Program.cs?name=snippet1&highlight=2,7-10)]</span><span class="sxs-lookup"><span data-stu-id="79c2b-134">[!code-csharp[Program](key-vault-configuration/samples/basic-sample/2.x/Program.cs?name=snippet1&highlight=2,7-10)]</span></span>

## <a name="creating-key-vault-secrets-and-loading-configuration-values-basic-sample"></a><span data-ttu-id="79c2b-135">シークレットの資格情報コンテナーを作成および構成値 (basic サンプル) の読み込み</span><span class="sxs-lookup"><span data-stu-id="79c2b-135">Creating key vault secrets and loading configuration values (basic-sample)</span></span>
1. <span data-ttu-id="79c2b-136">Key vault の作成し、ガイダンスに従って、アプリケーションの Azure Active Directory (Azure AD) を設定[Azure Key Vault の使用開始](https://azure.microsoft.com/documentation/articles/key-vault-get-started/)です。</span><span class="sxs-lookup"><span data-stu-id="79c2b-136">Create a key vault and set up Azure Active Directory (Azure AD) for the application following the guidance in [Get started with Azure Key Vault](https://azure.microsoft.com/documentation/articles/key-vault-get-started/).</span></span>
  * <span data-ttu-id="79c2b-137">使用して、資格情報コンテナーに機密情報を追加、 [AzureRM キー資格情報コンテナー PowerShell モジュール](/powershell/module/azurerm.keyvault)から使用可能な[PowerShell ギャラリー](https://www.powershellgallery.com/packages/AzureRM.KeyVault)、 [Azure Key Vault REST API](/rest/api/keyvault/)、または、 [Azure ポータル](https://portal.azure.com/)です。</span><span class="sxs-lookup"><span data-stu-id="79c2b-137">Add secrets to the key vault using the [AzureRM Key Vault PowerShell Module](/powershell/module/azurerm.keyvault) available from the [PowerShell Gallery](https://www.powershellgallery.com/packages/AzureRM.KeyVault), the [Azure Key Vault REST API](/rest/api/keyvault/), or the [Azure Portal](https://portal.azure.com/).</span></span> <span data-ttu-id="79c2b-138">機密情報は、いずれかとして作成*手動*または*証明書*シークレット。</span><span class="sxs-lookup"><span data-stu-id="79c2b-138">Secrets are created as either *Manual* or *Certificate* secrets.</span></span> <span data-ttu-id="79c2b-139">*証明書*シークレットはアプリやサービスで使用する証明書が、構成プロバイダーによってサポートされていません。</span><span class="sxs-lookup"><span data-stu-id="79c2b-139">*Certificate* secrets are certificates for use by apps and services but are not supported by the configuration provider.</span></span> <span data-ttu-id="79c2b-140">使用する必要があります、*手動*構成プロバイダーを使用するための名前と値のペアの機密情報を作成するオプションです。</span><span class="sxs-lookup"><span data-stu-id="79c2b-140">You should use the *Manual* option to create name-value pair secrets for use with the configuration provider.</span></span>
    * <span data-ttu-id="79c2b-141">単純なシークレットは、名前と値のペアとして作成されます。</span><span class="sxs-lookup"><span data-stu-id="79c2b-141">Simple secrets are created as name-value pairs.</span></span> <span data-ttu-id="79c2b-142">Azure Key Vault のシークレット名は、英数字とハイフンに制限されます。</span><span class="sxs-lookup"><span data-stu-id="79c2b-142">Azure Key Vault secret names are limited to alphanumeric characters and dashes.</span></span>
    * <span data-ttu-id="79c2b-143">階層型の値 (構成セクション) を使用して`--`(2 つのハイフン)、サンプルでは、区切り記号として。</span><span class="sxs-lookup"><span data-stu-id="79c2b-143">Hierarchical values (configuration sections) use `--` (two dashes) as a separator in the sample.</span></span> <span data-ttu-id="79c2b-144">サブキーのセクションを区切るために通常使用されるコロン[ASP.NET Core 構成](xref:fundamentals/configuration)、シークレット名に許可されていません。</span><span class="sxs-lookup"><span data-stu-id="79c2b-144">Colons, which are normally used to delimit a section from a subkey in [ASP.NET Core configuration](xref:fundamentals/configuration), aren't allowed in secret names.</span></span> <span data-ttu-id="79c2b-145">そのため、2 個のダッシュが使用され、コロン、シークレットは、アプリの構成に読み込まれるときに切り替わります。</span><span class="sxs-lookup"><span data-stu-id="79c2b-145">Therefore, two dashes are used and swapped for a colon when the secrets are loaded into the app's configuration.</span></span>
    * <span data-ttu-id="79c2b-146">2 つ作成*手動*機密情報の次の名前と値のペアを使用します。</span><span class="sxs-lookup"><span data-stu-id="79c2b-146">Create two *Manual* secrets with the following name-value pairs.</span></span> <span data-ttu-id="79c2b-147">最初のシークレットは、単純な名前と値を 2 番目のシークレット セクションとシークレットの名前のサブキーを使用して秘密の値を作成します。</span><span class="sxs-lookup"><span data-stu-id="79c2b-147">The first secret is a simple name and value, and the second secret creates a secret value with a section and subkey in the secret name:</span></span>
      * <span data-ttu-id="79c2b-148">`SecretName`: `secret_value_1`</span><span class="sxs-lookup"><span data-stu-id="79c2b-148">`SecretName`: `secret_value_1`</span></span>
      * <span data-ttu-id="79c2b-149">`Section--SecretName`: `secret_value_2`</span><span class="sxs-lookup"><span data-stu-id="79c2b-149">`Section--SecretName`: `secret_value_2`</span></span>
  * <span data-ttu-id="79c2b-150">Azure Active Directory とサンプル アプリを登録します。</span><span class="sxs-lookup"><span data-stu-id="79c2b-150">Register the sample app with Azure Active Directory.</span></span>
  * <span data-ttu-id="79c2b-151">Key vault にアクセスするアプリを承認します。</span><span class="sxs-lookup"><span data-stu-id="79c2b-151">Authorize the app to access the key vault.</span></span> <span data-ttu-id="79c2b-152">使用すると、 `Set-AzureRmKeyVaultAccessPolicy` 、key vault にアクセスするアプリを承認するために PowerShell コマンドレットを提供`List`と`Get`で機密データへのアクセス`-PermissionsToKeys list,get`です。</span><span class="sxs-lookup"><span data-stu-id="79c2b-152">When you use the `Set-AzureRmKeyVaultAccessPolicy` PowerShell cmdlet to authorize the app to access the key vault, provide `List` and `Get` access to secrets with `-PermissionsToKeys list,get`.</span></span>
2. <span data-ttu-id="79c2b-153">アプリの更新*される appsettings.json*の値を持つファイル`Vault`、 `ClientId`、および`ClientSecret`です。</span><span class="sxs-lookup"><span data-stu-id="79c2b-153">Update the app's *appsettings.json* file with the values of `Vault`, `ClientId`, and `ClientSecret`.</span></span>
3. <span data-ttu-id="79c2b-154">その構成値を取得するサンプル アプリを実行する`IConfigurationRoot`秘密の名前と同じ名前にします。</span><span class="sxs-lookup"><span data-stu-id="79c2b-154">Run the sample app, which obtains its configuration values from `IConfigurationRoot` with the same name as the secret name.</span></span>
  * <span data-ttu-id="79c2b-155">非階層型の値: の値は、`SecretName`で取得した`config["SecretName"]`です。</span><span class="sxs-lookup"><span data-stu-id="79c2b-155">Non-hierarchical values: The value for `SecretName` is obtained with `config["SecretName"]`.</span></span>
  * <span data-ttu-id="79c2b-156">階層型の値 (セクション)。 使用`:`(コロン) 表記または`GetSection`拡張メソッド。</span><span class="sxs-lookup"><span data-stu-id="79c2b-156">Hierarchical values (sections): Use `:` (colon) notation or the `GetSection` extension method.</span></span> <span data-ttu-id="79c2b-157">構成値を取得するのにには、これらのアプローチのいずれかを使用します。</span><span class="sxs-lookup"><span data-stu-id="79c2b-157">Use either of these approaches to obtain the configuration value:</span></span>
    * `config["Section:SecretName"]`
    * `config.GetSection("Section")["SecretName"]`

<span data-ttu-id="79c2b-158">アプリを実行すると、web ページには、秘密の読み込まれた値が示されます。</span><span class="sxs-lookup"><span data-stu-id="79c2b-158">When you run the app, a webpage shows the loaded secret values:</span></span>

![Azure キー資格情報コンテナーの構成プロバイダー経由で読み込まれているシークレットの値を表示されているブラウザー ウィンドウ](key-vault-configuration/_static/sample1.png)

## <a name="creating-prefixed-key-vault-secrets-and-loading-configuration-values-key-name-prefix-sample"></a><span data-ttu-id="79c2b-160">プレフィックスが指定された資格情報コンテナーの機密情報を作成および構成の値 (キーの名前のプレフィックス-サンプル) の読み込み</span><span class="sxs-lookup"><span data-stu-id="79c2b-160">Creating prefixed key vault secrets and loading configuration values (key-name-prefix-sample)</span></span>
<span data-ttu-id="79c2b-161">`AddAzureKeyVault`実装を受け入れるオーバー ロードも提供`IKeyVaultSecretManager`構成のキーに変換がどのキー資格情報コンテナーの機密情報を制御することができます。</span><span class="sxs-lookup"><span data-stu-id="79c2b-161">`AddAzureKeyVault` also provides an overload that accepts an implementation of `IKeyVaultSecretManager`, which allows you to control how key vault secrets are converted into configuration keys.</span></span> <span data-ttu-id="79c2b-162">たとえば、アプリの起動時に指定したプレフィックス値に基づいてシークレットの値を読み込むインターフェイスを実装することができます。</span><span class="sxs-lookup"><span data-stu-id="79c2b-162">For example, you can implement the interface to load secret values based on a prefix value you provide at app startup.</span></span> <span data-ttu-id="79c2b-163">これにより、たとえば、アプリのバージョンに基づくシークレットを読み込めません。</span><span class="sxs-lookup"><span data-stu-id="79c2b-163">This allows you, for example, to load secrets based on the version of the app.</span></span>

> [!WARNING]
> <span data-ttu-id="79c2b-164">複数のアプリのシークレットを key vault に配置するか、環境のシークレットを配置する、キー vault シークレットのプレフィックスを使用しない (たとえば、*開発*verus*運用*シークレット) を 1 つ資格情報コンテナー。</span><span class="sxs-lookup"><span data-stu-id="79c2b-164">Don't use prefixes on key vault secrets to place secrets for multiple apps into the same key vault or to place environmental secrets (for example, *development* verus *production* secrets) into the same vault.</span></span> <span data-ttu-id="79c2b-165">別のアプリとの開発と実稼働環境を最高レベルのセキュリティのアプリの環境を分離する個別のキー コンテナーを使用することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="79c2b-165">We recommend that different apps and development/production environments use separate key vaults to isolate app environments for the highest level of security.</span></span>

<span data-ttu-id="79c2b-166">Key vault にシークレットを作成する 2 つ目のサンプル アプリを使用して`5000-AppSecret`(key vault のシークレット名で許可されていない期間) を表すバージョン 5.0.0.0 のアプリのアプリのシークレット。</span><span class="sxs-lookup"><span data-stu-id="79c2b-166">Using the second sample app, you create a secret in the key vault for `5000-AppSecret` (periods aren't allowed in key vault secret names) representing an app secret for version 5.0.0.0 of your app.</span></span> <span data-ttu-id="79c2b-167">シークレットを作成する別のバージョン、5.1.0.0、`5100-AppSecret`です。</span><span class="sxs-lookup"><span data-stu-id="79c2b-167">For another version, 5.1.0.0, you create a secret for `5100-AppSecret`.</span></span> <span data-ttu-id="79c2b-168">各アプリのバージョンを読み込みます独自秘密の値としてその構成`AppSecret`、機密情報が読み込まれるバージョンを削除します。</span><span class="sxs-lookup"><span data-stu-id="79c2b-168">Each app version loads its own secret value into its configuration as `AppSecret`, stripping off the version as it loads the secret.</span></span> <span data-ttu-id="79c2b-169">サンプルの実装を次に示します。</span><span class="sxs-lookup"><span data-stu-id="79c2b-169">The sample's implementation is shown below:</span></span>

<span data-ttu-id="79c2b-170">[!code-csharp[構成ビルダー](key-vault-configuration/samples/key-name-prefix-sample/2.x/Program.cs?name=snippet1&highlight=12)]</span><span class="sxs-lookup"><span data-stu-id="79c2b-170">[!code-csharp[Configuration builder](key-vault-configuration/samples/key-name-prefix-sample/2.x/Program.cs?name=snippet1&highlight=12)]</span></span>

<span data-ttu-id="79c2b-171">[!code-csharp[PrefixKeyVaultSecretManager](key-vault-configuration/samples/key-name-prefix-sample/2.x/Startup.cs?name=snippet1)]</span><span class="sxs-lookup"><span data-stu-id="79c2b-171">[!code-csharp[PrefixKeyVaultSecretManager](key-vault-configuration/samples/key-name-prefix-sample/2.x/Startup.cs?name=snippet1)]</span></span>

<span data-ttu-id="79c2b-172">`Load`バージョンというプレフィックスが付いているものを検索する資格情報コンテナーのシークレットを反復処理するプロバイダーのアルゴリズムでメソッドが呼び出されます。</span><span class="sxs-lookup"><span data-stu-id="79c2b-172">The `Load` method is called by a provider algorithm that iterates through the vault secrets to find the ones that have the version prefix.</span></span> <span data-ttu-id="79c2b-173">バージョン プレフィックスがで検出されたときに`Load`、アルゴリズムを使用して、`GetKey`シークレット名の構成名を返すメソッド。</span><span class="sxs-lookup"><span data-stu-id="79c2b-173">When a version prefix is found with `Load`, the algorithm uses the `GetKey` method to return the configuration name of the secret name.</span></span> <span data-ttu-id="79c2b-174">シークレットの名前からバージョン プレフィックス取り除いてし、アプリの構成に名前と値のペアの読み込みのシークレット名の残りの部分を返します。</span><span class="sxs-lookup"><span data-stu-id="79c2b-174">It strips off the version prefix from the secret's name and returns the rest of the secret name for loading into the app's configuration name-value pairs.</span></span>

<span data-ttu-id="79c2b-175">このアプローチを実装する場合。</span><span class="sxs-lookup"><span data-stu-id="79c2b-175">When you implement this approach:</span></span>

1. <span data-ttu-id="79c2b-176">資格情報コンテナーの機密情報が読み込まれます。</span><span class="sxs-lookup"><span data-stu-id="79c2b-176">The key vault secrets are loaded.</span></span>
2. <span data-ttu-id="79c2b-177">文字列のシークレットを`5000-AppSecret`が一致します。</span><span class="sxs-lookup"><span data-stu-id="79c2b-177">The string secret for `5000-AppSecret` is matched.</span></span>
3. <span data-ttu-id="79c2b-178">バージョン、 `5000` (dash) のまま、キー名から取り除かは`AppSecret`シークレットの値を持つアプリの構成を読み込めません。</span><span class="sxs-lookup"><span data-stu-id="79c2b-178">The version, `5000` (with the dash), is stripped off of the key name leaving `AppSecret` to load with the secret value into the app's configuration.</span></span>

> [!NOTE]
> <span data-ttu-id="79c2b-179">独自に提供することも`KeyVaultClient`実装`AddAzureKeyVault`です。</span><span class="sxs-lookup"><span data-stu-id="79c2b-179">You can also provide your own `KeyVaultClient` implementation to `AddAzureKeyVault`.</span></span> <span data-ttu-id="79c2b-180">カスタムのクライアントを指定するには、クライアントの構成プロバイダーと、アプリの他の部分の 1 つのインスタンスを共有することができます。</span><span class="sxs-lookup"><span data-stu-id="79c2b-180">Supplying a custom client allows you to share a single instance of the client between the configuration provider and other parts of your app.</span></span>

1. <span data-ttu-id="79c2b-181">Key vault の作成し、ガイダンスに従って、アプリケーションの Azure Active Directory (Azure AD) を設定[Azure Key Vault の使用開始](https://azure.microsoft.com/documentation/articles/key-vault-get-started/)です。</span><span class="sxs-lookup"><span data-stu-id="79c2b-181">Create a key vault and set up Azure Active Directory (Azure AD) for the application following the guidance in [Get started with Azure Key Vault](https://azure.microsoft.com/documentation/articles/key-vault-get-started/).</span></span>
  * <span data-ttu-id="79c2b-182">使用して、資格情報コンテナーに機密情報を追加、 [AzureRM キー資格情報コンテナー PowerShell モジュール](/powershell/module/azurerm.keyvault)から使用可能な[PowerShell ギャラリー](https://www.powershellgallery.com/packages/AzureRM.KeyVault)、 [Azure Key Vault REST API](/rest/api/keyvault/)、または、 [Azure ポータル](https://portal.azure.com/)です。</span><span class="sxs-lookup"><span data-stu-id="79c2b-182">Add secrets to the key vault using the [AzureRM Key Vault PowerShell Module](/powershell/module/azurerm.keyvault) available from the [PowerShell Gallery](https://www.powershellgallery.com/packages/AzureRM.KeyVault), the [Azure Key Vault REST API](/rest/api/keyvault/), or the [Azure Portal](https://portal.azure.com/).</span></span> <span data-ttu-id="79c2b-183">機密情報は、いずれかとして作成*手動*または*証明書*シークレット。</span><span class="sxs-lookup"><span data-stu-id="79c2b-183">Secrets are created as either *Manual* or *Certificate* secrets.</span></span> <span data-ttu-id="79c2b-184">*証明書*シークレットはアプリやサービスで使用する証明書が、構成プロバイダーによってサポートされていません。</span><span class="sxs-lookup"><span data-stu-id="79c2b-184">*Certificate* secrets are certificates for use by apps and services but are not supported by the configuration provider.</span></span> <span data-ttu-id="79c2b-185">使用する必要があります、*手動*構成プロバイダーを使用するための名前と値のペアの機密情報を作成するオプションです。</span><span class="sxs-lookup"><span data-stu-id="79c2b-185">You should use the *Manual* option to create name-value pair secrets for use with the configuration provider.</span></span>
    * <span data-ttu-id="79c2b-186">階層型の値 (構成セクション) を使用して`--`(2 つのハイフン)、区切り記号として。</span><span class="sxs-lookup"><span data-stu-id="79c2b-186">Hierarchical values (configuration sections) use `--` (two dashes) as a separator.</span></span>
    * <span data-ttu-id="79c2b-187">2 つ作成*手動*次の名前/値ペアのシークレット。</span><span class="sxs-lookup"><span data-stu-id="79c2b-187">Create two *Manual* secrets with the following name-value pairs:</span></span>
      * <span data-ttu-id="79c2b-188">`5000-AppSecret`: `5.0.0.0_secret_value`</span><span class="sxs-lookup"><span data-stu-id="79c2b-188">`5000-AppSecret`: `5.0.0.0_secret_value`</span></span>
      * <span data-ttu-id="79c2b-189">`5100-AppSecret`: `5.1.0.0_secret_value`</span><span class="sxs-lookup"><span data-stu-id="79c2b-189">`5100-AppSecret`: `5.1.0.0_secret_value`</span></span>
  * <span data-ttu-id="79c2b-190">Azure Active Directory とサンプル アプリを登録します。</span><span class="sxs-lookup"><span data-stu-id="79c2b-190">Register the sample app with Azure Active Directory.</span></span>
  * <span data-ttu-id="79c2b-191">Key vault にアクセスするアプリを承認します。</span><span class="sxs-lookup"><span data-stu-id="79c2b-191">Authorize the app to access the key vault.</span></span> <span data-ttu-id="79c2b-192">使用すると、 `Set-AzureRmKeyVaultAccessPolicy` 、key vault にアクセスするアプリを承認するために PowerShell コマンドレットを提供`List`と`Get`で機密データへのアクセス`-PermissionsToKeys list,get`です。</span><span class="sxs-lookup"><span data-stu-id="79c2b-192">When you use the `Set-AzureRmKeyVaultAccessPolicy` PowerShell cmdlet to authorize the app to access the key vault, provide `List` and `Get` access to secrets with `-PermissionsToKeys list,get`.</span></span>
2. <span data-ttu-id="79c2b-193">アプリの更新*される appsettings.json*の値を持つファイル`Vault`、 `ClientId`、および`ClientSecret`です。</span><span class="sxs-lookup"><span data-stu-id="79c2b-193">Update the app's *appsettings.json* file with the values of `Vault`, `ClientId`, and `ClientSecret`.</span></span>
3. <span data-ttu-id="79c2b-194">その構成値を取得するサンプル アプリを実行する`IConfigurationRoot`プレフィックス付きのシークレットの名前と同じ名前にします。</span><span class="sxs-lookup"><span data-stu-id="79c2b-194">Run the sample app, which obtains its configuration values from `IConfigurationRoot` with the same name as the prefixed secret name.</span></span> <span data-ttu-id="79c2b-195">このサンプルではプレフィックスは、アプリのバージョンは、指定した、 `PrefixKeyVaultSecretManager` Azure Key Vault の構成プロバイダーを追加するとき。</span><span class="sxs-lookup"><span data-stu-id="79c2b-195">In this sample, the prefix is the app's version, which you provided to the `PrefixKeyVaultSecretManager` when you added the Azure Key Vault configuration provider.</span></span> <span data-ttu-id="79c2b-196">値は、`AppSecret`で取得した`config["AppSecret"]`です。</span><span class="sxs-lookup"><span data-stu-id="79c2b-196">The value for `AppSecret` is obtained with `config["AppSecret"]`.</span></span> <span data-ttu-id="79c2b-197">アプリによって生成される web ページは、読み込まれた値を示しています。</span><span class="sxs-lookup"><span data-stu-id="79c2b-197">The webpage generated by the app shows the loaded value:</span></span>

   ![ブラウザー ウィンドウが、アプリのバージョンが 5.0.0.0 である場合は、Azure キー資格情報コンテナーの構成プロバイダー経由で読み込まれる秘密の値を表示](key-vault-configuration/_static/sample2-1.png)

4. <span data-ttu-id="79c2b-199">プロジェクト ファイルでのアプリ アセンブリのバージョンを変更`5.0.0.0`に`5.1.0.0`アプリをもう一度実行します。</span><span class="sxs-lookup"><span data-stu-id="79c2b-199">Change the version of the app assembly in the project file from `5.0.0.0` to `5.1.0.0` and run the app again.</span></span> <span data-ttu-id="79c2b-200">このとき、返される秘密の値は`5.1.0.0_secret_value`します。</span><span class="sxs-lookup"><span data-stu-id="79c2b-200">This time, the secret value returned is `5.1.0.0_secret_value`.</span></span> <span data-ttu-id="79c2b-201">アプリによって生成される web ページは、読み込まれた値を示しています。</span><span class="sxs-lookup"><span data-stu-id="79c2b-201">The webpage generated by the app shows the loaded value:</span></span>

   ![ブラウザー ウィンドウが、アプリのバージョンが 5.1.0.0 である場合は、Azure キー資格情報コンテナーの構成プロバイダー経由で読み込まれる秘密の値を表示](key-vault-configuration/_static/sample2-2.png)

## <a name="controlling-access-to-the-clientsecret"></a><span data-ttu-id="79c2b-203">ClientSecret へのアクセスを制御</span><span class="sxs-lookup"><span data-stu-id="79c2b-203">Controlling access to the ClientSecret</span></span>
<span data-ttu-id="79c2b-204">使用して、[シークレット マネージャー ツール](xref:security/app-secrets)維持するために、`ClientSecret`プロジェクト ソース ツリーの外部でします。</span><span class="sxs-lookup"><span data-stu-id="79c2b-204">Use the [Secret Manager tool](xref:security/app-secrets) to maintain the `ClientSecret` outside of your project source tree.</span></span> <span data-ttu-id="79c2b-205">シークレット マネージャーでは、アプリ シークレットは、特定のプロジェクトに関連付けるしてそれらを複数のプロジェクト間で共有します。</span><span class="sxs-lookup"><span data-stu-id="79c2b-205">With Secret Manager, you associate app secrets with a specific project and share them across multiple projects.</span></span>

<span data-ttu-id="79c2b-206">証明書をサポートする環境での .NET Framework アプリを開発するときは、X.509 証明書を Azure Key Vault に認証できます。</span><span class="sxs-lookup"><span data-stu-id="79c2b-206">When developing a .NET Framework app in an environment that supports certificates, you can authenticate to Azure Key Vault with an X.509 certificate.</span></span> <span data-ttu-id="79c2b-207">X.509 証明書の秘密キーは、オペレーティング システムによって管理されます。</span><span class="sxs-lookup"><span data-stu-id="79c2b-207">The X.509 certificate's private key is managed by the OS.</span></span> <span data-ttu-id="79c2b-208">詳細については、次を参照してください。[クライアント シークレットではなく、証明書による認証](https://docs.microsoft.com/azure/key-vault/key-vault-use-from-web-application#authenticate-with-a-certificate-instead-of-a-client-secret)です。</span><span class="sxs-lookup"><span data-stu-id="79c2b-208">For more information, see [Authenticate with a Certificate instead of a Client Secret](https://docs.microsoft.com/azure/key-vault/key-vault-use-from-web-application#authenticate-with-a-certificate-instead-of-a-client-secret).</span></span> <span data-ttu-id="79c2b-209">使用して、`AddAzureKeyVault`を受け入れるオーバー ロード、`X509Certificate2`です。</span><span class="sxs-lookup"><span data-stu-id="79c2b-209">Use the `AddAzureKeyVault` overload that accepts an `X509Certificate2`.</span></span>

```csharp
var store = new X509Store(StoreLocation.CurrentUser);
store.Open(OpenFlags.ReadOnly);
var cert = store.Certificates.Find(X509FindType.FindByThumbprint, config["CertificateThumbprint"], false);

builder.AddAzureKeyVault(
    config["Vault"],
    config["ClientId"],
    cert.OfType<X509Certificate2>().Single(),
    new EnvironmentSecretManager(env.ApplicationName));
store.Close();

Configuration = builder.Build();
```

## <a name="reloading-secrets"></a><span data-ttu-id="79c2b-210">シークレットの再読み込み</span><span class="sxs-lookup"><span data-stu-id="79c2b-210">Reloading secrets</span></span>
<span data-ttu-id="79c2b-211">シークレットはまでキャッシュ`IConfigurationRoot.Reload()`と呼びます。</span><span class="sxs-lookup"><span data-stu-id="79c2b-211">Secrets are cached until `IConfigurationRoot.Reload()` is called.</span></span> <span data-ttu-id="79c2b-212">期限切れ、無効になっている、およびアプリケーションまでで、key vault に更新されたシークレットが守られていない`Reload`を実行します。</span><span class="sxs-lookup"><span data-stu-id="79c2b-212">Expired, disabled, and updated secrets in the key vault are not respected by the application until `Reload` is executed.</span></span>

```csharp
Configuration.Reload();
```

## <a name="disabled-and-expired-secrets"></a><span data-ttu-id="79c2b-213">無効になっており、有効期限が切れたシークレット</span><span class="sxs-lookup"><span data-stu-id="79c2b-213">Disabled and expired secrets</span></span>
<span data-ttu-id="79c2b-214">無効になっており、期限切れのシークレットをスロー、`KeyVaultClientException`です。</span><span class="sxs-lookup"><span data-stu-id="79c2b-214">Disabled and expired secrets throw a `KeyVaultClientException`.</span></span> <span data-ttu-id="79c2b-215">アプリを防ぐためがスローされることから、アプリを交換または無効/有効期限が切れてシークレットを更新します。</span><span class="sxs-lookup"><span data-stu-id="79c2b-215">To prevent your app from throwing, replace your app or update the disabled/expired secret.</span></span>

## <a name="troubleshooting"></a><span data-ttu-id="79c2b-216">トラブルシューティング</span><span class="sxs-lookup"><span data-stu-id="79c2b-216">Troubleshooting</span></span>
<span data-ttu-id="79c2b-217">アプリケーションは、プロバイダーを使用して構成の読み込みに失敗すると、エラー メッセージに書き込まれます。、 [ASP.NET のログ記録インフラストラクチャ](xref:fundamentals/logging)です。</span><span class="sxs-lookup"><span data-stu-id="79c2b-217">When the application fails to load configuration using the provider, an error message is written to the [ASP.NET Logging infrastructure](xref:fundamentals/logging).</span></span> <span data-ttu-id="79c2b-218">次の条件には、構成を読み込めないは禁止します。</span><span class="sxs-lookup"><span data-stu-id="79c2b-218">The following conditions will prevent configuration from loading:</span></span>
* <span data-ttu-id="79c2b-219">アプリは、Azure Active Directory に正しく構成されていません。</span><span class="sxs-lookup"><span data-stu-id="79c2b-219">The app isn't configured correctly in Azure Active Directory.</span></span>
* <span data-ttu-id="79c2b-220">Azure Key Vault に資格情報コンテナーが存在しません。</span><span class="sxs-lookup"><span data-stu-id="79c2b-220">The key vault doesn't exist in Azure Key Vault.</span></span>
* <span data-ttu-id="79c2b-221">アプリは、key vault にアクセスする承認されていません。</span><span class="sxs-lookup"><span data-stu-id="79c2b-221">The app isn't authorized to access the key vault.</span></span>
* <span data-ttu-id="79c2b-222">アクセス ポリシーに含まれていない`Get`と`List`アクセス許可。</span><span class="sxs-lookup"><span data-stu-id="79c2b-222">The access policy doesn't include `Get` and `List` permissions.</span></span>
* <span data-ttu-id="79c2b-223">Key vault に構成データ (名前と値のペア) は、不適切な名前、無効であるか有効期限が切れて、欠落しています。</span><span class="sxs-lookup"><span data-stu-id="79c2b-223">In the key vault, the configuration data (name-value pair) is incorrectly named, missing, disabled, or expired.</span></span>
* <span data-ttu-id="79c2b-224">アプリが正しくない資格情報コンテナーの名前 (`Vault`)、Azure AD アプリ Id (`ClientId`)、または Azure AD のキー (`ClientSecret`)。</span><span class="sxs-lookup"><span data-stu-id="79c2b-224">The app has the wrong key vault name (`Vault`), Azure AD App Id (`ClientId`), or Azure AD Key (`ClientSecret`).</span></span>
* <span data-ttu-id="79c2b-225">Azure AD のキー (`ClientSecret`) 有効期限が切れています。</span><span class="sxs-lookup"><span data-stu-id="79c2b-225">The Azure AD Key (`ClientSecret`) is expired.</span></span>
* <span data-ttu-id="79c2b-226">ロードしようとしている値のアプリの構成キー (名前) が正しくないです。</span><span class="sxs-lookup"><span data-stu-id="79c2b-226">The configuration key (name) is incorrect in the app for the value you're trying to load.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="79c2b-227">その他の技術情報</span><span class="sxs-lookup"><span data-stu-id="79c2b-227">Additional resources</span></span>
* <xref:fundamentals/configuration>
* [<span data-ttu-id="79c2b-228">Microsoft Azure: Key Vault</span><span class="sxs-lookup"><span data-stu-id="79c2b-228">Microsoft Azure: Key Vault</span></span>](https://azure.microsoft.com/services/key-vault/)
* [<span data-ttu-id="79c2b-229">Microsoft Azure: Key Vault のドキュメント</span><span class="sxs-lookup"><span data-stu-id="79c2b-229">Microsoft Azure: Key Vault Documentation</span></span>](https://docs.microsoft.com/azure/key-vault/)
* [<span data-ttu-id="79c2b-230">Azure Key Vault のキーを生成し、HSM で保護された転送する方法</span><span class="sxs-lookup"><span data-stu-id="79c2b-230">How to generate and transfer HSM-protected keys for Azure Key Vault</span></span>](https://docs.microsoft.com/azure/key-vault/key-vault-hsm-protected-keys)
* [<span data-ttu-id="79c2b-231">KeyVaultClient クラス</span><span class="sxs-lookup"><span data-stu-id="79c2b-231">KeyVaultClient Class</span></span>](https://docs.microsoft.com/dotnet/api/microsoft.azure.keyvault.keyvaultclient)