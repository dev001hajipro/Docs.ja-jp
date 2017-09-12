---
title: "ASP.NET Core での Grunt の使用"
author: rick-anderson
description: 
keywords: ASP.NET Core,
ms.author: riande
manager: wpickett
ms.date: 10/14/2016
ms.topic: article
ms.assetid: 471112e9-2c33-454b-96fc-32916102ce73
ms.technology: aspnet
ms.prod: asp.net-core
uid: client-side/using-grunt
ms.openlocfilehash: 8ae50514ce24c7f9e3bb1e347d5d860e1de43c5f
ms.sourcegitcommit: 9cdbfd0d670d70b9c354216aabee260c52dad5ee
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/12/2017
---
# <a name="using-grunt-in-aspnet-core"></a><span data-ttu-id="6a135-103">ASP.NET Core での Grunt の使用</span><span class="sxs-lookup"><span data-stu-id="6a135-103">Using Grunt in ASP.NET Core</span></span> 

<span data-ttu-id="6a135-104">によって[ノエル ライス](https://blog.falafel.com/falafel-software-recognized-sitefinity-website-year/)</span><span class="sxs-lookup"><span data-stu-id="6a135-104">By [Noel Rice](https://blog.falafel.com/falafel-software-recognized-sitefinity-website-year/)</span></span>

<span data-ttu-id="6a135-105">Grunt は、スクリプトの縮小、TypeScript のコンパイルをコード品質の「柔らかい」ツール、CSS 前プロセッサ、およびクライアントの開発をサポートするために行う必要がある反復的な面倒だけに関するを自動化する JavaScript タスク ランナーです。</span><span class="sxs-lookup"><span data-stu-id="6a135-105">Grunt is a JavaScript task runner that automates script minification, TypeScript compilation, code quality "lint" tools, CSS pre-processors, and just about any repetitive chore that needs doing to support client development.</span></span> <span data-ttu-id="6a135-106">Grunt は ASP.NET プロジェクト テンプレートが既定で Gulp を使用しても、完全に Visual Studio で、サポート (を参照してください[Gulp を使用して](using-gulp.md))。</span><span class="sxs-lookup"><span data-stu-id="6a135-106">Grunt is fully supported in Visual Studio, though the ASP.NET project templates use Gulp by default (see [Using Gulp](using-gulp.md)).</span></span>

<span data-ttu-id="6a135-107">この例では、その開始ポイントとして空の ASP.NET Core プロジェクトを使用して、最初からクライアントのビルド プロセスを自動化する方法を示します。</span><span class="sxs-lookup"><span data-stu-id="6a135-107">This example uses an empty ASP.NET Core project as its starting point, to show how to automate the client build process from scratch.</span></span>

<span data-ttu-id="6a135-108">完了の例では、ターゲット配置ディレクトリをクリーンアップ、結合の JavaScript ファイル、コードの品質を確認、JavaScript ファイルの内容を圧縮し、web アプリケーションのルートに配置します。</span><span class="sxs-lookup"><span data-stu-id="6a135-108">The finished example cleans the target deployment directory, combines JavaScript files, checks code quality, condenses JavaScript file content and deploys to the root of your web application.</span></span> <span data-ttu-id="6a135-109">次のパッケージが使用します。</span><span class="sxs-lookup"><span data-stu-id="6a135-109">We will use the following packages:</span></span>

* <span data-ttu-id="6a135-110">**grunt**:「Grunt タスク ランナー パッケージです。</span><span class="sxs-lookup"><span data-stu-id="6a135-110">**grunt**: The Grunt task runner package.</span></span>

* <span data-ttu-id="6a135-111">**grunt contrib クリーンアップ**: ファイルまたはディレクトリを削除するプラグイン。</span><span class="sxs-lookup"><span data-stu-id="6a135-111">**grunt-contrib-clean**: A plugin that removes files or directories.</span></span>

* <span data-ttu-id="6a135-112">**grunt contrib-jshint**: JavaScript コードの品質をレビューするプラグイン。</span><span class="sxs-lookup"><span data-stu-id="6a135-112">**grunt-contrib-jshint**: A plugin that reviews JavaScript code quality.</span></span>

* <span data-ttu-id="6a135-113">**grunt contrib-concat**: 1 つのファイルのファイルを結合するプラグイン。</span><span class="sxs-lookup"><span data-stu-id="6a135-113">**grunt-contrib-concat**: A plugin that joins files into a single file.</span></span>

* <span data-ttu-id="6a135-114">**grunt contrib uglify**: サイズを縮小する JavaScript の縮小をプラグインします。</span><span class="sxs-lookup"><span data-stu-id="6a135-114">**grunt-contrib-uglify**: A plugin that minifies JavaScript to reduce size.</span></span>

* <span data-ttu-id="6a135-115">**grunt contrib-ウォッチ**: ファイルのアクティビティを監視するプラグイン。</span><span class="sxs-lookup"><span data-stu-id="6a135-115">**grunt-contrib-watch**: A plugin that watches file activity.</span></span>

## <a name="preparing-the-application"></a><span data-ttu-id="6a135-116">アプリケーションを準備します。</span><span class="sxs-lookup"><span data-stu-id="6a135-116">Preparing the application</span></span>

<span data-ttu-id="6a135-117">を開始するには、新しい空の web アプリケーションを設定し、TypeScript ファイルの例を追加します。</span><span class="sxs-lookup"><span data-stu-id="6a135-117">To begin, set up a new empty web application and add TypeScript example files.</span></span> <span data-ttu-id="6a135-118">TypeScript ファイルは、自動的に既定の Visual Studio の設定を使用して JavaScript にコンパイルされ、Grunt を使用して処理する、原材料となります。</span><span class="sxs-lookup"><span data-stu-id="6a135-118">TypeScript files are automatically compiled into JavaScript using default Visual Studio settings and will be our raw material to process using Grunt.</span></span>

1.  <span data-ttu-id="6a135-119">Visual Studio で、新しい`ASP.NET Web Application`です。</span><span class="sxs-lookup"><span data-stu-id="6a135-119">In Visual Studio, create a new `ASP.NET Web Application`.</span></span>

2.  <span data-ttu-id="6a135-120">**新しい ASP.NET プロジェクト**ダイアログ ボックスで、選択 ASP.NET Core**空**テンプレートし、[ok] ボタンをクリックします。</span><span class="sxs-lookup"><span data-stu-id="6a135-120">In the **New ASP.NET Project** dialog, select the ASP.NET Core **Empty** template and click the OK button.</span></span>

3.  <span data-ttu-id="6a135-121">ソリューション エクスプ ローラーで、プロジェクトの構造を確認します。</span><span class="sxs-lookup"><span data-stu-id="6a135-121">In the Solution Explorer, review the project structure.</span></span> <span data-ttu-id="6a135-122">`\src`フォルダーに空`wwwroot`と`Dependencies`ノード。</span><span class="sxs-lookup"><span data-stu-id="6a135-122">The `\src` folder includes empty `wwwroot` and `Dependencies` nodes.</span></span>

    ![空の web ソリューション](using-grunt/_static/grunt-solution-explorer.png)

4.  <span data-ttu-id="6a135-124">という名前の新しいフォルダーを追加`TypeScript`プロジェクト ディレクトリにします。</span><span class="sxs-lookup"><span data-stu-id="6a135-124">Add a new folder named `TypeScript` to your project directory.</span></span>

5.  <span data-ttu-id="6a135-125">すべてのファイルを追加する前に説明があること確認 Visual Studio オプション 'コンパイル on save' の TypeScript ファイルをチェックします。</span><span class="sxs-lookup"><span data-stu-id="6a135-125">Before adding any files, let’s make sure that Visual Studio has the option 'compile on save' for TypeScript files checked.</span></span> <span data-ttu-id="6a135-126">*ツール > オプション > のテキスト エディター > Typescript > プロジェクト*</span><span class="sxs-lookup"><span data-stu-id="6a135-126">*Tools > Options > Text Editor > Typescript > Project*</span></span>

    ![オプションの TypeScript ファイルの自動 compliation の設定](using-grunt/_static/typescript-options.png)

6.  <span data-ttu-id="6a135-128">右クリックし、`TypeScript`ディレクトリおよび select**追加 > 新しい項目の**コンテキスト メニュー。</span><span class="sxs-lookup"><span data-stu-id="6a135-128">Right-click the `TypeScript` directory and select **Add > New Item** from the context menu.</span></span> <span data-ttu-id="6a135-129">選択、 **JavaScript ファイル**項目し、ファイルの名前*Tastes.ts* (注、 \*.ts 拡張子) です。</span><span class="sxs-lookup"><span data-stu-id="6a135-129">Select the **JavaScript file** item and name the file *Tastes.ts* (note the \*.ts extension).</span></span> <span data-ttu-id="6a135-130">TypeScript コードの下の行をファイルにコピー (を保存すると、新しい*Tastes.js* JavaScript ソース ファイルが表示されます)。</span><span class="sxs-lookup"><span data-stu-id="6a135-130">Copy the line of TypeScript code below into the file (when you save, a new *Tastes.js* file will appear with the JavaScript source).</span></span>
    
    ```typescript
    enum Tastes { Sweet, Sour, Salty, Bitter }
    ```

7.  <span data-ttu-id="6a135-131">2 番目のファイルを追加、 **TypeScript**ディレクトリし名前を付けます`Food.ts`です。</span><span class="sxs-lookup"><span data-stu-id="6a135-131">Add a second file to the **TypeScript** directory and name it `Food.ts`.</span></span> <span data-ttu-id="6a135-132">ファイルに次のコードをコピーします。</span><span class="sxs-lookup"><span data-stu-id="6a135-132">Copy the code below into the file.</span></span>

    ```typescript
    class Food {
      constructor(name: string, calories: number) {
        this._name = name;
        this._calories = calories;
      }
    
      private _name: string;
      get Name() {
        return this._name;
      }
    
      private _calories: number;
      get Calories() {
        return this._calories;
      }
    
      private _taste: Tastes;
      get Taste(): Tastes { return this._taste }
      set Taste(value: Tastes) {
        this._taste = value;
      }
    }
    ```

## <a name="configuring-npm"></a><span data-ttu-id="6a135-133">NPM を構成します。</span><span class="sxs-lookup"><span data-stu-id="6a135-133">Configuring NPM</span></span>

<span data-ttu-id="6a135-134">次に、NPM grunt と grunt タスク ダウンロードを構成します。</span><span class="sxs-lookup"><span data-stu-id="6a135-134">Next, configure NPM to download grunt and grunt-tasks.</span></span>

1. <span data-ttu-id="6a135-135">ソリューション エクスプ ローラーでプロジェクトを右クリックし、**追加 > 新しい項目の**コンテキスト メニュー。</span><span class="sxs-lookup"><span data-stu-id="6a135-135">In the Solution Explorer, right-click the project and select **Add > New Item** from the context menu.</span></span> <span data-ttu-id="6a135-136">選択、 **NPM 構成ファイル**項目で、既定の名前をそのまま使用*package.json*、 をクリックし、**追加**ボタンをクリックします。</span><span class="sxs-lookup"><span data-stu-id="6a135-136">Select the **NPM configuration file** item, leave the default name, *package.json*, and click the **Add** button.</span></span>

2. <span data-ttu-id="6a135-137">*Package.json*ファイル内、`devDependencies`中かっこをオブジェクトに、"grunt"を入力します。</span><span class="sxs-lookup"><span data-stu-id="6a135-137">In the *package.json* file, inside the `devDependencies` object braces, enter "grunt".</span></span> <span data-ttu-id="6a135-138">選択`grunt`intellisense を一覧表示し、Enter キーを押します。</span><span class="sxs-lookup"><span data-stu-id="6a135-138">Select `grunt` from the Intellisense list and press the Enter key.</span></span> <span data-ttu-id="6a135-139">Visual Studio が grunt パッケージ名、引用符で囲むし、コロンを追加します。</span><span class="sxs-lookup"><span data-stu-id="6a135-139">Visual Studio will quote the grunt package name, and add a colon.</span></span> <span data-ttu-id="6a135-140">コロンの右に、Intellisense の一覧の先頭から、パッケージの最新の安定したバージョンを選択します (キーを押して`Ctrl-Space`Intellisense が表示されない場合)。</span><span class="sxs-lookup"><span data-stu-id="6a135-140">To the right of the colon, select the latest stable version of the package from the top of the Intellisense list (press `Ctrl-Space` if Intellisense does not appear).</span></span>

    ![grun Intellisense](using-grunt/_static/devdependencies-grunt.png)
    
    > [!NOTE]
    > <span data-ttu-id="6a135-142">NPM を使用して[セマンティック バージョニング](http://semver.org/)の依存関係を整理します。</span><span class="sxs-lookup"><span data-stu-id="6a135-142">NPM uses [semantic versioning](http://semver.org/) to organize dependencies.</span></span> <span data-ttu-id="6a135-143">セマンティック バージョニング、SemVer とも呼ばれる、番号付けスキームでパッケージを識別する<major>.<minor>です。<patch>.Intellisense は、いくつかの一般的な選択肢のみを表示することによって、セマンティック バージョニングを簡略化します。</span><span class="sxs-lookup"><span data-stu-id="6a135-143">Semantic versioning, also known as SemVer, identifies packages with the numbering scheme <major>.<minor>.<patch>. Intellisense simplifies semantic versioning by showing only a few common choices.</span></span> <span data-ttu-id="6a135-144">Intellisense の一覧 (上記の例では 0.4.5) の最上位の項目は、パッケージの最新の安定バージョンと見なされます。</span><span class="sxs-lookup"><span data-stu-id="6a135-144">The top item in the Intellisense list (0.4.5 in the example above) is considered the latest stable version of the package.</span></span> <span data-ttu-id="6a135-145">キャレット (^) 記号が最新のメジャー バージョンと一致し、ティルダ (~) には、最新のマイナー バージョンが一致します。</span><span class="sxs-lookup"><span data-stu-id="6a135-145">The caret (^) symbol matches the most recent major version and the tilde (~) matches the most recent minor version.</span></span> <span data-ttu-id="6a135-146">参照してください、 [NPM semver バージョン パーサー参照](https://www.npmjs.com/package/semver)SemVer を提供するすべての表現性をガイドとして。</span><span class="sxs-lookup"><span data-stu-id="6a135-146">See the [NPM semver version parser reference](https://www.npmjs.com/package/semver) as a guide to the full expressivity that SemVer provides.</span></span>

3. <span data-ttu-id="6a135-147">追加の依存関係の詳細を読み込む grunt-contrib -\*パッケージ化*クリーン*、 *jshint*、 *concat*、 *uglify*、および*ウォッチ*次の例で示すようにします。</span><span class="sxs-lookup"><span data-stu-id="6a135-147">Add more dependencies to load grunt-contrib-\* packages for *clean*, *jshint*, *concat*, *uglify*, and *watch* as shown in the example below.</span></span> <span data-ttu-id="6a135-148">バージョンは、例では、一致する必要はありません。</span><span class="sxs-lookup"><span data-stu-id="6a135-148">The versions do not need to match the example.</span></span>

    ```json
    "devDependencies": {
      "grunt": "0.4.5",
      "grunt-contrib-clean": "0.6.0",
      "grunt-contrib-jshint": "0.11.0",
      "grunt-contrib-concat": "0.5.1",
      "grunt-contrib-uglify": "0.8.0",
      "grunt-contrib-watch": "0.6.1"
    }
    ```

4. <span data-ttu-id="6a135-149">保存、 *package.json*ファイル。</span><span class="sxs-lookup"><span data-stu-id="6a135-149">Save the *package.json* file.</span></span>

<span data-ttu-id="6a135-150">DevDependencies 項目ごとに、パッケージは、各パッケージが必要なすべてのファイルと共にダウンロードされます。</span><span class="sxs-lookup"><span data-stu-id="6a135-150">The packages for each devDependencies item will download, along with any files that each package requires.</span></span> <span data-ttu-id="6a135-151">内のパッケージ ファイルを見つけることができます、`node_modules`ディレクトリを有効にして、 **すべてのファイル**ソリューション エクスプ ローラーでボタンをクリックします。</span><span class="sxs-lookup"><span data-stu-id="6a135-151">You can find the package files in the `node_modules` directory by enabling the **Show All Files** button in the Solution Explorer.</span></span>

![grunt node_modules](using-grunt/_static/node-modules.png)

> [!NOTE]
> <span data-ttu-id="6a135-153">右クリックして、ソリューション エクスプ ローラー内の依存関係を手動で復元する必要がある場合、`Dependencies\NPM`を選択して、**パッケージの復元**メニュー オプション。</span><span class="sxs-lookup"><span data-stu-id="6a135-153">If you need to, you can manually restore dependencies in Solution Explorer by right-clicking on `Dependencies\NPM` and selecting the **Restore Packages** menu option.</span></span>

![パッケージを復元します。](using-grunt/_static/restore-packages.png)

## <a name="configuring-grunt"></a><span data-ttu-id="6a135-155">Grunt を構成します。</span><span class="sxs-lookup"><span data-stu-id="6a135-155">Configuring Grunt</span></span>

<span data-ttu-id="6a135-156">Grunt、という名前のマニフェストを使用して構成*Gruntfile.js*を定義、読み込まれ、タスクを手動で実行したりできる Visual Studio でのイベントに基づいて自動的に実行するように構成を登録します。</span><span class="sxs-lookup"><span data-stu-id="6a135-156">Grunt is configured using a manifest named *Gruntfile.js* that defines, loads and registers tasks that can be run manually or configured to run automatically based on events in Visual Studio.</span></span>

1.  <span data-ttu-id="6a135-157">プロジェクトを右クリックし **追加 > 新しい項目の**します。</span><span class="sxs-lookup"><span data-stu-id="6a135-157">Right-click the project and select **Add > New Item**.</span></span> <span data-ttu-id="6a135-158">選択、 **Grunt 構成ファイル**オプションで、既定の名前、 *Gruntfile.js*、 をクリックし、**追加**ボタンをクリックします。</span><span class="sxs-lookup"><span data-stu-id="6a135-158">Select the **Grunt Configuration file** option, leave the default name, *Gruntfile.js*, and click the **Add** button.</span></span>

    <span data-ttu-id="6a135-159">最初のコードには、モジュール定義が含まれています。 および`grunt.initConfig()`メソッドです。</span><span class="sxs-lookup"><span data-stu-id="6a135-159">The initial code includes a module definition and the `grunt.initConfig()` method.</span></span> <span data-ttu-id="6a135-160">`initConfig()`パッケージごとにオプションの設定に使用されると、モジュールの残りの部分がロードされ、タスクを登録します。</span><span class="sxs-lookup"><span data-stu-id="6a135-160">The `initConfig()` is used to set options for each package, and the remainder of the module will load and register tasks.</span></span>
    
    ```javascript
    module.exports = function (grunt) {
      grunt.initConfig({
      });
    };
    ```

2. <span data-ttu-id="6a135-161">内部、`initConfig()`メソッドのオプションを追加、`clean`タスクの例で示すように*Gruntfile.js*下です。</span><span class="sxs-lookup"><span data-stu-id="6a135-161">Inside the `initConfig()` method, add options for the `clean` task as shown in the example *Gruntfile.js* below.</span></span> <span data-ttu-id="6a135-162">Clean タスクは、ディレクトリの文字列の配列を受け入れます。</span><span class="sxs-lookup"><span data-stu-id="6a135-162">The clean task accepts an array of directory strings.</span></span> <span data-ttu-id="6a135-163">このタスクは、wwwroot/lib からファイルを削除し、全体/一時ディレクトリを削除します。</span><span class="sxs-lookup"><span data-stu-id="6a135-163">This task removes files from wwwroot/lib and removes the entire /temp directory.</span></span>

    ```javascript
    module.exports = function (grunt) {
      grunt.initConfig({
        clean: ["wwwroot/lib/*", "temp/"],
      });
    };
    ```

3. <span data-ttu-id="6a135-164">下 initConfig() メソッド呼び出しを追加して`grunt.loadNpmTasks()`です。</span><span class="sxs-lookup"><span data-stu-id="6a135-164">Below the initConfig() method, add a call to `grunt.loadNpmTasks()`.</span></span> <span data-ttu-id="6a135-165">これにより、タスク実行可能な Visual Studio から。</span><span class="sxs-lookup"><span data-stu-id="6a135-165">This will make the task runnable from Visual Studio.</span></span>

    ```javascript
    grunt.loadNpmTasks("grunt-contrib-clean");
    ```

4. <span data-ttu-id="6a135-166">保存*Gruntfile.js*です。</span><span class="sxs-lookup"><span data-stu-id="6a135-166">Save *Gruntfile.js*.</span></span> <span data-ttu-id="6a135-167">ファイルは次のスクリーン ショットのようになります。</span><span class="sxs-lookup"><span data-stu-id="6a135-167">The file should look something like the screenshot below.</span></span>

    ![初期の gruntfile](using-grunt/_static/gruntfile-js-initial.png)

5. <span data-ttu-id="6a135-169">右クリック*Gruntfile.js*選択**タスク ランナー エクスプ ローラー**コンテキスト メニュー。</span><span class="sxs-lookup"><span data-stu-id="6a135-169">Right-click *Gruntfile.js* and select **Task Runner Explorer** from the context menu.</span></span> <span data-ttu-id="6a135-170">タスク ランナー エクスプ ローラー ウィンドウが開きます。</span><span class="sxs-lookup"><span data-stu-id="6a135-170">The Task Runner Explorer window will open.</span></span>

    ![タスク ランナー エクスプ ローラーのメニュー](using-grunt/_static/task-runner-explorer-menu.png)

6. <span data-ttu-id="6a135-172">いることを確認`clean`の下に表示**タスク**タスク ランナー エクスプ ローラーでします。</span><span class="sxs-lookup"><span data-stu-id="6a135-172">Verify that `clean` shows under **Tasks** in the Task Runner Explorer.</span></span>

    ![タスク ランナー エクスプ ローラーのタスク一覧](using-grunt/_static/task-runner-explorer-tasks.png)

7. <span data-ttu-id="6a135-174">Clean タスクを右クリックし **実行**コンテキスト メニュー。</span><span class="sxs-lookup"><span data-stu-id="6a135-174">Right-click the clean task and select **Run** from the context menu.</span></span> <span data-ttu-id="6a135-175">コマンド ウィンドウには、タスクの進行状況が表示されます。</span><span class="sxs-lookup"><span data-stu-id="6a135-175">A command window displays progress of the task.</span></span>

    ![タスク ランナー エクスプ ローラーの実行のクリーン タスク](using-grunt/_static/task-runner-explorer-run-clean.png)
    
    > [!NOTE]
    > <span data-ttu-id="6a135-177">まだクリーンアップするファイルまたはディレクトリはありません。</span><span class="sxs-lookup"><span data-stu-id="6a135-177">There are no files or directories to clean yet.</span></span> <span data-ttu-id="6a135-178">必要に応じて、ソリューション エクスプ ローラーでそれらを手動で作成し、テスト用に clean タスクを実行できます。</span><span class="sxs-lookup"><span data-stu-id="6a135-178">If you like, you can manually create them in the Solution Explorer and then run the clean task as a test.</span></span>
    
8. <span data-ttu-id="6a135-179">InitConfig() メソッドでのエントリを追加`concat`次のコードを使用します。</span><span class="sxs-lookup"><span data-stu-id="6a135-179">In the initConfig() method, add an entry for `concat` using the code below.</span></span>

    <span data-ttu-id="6a135-180">`src`プロパティ配列を組み合わせる必要があります順番、結合のファイルを一覧表示します。</span><span class="sxs-lookup"><span data-stu-id="6a135-180">The `src` property array lists files to combine, in the order that they should be combined.</span></span> <span data-ttu-id="6a135-181">`dest`プロパティが生成される結合されたファイルにパスを割り当てます。</span><span class="sxs-lookup"><span data-stu-id="6a135-181">The `dest` property assigns the path to the combined file that is produced.</span></span>

    ```javascript
    concat: {
      all: {
        src: ['TypeScript/Tastes.js', 'TypeScript/Food.js'],
        dest: 'temp/combined.js'
      }
    },
    ```
    
    > [!NOTE]
    > <span data-ttu-id="6a135-182">`all`上記のコードでのプロパティは、ターゲットの名前。</span><span class="sxs-lookup"><span data-stu-id="6a135-182">The `all` property in the code above is the name of a target.</span></span> <span data-ttu-id="6a135-183">ターゲットは、複数のビルド環境を許可する Grunt タスクの一部で使用されます。</span><span class="sxs-lookup"><span data-stu-id="6a135-183">Targets are used in some Grunt tasks to allow multiple build environments.</span></span> <span data-ttu-id="6a135-184">Intellisense を使用して組み込みのターゲットを表示または、自分に割り当てることができます。</span><span class="sxs-lookup"><span data-stu-id="6a135-184">You can view the built-in targets using Intellisense or assign your own.</span></span>
    
9. <span data-ttu-id="6a135-185">追加、`jshint`タスクの次のコードを使用します。</span><span class="sxs-lookup"><span data-stu-id="6a135-185">Add the `jshint` task using the code below.</span></span>

    <span data-ttu-id="6a135-186">Jshint コード品質のユーティリティは、一時ディレクトリにあるすべての JavaScript ファイルに対して実行されます。</span><span class="sxs-lookup"><span data-stu-id="6a135-186">The jshint code-quality utility is run against every JavaScript file found in the temp directory.</span></span>
    
    ```javascript
    jshint: {
      files: ['temp/*.js'],
      options: {
        '-W069': false,
      }
    },
    ```

    > [!NOTE]
    > <span data-ttu-id="6a135-187">オプション"-W069"がエラー時に生成される jshint JavaScript の使用に右山かっこ構文例: ドット付き表記ではなくプロパティを割り当てる`Tastes["Sweet"]`の代わりに`Tastes.Sweet`です。</span><span class="sxs-lookup"><span data-stu-id="6a135-187">The option "-W069" is an error produced by jshint when JavaScript uses bracket syntax to assign a property instead of dot notation, i.e. `Tastes["Sweet"]` instead of `Tastes.Sweet`.</span></span> <span data-ttu-id="6a135-188">オプションは、残りのプロセスの続行を許可する警告をオフにします。</span><span class="sxs-lookup"><span data-stu-id="6a135-188">The option turns off the warning to allow the rest of the process to continue.</span></span>

10.  <span data-ttu-id="6a135-189">追加、`uglify`タスクの次のコードを使用します。</span><span class="sxs-lookup"><span data-stu-id="6a135-189">Add the `uglify` task using the code below.</span></span>

    <span data-ttu-id="6a135-190">タスクの縮小、 *combined.js*ファイルが、一時ディレクトリにあるし、標準の命名規則に従った wwwroot/lib で結果ファイルを作成*\<ファイル名\>min.js*。.</span><span class="sxs-lookup"><span data-stu-id="6a135-190">The task minifies the *combined.js* file found in the temp directory and creates the result file in wwwroot/lib following the standard naming convention *\<file name\>.min.js*.</span></span>
    
    ```javascript
    uglify: {
      all: {
        src: ['temp/combined.js'],
        dest: 'wwwroot/lib/combined.min.js'
      }
    },
    ```

11. <span data-ttu-id="6a135-191">Grunt contrib クリーンアップをロードする呼び出し grunt.loadNpmTasks()、jshint、concat の同時呼び出しが含まれてし、uglify 次のコードを使用します。</span><span class="sxs-lookup"><span data-stu-id="6a135-191">Under the call grunt.loadNpmTasks() that loads grunt-contrib-clean, include the same call for jshint, concat and uglify using the code below.</span></span>
    
    ```javascript
    grunt.loadNpmTasks('grunt-contrib-jshint');
    grunt.loadNpmTasks('grunt-contrib-concat');
    grunt.loadNpmTasks('grunt-contrib-uglify');
    ```

12. <span data-ttu-id="6a135-192">保存*Gruntfile.js*です。</span><span class="sxs-lookup"><span data-stu-id="6a135-192">Save *Gruntfile.js*.</span></span> <span data-ttu-id="6a135-193">ファイルは次の例のようになります。</span><span class="sxs-lookup"><span data-stu-id="6a135-193">The file should look something like the example below.</span></span>

    ![完全な grunt ファイルの例](using-grunt/_static/gruntfile-js-complete.png)

13. <span data-ttu-id="6a135-195">タスク ランナー エクスプ ローラーのタスク一覧には、通知`clean`、 `concat`、`jshint`と`uglify`タスク。</span><span class="sxs-lookup"><span data-stu-id="6a135-195">Notice that the Task Runner Explorer Tasks list includes `clean`, `concat`, `jshint` and `uglify` tasks.</span></span> <span data-ttu-id="6a135-196">順序で各タスクを実行し、ソリューション エクスプ ローラーで結果を確認します。</span><span class="sxs-lookup"><span data-stu-id="6a135-196">Run each task in order and observe the results in Solution Explorer.</span></span> <span data-ttu-id="6a135-197">各タスクは、エラーなく実行する必要があります。</span><span class="sxs-lookup"><span data-stu-id="6a135-197">Each task should run without errors.</span></span>
    
    ![各タスクの実行タスク ランナー エクスプ ローラー](using-grunt/_static/task-runner-explorer-run-each-task.png)
    
    <span data-ttu-id="6a135-199">Concat タスクが新たに作成*combined.js*ファイルし、temp ディレクトリに配置します。</span><span class="sxs-lookup"><span data-stu-id="6a135-199">The concat task creates a new *combined.js* file and places it into the temp directory.</span></span> <span data-ttu-id="6a135-200">単に jshint タスクを実行し、出力は生成されません。</span><span class="sxs-lookup"><span data-stu-id="6a135-200">The jshint task simply runs and doesn’t produce output.</span></span> <span data-ttu-id="6a135-201">新たに作成 uglify タスク*combined.min.js*ファイルし、wwwroot/lib に配置します。</span><span class="sxs-lookup"><span data-stu-id="6a135-201">The uglify task creates a new *combined.min.js* file and places it into wwwroot/lib.</span></span> <span data-ttu-id="6a135-202">完了時に、ソリューションはようになります次のスクリーン ショット。</span><span class="sxs-lookup"><span data-stu-id="6a135-202">On completion, the solution should look something like the screenshot below:</span></span>
    
    ![ソリューション エクスプ ローラーのすべてのタスクします。](using-grunt/_static/solution-explorer-after-all-tasks.png)
    
    > [!NOTE]
    > <span data-ttu-id="6a135-204">各パッケージのオプションの詳細については、次を参照してください。 [https://www.npmjs.com/](https://www.npmjs.com/)とメイン ページで、[検索] ボックスで、パッケージ名を検索します。</span><span class="sxs-lookup"><span data-stu-id="6a135-204">For more information on the options for each package, visit [https://www.npmjs.com/](https://www.npmjs.com/) and lookup the package name in the search box on the main page.</span></span> <span data-ttu-id="6a135-205">たとえば、すべてのパラメーターを説明するドキュメントのリンクを取得する grunt contrib クリーンアップ パッケージを検索できます。</span><span class="sxs-lookup"><span data-stu-id="6a135-205">For example, you can look up the grunt-contrib-clean package to get a documentation link that explains all of its parameters.</span></span>

### <a name="all-together-now"></a><span data-ttu-id="6a135-206">まとめ</span><span class="sxs-lookup"><span data-stu-id="6a135-206">All together now</span></span>

<span data-ttu-id="6a135-207">使用して、Grunt`registerTask()`特定のシーケンスで一連のタスクを実行するメソッド。</span><span class="sxs-lookup"><span data-stu-id="6a135-207">Use the Grunt `registerTask()` method to run a series of tasks in a particular sequence.</span></span> <span data-ttu-id="6a135-208">たとえば、クリーンな順序で上記の手順 -> 例を実行する concat]-> [jshint]-> [uglify、モジュールに、以下のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="6a135-208">For example, to run the example steps above in the order clean -> concat -> jshint -> uglify, add the code below to the module.</span></span> <span data-ttu-id="6a135-209">コードは、initConfig 外の loadNpmTasks() 呼び出しと同じレベルに追加する必要があります。</span><span class="sxs-lookup"><span data-stu-id="6a135-209">The code should be added to the same level as the loadNpmTasks() calls, outside initConfig.</span></span>

```javascript
grunt.registerTask("all", ['clean', 'concat', 'jshint', 'uglify']);
```

<span data-ttu-id="6a135-210">タスク ランナー エクスプ ローラーの エイリアスのタスクに新しいタスクが表示されます。</span><span class="sxs-lookup"><span data-stu-id="6a135-210">The new task shows up in Task Runner Explorer under Alias Tasks.</span></span> <span data-ttu-id="6a135-211">右クリックし、その他のタスクと同様に実行することができます。</span><span class="sxs-lookup"><span data-stu-id="6a135-211">You can right-click and run it just as you would other tasks.</span></span> <span data-ttu-id="6a135-212">`all`タスクは実行`clean`、 `concat`、`jshint`と`uglify`の順序で。</span><span class="sxs-lookup"><span data-stu-id="6a135-212">The `all` task will run `clean`, `concat`, `jshint` and `uglify`, in order.</span></span>

![エイリアス grunt タスク](using-grunt/_static/alias-tasks.png)

## <a name="watching-for-changes"></a><span data-ttu-id="6a135-214">変更の監視</span><span class="sxs-lookup"><span data-stu-id="6a135-214">Watching for changes</span></span>

<span data-ttu-id="6a135-215">A`watch`タスクは、ファイルおよびディレクトリで、監視を保持します。</span><span class="sxs-lookup"><span data-stu-id="6a135-215">A `watch` task keeps an eye on files and directories.</span></span> <span data-ttu-id="6a135-216">ウォッチは変更を検出した場合、タスクを自動的にトリガーされます。</span><span class="sxs-lookup"><span data-stu-id="6a135-216">The watch triggers tasks automatically if it detects changes.</span></span> <span data-ttu-id="6a135-217">変更を監視する initConfig に次のコードを追加\*TypeScript ディレクトリ内の .js ファイル。</span><span class="sxs-lookup"><span data-stu-id="6a135-217">Add the code below to initConfig to watch for changes to \*.js files in the TypeScript directory.</span></span> <span data-ttu-id="6a135-218">JavaScript ファイルが変更された場合、`watch`実行は、`all`タスク。</span><span class="sxs-lookup"><span data-stu-id="6a135-218">If a JavaScript file is changed, `watch` will run the `all` task.</span></span>

```javascript
watch: {
  files: ["TypeScript/*.js"],
  tasks: ["all"]
}
```

<span data-ttu-id="6a135-219">呼び出しを追加して`loadNpmTasks()`を表示する、`watch`タスク ランナー エクスプ ローラーで作業します。</span><span class="sxs-lookup"><span data-stu-id="6a135-219">Add a call to `loadNpmTasks()` to show the `watch` task in Task Runner Explorer.</span></span>

```javascript
grunt.loadNpmTasks('grunt-contrib-watch');
```

<span data-ttu-id="6a135-220">タスク ランナー エクスプ ローラーで、監視タスクを右クリックし、コンテキスト メニューの [実行] を選択します。</span><span class="sxs-lookup"><span data-stu-id="6a135-220">Right-click the watch task in Task Runner Explorer and select Run from the context menu.</span></span> <span data-ttu-id="6a135-221">「待機しています...」を実行中の監視タスクを示しています。 コマンド ウィンドウが表示されます。</span><span class="sxs-lookup"><span data-stu-id="6a135-221">The command window that shows the watch task running will display a "Waiting…"</span></span> <span data-ttu-id="6a135-222">メッセージが表示されます。</span><span class="sxs-lookup"><span data-stu-id="6a135-222">message.</span></span> <span data-ttu-id="6a135-223">TypeScript ファイルの 1 つを開き、、スペースを追加し、ファイルを保存します。</span><span class="sxs-lookup"><span data-stu-id="6a135-223">Open one of the TypeScript files, add a space, and then save the file.</span></span> <span data-ttu-id="6a135-224">監視タスクをトリガーされ順序で実行するその他のタスクをトリガーします。</span><span class="sxs-lookup"><span data-stu-id="6a135-224">This will trigger the watch task and trigger the other tasks to run in order.</span></span> <span data-ttu-id="6a135-225">次のスクリーン ショットは、サンプルの実行を示します。</span><span class="sxs-lookup"><span data-stu-id="6a135-225">The screenshot below shows a sample run.</span></span>

![タスクの出力を実行しています。](using-grunt/_static/watch-running.png)

## <a name="binding-to-visual-studio-events"></a><span data-ttu-id="6a135-227">Visual Studio イベントへのバインド</span><span class="sxs-lookup"><span data-stu-id="6a135-227">Binding to Visual Studio events</span></span>

<span data-ttu-id="6a135-228">タスクをバインドするには Visual Studio で作業するたびに、タスクを手動で開始する場合を除き、**する前にビルド**、**後にビルド**、**クリーン**、および**プロジェクトのオープン**イベント。</span><span class="sxs-lookup"><span data-stu-id="6a135-228">Unless you want to manually start your tasks every time you work in Visual Studio, you can bind tasks to **Before Build**, **After Build**, **Clean**, and **Project Open** events.</span></span>

<span data-ttu-id="6a135-229">バインドしてみます`watch`Visual Studio が開くたびに実行されるようにします。</span><span class="sxs-lookup"><span data-stu-id="6a135-229">Let’s bind `watch` so that it runs every time Visual Studio opens.</span></span> <span data-ttu-id="6a135-230">タスク ランナー エクスプ ローラーで、監視タスクを右クリックし、選択**バインド > プロジェクトを開く**コンテキスト メニュー。</span><span class="sxs-lookup"><span data-stu-id="6a135-230">In Task Runner Explorer, right-click the watch task and select **Bindings > Project Open** from the context menu.</span></span>

![プロジェクトを開くにはタスクをバインドします。](using-grunt/_static/bindings-project-open.png)

<span data-ttu-id="6a135-232">アンロードし、プロジェクトの再読み込みします。</span><span class="sxs-lookup"><span data-stu-id="6a135-232">Unload and reload the project.</span></span> <span data-ttu-id="6a135-233">プロジェクトが再度読み込まれると、自動的に実行されている、監視タスクが開始されます。</span><span class="sxs-lookup"><span data-stu-id="6a135-233">When the project loads again, the watch task will start running automatically.</span></span>

## <a name="summary"></a><span data-ttu-id="6a135-234">概要</span><span class="sxs-lookup"><span data-stu-id="6a135-234">Summary</span></span>

<span data-ttu-id="6a135-235">Grunt は、ほとんどのクライアント ビルド タスクの自動化に使用できる強力なタスク ランナーです。</span><span class="sxs-lookup"><span data-stu-id="6a135-235">Grunt is a powerful task runner that can be used to automate most client-build tasks.</span></span> <span data-ttu-id="6a135-236">Grunt は、そのパッケージ、およびツールを Visual Studio との統合機能を提供する NPM を活用します。</span><span class="sxs-lookup"><span data-stu-id="6a135-236">Grunt leverages NPM to deliver its packages, and features tooling integration with Visual Studio.</span></span> <span data-ttu-id="6a135-237">Visual Studio のタスク ランナー エクスプ ローラーでは、構成ファイルに対する変更を検出し、タスクの実行、実行中のタスクを表示、および Visual Studio のイベントにタスクを関連付ける便利なインターフェイスを提供します。</span><span class="sxs-lookup"><span data-stu-id="6a135-237">Visual Studio's Task Runner Explorer detects changes to configuration files and provides a convenient interface to run tasks, view running tasks, and bind tasks to Visual Studio events.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="6a135-238">その他の技術情報</span><span class="sxs-lookup"><span data-stu-id="6a135-238">Additional resources</span></span>

   * [<span data-ttu-id="6a135-239">Gulp の使用</span><span class="sxs-lookup"><span data-stu-id="6a135-239">Using Gulp</span></span>](using-gulp.md)