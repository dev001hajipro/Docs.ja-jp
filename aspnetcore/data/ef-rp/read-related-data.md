---
title: "Razor ページと EF Core - 関連データの読み取り - 6/8"
author: rick-anderson
description: "このチュートリアルでは、関連データ (Entity Framework がナビゲーション プロパティに読み込むデータ) の読み取りと表示を行います。"
manager: wpickett
ms.author: riande
ms.date: 11/05/2017
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: get-started-article
uid: data/ef-rp/read-related-data
ms.openlocfilehash: ccb1e95ae2b43fd0a4c4b1ac9ed58a4d474ab3b6
ms.sourcegitcommit: 18d1dc86770f2e272d93c7e1cddfc095c5995d9e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/31/2018
---
# <a name="reading-related-data---ef-core-with-razor-pages-6-of-8"></a>関連データの読み取り - EF Core と Razor ページ (6/8)

作成者: [Tom Dykstra](https://github.com/tdykstra)、[Jon P Smith](https://twitter.com/thereformedprog)、[Rick Anderson](https://twitter.com/RickAndMSFT)

[!INCLUDE[about the series](../../includes/RP-EF/intro.md)]

このチュートリアルでは、関連データが読み取られ、表示されます。 関連データとは、EF Core がナビゲーション プロパティに読み込むデータのことです。

解決できない問題が発生した場合は、[このステージの完成したアプリ](https://github.com/aspnet/Docs/tree/master/aspnetcore/data/ef-rp/intro/samples/StageSnapShots/cu-part6-related)をダウンロードしてください。

以下の図は、このチュートリアルの完成したページを示しています。

![Courses/Index ページ](read-related-data/_static/courses-index.png)

![Instructors/Index ページ](read-related-data/_static/instructors-index.png)

## <a name="eager-explicit-and-lazy-loading-of-related-data"></a>関連データの一括読み込み、明示的読み込み、遅延読み込み

EF Core がエンティティのナビゲーション プロパティに関連データを読み込むには、複数の方法があります。

* [一括読み込み](https://docs.microsoft.com/ef/core/querying/related-data#eager-loading)。 一括読み込みは、エンティティの 1 つの型に対するクエリが関連エンティティも読み込む場合です。 エンティティが読み取られるときに、その関連データが取得されます。 これは通常、必要なすべてのデータを取得する 1 つの結合クエリになります。 EF Core は、一部の型の一括読み込みに対して複数のクエリを発行します。 複数のクエリを発行することで、1 つのクエリしかなかった EF6 の一部のクエリよりも、効率を高めることができます。 一括読み込みは、`Include` メソッドと `ThenInclude` メソッドを使用して指定されます。

 ![一括読み込みの例](read-related-data/_static/eager-loading.png)
 
 一括読み込みでは、コレクション ナビゲーションが含まれるときに、複数のクエリが送信されます。

 * メイン クエリに 1 つのクエリ 
 * 読み込みツリー内のコレクション "エッジ" ごとに 1 つのクエリ

* `Load` で分離したクエリ: データは分離したクエリで取得でき、EF Core がナビゲーション プロパティを "修正" します。 "修正" は、ナビゲーション プロパティが EF Core によって自動的に入力されることを意味します。 `Load` で分離したクエリは、一括読み込みよりも明示的読み込みに似ています。

 ![分離したクエリの例](read-related-data/_static/separate-queries.png)

 注: EF Core は、コンテキスト インスタンスに以前に読み込まれたその他のエンティティに対して、ナビゲーション プロパティを自動的に修正します。 ナビゲーション プロパティのデータが明示的に含まれ*ない*場合でも、関連エンティティの一部またはすべてが以前に読み込まれていれば、プロパティを設定することができます。

* [明示的読み込み](https://docs.microsoft.com/ef/core/querying/related-data#explicit-loading)。 エンティティが最初に読み込まれるときに、関連データは取得されません。 必要なときに関連するデータを取得するコードを記述する必要があります。 分離したクエリによる明示的読み込みにより、複数のクエリが DB に送信されます。 明示的読み込みでは、コードで読み込まれるナビゲーション プロパティを指定します。 明示的読み込みを行うには、`Load` メソッドを使用します。 例:

 ![明示的読み込みの例](read-related-data/_static/explicit-loading.png)

* [遅延読み込み](https://docs.microsoft.com/ef/core/querying/related-data#lazy-loading)。 [EF Core では、現在、遅延読み込みをサポートしていません](https://github.com/aspnet/EntityFrameworkCore/issues/3797)。 エンティティが最初に読み込まれるときに、関連データは取得されません。 ナビゲーション プロパティに初めてアクセスすると、そのナビゲーション プロパティに必要なデータが自動的に取得されます。 初めてナビゲーション プロパティにアクセスされるたびに、クエリが DB に送信されます。

* `Select` 演算子は必要な関連データのみを読み込みます。

## <a name="create-a-courses-page-that-displays-department-name"></a>部門名を表示する Courses ページを作成する

Course エンティティには、`Department` エンティティを含むナビゲーション プロパティが含まれています。 `Department` エンティティには、コースが割り当てられる部門が含まれています。

コースの一覧で割り当てられている部門の名前を表示するには:

* `Department` エンティティから `Name` プロパティを取得します。
* `Department` エンティティは `Course.Department` ナビゲーション プロパティから取得されます。

![ourse.Department](read-related-data/_static/dep-crs.png)

<a name="scaffold"></a>
### <a name="scaffold-the-course-model"></a>Course モデルのスキャフォールディング

* Visual Studio を終了します。
* プロジェクト ディレクトリ (*Program.cs*、*Startup.cs*、および *.csproj* ファイルを含むディレクトリ) でコマンド ウィンドウを開きます。
* 次のコマンドを実行します。

 ```console
dotnet aspnet-codegenerator razorpage -m Course -dc SchoolContext -udl -outDir Pages\Courses --referenceScriptLibraries
 ```

上記のコマンドは、`Course` モデルをスキャフォールディングします。 Visual Studio でプロジェクトを開きます。

プロジェクトをビルドします。 ビルドにより、次のようなエラーが生成されます。

`1>Pages/Courses/Index.cshtml.cs(26,37,26,43): error CS1061: 'SchoolContext' does not
 contain a definition for 'Course' and no extension method 'Course' accepting a first
 argument of type 'SchoolContext' could be found (are you missing a using directive or
 an assembly reference?)`

 `_context.Course` を `_context.Courses` にグローバルに変更します (つまり、"s" を `Course` に追加します)。 7 回の出現が見つかり、更新されます。

*Pages/Courses/Index.cshtml.cs* を開き、`OnGetAsync` メソッドを調べます。 スキャフォールディング エンジンは、`Department` ナビゲーション プロパティに一括読み込みを指定しました。 `Include` メソッドが一括読み込みを指定します。

アプリを実行し、**[Courses]** リンクを選択します。 Department 列に `DepartmentID` が表示されますが、これには役に立ちません。

`OnGetAsync` メソッドを次のコードで更新します。

[!code-csharp[Main](intro/samples/cu/Pages/Courses/Index.cshtml.cs?name=snippet_RevisedIndexMethod)]

上のコードは `AsNoTracking` を追加します。 `AsNoTracking` は、返されるエンティティが追跡されないため、パフォーマンスが向上します。 これらのエンティティは現在のコンテキストでは更新されないため、追跡されません。

次の強調表示されているマークアップで *Views/Courses/Index.cshtml* を更新します。

[!code-html[](intro/samples/cu/Pages/Courses/Index.cshtml?highlight=4,7,15-17,34-36,44)]

スキャフォールディング コードに、次の変更が行われました。

* 見出しが Index から Courses に変更されました。
* `CourseID` プロパティ値を示す **Number** 列が追加されました。 既定では、主キーは、通常、エンドユーザーにとって意味がないため、スキャフォールディングされません。 ただし、このケースでは、主キーは意味があります。
* 部門名が表示されるように、**Department** 列を変更しました。 コードは、`Department` ナビゲーション プロパティに読み込まれる `Department` エンティティの `Name` プロパティを表示します。

  ```html
  @Html.DisplayFor(modelItem => item.Department.Name)
  ```

アプリを実行し、**[Courses]** タブを選択して部門名のリストを表示します。

![Courses/Index ページ](read-related-data/_static/courses-index.png)

<a name="select"></a>
### <a name="loading-related-data-with-select"></a>Select を使用した関連データの読み込み

`OnGetAsync` メソッドは、`Include` メソッドを使用して関連データを読み込みます。

[!code-csharp[Main](intro/samples/cu/Pages/Courses/Index.cshtml.cs?name=snippet_RevisedIndexMethod&highlight=4)]

`Select` 演算子は必要な関連データのみを読み込みます。 `Department.Name` のような単一の項目の場合、SQL INNER JOIN が使用されます。 コレクションの場合は、別のデータベース アクセスが使用されますが、コレクションの `Include`  演算子でも同じです。

次のコードは、`Select` メソッドを使用して関連データを読み込みます。

[!code-csharp[Main](intro/samples/cu/Pages/Courses/IndexSelect.cshtml.cs?name=snippet_RevisedIndexMethod&highlight=4)]

`CourseViewModel`:

[!code-csharp[Main](intro/samples/cu/Models/SchoolViewModels/CourseViewModel.cs?name=snippet)]

完全な例については、[IndexSelect.cshtml](https://github.com/aspnet/Docs/tree/master/aspnetcore/data/ef-rp/intro/samples/cu/Pages/Courses/IndexSelect.cshtml) と [IndexSelect.cshtml.cs](https://github.com/aspnet/Docs/tree/master/aspnetcore/data/ef-rp/intro/samples/cu/Pages/Courses/IndexSelect.cshtml.cs) を参照してください。

## <a name="create-an-instructors-page-that-shows-courses-and-enrollments"></a>コース登録を示す Instructors ページを作成する

このセクションでは、Instructors ページが作成されます。

<a name="IP"></a>
![Instructors/Index ページ](read-related-data/_static/instructors-index.png)

このページは、次の方法で関連データを読み取って表示します。

* インストラクターのリストには `OfficeAssignment` エンティティからの関連データが表示されます (上の図の Office)。 `Instructor` エンティティと `OfficeAssignment` エンティティは、一対ゼロまたは一対一のリレーションシップです。 `OfficeAssignment` エンティティには一括読み込みが使用されています。 一括読み込みは一般的に、関連データを表示する必要がある場合により効率的です。 この場合、インストラクターへのオフィスの割り当てが表示されます。
* ユーザーがインストラクターを選択 (上の図では Harui) すると、関連 `Course` エンティティが表示されます。 `Instructor` エンティティと `Course` エンティティは多対多リレーションシップです。 `Course` エンティティとその関連 `Department` エンティティの一括読み込みが使用されます。 このケースでは、選択したインストラクターのコースのみが必要なため、分離したクエリの方が効率的な場合があります。 この例では、ナビゲーション プロパティ内のエンティティのナビゲーション プロパティに一括読み込みを使用する方法を示します。
* ユーザーがコースを選択すると (上の図では Chemistry (化学))、`Enrollments` エンティティからの関連データが表示されます。 上の図では、受講者名とグレードが表示されています。 `Course` エンティティと `Enrollment` エンティティは一対多リレーションシップです。

### <a name="create-a-view-model-for-the-instructor-index-view"></a>Instructor インデックス ビューのビュー モデルを作成する

Instructors ページには、3 つの異なるテーブルからのデータが表示されます。 3 つのテーブルを表す 3 つのエンティティを含むビュー モデルが作成されます。

次のコードを使用して、*SchoolViewModels* フォルダー内に *InstructorIndexData.cs* を作成します。

[!code-csharp[Main](intro/samples/cu/Models/SchoolViewModels/InstructorIndexData.cs)]

### <a name="scaffold-the-instructor-model"></a>Instructor モデルのスキャフォールディング

* Visual Studio を終了します。
* プロジェクト ディレクトリ (*Program.cs*、*Startup.cs*、および *.csproj* ファイルを含むディレクトリ) でコマンド ウィンドウを開きます。
* 次のコマンドを実行します。

 ```console
dotnet aspnet-codegenerator razorpage -m Instructor -dc SchoolContext -udl -outDir Pages\Instructors --referenceScriptLibraries
 ```

上記のコマンドは、`Instructor` モデルをスキャフォールディングします。 Visual Studio でプロジェクトを開きます。

プロジェクトをビルドします。 ビルドがエラーを生成します。

`_context.Instructor` を `_context.Instructors` にグローバルに変更します (つまり、"s" を `Instructor` に追加します)。 7 回の出現が見つかり、更新されます。

アプリを実行し、Instructors ページに移動します。

*Pages/Instructors/Index.cshtml.cs* を次のコードで置き換えます。

[!code-csharp[Main](intro/samples/cu/Pages/Instructors/Index1.cshtml.cs?name=snippet_all&highlight=2,20-)]

`OnGetAsync` メソッドは、選択したインストラクターの ID の任意のルート データを受け取ります。

*Pages/Instructors/Index.cshtml* ページでクエリを調べます。

[!code-csharp[Main](intro/samples/cu/Pages/Instructors/Index1.cshtml.cs?name=snippet_ThenInclude)]

クエリには次の 2 つが含まれています。

* `OfficeAssignment`: [Instructors ビュー](#IP)に表示されます。
* `CourseAssignments`: 担当したコースを取り込みます。


### <a name="update-the-instructors-index-page"></a>Instructors/Index ページを更新する

次のマークアップを使用して *Pages/Instructors/Index.cshtml* を更新します。

[!code-html[](intro/samples/cu/Pages/Instructors/IndexRRD.cshtml?range=1-65&highlight=1,5,8,16-21,25-32,43-57)]

上記のマークアップは、次の変更を加えます。

* `page` ディレクティブを `@page` から `@page "{id:int?}"` に更新します。 `"{id:int?}"` はルート テンプレートです。 ルート テンプレートは、URL 内の整数クエリ文字列をルート データに変更します。 たとえば、インストラクターの **[Select]** リンクをクリックすると、ページ ディレクティブは次のような URL を生成します。

    `http://localhost:1234/Instructors?id=2`

    ページ ディレクティブが `@page "{id:int?}"` の場合、上記の URL は次のようになります。

    `http://localhost:1234/Instructors/2`

* ページ タイトルは **Instructors** です。
* `item.OfficeAssignment` が null ではない場合にのみ `item.OfficeAssignment.Location` を表示する **Office** 列を追加しました。 これは、一対ゼロまたは一対一のリレーションシップであるため、関連する OfficeAssignment エンティティがない場合があります。

  ```html
  @if (item.OfficeAssignment != null)
  {
      @item.OfficeAssignment.Location
  }
  ```

* インストラクターごとに担当したコースを表示する **Courses** 列を追加しました。 この Razor 構文の詳細については、「[Explicit Line Transition with `@:`](xref:mvc/views/razor#explicit-line-transition-with-)」(@: による明示的な行の遷移) を参照してください。

* 選択したインストラクターの `tr` 要素に `class="success"` を動的に追加するコードを追加しました。 これは、ブートストラップ クラスを使用して、選択した行の背景色を設定します。

  ```html
  string selectedRow = "";
  if (item.CourseID == Model.CourseID)
  {
      selectedRow = "success";
  }
  <tr class="@selectedRow">
  ```

* **Select** とラベル付けされるハイパーリンクを追加しました。 このリンクは、選択したインストラクターの ID を `Index` メソッドに送信し、背景色を設定します。

  ```html
  <a asp-action="Index" asp-route-id="@item.ID">Select</a> |
  ```

アプリを実行し、**[Instructors]** タブを選択します。関連する `OfficeAssignment` エンティティから `Location` (オフィス) がページに表示されます。 OfficeAssignment` が null の場合、空のテーブル セルが表示されます。

![何も選択されていない Instructors/Index ページ](read-related-data/_static/instructors-index-no-selection.png)

**[Select]** リンクをクリックします。 行のスタイルが変更されます。

### <a name="add-courses-taught-by-selected-instructor"></a>選択したインストラクターが担当するコースを追加する

次のコードを使用して、*Pages/Instructors/Index.cshtml.cs* 内の `OnGetAsync` メソッドを更新します。

[!code-csharp[Main](intro/samples/cu/Pages/Instructors/Index2.cshtml.cs?name=snippet_OnGetAsync&highlight=1,8,16-)]

更新されたクエリを確認します。

[!code-csharp[Main](intro/samples/cu/Pages/Instructors/Index2.cshtml.cs?name=snippet_ThenInclude)]

上記のクエリは `Department` エンティティを追加します。

次のコードは、インストラクターが選択されたとき (`id != null`) に実行されます。 選択されたインストラクターがビュー モデルのインストラクターのリストから取得されます。 ビュー モデルの `Courses` プロパティが `Course` エンティティと共にそのインストラクターの `CourseAssignments` ナビゲーション プロパティから読み込まれます。

[!code-csharp[Main](intro/samples/cu/Pages/Instructors/Index2.cshtml.cs?name=snippet_ID)]

`Where` メソッドはコレクションを返します。 上記の `Where` メソッドでは、1 つの `Instructor` エンティティのみが返されます。 `Single` メソッドはコレクションを 1 つの `Instructor` エンティティに変換します。 `Instructor` エンティティは `CourseAssignments` プロパティへのアクセスを提供します。 `CourseAssignments` は関連する `Course` エンティティへのアクセスを提供します。

![インストラクター対コース m:M](complex-data-model/_static/courseassignment.png)

コレクションに 1 つの項目しかない場合は、`Single` メソッドがコレクションで使用されます。 コレクションが空の場合、または複数の項目がある場合、`Single` メソッドは例外をスローします。 代わりに、コレクションが空の場合に既定値を返す (この場合は null) `SingleOrDefault` を使用します。 空のコレクションで `SingleOrDefault` を使用すると、次のようになります。

* (null 参照で `Courses` プロパティを見つけようとすると) 例外になります。
* 例外メッセージに問題の原因が明確に示されない場合があります。

次のコードは、コースが選択されたときにビュー モデルの `Enrollments` プロパティを設定します。

[!code-csharp[Main](intro/samples/cu/Pages/Instructors/Index2.cshtml.cs?name=snippet_courseID)]

*Pages/Courses/Index.cshtml* Razor ページの末尾に次のマークアップを追加します。

[!code-html[](intro/samples/cu/Pages/Instructors/IndexRRD.cshtml?range=60-102&highlight=7-)]

上記のマークアップは、インストラクターが選択されたときに、インストラクターに関連するコースのリストを表示します。

アプリをテストします。 Instructors ページの **[Select]** リンクをクリックします。

![インストラクターが選択された Instructors/Index ページ](read-related-data/_static/instructors-index-instructor-selected.png)

### <a name="show-student-data"></a>受講者データを表示する

このセクションでは、選択したコースの受講者データを表示するため、アプリが更新されます。

次のコードを使用して、*Pages/Instructors/Index.cshtml.cs* 内の `OnGetAsync` メソッドのクエリを更新します。

[!code-csharp[Main](intro/samples/cu/Pages/Instructors/Index.cshtml.cs?name=snippet_ThenInclude&highlight=6-9)]

*Pages/Instructors/Index.cshtml* を更新します。 ファイルの末尾に次のマークアップを追加します。

[!code-html[](intro/samples/cu/Pages/Instructors/IndexRRD.cshtml?range=103-)]

上記のマークアップは、選択したコースに登録されている受講者のリストを表示します。

ページを更新し、インストラクターを選択します。 コースを選択して、登録済みの受講者とその成績のリストを表示します。

![インストラクターとコースが選択された Instructors/Index ページ](read-related-data/_static/instructors-index.png)

## <a name="using-single"></a>Single を使用する

`Single` メソッドは、`Where` メソッドを別に呼び出す代わりに、`Where` 条件で渡すことができます。

[!code-csharp[Main](intro/samples/cu/Pages/Instructors/IndexSingle.cshtml.cs?name=snippet_single&highlight=21,28-29)]

上記の `Single` アプローチでは、`Where` を使用すること以上のメリットは提供されません。 一部の開発者は、`Single` アプローチ スタイルを選択します。

## <a name="explicit-loading"></a>明示的読み込み

現在のコードは、`Enrollments` と `Students` に一括読み込みを指定します。

[!code-csharp[Main](intro/samples/cu/Pages/Instructors/Index.cshtml.cs?name=snippet_ThenInclude&highlight=6-9)]

ユーザーがコースの登録を表示することはほとんどないとします。 その場合、最適化は要求された場合にのみ登録データを読み込むことです。 このセクションでは、`Enrollments` と `Students` の明示的読み込みを使用するために `OnGetAsync` が更新されます。

次のコードを使用して `OnGetAsync` を更新します。

[!code-csharp[Main](intro/samples/cu/Pages/Instructors/IndexXp.cshtml.cs?name=snippet_OnGetAsync&highlight=9-13,29-35)]

上記のコードは、登録と学生データの *ThenInclude* メソッド呼び出しを破棄します。 コースが選択されると、強調表示されたコードが以下を取得します。

* 選択したコースの `Enrollment` エンティティ。
* 各 `Enrollment` の `Student` エンティティ。

上記のコードでは、`.AsNoTracking()` がコメント アウトされていることに注目してください。 追跡対象のエンティティに対して、ナビゲーション プロパティのみを明示的に読み込むことができます。

アプリをテストします。 ユーザーの観点からは、アプリの動作は以前のバージョンと同じです。

次のチュートリアルでは、関連データの更新方法を示します。

>[!div class="step-by-step"]
>[前へ](xref:data/ef-rp/complex-data-model)
>[次へ](xref:data/ef-rp/update-related-data)
