---
uid: web-api/overview/data/using-web-api-with-entity-framework/part-6
title: JavaScript クライアントを作成 |Microsoft ドキュメント
author: MikeWasson
description: ''
ms.author: aspnetcontent
manager: wpickett
ms.date: 06/16/2014
ms.topic: article
ms.assetid: 20360326-b123-4b1e-abae-1d350edf4ce4
ms.technology: dotnet-webapi
ms.prod: .net-framework
msc.legacyurl: /web-api/overview/data/using-web-api-with-entity-framework/part-6
msc.type: authoredcontent
ms.openlocfilehash: 29d50e448e6d282c7db06b9d1946ac221347e1ea
ms.sourcegitcommit: f8852267f463b62d7f975e56bea9aa3f68fbbdeb
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/06/2018
---
<a name="create-the-javascript-client"></a>JavaScript クライアントを作成します。
====================
作成者 [Mike Wasson](https://github.com/MikeWasson)

[完成したプロジェクトのダウンロード](https://github.com/MikeWasson/BookService)

このセクションでは、HTML、JavaScript を使用して、アプリケーション用のクライアントを作成し、 [Knockout.js](http://knockoutjs.com/)ライブラリです。 クライアント アプリのビルドは段階的に。

- 書籍の一覧を表示しています。
- 書籍の詳細を表示しています。
- 新しいブックを追加します。

Knockout ライブラリは、モデル View-viewmodel (MVVM) パターンを使用します。

- **モデル**(当社の場合、ブックおよび作成者) に、ビジネス ドメイン内のデータのサーバー側表現です。
- **ビュー**は、プレゼンテーション層 (HTML)。
- **ビュー モデル**JavaScript オブジェクト モデルを保持します。 ビュー モデルは、UI のコードの抽象化です。 HTML 形式の知識がないです。 代わりに、そのなどを表す抽象機能、ビューの&quot;書籍の一覧&quot;です。

ビュー データにバインドされてビュー モデルです。 ビュー モデルへの更新は、ビューに自動的に反映されます。 ビュー モデルは、ボタンのクリックしてなどもイベントと、ビューから取得します。

![](part-6/_static/image1.png)

この方法が便利レイアウトと、アプリの UI を変更する任意のコードを書き直すことがなく、バインドを変更できるためです。 たとえば、としてアイテムの一覧を表示する場合があります、 `<ul>`、後で、テーブルに変更します。

## <a name="add-the-knockout-library"></a>Knockout ライブラリを追加します。

Visual Studio から、**ツール**メニューの **ライブラリ パッケージ マネージャー**です。 選択し、 **Package Manager Console**です。 パッケージ マネージャー コンソール ウィンドウで、次のコマンドを入力します。

[!code-console[Main](part-6/samples/sample1.cmd)]

このコマンドは、Scripts フォルダーに Knockout ファイルを追加します。

## <a name="create-the-view-model"></a>ビュー モデルを作成します。

スクリプト フォルダーに app.js をという名前の JavaScript ファイルを追加します。 (ソリューション エクスプ ローラーで、Scripts フォルダーを右クリックし、選択**追加**選択してから、 **JavaScript ファイル**)。次のコードを貼り付けます。

[!code-javascript[Main](part-6/samples/sample2.js)]

Knockout で、`observable`クラスは、データ バインディングを使用します。 監視対象の内容を変更するときに、観測可能なオブジェクトに通知のすべてのデータ バインド コントロール自体を更新できるようにします。 (、`observableArray`クラスは、配列のバージョンの*observable*)。開始するには、ビュー モデルには、次の 2 つの観測可能なオブジェクトがあります。

- `books` 書籍の一覧を保持します。
- `error` AJAX 呼び出しが失敗した場合は、エラー メッセージに含まれています。

`getAllBooks`メソッドは、書籍の一覧を取得するための AJAX 呼び出しを使用します。 結果をプッシュし、`books`配列。

`ko.applyBindings`メソッドは、Knockout ライブラリの一部です。 ビュー モデルとしてパラメーターを受け取るし、データ バインディングを設定します。

## <a name="add-a-script-bundle"></a>スクリプト バンドルを追加します。

バンドルは、ASP.NET 4.5 を結合または 1 つのファイルを複数のファイルにバンドルしやすく機能です。 ページ読み込み時間を向上させることができるサーバーに要求の数を削減するバンドルします。

アプリのファイルを開く\_Start/BundleConfig.cs です。 RegisterBundles メソッドに次のコードを追加します。

[!code-csharp[Main](part-6/samples/sample3.cs)]

> [!div class="step-by-step"]
> [前へ](part-5.md)
> [次へ](part-7.md)
