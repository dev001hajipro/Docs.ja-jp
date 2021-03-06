---
uid: web-forms/overview/ajax-control-toolkit/animation/picking-one-animation-out-of-a-list-cs
title: (C#) リストから 1 つのアニメーションをピッキング |Microsoft ドキュメント
author: wenz
description: アニメーション コントロール、ASP.NET AJAX コントロール Toolkit ではなくコントロールだけアニメーションをコントロールに追加するために全体のフレームワークです。 フレームワークも許可しています.
ms.author: aspnetcontent
manager: wpickett
ms.date: 06/02/2008
ms.topic: article
ms.assetid: 06a776fe-7c73-4ca7-8e02-5260a86edc03
ms.technology: dotnet-webforms
ms.prod: .net-framework
msc.legacyurl: /web-forms/overview/ajax-control-toolkit/animation/picking-one-animation-out-of-a-list-cs
msc.type: authoredcontent
ms.openlocfilehash: 2d4aac447fcdfbf296560091cfcdf5eb51997a7b
ms.sourcegitcommit: f8852267f463b62d7f975e56bea9aa3f68fbbdeb
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/06/2018
---
<a name="picking-one-animation-out-of-a-list-c"></a>選択リスト (c#) から 1 つのアニメーション
====================
によって[Christian Wenz](https://github.com/wenz)

[コードをダウンロードする](http://download.microsoft.com/download/f/9/a/f9a26acd-8df4-4484-8a18-199e4598f411/Animation5.cs.zip)または[PDF のダウンロード](http://download.microsoft.com/download/6/7/1/6718d452-ff89-4d3f-a90e-c74ec2d636a3/animation5CS.pdf)

> アニメーション コントロール、ASP.NET AJAX コントロール Toolkit ではなくコントロールだけアニメーションをコントロールに追加するために全体のフレームワークです。 フレームワークでは、プログラマによっては一部の JavaScript コードの評価、アニメーションのリストから 1 つのアニメーションを選択することもできます。


## <a name="overview"></a>概要

アニメーション コントロール、ASP.NET AJAX コントロール Toolkit ではなくコントロールだけアニメーションをコントロールに追加するために全体のフレームワークです。 フレームワークでは、プログラマによっては一部の JavaScript コードの評価、アニメーションのリストから 1 つのアニメーションを選択することもできます。

## <a name="steps"></a>手順

まず、含める、 `ScriptManager` ; ページで次に、ASP.NET AJAX ライブラリが読み込まれるコントロール ツールキットを使用できるようにします。

[!code-aspx[Main](picking-one-animation-out-of-a-list-cs/samples/sample1.aspx)]

アニメーションは、次のようなテキストのパネルに適用されます。

[!code-aspx[Main](picking-one-animation-out-of-a-list-cs/samples/sample2.aspx)]

パネルの関連付けられている CSS クラス、nice の背景色を定義し、パネルの固定幅を設定します。

[!code-css[Main](picking-one-animation-out-of-a-list-cs/samples/sample3.css)]

次に、追加、`AnimationExtender`のページを提供する、 `ID`、`TargetControlID`属性と、任意 `runat="server":`

[!code-aspx[Main](picking-one-animation-out-of-a-list-cs/samples/sample4.aspx)]

内で、`<Animations>`ノードを使用して`<OnLoad>`ページが完全に読み込まれた後に、アニメーションを実行します。 正規のアニメーションでは、いずれかではなく、`<Case>`要素が関係します。 その SelectScript 属性の値が評価されます。戻り値を数値にする必要があります。 この数は、サブアニメーション内のいずれかによって&lt;ケース&gt;を実行します。 たとえば、SelectScript は 2 に評価されると、コントロール Toolkit の実行内で 3 番目のアニメーション&lt;ケース&gt;(カウント 0 から始まる)。

次のマークアップを次の 3 つサブアニメーションを定義します。 幅をサイズ変更、サイズの高さを変更するとフェードアウトします。JavaScript コード (`Math.floor(3 * Math.random())`) 3 つのアニメーションのいずれかが実行されるため、0 ~ 2 の数値を取得します。

[!code-aspx[Main](picking-one-animation-out-of-a-list-cs/samples/sample5.aspx)]


[![考えられる 3 つのアニメーションのいずれかの: パネルが広くなります。](picking-one-animation-out-of-a-list-cs/_static/image2.png)](picking-one-animation-out-of-a-list-cs/_static/image1.png)

考えられる 3 つのアニメーションのいずれかの: パネルが広くなる ([フルサイズのイメージを表示するをクリックして](picking-one-animation-out-of-a-list-cs/_static/image3.png))

> [!div class="step-by-step"]
> [前へ](animation-depending-on-a-condition-cs.md)
> [次へ](animating-in-response-to-user-interaction-cs.md)
