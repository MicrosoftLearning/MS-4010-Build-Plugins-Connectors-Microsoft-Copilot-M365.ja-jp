---
lab:
  title: 演習 3 - アクセス制御リストを使用して安全なアクセスを確保する
  module: 'LAB 02: Integrate external content with Copilot for Microsoft 365 using Microsoft Graph connectors built with .NET'
---

# 演習 3 - アクセス制御リストを使用して安全なアクセスを確保する

この演習では、選択した項目に ACL を構成して、ローカル Markdown ファイルのインポートを担当するコードを更新します。

## 開始する前に

このステップは、完了までに約 **XX 分**かかります。

## タスク 1 - 組織内のすべてのユーザーが利用できるコンテンツをインポートする

前の演習で外部コンテンツをインポートするためのコードを実装したときに、組織内のすべてのユーザーが使用できるように構成しました。 使用したコードを次に示します。

```csharp
static IEnumerable<ExternalItem> Transform(IEnumerable<DocsArticle> content)
{
  var baseUrl = new Uri("https://learn.microsoft.com/graph/");

  return content.Select(a =>
  {
    var docId = GetDocId(a.RelativePath ?? "");

    return new ExternalItem
    {
      Id = docId,
      Properties = new()
      {
        AdditionalData = new Dictionary<string, object> {
            { "title", a.Title ?? "" },
            { "description", a.Description ?? "" },
            { "url", new Uri(baseUrl, a.RelativePath!.Replace(".md", "")).ToString() }
        }
      },
      Content = new()
      {
        Value = a.Content ?? "",
        Type = ExternalItemContentType.Html
      },
      Acl = new()
      {
          new()
          {
            Type = AclType.Everyone,
            Value = "everyone",
            AccessType = AccessType.Grant
          }
      }
    };
  });
}
```

すべてのユーザーにアクセス権を付与するように ACL を構成しました。 インポート用に選択した Markdown ページに合わせて調整してみましょう。

## タスク 2 - ユーザーの選択に使用できるコンテンツをインポートする

まず、インポートするページの 1 つを、特定のユーザーのみがアクセスできるように構成します。

Web ブラウザーで以下を行います。

1. Azure portal ([https://portal.azure.com](https://portal.azure.com)) に移動し、職場または学校のアカウントでサインインします。
1. サイド バーから **[Microsoft Entra ID]** を選択します。
1. ナビゲーションから、**[ユーザー]** を選択します。
1. ユーザーの一覧から、名前を選択していずれかのユーザーを開きます。
1. **Object ID** プロパティの値をコピーします。

   :::image type="content" source="../media/8-user.png" alt-text="Azure AD ポータルのユーザーのプロファイルのスクリーンショット":::

この値を使用して、特定の Markdown ページの新しい ACL を定義します。

コード エディタで次を行います。

1. **ContentService.cs** ファイルを開き、`Transform` メソッドを検索します。
1. `Select` デリゲート内で、インポートされたすべての項目に適用される既定の ACL を定義します。

   ```csharp
   var acl = new Acl
   {
     Type = AclType.Everyone,
     Value = "everyone",
     AccessType = AccessType.Grant
   };
   ```

1. 次に、マークダウン ファイルのデフォルト ACL を、 `use-the-api.md` で終わる名前でオーバーライドします。

   ```csharp
   if (a.RelativePath!.EndsWith("use-the-api.md"))
   {
     acl = new()
     {
       Type = AclType.User,
       // AdeleV
       Value = "6de8ec04-6376-4939-ab47-83a2c85ab5f5",
       AccessType = AccessType.Grant
     };
   }
   ```

1. 最後に、定義された ACL を使用するように外部項目を返すコードを更新します。

   ```csharp
   return new ExternalItem
   {
     Id = docId,
     Properties = new()
     {
       AdditionalData = new Dictionary<string, object> {
         { "title", a.Title ?? "" },
         { "description", a.Description ?? "" },
         { "url", new Uri(baseUrl, a.RelativePath!.Replace(".md", "")).   ToString() }
       }
     },
     Content = new()
     {
       Value = a.Content ?? "",
       Type = ExternalItemContentType.Html
     },
     Acl = new()
     {
       acl
     }
   };
   ```

1. 更新された `Transform` メソッドは次のようになります。

   ```csharp
   static IEnumerable<ExternalItem> Transform(IEnumerable<DocsArticle>    content)
   {
     var baseUrl = new Uri("https://learn.microsoft.com/graph/");
   
     return content.Select(a =>
     {
       var acl = new Acl
       {
         Type = AclType.Everyone,
         Value = "everyone",
         AccessType = AccessType.Grant
       };
   
       if (a.RelativePath!.EndsWith("use-the-api.md"))
       {
         acl = new()
         {
           Type = AclType.User,
           // AdeleV
           Value = "6de8ec04-6376-4939-ab47-83a2c85ab5f5",
           AccessType = AccessType.Grant
         };
       }
   
       var docId = GetDocId(a.RelativePath ?? "");
   
       return new ExternalItem
       {
         Id = docId,
         Properties = new()
         {
           AdditionalData = new Dictionary<string, object> {
             { "title", a.Title ?? "" },
             { "description", a.Description ?? "" },
             { "url", new Uri(baseUrl, a.RelativePath!.Replace(".md", "")).   ToString() }
           }
         },
         Content = new()
         {
           Value = a.Content ?? "",
           Type = ExternalItemContentType.Html
         },
         Acl = new()
         {
           acl
         }
       };
     });
   }
   ```

1. 変更を保存。

## タスク 3 - 選択したグループで使用できるコンテンツをインポートする

次に、選択したユーザー グループのみが別のページにアクセスできるようにコードを拡張しましょう。

Web ブラウザーで以下を行います。

1. Azure portal ([https://portal.azure.com](https://portal.azure.com)) に移動し、職場または学校のアカウントでサインインします。
1. サイド バーから **[Microsoft Entra ID]** を選択します。
1. ナビゲーションから、**[グループ]** を選択します。
1. グループの一覧から、グループの名前を選択して、いずれかのグループを開きます。
1. **Object ID** プロパティの値をコピーします。

:::image type="content" source="../media/8-group.png" alt-text="グループ ページが開いている Azure portal のスクリーンショット。":::

この値を使用して、特定の Markdown ページの新しい ACL を定義します。

コード エディタで次を行います。

1. **ContentService.cs** ファイルを開き、`Transform` メソッドを検索します
1. 前に定義した `if` 句を拡張し、名前が `traverse-the-graph.md` で終わるマークダウン ファイルの ACL を定義する追加の条件を指定します。

   ```csharp
   if (a.RelativePath!.EndsWith("use-the-api.md"))
   {
     acl = new()
     {
       Type = AclType.User,
       // AdeleV
       Value = "6de8ec04-6376-4939-ab47-83a2c85ab5f5",
       AccessType = AccessType.Grant
     };
   }
   else if (a.RelativePath.EndsWith("traverse-the-graph.md"))
   {
     acl = new()
     {
       Type = AclType.Group,
       // Sales and marketing
       Value = "a9fd282f-4634-4cba-9dd4-631a2ee83cd3",
       AccessType = AccessType.Grant
     };
   }
   ```

1. 更新された `Transform` メソッドは次のようになります。

   ```csharp
   static IEnumerable<ExternalItem> Transform(IEnumerable<DocsArticle>    content)
   {
     var baseUrl = new Uri("https://learn.microsoft.com/graph/");
   
     return content.Select(a =>
     {
       var acl = new Acl
       {
         Type = AclType.Everyone,
         Value = "everyone",
         AccessType = AccessType.Grant
       };
   
       if (a.RelativePath!.EndsWith("use-the-api.md"))
       {
         acl = new()
         {
           Type = AclType.User,
           // AdeleV
           Value = "6de8ec04-6376-4939-ab47-83a2c85ab5f5",
           AccessType = AccessType.Grant
         };
       }
       else if (a.RelativePath.EndsWith("traverse-the-graph.md"))
       {
         acl = new()
         {
           Type = AclType.Group,
           // Sales and marketing
           Value = "a9fd282f-4634-4cba-9dd4-631a2ee83cd3",
           AccessType = AccessType.Grant
         };
       }
   
       var docId = GetDocId(a.RelativePath ?? "");
   
       return new ExternalItem
       {
         Id = docId,
         Properties = new()
         {
           AdditionalData = new Dictionary<string, object> {
               { "title", a.Title ?? "" },
               { "description", a.Description ?? "" },
               { "url", new Uri(baseUrl, a.RelativePath!.Replace(".md",    "")).ToString() }
           }
         },
         Content = new()
         {
           Value = a.Content ?? "",
           Type = ExternalItemContentType.Html
         },
         Acl = new()
         {
             acl
         }
       };
     });
   }
   ```

1. 変更を保存。

## タスク 4 - 新しい ACL を適用する

最後の手順では、新しく構成された ACL を適用します。

1. ターミナルを開いて、作業ディレクトリをプロジェクトに変更します。
1. `dotnet build` コマンドを実行してプロジェクトをビルドします。
1. `dotnet run -- load-content` コマンドを実行して、コンテンツの読み込みを開始します。

[次の演習に進んでください...](./5-exercise-enable-inline-results.md)