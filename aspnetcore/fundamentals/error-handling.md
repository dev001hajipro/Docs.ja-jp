---
title: "ASP.NET Core のエラー処理"
author: ardalis
description: "ASP.NET Core アプリケーションでエラーを処理する方法について説明します。"
manager: wpickett
ms.author: tdykstra
ms.custom: H1Hack27Feb2017
ms.date: 11/30/2016
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: fundamentals/error-handling
ms.openlocfilehash: 5b0cda7b79b8a9523d1ba6a9b321d22d3ccc753a
ms.sourcegitcommit: 18d1dc86770f2e272d93c7e1cddfc095c5995d9e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/30/2018
---
# <a name="introduction-to-error-handling-in-aspnet-core"></a>ASP.NET Core でのエラー処理の概要

執筆: [Steve Smith](https://ardalis.com/)、[Tom Dykstra](https://github.com/tdykstra/)

この記事では、ASP.NET Core アプリでエラーを処理するための一般的な手法について取り上げます。

[サンプル コードを表示またはダウンロード](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/error-handling/sample)します ([ダウンロード方法](xref:tutorials/index#how-to-download-a-sample))。

## <a name="the-developer-exception-page"></a>開発者例外ページ

例外に関する詳細を表示するページを表示するようにアプリを構成するには、`Microsoft.AspNetCore.Diagnostics` NuGet パッケージをインストールし、[Startup クラスの Configure メソッド](startup.md)に次の行を追加します。

[!code-csharp[Main](error-handling/sample/Startup.cs?name=snippet_DevExceptionPage&highlight=7)]

`app.UseMvc` のように、例外をキャッチするミドルウェアの前に `UseDeveloperExceptionPage` を挿入します。

>[!WARNING]
> **アプリを開発環境で実行するときにのみ**、開発者例外ページを有効にします。 アプリを実稼働環境で実行するときは、詳細な例外情報を公開しません。 [環境の構成についてはこちらをご覧ください](environments.md)。

開発者例外ページを表示するには、環境を `Development` に設定してサンプル アプリケーションを実行し、アプリの基礎 URL に `?throw=true` を追加します。 このページには、例外と要求に関する情報を含むタブがいくつかあります。 最初のタブにはスタック トレースがあります。 

![スタック トレース](error-handling/_static/developer-exception-page.png)

次のタブには、クエリ文字列パラメーターがあればそれが表示されます。

![クエリ文字列のパラメーター](error-handling/_static/developer-exception-page-query.png)

この要求には Cookie が与えられませんでしたが、与えられた場合、**Cookies** タブに表示されます。最後のタブに渡されたヘッダーを表示できます。

![ヘッダー](error-handling/_static/developer-exception-page-headers.png)

## <a name="configuring-a-custom-exception-handling-page"></a>カスタム例外処理ページを構成する

アプリを `Development` 環境で実行しないときに使用する例外ハンドラー ページを構成することをお勧めします。

[!code-csharp[Main](error-handling/sample/Startup.cs?name=snippet_DevExceptionPage&highlight=11)]

MVC アプリでは、`HttpGet` など、HTTP メソッド属性でエラー ハンドラー アクション メソッドを明示的に修飾しないでください。 明示的な動詞を使用すると、要求がメソッドに届かないことがあります。

```csharp
[Route("/Error")]
public IActionResult Index()
{
    // Handle error here
}
```

## <a name="configuring-status-code-pages"></a>ステータス コード ページを構成する

アプリは既定で、500 (内部サーバー エラー) や 404 (見つかりません) など、HTTP ステータス コードをリッチ ページで表示しません。 `Configure` メソッドに次の行を追加することで `StatusCodePagesMiddleware` を構成できます。

```csharp
app.UseStatusCodePages();
```

既定では、このミドルウェアは、404 など、一般的なステータス コードに単純なテキストのみのハンドラーを追加します。

![404 ページ](error-handling/_static/default-404-status-code.png)

ミドルウェアは、さまざまな拡張メソッドに対応しています。 ラムダ式を受け取るものや、コンテンツの種類と書式設定文字列を受け取るものがあります。

[!code-csharp[Main](error-handling/sample/Startup.cs?name=snippet_StatusCodePages)]

```csharp
app.UseStatusCodePages("text/plain", "Status code page, status code: {0}");
```

リダイレクト拡張メソッドもあります。 302 ステータス コードをクライアントに送信するものや、元のステータス コードをクライアントに返すが、リダイレクト URL のハンドラーを実行するものがあります。

[!code-csharp[Main](error-handling/sample/Startup.cs?name=snippet_StatusCodePagesWithRedirect)]

```csharp
app.UseStatusCodePagesWithReExecute("/error/{0}");
```

特定の要求に対してステータス コード ページを無効にする場合、次の方法で可能です。

```csharp
var statusCodePagesFeature = context.Features.Get<IStatusCodePagesFeature>();
if (statusCodePagesFeature != null)
{
  statusCodePagesFeature.Enabled = false;
}
```

## <a name="exception-handling-code"></a>例外処理コード

例外処理ページのコードは例外をスローすることがあります。 実稼働のエラー ページは純粋に静的なコンテンツで構成することをお勧めします。

また、応答のヘッダーの送信後、応答のステータス コードを変更できなくなり、例外ページやハンドラーを実行できなくなることに注意してください。 応答は完了している必要があります。あるいは、接続が中止となっている必要があります。

## <a name="server-exception-handling"></a>サーバー例外処理

アプリの例外処理ロジックに加え、アプリをホストしている[サーバー](servers/index.md)がいくつかの例外処理を実行します。 サーバーがヘッダーの送信前に例外をキャッチすると、サーバーは 500 内部サーバー エラーを本文なしで送信します。 サーバーがヘッダーの送信後に例外をキャッチすると、サーバーは接続を閉じます。 アプリで処理されない要求はサーバーで処理されます。 例外が発生すると、サーバーの例外処理で処理されます。 カスタムのエラー ページ、例外処理ミドルウェア、フィルターを構成しても、この動作は変わりません。

## <a name="startup-exception-handling"></a>起動時の例外処理

アプリの起動中に起こる例外はホスティング層だけが処理できます。 `captureStartupErrors` と `detailedErrors` キーを利用し、[起動中のエラーに対するホストの動作を構成](hosting.md#detailed-errors)できます。

ホスティングでは、ホスト アドレス/ポート バインディング後にエラーが発生した場合のみ、キャプチャされた起動時エラーに対してエラー ページを表示できます。 何らかの理由でバインディングに失敗した場合、ホスティング層は重要な例外をログに記録し、dotnet プロセスがクラッシュします。エラー ページは表示されません。

## <a name="aspnet-mvc-error-handling"></a>ASP.NET MVC エラー処理

[MVC](xref:mvc/overview) アプリには、エラー処理の追加オプションがいくつかあります。例外フィルターの構成やモデル検証の実行などです。

### <a name="exception-filters"></a>例外フィルター

例外フィルターはグローバルに構成するか、MVC アプリのコントローラーまたはアクション単位で構成できます。 このようなフィルターはコントローラー アクションや別のフィルターの実行中に発生した例外が未処理のときにそれを処理し、それ以外では呼び出されません。 例外フィルターの詳細は[フィルター](../mvc/controllers/filters.md)で確認できます。

>[!TIP]
> 例外フィルターは、MVC アクション内で発生する例外をトラップするのには適していますが、エラー処理ミドルウェアほどの柔軟性はありません。 一般的なケースにはミドルウェアを選択し、MVC アクションで選択されたのとは*異なる*方法でエラー処理を行う必要がある場合にのみフィルターを使用します。

### <a name="handling-model-state-errors"></a>モデル状態エラーの処理

[モデル検証](../mvc/models/validation.md)は各コントローラー アクションを呼び出す前に行われます。`ModelState.IsValid` を検査し、適切に対処するのはアクション メソッドの仕事です。

一部のアプリでは、モデル検証エラーを処理するとき、標準的な規則に従います。その場合、そのようなポリシーを実装する場所として[フィルター](../mvc/controllers/filters.md)が適していることがあります。 無効なモデル状態でアクションの動作をテストしてください。 詳細については、[コントローラー ロジックのテスト](../mvc/controllers/testing.md)に関するページを参照してください。



