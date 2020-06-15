## 3_Bot:
想定時間: 30-40 分

## ボットの構築

Bot Framework に触れた経験があることを前提としています。経験があれば、問題ありません。経験がない場合でも、心配しすぎないでください。このセクションで十分に学習します。[この Microsoft Virtual Academy コース](https://mva.microsoft.com/ja-jp/training-courses/creating-bots-in-the-microsoft-bot-framework-using-c-17590#!)を修了し、[ドキュメント](https://docs.microsoft.com/ja-jp/bot-framework/)を確認することをお勧めします。

### ラボ: ボット開発の設定

C# SDK を使用してボットを開発します。  開始するには、次の 2 つのことが必要です。
1. Bot Framework プロジェクトのテンプレートは、[ここ](http://aka.ms/bf-bc-vstemplate)でダウンロードできます。  このファイルは "Bot Application.zip" と呼ばれ、\Documents\Visual Studio 2019\Templates\ProjectTemplates\Visual C#\ ディレクトリに保存する必要があります。  ここに zip ファイル全体をドロップするだけです。解凍する必要はありません。  
2. ボットをローカルでテストするために、Bot フレームワーク エミュレーターを[ここ](https://github.com/Microsoft/BotFramework-Emulator/releases/download/v3.5.33/botframework-emulator-Setup-3.5.33.exe)から Bot Framework Emulator をダウンロードしてください。  Emulator は、ブラウザーに応じて、「c:\Users\`_your-username_`\AppData\Local\botframework\app-3.5.33\botframework-emulator.exe」フォルダーにインストールされます。 

### ラボ: 単純なボットを作成して実行する

Visual Studio で、「ファイル」 --> 「新しいプロジェクト」に移動し、「PictureBot」という名前のボット アプリケーションを作成します。必ず「PictureBot」という名前を付けてください。そうしないと、後で問題が発生する可能性があります。  

![新しいボット アプリケーション](./resources/assets/NewBotApplication.png) 

>**単純なボットを作成して実行する**ラボの残りの部分はオプションです。前提条件に従って、Bot Framework の操作経験が必要です。F5 キーを押すと、正しくビルドされていることを確認できます。次のラボに進みましょう。

サンプルのボット コードのファイルの内容を調べます。これは、メッセージとその文字数を繰り返すエコー ボットです。  特に、 **注**
+ App_Start の下の **WebApiConfig.cs** にあるルート テンプレートは api/{controller}/{id} で、ID は省略可能です。  そのため、必ず最後に API/メッセージが追加されたボットのエンドポイントを呼び出します。  
+ Controllers の下の **MessagesController.cs** は、ボットへのエントリポイントです。ボットはさまざまな種類のアクティビティに対応でき、メッセージを送信すると RootDialog が呼び出されます。  
+ Dialogs の下の **RootDialog.cs** にある "StartAsync" はユーザーからのメッセージを待機しているエントリ ポイントであり、"MessageReceiveAsync" は受信したメッセージを処理し、さらにメッセージを待機するメソッドです。  "context.PostAsync" を使用して、ボットからのメッセージをユーザーに送信します。  

F5 キーを押して、サンプル コードを実行します。  NuGet によって、適切な依存関係がが自動的にダウンロードされます。  

http://localhost:3979/ のような URL で、既定の Web ブラウザーでコードを起動します。  

> 楽しい余談: なぜこのポート番号でしょうか?  これは、プロジェクトのプロパティとして設定されます。  ソリューション エクスプローラーで、「プロパティ」をダブルクリックし、「Web」タブを選択します。  プロジェクトの URL は「サーバー」セクションで設定されます。  

![ボット プロジェクトの URL](./resources/assets/BotProjectUrl.jpg) 

プロジェクトがまだ実行されていることを確認し (プロジェクトのプロパティを確認するために停止した場合は、F5 キーをもう一度押す)、Bot Framework Emulator を起動します  (インストールしたばかりの場合は、ローカル コンピューターでの検索時にインデックスが表示されない可能性があるため、c:\Users\your-username\AppData\Local\botframework\app-3.5.27\botframework-emulator.exe. にインストールされていることを確認してください)。  ボットの URL が上記でコードを起動したポート番号と一致していて、最後に API/メッセージが追加されていることを確認します。  ボットと会話できる状態になっているはずです。  

![Bot Emulator](./resources/assets/BotEmulator.png) 

### ラボ: LUIS を使用するようにボットを更新する

LUIS を使用するには、ボットを更新する必要があります。  これを行うには、[LuisDialog クラス](https://docs.botframework.com/ja-jp/csharp/builder/sdkreference/d8/df9/class_microsoft_1_1_bot_1_1_builder_1_1_dialogs_1_1_luis_dialog.html)を使用します。  

**RootDialog.cs** ファイルで、次の名前空間への参照を追加します。

```csharp

using Microsoft.Bot.Builder.Luis;
using Microsoft.Bot.Builder.Luis.Models;

```

次に、RootDialog クラスを変更して、IDialog<object> ではなく、LuisDialog<object> から派生させます。  次に、LUIS アプリ ID と LUIS キーがある LuisModel 属性をクラスに付与します。  これらの値が見つからない場合は、http://luis.ai に戻ります。  アプリケーションをクリックし、「Publish App」 (アプリを公開する) ページに移動します。LUIS アプリ ID と LUIS キーは、エンドポイント URL から取得できます (ヒント: LUIS アプリ ID にはハイフンが含まれ、LUIS キーには含まれません)。

```csharp

using System;
using System.Threading.Tasks;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Connector;
using Microsoft.Bot.Builder.Luis;
using Microsoft.Bot.Builder.Luis.Models;

namespace PictureBot.Dialogs
{
    [LuisModel("96f65e22-7dcc-4f4d-a83a-d2aca5c72b24", "1234bb84eva3481a80c8a2a0fa2122f0")]
    [Serializable]
    public class RootDialog : LuisDialog<object>
    {

```

> 楽しい余談: [Autofac](https://autofac.org/) を使用して、構成ファイルに正しく格納できるように、LuisModel 属性をハードコードせずに、クラスに動的に読み込むことができます。  この例は、[AlarmBot サンプル](https://github.com/Microsoft/BotBuilder/blob/master/CSharp/Samples/AlarmBot/Models/AlarmModule.cs#L24)にあります。  

次に、クラス内の 2 つの既存のメソッド (StartAsync と MessageReceiveAsync) を削除します。  LuisDialog では、LUIS サービスを呼び出し、応答に基づいて適切なメソッドにルーティングする StartAsync の実装が既に行われています。  

最後に、各意図のメソッドを追加します。  対応するメソッドが、最高スコアの意図に対して呼び出されます。  まず、各意図に対して簡単なメッセージを表示します。  

```csharp

        [LuisIntent("")]
        [LuisIntent("None")]
        public async Task None(IDialogContext context, LuisResult result)
        {
            await context.PostAsync("Hmmmm, I didn't understand that.  I'm still learning!");
        }

        [LuisIntent("Greeting")]
        public async Task Greeting(IDialogContext context, LuisResult result)
        {
            await context.PostAsync("Hello!  I am a Photo Organization Bot.  I can search your photos, share your photos on Twitter, and order prints of your photos.  You can ask me things like 'find pictures of food'.");
        }

        [LuisIntent("SearchPics")]
        public async Task SearchPics(IDialogContext context, LuisResult result)
        {
            await context.PostAsync("Searching for your pictures...");
        }

        [LuisIntent("OrderPic")]
        public async Task OrderPic(IDialogContext context, LuisResult result)
        {
            await context.PostAsync("Ordering your pictures...");
        }

        [LuisIntent("SharePic")]
        public async Task SharePic(IDialogContext context, LuisResult result)
        {
            await context.PostAsync("Posting your pictures to Twitter...");
        } 

```

コードを変更したら、F5 キーをクリックして Visual Studio で実行し、Bot Framework Emulator で新しい会話を開始します。  ボットとチャットして、期待どおりの応答を得られることを確認してみましょう。  予期しない結果が生じた場合は、それをメモして LUIS を修正します。  

![ボットで LUIS をテストする](./resources/assets/BotTestLuis.jpg) 

上記のスクリーンショットでは、ボットに対して "order prints"　(プリントを注文して) と言ったときに、別の応答を得ることを期待していました。これは "OrderPic" という意図ではなく、"SearchPics" という意図にマッピングされたようです。  Http://luis.ai に戻ることで LUIS モデルを更新できます。  適切なアプリケーションをクリックし、左側のサイドバーの「Intents」 (意図) をクリックします。  これを新しい発話として手動で追加したり、LUIS の "suggested utterances" (推奨される発話) 機能を利用してモデルを改善したりできます。  "SearchPics" という意図 (または発話のラベルが間違っていた意図) をクリックし、「Suggested utterances」 (推奨される発話) をクリックします。  ラベルが間違っている発話のチェックボックスをオンにして、「Reassign Intent」 (意図の再割り当て) をクリックし、正しい意図を選択します。  

![LUIS で意図を再度割り当てる](./resources/assets/LuisReassignIntent.jpg) 

これらの変更をボットに反映させるには、LUIS モデルを再トレーニングして再度公開する必要があります。  左側のサイドバーで「Publish App」 (アプリの公開) をクリックし、「Train」 (トレーニング) ボタンをクリックして、下の方にある「Publish」 (公開) ボタンをクリックします。  その後、エミュレーターでボットに戻って再度実行できます。  

> 楽しい余談: 提案された発話は非常に効果的です。  LUIS は、どの発話を表面化するかについてスマートに決定します。  人間参加型 (human-in-the-loop) で手動でラベル付けすると、改善するために最大限に役立つものを選びます。  たとえば、LUIS モデルで、特定の発話が 47% の信頼度で Intent1 にマッピングされ、48% の信頼度で Intent2 にマッピングされると予測された場合、このモデルが 2 つの意図の間に非常に近いため、手動でマッピングする人間に対して表面化する有力な候補になります。  

LUIS モデルを使用してユーザーの意図を把握できたので、Azure Cognitive Search を統合して写真を検索してみましょう。  

### ラボ: Azure Cognitive Search 用にボットを構成する

最初に、Azure Cognitive Search インデックスに接続するための関連情報をボットに提供する必要があります。  接続情報を格納するのに最適な場所は、構成ファイルです。  

Web.config を開き、「appSettings」セクションで以下を追加します。

```xml
    <!-- Azure Cognitive Search の設定 -->
    <add key="SearchDialogsServiceName" value="" />
    <add key="SearchDialogsServiceKey" value="" />
    <add key="SearchDialogsIndexName" value="images" />
```

SearchDialogs ServiceName の値を、以前に作成した Azure Cognitive Search サービスの名前に設定します。  必要に応じて、[Azure portal](https://portal.azure.com) に戻ってこれを確認します。  

SearchDialogsServiceKey の値をこのサービスのキーに設定します。  これは、[Azure portal](https://portal.azure.com) で Azure Cognitive Search の「キー」セクションで確認できます。  次のスクリーンショットでは、SearchDialogsServiceName が「aiimmersionsearch」、SearchDialogsServiceKey が「375...」です。  

![Azure Cognitive Search の設定](./resources/assets/AzureSearchSettings.jpg) 

### ラボ: Azure Cognitive Search を使用するようにボットを更新する

次に、Azure Cognitive Search を呼び出すようにボットを更新します。  まず、「ツール」 --> 「NuGet パッケージ マネージャー」 --> 「ソリューションの NuGet パッケージの管理」を開きます。  検索ボックスに「Microsoft.Azure.Search」と入力します。  対応するライブラリを選択し、プロジェクトを示すチェックボックスをオンにしてインストールします。  他の依存関係もインストールされる可能性があります。インストールされているパッケージで、"Newtonsoft.Json" パッケージの更新が必要な場合もあります。

![Azure Cognitive Search NuGet](./resources/assets/AzureSearchNuGet.jpg) 

Visual Studio のソリューション エクスプローラーでプロジェクトを右クリックし、「追加」 --> 「新しいフォルダー」を選択します。  "Models" という名前のフォルダーを作成します。  次に、"Models" フォルダーを右クリックして、「追加」 > 「既存の項目」を選択します。  "Models" フォルダーにこれら 2 つのファイルを追加するには、この操作を 2 回行います (必要に応じて名前空間を調整してください)。
1. [ImageMapper.cs](./resources/code/Models/ImageMapper.cs)
2. [SearchHit.cs](./resources/code/Models/SearchHit.cs)

>このリポジトリ内のファイルは、[resources/code/Models](./resources/code/Models) にあります。

次に、Visual Studio のソリューション エクスプローラーで "Dialogs" フォルダーを右クリックし、「追加」 --> 「クラス」を選択します。  クラス "SearchDialog.cs" を呼び出します。[ここ](./resources/code/SearchDialog.cs)からコンテンツを追加します。

最後に、SearchDialog を呼び出すように RootDialog を更新する必要があります。  "Dialogs" フォルダー内の RootDialog.cs で、"SearchPics" メソッドを更新し、次の "ResumeAfter" メソッドを追加します。

```csharp

        [LuisIntent("SearchPics")]
        public async Task SearchPics(IDialogContext context, LuisResult result)
        {
            // 検索する必要がある検索語句が LUIS によって識別されたかどうかを確認します。  
            string facet = null;
            EntityRecommendation rec;
            if (result.TryFindEntity("facet", out rec)) facet = rec.Entity;

            // 何を検索すべきかわからない (たとえば、ユーザーが
            // 「find pictures of x」 (x の写真を検索) と言う代わりに「find pictures」 (写真を検索) または「search」 (検索) と言った) 場合、
            // 検索語句の入力を求めるプロンプトが表示されます。  
            if (string.IsNullOrEmpty(facet))
            {
                PromptDialog.Text(context, ResumeAfterSearchTopicClarification,
                    "What kind of picture do you want to search for?");
            }
            その他
            {
                await context.PostAsync("Searching pictures...");
                context.Call(new SearchDialog(facet), ResumeAfterSearchDialog);
            }
        }

        private async Task ResumeAfterSearchTopicClarification(IDialogContext context, IAwaitable<string> result)
        {
            string searchTerm = await result;
            context.Call(new SearchDialog(searchTerm), ResumeAfterSearchDialog);
        }

        private async Task ResumeAfterSearchDialog(IDialogContext context, IAwaitable<object> result)
        {
            await context.PostAsync("Done searching pictures");
        }

```

F5 キーを押してボットを再度実行します。  Bot Emulator で、"find dog pics" (犬の写真を探す) または「search for happiness photos」 (幸せそうな写真を検索) と検索してみてください。  写真のタグが要求されたときに、結果が表示されていることを確認します。  


### [4_Bot_Enhancements](./4_Bot_Enhancements.md) に進みましょう

[README](./0_README.md) に戻る