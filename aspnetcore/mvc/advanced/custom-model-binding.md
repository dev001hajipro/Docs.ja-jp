---
title: "カスタム モデル バインディング"
author: ardalis
description: "ASP.NET Core mvc モデル バインディングをカスタマイズします。"
keywords: "ASP.NET Core、モデル バインディング、カスタム モデル バインダー"
ms.author: riande
manager: wpickett
ms.date: 4/10/2017
ms.topic: article
ms.assetid: ebd98159-a028-4a94-b06c-43981c79c6be
ms.technology: aspnet
ms.prod: asp.net-core
uid: mvc/advanced/custom-model-binding
ms.openlocfilehash: 8dae9cf1597ae2bddf2943af4a7ab9d50620825b
ms.sourcegitcommit: 26166785ad181a8519cb008800d71d96499b0499
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/01/2017
---
# <a name="custom-model-binding"></a><span data-ttu-id="ab595-104">カスタム モデル バインディング</span><span class="sxs-lookup"><span data-stu-id="ab595-104">Custom Model Binding</span></span>

<span data-ttu-id="ab595-105">によって[Steve Smith](http://ardalis.com)</span><span class="sxs-lookup"><span data-stu-id="ab595-105">By [Steve Smith](http://ardalis.com)</span></span>

<span data-ttu-id="ab595-106">モデル バインディングでは、モデルの種類 (渡されるとメソッドの引数として) ではなく HTTP 要求よりもを直接操作するコント ローラーのアクションを許可します。</span><span class="sxs-lookup"><span data-stu-id="ab595-106">Model binding allows controller actions to work directly with model types (passed in as method arguments), rather than HTTP requests.</span></span> <span data-ttu-id="ab595-107">入力方向の要求データとアプリケーションのモデル間のマッピングは、モデル バインダーによって処理されます。</span><span class="sxs-lookup"><span data-stu-id="ab595-107">Mapping between incoming request data and application models is handled by model binders.</span></span> <span data-ttu-id="ab595-108">開発者は、カスタム モデル バインダー (ただし、通常、独自のプロバイダーを記述する必要はありません) を実装することによって、組み込みのモデル バインディング機能を拡張できます。</span><span class="sxs-lookup"><span data-stu-id="ab595-108">Developers can extend the built-in model binding functionality by implementing custom model binders (though typically, you don't need to write your own provider).</span></span>

[<span data-ttu-id="ab595-109">GitHub から表示またはダウンロードのサンプル</span><span class="sxs-lookup"><span data-stu-id="ab595-109">View or download sample from GitHub</span></span>](https://github.com/aspnet/Docs/tree/master/aspnetcore/mvc/advanced/custom-model-binding/)

## <a name="default-model-binder-limitations"></a><span data-ttu-id="ab595-110">既定のモデル バインダーの制限事項</span><span class="sxs-lookup"><span data-stu-id="ab595-110">Default model binder limitations</span></span>

<span data-ttu-id="ab595-111">既定のモデル バインダーは、ほとんどの一般的な .NET Core データ型をサポートし、ほとんどの開発者のニーズを満たす必要があります。</span><span class="sxs-lookup"><span data-stu-id="ab595-111">The default model binders support most of the common .NET Core data types and should meet most developers\` needs.</span></span> <span data-ttu-id="ab595-112">テキスト ベースの入力を要求の種類のモデルに直接バインドを享受できます。</span><span class="sxs-lookup"><span data-stu-id="ab595-112">They expect to bind text-based input from the request directly to model types.</span></span> <span data-ttu-id="ab595-113">それをバインドする前に、入力を変換する必要があります。</span><span class="sxs-lookup"><span data-stu-id="ab595-113">You might need to transform the input prior to binding it.</span></span> <span data-ttu-id="ab595-114">たとえばがある場合、モデル データの検索に使用できるキー。</span><span class="sxs-lookup"><span data-stu-id="ab595-114">For example, when you have a key that can be used to look up model data.</span></span> <span data-ttu-id="ab595-115">キーに基づいてデータをフェッチするのにカスタム モデル バインダーを使用することができます。</span><span class="sxs-lookup"><span data-stu-id="ab595-115">You can use a custom model binder to fetch data based on the key.</span></span>

## <a name="model-binding-review"></a><span data-ttu-id="ab595-116">モデル バインディングのレビュー</span><span class="sxs-lookup"><span data-stu-id="ab595-116">Model binding review</span></span>

<span data-ttu-id="ab595-117">モデル バインドは、上で動作の種類に固有の定義を使用します。</span><span class="sxs-lookup"><span data-stu-id="ab595-117">Model binding uses specific definitions for the types it operates on.</span></span> <span data-ttu-id="ab595-118">A*単純型*が入力内の 1 つの文字列から変換されます。</span><span class="sxs-lookup"><span data-stu-id="ab595-118">A *simple type* is converted from a single string in the input.</span></span> <span data-ttu-id="ab595-119">A*複合型*が複数の入力値から変換されます。</span><span class="sxs-lookup"><span data-stu-id="ab595-119">A *complex type* is converted from multiple input values.</span></span> <span data-ttu-id="ab595-120">フレームワークの存在に基づいて差を決定する、`TypeConverter`です。</span><span class="sxs-lookup"><span data-stu-id="ab595-120">The framework determines the difference based on the existence of a `TypeConverter`.</span></span> <span data-ttu-id="ab595-121">単純ながある場合に実行する型コンバーターを作成することをお勧めします。 `string`  ->  `SomeType`外部リソースを必要としないマッピングします。</span><span class="sxs-lookup"><span data-stu-id="ab595-121">We recommended you create a type converter if you have a simple `string` -> `SomeType` mapping that doesn't require external resources.</span></span>

<span data-ttu-id="ab595-122">独自のカスタム モデル バインダーを作成する前にどのように既存のモデルを確認する価値のバインダーが実装されているをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="ab595-122">Before creating your own custom model binder, it's worth reviewing how existing model binders are implemented.</span></span> <span data-ttu-id="ab595-123">検討してください、 [ByteArrayModelBinder](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.modelbinding.binders.bytearraymodelbinder)これは、base64 でエンコードされた文字列をバイト配列に変換を使用できます。</span><span class="sxs-lookup"><span data-stu-id="ab595-123">Consider the [ByteArrayModelBinder](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.modelbinding.binders.bytearraymodelbinder) which can be used to convert base64-encoded strings into byte arrays.</span></span> <span data-ttu-id="ab595-124">バイト配列は多くの場合、ファイルまたはデータベース BLOB フィールドとして格納されます。</span><span class="sxs-lookup"><span data-stu-id="ab595-124">The byte arrays are often stored as files or database BLOB fields.</span></span>

### <a name="working-with-the-bytearraymodelbinder"></a><span data-ttu-id="ab595-125">ByteArrayModelBinder の操作</span><span class="sxs-lookup"><span data-stu-id="ab595-125">Working with the ByteArrayModelBinder</span></span>

<span data-ttu-id="ab595-126">バイナリ データを表す、Base64 でエンコードされた文字列を使用できます。</span><span class="sxs-lookup"><span data-stu-id="ab595-126">Base64-encoded strings can be used to represent binary data.</span></span> <span data-ttu-id="ab595-127">たとえば、次の図は、文字列としてエンコードできます。</span><span class="sxs-lookup"><span data-stu-id="ab595-127">For example, the following image can be encoded as a string.</span></span>

<span data-ttu-id="ab595-128">![dotnet bot](custom-model-binding/images/bot.png "dotnet bot")</span><span class="sxs-lookup"><span data-stu-id="ab595-128">![dotnet bot](custom-model-binding/images/bot.png "dotnet bot")</span></span>

<span data-ttu-id="ab595-129">エンコードされた文字列の一部は、次の図で示されます。</span><span class="sxs-lookup"><span data-stu-id="ab595-129">A small portion of the encoded string is shown in the following image:</span></span>

<span data-ttu-id="ab595-130">![エンコードされた dotnet bot](custom-model-binding/images/encoded-bot.png "dotnet bot エンコード")</span><span class="sxs-lookup"><span data-stu-id="ab595-130">![dotnet bot encoded](custom-model-binding/images/encoded-bot.png "dotnet bot encoded")</span></span>

<span data-ttu-id="ab595-131">指示に従って、[サンプルの README](https://github.com/aspnet/Docs/blob/master/aspnetcore/mvc/advanced/custom-model-binding/sample/CustomModelBindingSample/README.md)を base64 でエンコードされた文字列をファイルに変換します。</span><span class="sxs-lookup"><span data-stu-id="ab595-131">Follow the instructions in the [sample's README](https://github.com/aspnet/Docs/blob/master/aspnetcore/mvc/advanced/custom-model-binding/sample/CustomModelBindingSample/README.md) to convert the base64-encoded string into a file.</span></span>

<span data-ttu-id="ab595-132">ASP.NET Core MVC の base64 でエンコードされた文字列を撮影して使用することができます、`ByteArrayModelBinder`バイト配列に変換します。</span><span class="sxs-lookup"><span data-stu-id="ab595-132">ASP.NET Core MVC can take a base64-encoded strings and use a `ByteArrayModelBinder` to convert it into a byte array.</span></span> <span data-ttu-id="ab595-133">[ByteArrayModelBinderProvider](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.modelbinding.binders.bytearraymodelbinderprovider)を実装する[IModelBinderProvider](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.modelbinding.imodelbinderprovider)マップ`byte[]`引数`ByteArrayModelBinder`:</span><span class="sxs-lookup"><span data-stu-id="ab595-133">The [ByteArrayModelBinderProvider](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.modelbinding.binders.bytearraymodelbinderprovider) which implements [IModelBinderProvider](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.modelbinding.imodelbinderprovider) maps `byte[]` arguments to `ByteArrayModelBinder`:</span></span>

```csharp
public IModelBinder GetBinder(ModelBinderProviderContext context)
{
    if (context == null)
    {
        throw new ArgumentNullException(nameof(context));
    }

    if (context.Metadata.ModelType == typeof(byte[]))
    {
        return new ByteArrayModelBinder();
    }

    return null;
}
```

<span data-ttu-id="ab595-134">独自のカスタム モデル バインダーを作成するときにすることができます独自に実装`IModelBinderProvider`入力するかを使用して、 [ModelBinderAttribute](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.modelbinderattribute)です。</span><span class="sxs-lookup"><span data-stu-id="ab595-134">When creating your own custom model binder, you can implement your own `IModelBinderProvider` type, or use the [ModelBinderAttribute](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.modelbinderattribute).</span></span>

<span data-ttu-id="ab595-135">使用する次の例に示します`ByteArrayModelBinder`を base64 でエンコードされた文字列に変換する、`byte[]`し、結果をファイルに保存します。</span><span class="sxs-lookup"><span data-stu-id="ab595-135">The following example shows how to use `ByteArrayModelBinder` to convert a base64-encoded string to a `byte[]` and save the result to a file:</span></span>

<span data-ttu-id="ab595-136">[!code-csharp[Main](custom-model-binding/sample/CustomModelBindingSample/Controllers/ImageController.cs?name=post1&highlight=3)]</span><span class="sxs-lookup"><span data-stu-id="ab595-136">[!code-csharp[Main](custom-model-binding/sample/CustomModelBindingSample/Controllers/ImageController.cs?name=post1&highlight=3)]</span></span>

<span data-ttu-id="ab595-137">などのツールを使用してこの api メソッドに base64 でエンコードされた文字列を投稿する[Postman](https://www.getpostman.com/):</span><span class="sxs-lookup"><span data-stu-id="ab595-137">You can POST a base64-encoded string to this api method using a tool like [Postman](https://www.getpostman.com/):</span></span>

<span data-ttu-id="ab595-138">![postman](custom-model-binding/images/postman.png "postman")</span><span class="sxs-lookup"><span data-stu-id="ab595-138">![postman](custom-model-binding/images/postman.png "postman")</span></span>

<span data-ttu-id="ab595-139">バインダーは、要求データを適切に名前付きプロパティまたは引数をバインドできます、限りモデル バインドは成功します。</span><span class="sxs-lookup"><span data-stu-id="ab595-139">As long as the binder can bind request data to appropriately named properties or arguments, model binding will succeed.</span></span> <span data-ttu-id="ab595-140">次の例は、使用する方法を示しています。`ByteArrayModelBinder`ビュー モデルを含む。</span><span class="sxs-lookup"><span data-stu-id="ab595-140">The following example shows how to use `ByteArrayModelBinder` with a view model:</span></span>

<span data-ttu-id="ab595-141">[!code-csharp[Main](custom-model-binding/sample/CustomModelBindingSample/Controllers/ImageController.cs?name=post2&highlight=2)]</span><span class="sxs-lookup"><span data-stu-id="ab595-141">[!code-csharp[Main](custom-model-binding/sample/CustomModelBindingSample/Controllers/ImageController.cs?name=post2&highlight=2)]</span></span>

## <a name="custom-model-binder-sample"></a><span data-ttu-id="ab595-142">カスタム モデル バインダーのサンプル</span><span class="sxs-lookup"><span data-stu-id="ab595-142">Custom model binder sample</span></span>

<span data-ttu-id="ab595-143">このセクションで、カスタム モデル バインダーを実装します。</span><span class="sxs-lookup"><span data-stu-id="ab595-143">In this section we'll implement a custom model binder that:</span></span>

- <span data-ttu-id="ab595-144">受信した要求データを厳密に型指定されたキーの引数に変換します。</span><span class="sxs-lookup"><span data-stu-id="ab595-144">Converts incoming request data into strongly typed key arguments.</span></span>
- <span data-ttu-id="ab595-145">Entity Framework のコアを使用すると、関連するエンティティをフェッチします。</span><span class="sxs-lookup"><span data-stu-id="ab595-145">Uses Entity Framework Core to fetch the associated entity.</span></span>
- <span data-ttu-id="ab595-146">関連するエンティティを引数としてアクション メソッドに渡します。</span><span class="sxs-lookup"><span data-stu-id="ab595-146">Passes the associated entity as an argument to the action method.</span></span>

<span data-ttu-id="ab595-147">次のサンプルは、`ModelBinder`属性を`Author`モデル。</span><span class="sxs-lookup"><span data-stu-id="ab595-147">The following sample uses the `ModelBinder` attribute on the `Author` model:</span></span>

<span data-ttu-id="ab595-148">[!code-csharp[Main](custom-model-binding/sample/CustomModelBindingSample/Data/Author.cs?highlight=10)]</span><span class="sxs-lookup"><span data-stu-id="ab595-148">[!code-csharp[Main](custom-model-binding/sample/CustomModelBindingSample/Data/Author.cs?highlight=10)]</span></span>

<span data-ttu-id="ab595-149">上記のコードでは、`ModelBinder`属性の型を指定`IModelBinder`バインドを使用する必要のある`Author`アクションのパラメーターです。</span><span class="sxs-lookup"><span data-stu-id="ab595-149">In the preceding code, the `ModelBinder` attribute specifies the type of `IModelBinder` that should be used to bind `Author` action parameters.</span></span> 

<span data-ttu-id="ab595-150">`AuthorEntityBinder`に連結するため、`Author`エンティティ フレームワークのコアを使用してデータ ソースからエンティティをフェッチしてパラメーターと`authorId`:</span><span class="sxs-lookup"><span data-stu-id="ab595-150">The `AuthorEntityBinder` is used to bind an `Author` parameter by fetching the entity from a data source using Entity Framework Core and an `authorId`:</span></span>

<span data-ttu-id="ab595-151">[!code-csharp[Main](custom-model-binding/sample/CustomModelBindingSample/Binders/AuthorEntityBinder.cs?name=demo)]</span><span class="sxs-lookup"><span data-stu-id="ab595-151">[!code-csharp[Main](custom-model-binding/sample/CustomModelBindingSample/Binders/AuthorEntityBinder.cs?name=demo)]</span></span>

<span data-ttu-id="ab595-152">次のコードを使用する方法を示しています、`AuthorEntityBinder`アクション メソッドで。</span><span class="sxs-lookup"><span data-stu-id="ab595-152">The following code shows how to use the `AuthorEntityBinder` in an action method:</span></span>

<span data-ttu-id="ab595-153">[!code-csharp[Main](custom-model-binding/sample/CustomModelBindingSample/Controllers/BoundAuthorsController.cs?name=demo2&highlight=2)]</span><span class="sxs-lookup"><span data-stu-id="ab595-153">[!code-csharp[Main](custom-model-binding/sample/CustomModelBindingSample/Controllers/BoundAuthorsController.cs?name=demo2&highlight=2)]</span></span>

<span data-ttu-id="ab595-154">`ModelBinder`を適用する属性を使用することができます、`AuthorEntityBinder`パラメーターを既定の規則を使用しません。</span><span class="sxs-lookup"><span data-stu-id="ab595-154">The `ModelBinder` attribute can be used to apply the `AuthorEntityBinder` to parameters that do not use default conventions:</span></span>

<span data-ttu-id="ab595-155">[!code-csharp[Main](custom-model-binding/sample/CustomModelBindingSample/Controllers/BoundAuthorsController.cs?name=demo1&highlight=2)]</span><span class="sxs-lookup"><span data-stu-id="ab595-155">[!code-csharp[Main](custom-model-binding/sample/CustomModelBindingSample/Controllers/BoundAuthorsController.cs?name=demo1&highlight=2)]</span></span>

<span data-ttu-id="ab595-156">引数の名前は、既定ではないため、この例では`authorId`、パラメーターを使用して、指定されている`ModelBinder`属性。</span><span class="sxs-lookup"><span data-stu-id="ab595-156">In this example, since the name of the argument is not the default `authorId`, it's specified on the parameter using `ModelBinder` attribute.</span></span> <span data-ttu-id="ab595-157">コント ローラーとアクションの両方のメソッドは、アクション メソッド内のエンティティの検索と比較して簡素化することに注意してください。</span><span class="sxs-lookup"><span data-stu-id="ab595-157">Note that both the controller and action method are simplified compared to looking up the entity in the action method.</span></span> <span data-ttu-id="ab595-158">Entity Framework のコアを使用して、作成者をフェッチするためのロジックは、モデル バインダーに移動されます。</span><span class="sxs-lookup"><span data-stu-id="ab595-158">The logic to fetch the author using Entity Framework Core is moved to the model binder.</span></span> <span data-ttu-id="ab595-159">作成者モデルにバインドし、次に役立つことがいくつかのメソッドがあるときはかなり単純化できます、[ドライ原則](http://deviq.com/don-t-repeat-yourself/)です。</span><span class="sxs-lookup"><span data-stu-id="ab595-159">This can be considerable simplification when you have several methods that bind to the author model, and can help you to follow the [DRY principle](http://deviq.com/don-t-repeat-yourself/).</span></span>

<span data-ttu-id="ab595-160">適用することができます、`ModelBinder`属性個々 のモデルのプロパティを (など、viewmodel で) またはアクション メソッドのパラメーターを特定のモデル バインダーまたはその型またはアクションだけのモデル名を指定します。</span><span class="sxs-lookup"><span data-stu-id="ab595-160">You can apply the `ModelBinder` attribute to individual model properties (such as on a viewmodel) or to action method parameters to specify a certain model binder or model name for just that type or action.</span></span>

### <a name="implementing-a-modelbinderprovider"></a><span data-ttu-id="ab595-161">ModelBinderProvider を実装します。</span><span class="sxs-lookup"><span data-stu-id="ab595-161">Implementing a ModelBinderProvider</span></span>

<span data-ttu-id="ab595-162">実装する属性を適用するには、代わりに`IModelBinderProvider`です。</span><span class="sxs-lookup"><span data-stu-id="ab595-162">Instead of applying an attribute, you can implement `IModelBinderProvider`.</span></span> <span data-ttu-id="ab595-163">これは、組み込みフレームワーク バインダーの実装方法です。</span><span class="sxs-lookup"><span data-stu-id="ab595-163">This is how the built-in framework binders are implemented.</span></span> <span data-ttu-id="ab595-164">型を指定する場合に、バインダーが動作するか、それによって生成される値の引数の型を指定する**いない**入力、バインダーを受け入れます。</span><span class="sxs-lookup"><span data-stu-id="ab595-164">When you specify the type your binder operates on, you specify the type of argument it produces, **not** the input your binder accepts.</span></span> <span data-ttu-id="ab595-165">次のバインダー プロバイダーが連動、`AuthorEntityBinder`です。</span><span class="sxs-lookup"><span data-stu-id="ab595-165">The following binder provider works with the `AuthorEntityBinder`.</span></span> <span data-ttu-id="ab595-166">使用する必要はありません、プロバイダーの MVC のコレクションに追加されたとき、`ModelBinder`属性に`Author`または`Author`パラメーターを入力します。</span><span class="sxs-lookup"><span data-stu-id="ab595-166">When it's added to MVC's collection of providers, you don't need to use the `ModelBinder` attribute on `Author` or `Author` typed parameters.</span></span>

<span data-ttu-id="ab595-167">[!code-csharp[Main](custom-model-binding/sample/CustomModelBindingSample/Binders/AuthorEntityBinderProvider.cs?highlight=17-20)]</span><span class="sxs-lookup"><span data-stu-id="ab595-167">[!code-csharp[Main](custom-model-binding/sample/CustomModelBindingSample/Binders/AuthorEntityBinderProvider.cs?highlight=17-20)]</span></span>

> <span data-ttu-id="ab595-168">注: 上記のコードを返します、`BinderTypeModelBinder`です。</span><span class="sxs-lookup"><span data-stu-id="ab595-168">Note: The preceding code returns a `BinderTypeModelBinder`.</span></span> <span data-ttu-id="ab595-169">`BinderTypeModelBinder`モデル バインダーのファクトリとして機能し、依存性の注入 (DI) を提供します。</span><span class="sxs-lookup"><span data-stu-id="ab595-169">`BinderTypeModelBinder` acts as a factory for model binders and provides dependency injection (DI).</span></span> <span data-ttu-id="ab595-170">`AuthorEntityBinder` DI EF コアにアクセスする必要があります。</span><span class="sxs-lookup"><span data-stu-id="ab595-170">The `AuthorEntityBinder` requires DI to access EF Core.</span></span> <span data-ttu-id="ab595-171">使用して`BinderTypeModelBinder`DI からサービスが、モデル バインダーに必要な場合です。</span><span class="sxs-lookup"><span data-stu-id="ab595-171">Use `BinderTypeModelBinder` if your model binder requires services from DI.</span></span>

<span data-ttu-id="ab595-172">カスタム モデル バインダー プロバイダーを使用する追加の`ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="ab595-172">To use a custom model binder provider, add it in `ConfigureServices`:</span></span>

<span data-ttu-id="ab595-173">[!code-csharp[Main](custom-model-binding/sample/CustomModelBindingSample/Startup.cs?name=callout&highlight=5-9)]</span><span class="sxs-lookup"><span data-stu-id="ab595-173">[!code-csharp[Main](custom-model-binding/sample/CustomModelBindingSample/Startup.cs?name=callout&highlight=5-9)]</span></span>

<span data-ttu-id="ab595-174">モデル バインダーを評価するときに、プロバイダーのコレクションは順序どおりにチェックします。</span><span class="sxs-lookup"><span data-stu-id="ab595-174">When evaluating model binders, the collection of providers is examined in order.</span></span> <span data-ttu-id="ab595-175">バインダーを返します最初のプロバイダーが使用されます。</span><span class="sxs-lookup"><span data-stu-id="ab595-175">The first provider that returns a binder is used.</span></span>

<span data-ttu-id="ab595-176">次の図は、デバッガーからのモデル バインダーに既定値を示します。</span><span class="sxs-lookup"><span data-stu-id="ab595-176">The following image shows the default model binders from the debugger.</span></span>

<span data-ttu-id="ab595-177">![既定のモデル バインダー](custom-model-binding/images/default-model-binders.png "既定のモデル バインダー")</span><span class="sxs-lookup"><span data-stu-id="ab595-177">![default model binders](custom-model-binding/images/default-model-binders.png "default model binders")</span></span>

<span data-ttu-id="ab595-178">コレクションの末尾には、プロバイダーを追加すると、組み込みのモデル バインダー前に、カスタムのバインダーに呼び出される可能性があります。</span><span class="sxs-lookup"><span data-stu-id="ab595-178">Adding your provider to the end of the collection may result in a built-in model binder being called before your custom binder has a chance.</span></span> <span data-ttu-id="ab595-179">この例では、カスタムのプロバイダーがの使用されることを確認するコレクションの先頭に追加される`Author`アクションの引数。</span><span class="sxs-lookup"><span data-stu-id="ab595-179">In this example, the custom provider is added to the beginning of the collection to ensure it is used for `Author` action arguments.</span></span>

<span data-ttu-id="ab595-180">[!code-csharp[Main](custom-model-binding/sample/CustomModelBindingSample/Startup.cs?name=callout&highlight=5-9)]</span><span class="sxs-lookup"><span data-stu-id="ab595-180">[!code-csharp[Main](custom-model-binding/sample/CustomModelBindingSample/Startup.cs?name=callout&highlight=5-9)]</span></span>

## <a name="recommendations-and-best-practices"></a><span data-ttu-id="ab595-181">推奨事項とベスト プラクティス</span><span class="sxs-lookup"><span data-stu-id="ab595-181">Recommendations and best practices</span></span>

<span data-ttu-id="ab595-182">カスタム モデル バインダー。</span><span class="sxs-lookup"><span data-stu-id="ab595-182">Custom model binders:</span></span>
- <span data-ttu-id="ab595-183">ステータス コードを設定するか、結果を返すにしないでください (たとえば、404 Not Found)。</span><span class="sxs-lookup"><span data-stu-id="ab595-183">Should not attempt to set status codes or return results (for example, 404 Not Found).</span></span> <span data-ttu-id="ab595-184">モデル バインドに失敗した場合、[アクション フィルター](xref:mvc/controllers/filters)自体アクション メソッド内のロジックは、エラーを処理する必要がありますか。</span><span class="sxs-lookup"><span data-stu-id="ab595-184">If model binding fails, an [action filter](xref:mvc/controllers/filters) or logic within the action method itself should handle the failure.</span></span>
- <span data-ttu-id="ab595-185">コードの繰り返しとアクション メソッドから横断的関心事を排除する最も便利です。</span><span class="sxs-lookup"><span data-stu-id="ab595-185">Are most useful for eliminating repetitive code and cross-cutting concerns from action methods.</span></span>
- <span data-ttu-id="ab595-186">通常、カスタムの型を文字列に変換には使用できません、 [ `TypeConverter` ](https://msdn.microsoft.com/library/ayybcxe5.aspx)は通常より良いオプション。</span><span class="sxs-lookup"><span data-stu-id="ab595-186">Typically should not be used to convert a string into a custom type, a [`TypeConverter`](https://msdn.microsoft.com/library/ayybcxe5.aspx) is usually a better option.</span></span>