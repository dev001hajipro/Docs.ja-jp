---
title: "分散キャッシュの使用"
author: ardalis
description: 
keywords: ASP.NET Core,
ms.author: riande
manager: wpickett
ms.date: 02/14/2017
ms.topic: article
ms.assetid: 870f082d-6d43-453d-b311-45f3aeb4d2c5
ms.technology: aspnet
ms.prod: asp.net-core
uid: performance/caching/distributed
ms.openlocfilehash: abf680fef9de175082c1e4f4cebc2b9648f18a28
ms.sourcegitcommit: 9cdbfd0d670d70b9c354216aabee260c52dad5ee
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/12/2017
---
# <a name="working-with-a-distributed-cache"></a><span data-ttu-id="4698e-103">分散キャッシュの使用</span><span class="sxs-lookup"><span data-stu-id="4698e-103">Working with a distributed cache</span></span>

<span data-ttu-id="4698e-104">によって[Steve Smith](https://ardalis.com/)</span><span class="sxs-lookup"><span data-stu-id="4698e-104">By [Steve Smith](https://ardalis.com/)</span></span>

<span data-ttu-id="4698e-105">分散キャッシュでは、クラウドまたはサーバー ファーム環境でホストされている場合に特に ASP.NET Core アプリケーションのスケーラビリティとパフォーマンスを向上できます。</span><span class="sxs-lookup"><span data-stu-id="4698e-105">Distributed caches can improve the performance and scalability of ASP.NET Core apps, especially when hosted in a cloud or server farm environment.</span></span> <span data-ttu-id="4698e-106">この記事では、ASP.NET Core の組み込みの分散キャッシュの抽象化と実装を操作する方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="4698e-106">This article explains how to work with ASP.NET Core's built-in distributed cache abstractions and implementations.</span></span>

[<span data-ttu-id="4698e-107">サンプル コードを表示またはダウンロードする</span><span class="sxs-lookup"><span data-stu-id="4698e-107">View or download sample code</span></span>](https://github.com/aspnet/Docs/tree/master/aspnetcore/performance/caching/distributed/sample)

## <a name="what-is-a-distributed-cache"></a><span data-ttu-id="4698e-108">分散キャッシュとは</span><span class="sxs-lookup"><span data-stu-id="4698e-108">What is a Distributed Cache</span></span>

<span data-ttu-id="4698e-109">分散キャッシュが複数のアプリケーション サーバーで共有 (を参照してください[キャッシュ基本](memory.md#caching-basics))。</span><span class="sxs-lookup"><span data-stu-id="4698e-109">A distributed cache is shared by multiple app servers (see [Caching Basics](memory.md#caching-basics)).</span></span> <span data-ttu-id="4698e-110">キャッシュ内の情報が個々 の web サーバーのメモリに格納されず、キャッシュされたデータはすべてのアプリのサーバーに使用できます。</span><span class="sxs-lookup"><span data-stu-id="4698e-110">The information in the cache is not stored in the memory of individual web servers, and the cached data is available to all of the app's servers.</span></span> <span data-ttu-id="4698e-111">これは、いくつかの利点を提供します。</span><span class="sxs-lookup"><span data-stu-id="4698e-111">This provides several advantages:</span></span>

1. <span data-ttu-id="4698e-112">キャッシュされたデータがすべての web サーバーに生じます。</span><span class="sxs-lookup"><span data-stu-id="4698e-112">Cached data is coherent on all web servers.</span></span> <span data-ttu-id="4698e-113">ユーザーによっては、web サーバーは、要求を処理、異なる結果が表示されません。</span><span class="sxs-lookup"><span data-stu-id="4698e-113">Users don't see different results depending on which web server handles their request</span></span>

2. <span data-ttu-id="4698e-114">キャッシュされたデータが回収されなかったは、web サーバーを再起動し展開します。</span><span class="sxs-lookup"><span data-stu-id="4698e-114">Cached data survives web server restarts and deployments.</span></span> <span data-ttu-id="4698e-115">個々 の web サーバーを削除またはキャッシュに影響を与えずに追加することができます。</span><span class="sxs-lookup"><span data-stu-id="4698e-115">Individual web servers can be removed or added without impacting the cache.</span></span>

3. <span data-ttu-id="4698e-116">ソースのデータ ストアでは、(より、複数のメモリ内キャッシュまたはなしですべてのキャッシュ) に加えられた以下の要求があります。</span><span class="sxs-lookup"><span data-stu-id="4698e-116">The source data store has fewer requests made to it (than with multiple in-memory caches or no cache at all).</span></span>

> [!NOTE]
> <span data-ttu-id="4698e-117">SQL Server 分散キャッシュを使用する場合は、別のデータベース インスタンスは、アプリのソース データのよりも、キャッシュの使用は、true を返します。 これらの利点の一部はのみです。</span><span class="sxs-lookup"><span data-stu-id="4698e-117">If using a SQL Server Distributed Cache, some of these advantages are only true if a separate database instance is used for the cache than for the app's source data.</span></span>

<span data-ttu-id="4698e-118">、任意のキャッシュのように、分散キャッシュ大幅にアプリの応答性が向上、ので、通常のリレーショナル データベース (または web サービス) よりも高速のキャッシュからデータを取得できます。</span><span class="sxs-lookup"><span data-stu-id="4698e-118">Like any cache, a distributed cache can dramatically improve an app's responsiveness, since typically data can be retrieved from the cache much faster than from a relational database (or web service).</span></span>

<span data-ttu-id="4698e-119">キャッシュの構成は、特定の実装です。</span><span class="sxs-lookup"><span data-stu-id="4698e-119">Cache configuration is implementation specific.</span></span> <span data-ttu-id="4698e-120">この記事では、Redis 両方を構成して、SQL Server の分散キャッシュ方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="4698e-120">This article describes how to configure both Redis and SQL Server distributed caches.</span></span> <span data-ttu-id="4698e-121">アプリが共通を使用して、キャッシュと対話する実装の種類を選択するに関係なく`IDistributedCache`インターフェイスです。</span><span class="sxs-lookup"><span data-stu-id="4698e-121">Regardless of which implementation is selected, the app interacts with the cache using a common `IDistributedCache` interface.</span></span>

## <a name="the-idistributedcache-interface"></a><span data-ttu-id="4698e-122">IDistributedCache インターフェイス</span><span class="sxs-lookup"><span data-stu-id="4698e-122">The IDistributedCache Interface</span></span>

<span data-ttu-id="4698e-123">`IDistributedCache`インターフェイスには、同期および非同期のメソッドが含まれています。</span><span class="sxs-lookup"><span data-stu-id="4698e-123">The `IDistributedCache` interface includes synchronous and asynchronous methods.</span></span> <span data-ttu-id="4698e-124">インターフェイスは、追加、取得、および実装では、分散キャッシュから削除する項目を使用します。</span><span class="sxs-lookup"><span data-stu-id="4698e-124">The interface allows items to be added, retrieved, and removed from the distributed cache implementation.</span></span> <span data-ttu-id="4698e-125">`IDistributedCache`インターフェイスには、次のメソッドが含まれています。</span><span class="sxs-lookup"><span data-stu-id="4698e-125">The `IDistributedCache` interface includes the following methods:</span></span>

<span data-ttu-id="4698e-126">**Get、されます。**</span><span class="sxs-lookup"><span data-stu-id="4698e-126">**Get, GetAsync**</span></span>

<span data-ttu-id="4698e-127">文字列のキーは、としてキャッシュされたアイテムを取得、`byte[]`場合、キャッシュ内に存在します。</span><span class="sxs-lookup"><span data-stu-id="4698e-127">Takes a string key and retrieves a cached item as a `byte[]` if found in the cache.</span></span>

<span data-ttu-id="4698e-128">**セット、SetAsync**</span><span class="sxs-lookup"><span data-stu-id="4698e-128">**Set, SetAsync**</span></span>

<span data-ttu-id="4698e-129">項目を追加 (として`byte[]`) 文字列キーを使用してキャッシュします。</span><span class="sxs-lookup"><span data-stu-id="4698e-129">Adds an item (as `byte[]`) to the cache using a string key.</span></span>

<span data-ttu-id="4698e-130">**更新、RefreshAsync**</span><span class="sxs-lookup"><span data-stu-id="4698e-130">**Refresh, RefreshAsync**</span></span>

<span data-ttu-id="4698e-131">スライド式有効期限タイムアウトをリセットする (存在する場合)、そのキーに基づく、キャッシュ内の項目を更新します。</span><span class="sxs-lookup"><span data-stu-id="4698e-131">Refreshes an item in the cache based on its key, resetting its sliding expiration timeout (if any).</span></span>

<span data-ttu-id="4698e-132">**に対して removeasync を実行を削除します。**</span><span class="sxs-lookup"><span data-stu-id="4698e-132">**Remove, RemoveAsync**</span></span>

<span data-ttu-id="4698e-133">そのキーに基づくキャッシュ エントリを削除します。</span><span class="sxs-lookup"><span data-stu-id="4698e-133">Removes a cache entry based on its key.</span></span>

<span data-ttu-id="4698e-134">使用する、`IDistributedCache`インターフェイス。</span><span class="sxs-lookup"><span data-stu-id="4698e-134">To use the `IDistributedCache` interface:</span></span>

   1. <span data-ttu-id="4698e-135">必要な NuGet パッケージをプロジェクト ファイルに追加します。</span><span class="sxs-lookup"><span data-stu-id="4698e-135">Add the required NuGet packages to your project file.</span></span>

   2. <span data-ttu-id="4698e-136">特定の実装を構成する`IDistributedCache`で、`Startup`クラスの`ConfigureServices`メソッドがコンテナーに追加します。</span><span class="sxs-lookup"><span data-stu-id="4698e-136">Configure the specific implementation of `IDistributedCache` in your `Startup` class's `ConfigureServices` method, and add it to the container there.</span></span>

   3. <span data-ttu-id="4698e-137">アプリから [`Middleware](../../fundamentals/middleware.md) or MVC controller classes, request an instance of `IDistributedCache' コンス トラクターからです。</span><span class="sxs-lookup"><span data-stu-id="4698e-137">From the app's [`Middleware](../../fundamentals/middleware.md) or MVC controller classes, request an instance of `IDistributedCache\` from the constructor.</span></span> <span data-ttu-id="4698e-138">によって、インスタンスが提供されます[依存性の注入](../../fundamentals/dependency-injection.md)(DI)。</span><span class="sxs-lookup"><span data-stu-id="4698e-138">The instance will be provided by [Dependency Injection](../../fundamentals/dependency-injection.md) (DI).</span></span>

> [!NOTE]
> <span data-ttu-id="4698e-139">シングルトンまたはスコープ ベースの有効期間を使用する必要はありません`IDistributedCache`インスタンス (少なくとも組み込みの実装のため)。</span><span class="sxs-lookup"><span data-stu-id="4698e-139">There is no need to use a Singleton or Scoped lifetime for `IDistributedCache` instances (at least for the built-in implementations).</span></span> <span data-ttu-id="4698e-140">必要に応じていずれかの場所にインスタンスを作成することも (を使用せずに[依存性の注入](../../fundamentals/dependency-injection.md))、これが難しく、コードをテストするには、違反している、[明示的な依存関係の原則](http://deviq.com/explicit-dependencies-principle/)です。</span><span class="sxs-lookup"><span data-stu-id="4698e-140">You can also create an instance wherever you might need one (instead of using [Dependency Injection](../../fundamentals/dependency-injection.md)), but this can make your code harder to test, and violates the [Explicit Dependencies Principle](http://deviq.com/explicit-dependencies-principle/).</span></span>

<span data-ttu-id="4698e-141">次の例は、のインスタンスを使用する方法を示しています。`IDistributedCache`では、単純なミドルウェア コンポーネント。</span><span class="sxs-lookup"><span data-stu-id="4698e-141">The following example shows how to use an instance of `IDistributedCache` in a simple middleware component:</span></span>

<span data-ttu-id="4698e-142">[!code-csharp[Main](./distributed/sample/src/DistCacheSample/StartTimeHeader.cs?highlight=15,18,21,27,28,29,30,31)]</span><span class="sxs-lookup"><span data-stu-id="4698e-142">[!code-csharp[Main](./distributed/sample/src/DistCacheSample/StartTimeHeader.cs?highlight=15,18,21,27,28,29,30,31)]</span></span>

<span data-ttu-id="4698e-143">上記のコードでキャッシュされた値が、読み取りが書き込まれていません。</span><span class="sxs-lookup"><span data-stu-id="4698e-143">In the code above, the cached value is read, but never written.</span></span> <span data-ttu-id="4698e-144">このサンプルでは、値は、サーバーの起動、および変更されない場合にのみ設定されます。</span><span class="sxs-lookup"><span data-stu-id="4698e-144">In this sample, the value is only set when a server starts up, and doesn't change.</span></span> <span data-ttu-id="4698e-145">マルチ サーバーのシナリオで開始する最も最近使用したサーバーに他のサーバーで設定した前の値が上書きされます。</span><span class="sxs-lookup"><span data-stu-id="4698e-145">In a multi-server scenario, the most recent server to start will overwrite any previous values that were set by other servers.</span></span> <span data-ttu-id="4698e-146">`Get`と`Set`メソッドを使用して、`byte[]`型です。</span><span class="sxs-lookup"><span data-stu-id="4698e-146">The `Get` and `Set` methods use the `byte[]` type.</span></span> <span data-ttu-id="4698e-147">使用して文字列値を変換する必要がありますので、 `Encoding.UTF8.GetString` (の`Get`) および`Encoding.UTF8.GetBytes`(の`Set`)。</span><span class="sxs-lookup"><span data-stu-id="4698e-147">Therefore, the string value must be converted using `Encoding.UTF8.GetString` (for `Get`) and `Encoding.UTF8.GetBytes` (for `Set`).</span></span>

<span data-ttu-id="4698e-148">次のコードから*Startup.cs*設定される値を示しています。</span><span class="sxs-lookup"><span data-stu-id="4698e-148">The following code from *Startup.cs* shows the value being set:</span></span>

<span data-ttu-id="4698e-149">[!code-csharp[Main](./distributed/sample/src/DistCacheSample/Startup.cs?highlight=2,4,5,6&range=58-66)]</span><span class="sxs-lookup"><span data-stu-id="4698e-149">[!code-csharp[Main](./distributed/sample/src/DistCacheSample/Startup.cs?highlight=2,4,5,6&range=58-66)]</span></span>

> [!NOTE]
> <span data-ttu-id="4698e-150">`IDistributedCache`で構成された、`ConfigureServices`メソッドが使用できる、`Configure`メソッドのパラメーターとして。</span><span class="sxs-lookup"><span data-stu-id="4698e-150">Since `IDistributedCache` is configured in the `ConfigureServices` method, it is available to the `Configure` method as a parameter.</span></span> <span data-ttu-id="4698e-151">パラメーターとして追加すると、DI で指定する構成済みのインスタンスが許可されます。</span><span class="sxs-lookup"><span data-stu-id="4698e-151">Adding it as a parameter will allow the configured instance to be provided through DI.</span></span>

## <a name="using-a-redis-distributed-cache"></a><span data-ttu-id="4698e-152">キャッシュ、Redis を使用した分散</span><span class="sxs-lookup"><span data-stu-id="4698e-152">Using a Redis Distributed Cache</span></span>

<span data-ttu-id="4698e-153">[Redis](https://redis.io/)分散キャッシュとして使用される多くの場合、オープン ソースのインメモリ データ ストアです。</span><span class="sxs-lookup"><span data-stu-id="4698e-153">[Redis](https://redis.io/) is an open source in-memory data store, which is often used as a distributed cache.</span></span> <span data-ttu-id="4698e-154">ローカルで行うこともでき、構成することができます、 [Azure Redis Cache](https://azure.microsoft.com/services/cache/) ASP.NET Core の Azure でホストされるアプリにします。</span><span class="sxs-lookup"><span data-stu-id="4698e-154">You can use it locally, and you can configure an [Azure Redis Cache](https://azure.microsoft.com/services/cache/) for your Azure-hosted ASP.NET Core apps.</span></span> <span data-ttu-id="4698e-155">キャッシュ実装を使用して、ASP.NET Core アプリケーションの構成、`RedisDistributedCache`インスタンス。</span><span class="sxs-lookup"><span data-stu-id="4698e-155">Your ASP.NET Core app configures the cache implementation using a `RedisDistributedCache` instance.</span></span>

<span data-ttu-id="4698e-156">Redis の実装を構成する`ConfigureServices`のインスタンスを要求することによって、アプリ コードでアクセスおよび`IDistributedCache`(上記のコードを参照してください)。</span><span class="sxs-lookup"><span data-stu-id="4698e-156">You configure the Redis implementation in `ConfigureServices` and access it in your app code by requesting an instance of `IDistributedCache` (see the code above).</span></span>

<span data-ttu-id="4698e-157">サンプル コードで、`RedisCache`のサーバーが構成されている実装を使用して、`Staging`環境。</span><span class="sxs-lookup"><span data-stu-id="4698e-157">In the sample code, a `RedisCache` implementation is used when the server is configured for a `Staging` environment.</span></span> <span data-ttu-id="4698e-158">したがって、`ConfigureStagingServices`メソッドは、構成、 `RedisCache`:</span><span class="sxs-lookup"><span data-stu-id="4698e-158">Thus the `ConfigureStagingServices` method configures the `RedisCache`:</span></span>

<span data-ttu-id="4698e-159">[!code-csharp[Main](./distributed/sample/src/DistCacheSample/Startup.cs?highlight=8,9,10,11,12,13&range=27-40)]</span><span class="sxs-lookup"><span data-stu-id="4698e-159">[!code-csharp[Main](./distributed/sample/src/DistCacheSample/Startup.cs?highlight=8,9,10,11,12,13&range=27-40)]</span></span>

> [!NOTE]
> <span data-ttu-id="4698e-160">ローカル コンピューターに Redis をインストールするには、chocolatey パッケージをインストール[https://chocolatey.org/packages/redis-64/](https://chocolatey.org/packages/redis-64/)実行と`redis-server`コマンド プロンプトからです。</span><span class="sxs-lookup"><span data-stu-id="4698e-160">To install Redis on your local machine, install the chocolatey package [https://chocolatey.org/packages/redis-64/](https://chocolatey.org/packages/redis-64/) and run `redis-server` from a command prompt.</span></span>

## <a name="using-a-sql-server-distributed-cache"></a><span data-ttu-id="4698e-161">SQL Server を使用してキャッシュを分散</span><span class="sxs-lookup"><span data-stu-id="4698e-161">Using a SQL Server Distributed Cache</span></span>

<span data-ttu-id="4698e-162">SqlServerCache 実装では、バッキング ストアとして SQL Server データベースを使用する分散キャッシュを許可します。</span><span class="sxs-lookup"><span data-stu-id="4698e-162">The SqlServerCache implementation allows the distributed cache to use a SQL Server database as its backing store.</span></span> <span data-ttu-id="4698e-163">SQL サーバーを作成するには、ツール、sql キャッシュを使用するテーブルは指定した名前とスキーマを持つテーブルを作成します。</span><span class="sxs-lookup"><span data-stu-id="4698e-163">To create SQL Server table you can use sql-cache tool, the tool creates a table with the name and schema you specify.</span></span>

<span data-ttu-id="4698e-164">Sql キャッシュのツールを使用する追加`SqlConfig.Tools`を`<ItemGroup>`の要素、 *.csproj*ファイル dotnet 復元を実行しています。</span><span class="sxs-lookup"><span data-stu-id="4698e-164">To use the sql-cache tool, add `SqlConfig.Tools` to the `<ItemGroup>` element of the *.csproj* file and run dotnet restore.</span></span>

<span data-ttu-id="4698e-165">[!code-csharp[Main](./distributed/sample/src/DistCacheSample/DistCacheSample.csproj?range=23-25)]</span><span class="sxs-lookup"><span data-stu-id="4698e-165">[!code-csharp[Main](./distributed/sample/src/DistCacheSample/DistCacheSample.csproj?range=23-25)]</span></span>

<span data-ttu-id="4698e-166">SqlConfig.Tools をテストするには、次のコマンドを実行</span><span class="sxs-lookup"><span data-stu-id="4698e-166">Test SqlConfig.Tools by running the following command</span></span>

```none
C:\DistCacheSample\src\DistCacheSample>dotnet sql-cache create --help
   ```

<span data-ttu-id="4698e-167">sql キャッシュ ツール「sql キャッシュの作成」コマンドを実行している sql server にテーブルを作成することができますので使用状況、オプションおよびコマンドのヘルプが表示されます。</span><span class="sxs-lookup"><span data-stu-id="4698e-167">sql-cache tool  will display usage, options and command help, now you can create tables into sql server, running "sql-cache create" command :</span></span>

```none
C:\DistCacheSample\src\DistCacheSample>dotnet sql-cache create "Data Source=(localdb)\v11.0;Initial Catalog=DistCache;Integrated Security=True;" dbo TestCache
   info: Microsoft.Extensions.Caching.SqlConfig.Tools.Program[0]
       Table and index were created successfully.
   ```

<span data-ttu-id="4698e-168">作成されたテーブルには、次のスキーマがあります。</span><span class="sxs-lookup"><span data-stu-id="4698e-168">The created table has the following schema:</span></span>

![SqlServer キャッシュ テーブル](distributed/_static/SqlServerCacheTable.png)

<span data-ttu-id="4698e-170">すべてのキャッシュ実装と同様にアプリを取得および設定のインスタンスを使用してキャッシュ値`IDistributedCache`ではなく、`SqlServerCache`です。</span><span class="sxs-lookup"><span data-stu-id="4698e-170">Like all cache implementations, your app should get and set cache values using an instance of `IDistributedCache`, not a `SqlServerCache`.</span></span> <span data-ttu-id="4698e-171">このサンプルを実装`SqlServerCache`で、`Production`環境 (で構成されているように`ConfigureProductionServices`)。</span><span class="sxs-lookup"><span data-stu-id="4698e-171">The sample implements `SqlServerCache` in the `Production` environment (so it is configured in `ConfigureProductionServices`).</span></span>

<span data-ttu-id="4698e-172">[!code-csharp[Main](./distributed/sample/src/DistCacheSample/Startup.cs?highlight=7,8,9,10,11,12&range=42-56)]</span><span class="sxs-lookup"><span data-stu-id="4698e-172">[!code-csharp[Main](./distributed/sample/src/DistCacheSample/Startup.cs?highlight=7,8,9,10,11,12&range=42-56)]</span></span>

> [!NOTE]
> <span data-ttu-id="4698e-173">`ConnectionString` (および必要に応じて、`SchemaName`と`TableName`) 資格情報を含めることは、通常 (UserSecrets) などのソース管理の外部に格納する必要があります。</span><span class="sxs-lookup"><span data-stu-id="4698e-173">The `ConnectionString` (and optionally, `SchemaName` and `TableName`) should typically be stored outside of source control (such as UserSecrets), as they may contain credentials.</span></span>

## <a name="recommendations"></a><span data-ttu-id="4698e-174">推奨事項</span><span class="sxs-lookup"><span data-stu-id="4698e-174">Recommendations</span></span>

<span data-ttu-id="4698e-175">のどの実装を決定する際に`IDistributedCache`Redis 間を選択して、アプリの権利は、既存のインフラストラクチャと環境、パフォーマンス要件、およびチームの経験に基づいて、SQL Server、します。</span><span class="sxs-lookup"><span data-stu-id="4698e-175">When deciding which implementation of `IDistributedCache` is right for your app, choose between Redis and SQL Server based on your existing infrastructure and environment, your performance requirements, and your team's experience.</span></span> <span data-ttu-id="4698e-176">チームがより快適な Redis の操作の場合は、最適な選択肢であります。</span><span class="sxs-lookup"><span data-stu-id="4698e-176">If your team is more comfortable working with Redis, it's an excellent choice.</span></span> <span data-ttu-id="4698e-177">チームが SQL Server を希望する場合で同様に実装することを確信できます。</span><span class="sxs-lookup"><span data-stu-id="4698e-177">If your team prefers SQL Server, you can be confident in that implementation as well.</span></span> <span data-ttu-id="4698e-178">従来のキャッシュ ソリューションがデータをメモリ内データの取得を高速を格納することに注意してください。</span><span class="sxs-lookup"><span data-stu-id="4698e-178">Note that A traditional caching solution stores data in-memory which allows for fast retrieval of data.</span></span> <span data-ttu-id="4698e-179">一般的に使用されるデータをキャッシュに格納し、SQL Server または Azure ストレージなどのバックエンドの永続的なストアでデータ全体を格納する必要があります。</span><span class="sxs-lookup"><span data-stu-id="4698e-179">You should store commonly used data in a cache and store the entire data in a backend persistent store such as SQL Server or Azure Storage.</span></span> <span data-ttu-id="4698e-180">Redis Cache は、SQL キャッシュと比較して、高いスループットと低待機時間ににより、キャッシュ ソリューションです。</span><span class="sxs-lookup"><span data-stu-id="4698e-180">Redis Cache is a caching solution which gives you high throughput and low latency as compared to SQL Cache.</span></span>

<span data-ttu-id="4698e-181">その他のリソース:</span><span class="sxs-lookup"><span data-stu-id="4698e-181">Additional resources:</span></span>

* [<span data-ttu-id="4698e-182">メモリ キャッシュでは、</span><span class="sxs-lookup"><span data-stu-id="4698e-182">In memory caching</span></span>](memory.md)
* [<span data-ttu-id="4698e-183">Redis の Azure キャッシュ</span><span class="sxs-lookup"><span data-stu-id="4698e-183">Redis Cache on Azure</span></span>](https://azure.microsoft.com/documentation/services/redis-cache/)
* [<span data-ttu-id="4698e-184">Azure SQL データベース</span><span class="sxs-lookup"><span data-stu-id="4698e-184">SQL Database on Azure</span></span>](https://azure.microsoft.com/documentation/services/sql-database/)