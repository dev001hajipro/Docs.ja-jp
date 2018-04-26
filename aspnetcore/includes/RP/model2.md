`Movie` クラスに次のプロパティを追加します。

[!code-csharp[Main](../../tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie/Models/MovieNoEF.cs?name=snippet_MovieNoEF)]

`ID` フィールドは、データベースで主キー用に必要です。

<a name="dc"></a>
### <a name="add-a-database-context-class"></a>データベース コンテキスト クラスの追加

*MovieContext.cs* という次の `DbContext` 派生クラスを *Models* フォルダーに追加します。
[!code-csharp[Main](../../tutorials/razor-pages/razor-pages-start/snapshot_sample/RazorPagesMovie/Models/MovieContext.cs)]

上記のコードによって、エンティティ セットの `DbSet` プロパティが作成されます。 Entity Framework の用語では、エンティティ セットは通常はデータベース テーブルに対応し、エンティティはテーブルの行に対応します。
