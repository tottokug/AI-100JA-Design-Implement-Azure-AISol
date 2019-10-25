## 3_LUIS:
予想時間: 15-20 分

### ラボ 3.1: LUIS を使用するようにボットを更新する

LUIS を使用するには、ボットを更新する必要があります。  これを行うには、[LuisDialog クラス](https://docs.botframework.com/ja-jp/csharp/builder/sdkreference/d8/df9/class_microsoft_1_1_bot_1_1_builder_1_1_dialogs_1_1_luis_dialog.html)を使用します。  

**RootDialog.cs** ファイルで、次の名前空間への参照を追加します。

```csharp

using Microsoft.Bot.Builder.Luis;
using Microsoft.Bot.Builder.Luis.Models;

```

次に、LUIS アプリ ID と LUIS キーがある LuisModel 属性をクラスに付与します。  これらの値が見つからない場合は、http://luis.ai に戻ります。  アプリケーションをクリックし、「Publish App」(アプリを公開する) ページに移動します。LUIS アプリ ID と LUIS キーは、エンドポイント URL から取得できます (ヒント: LUIS アプリ ID にはハイフンが含まれ、LUIS キーには含まれません)。LUIS アプリ ID と LUIS キーとともに、次のコード行を `[Serializable]` のすぐ上に配置する必要があります。

```csharp

namespace PictureBot.Dialogs
{
    [LuisModel("YOUR-APP-ID", "YOUR-SUBSCRIPTION-KEY")]
    [Serializable]

...
```

> 楽しい余談: [Autofac](https://autofac.org/) を使用して、構成ファイルに正しく格納できるように、LuisModel 属性をハードコードせずに、クラスに動的に読み込むことができます。  この例は、[AlarmBot サンプル](https://github.com/Microsoft/BotBuilder/blob/master/CSharp/Samples/AlarmBot/Models/AlarmModule.cs#L24)にあります。  


"Greeting" の意図 (DispatchDialog 内に既に存在している以下のすべてのコード) を追加することで、簡単に開始できます。

```csharp
        [LuisIntent("Greeting")]
        [ScorableGroup(1)]
        public async Task Greeting(IDialogContext context, LuisResult result)
        {
            // SCorables について指導できるように、ロジックが重複しています。  
            await context.PostAsync("Hello from LUIS!  I am a Photo Organization Bot.  I can search your photos, share your photos on Twitter, and order prints of your photos.  You can ask me things like 'find pictures of food'.");
        }
```

F5 キーを押してアプリを実行します。Bot Emulator で、ボットにさまざまな方法で "hello" と送信してみてください。"whats up" と "hello" を送信するとどうなりますか?

LUIS の "None" という意図は、発話が意図にマッピングされていないことを意味します。  このような状況では、次のレベルの ScorableGroup に落とし込みましょう。  次のように、RootDialog クラスに "None" メソッドを追加します。
```csharp
        [LuisIntent("")]
        [LuisIntent("None")]
        public async Task None(IDialogContext context, LuisResult result)
        {
            // Luis が勝利の意図として "None" と返したため、
            // 次のレベルの ScorableGroups にドロップダウンします。  
            ContinueWithNextGroup();
        }
```

最後に、残りのインテント用のメソッドを追加します。  対応するメソッドが、最高スコアの意図に対して呼び出されます。"SearchPics" と "SharePics" のコードについて、近くにいる人と話します。このコードでは他に何をしますか? 


```csharp
        [LuisIntent("SearchPics")]
        public async Task SearchPics(IDialogContext context, LuisResult result)
        {
            // 検索する必要がある検索語句が LUIS によって識別されたかどうかを確認します。  
            string facet = null;
            EntityRecommendation rec;
            if (result.TryFindEntity("facet", out rec)) facet = rec.Entity;

            // 何を検索すべきかわからない (たとえば、ユーザーが
            // 「find pictures of x」(x の画像を検索) と言う代わりに「find pictures」(画像を検索) または「search」(検索) と言った) 場合、
            // 検索語句の入力を求めるプロンプトが表示されます。  
            if (string.IsNullOrEmpty(facet))
            {
                PromptDialog.Text(context, ResumeAfterSearchTopicClarification,
                    "What kind of picture do you want to search for?");
            }
            else
            {
                await context.PostAsync("Searching pictures...");
                context.Call(new SearchDialog(facet), ResumeAfterSearchDialog);
            }
        }

        [LuisIntent("OrderPic")]
        public async Task OrderPic(IDialogContext context, LuisResult result)
        {
            await context.PostAsync("Ordering your pictures...");
        }

        [LuisIntent("SharePic")]
        public async Task SharePic(IDialogContext context, LuisResult result)
        {
            PromptDialog.Confirm(context, AfterShareAsync,
                "Are you sure you want to tweet this picture?");
        }

        private async Task AfterShareAsync(IDialogContext context, IAwaitable<bool> result)
        {
            if (result.GetAwaiter().GetResult() == true)
            {
                // はい、写真を共有します。
                await context.PostAsync("Posting tweet.");
            }
            else
            {
                // いいえ、写真を共有しないでください。  
                await context.PostAsync("OK, I won't share it.");
            }
        }


```



> ヒント: "SharePic" メソッドには、はい/いいえの確認のためのプロンプトを表示する方法と、ScorableGroup を設定する方法を示す小さなコードが含まれています。Twitter の開発者アカウントなどで全員を設定する時間を費やしたくないため、このコードでは実際にはツイートを投稿しませんが、必要に応じて実装しても構いません。

追加した意図に対してスコラブル グループが実装されていないことに気付いたかもしれません。すべての`LuisIntent`がスコラブル グループの優先度 1 に設定できるようにコードを変更します。LUIS 機能を追加したので、`ResumeAfterChoice`メソッドの 2 つの`await`行のコメントを解除できます。  

コードを変更したら、F5 キーをクリックして Visual Studio で実行し、Bot Framework Emulator で新しい会話を開始します。  ボットとチャットして、期待どおりの応答を得られることを確認してみましょう。  予期しない結果が生じた場合は、それをメモして [LUIS を修正](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/Train-Test)します。

LUIS モデルを再トレーニングして再度公開する必要があることを忘れないでください。  その後、エミュレーターでボットに戻ってやり直すことができます。  

> 楽しい余談: 提案された発話は非常に効果的です。  LUIS は、どの発話を表面化するかについてスマートに決定します。  人間参加型 (human-in-the-loop) で手動でラベル付けすると、改善するために最大限に役立つものを選びます。  たとえば、LUIS モデルで、特定の発話が 47% の信頼度で Intent1 にマッピングされ、48% の信頼度で Intent2 にマッピングされると予測された場合、このモデルが 2 つの意図の間に非常に近いため、手動でマッピングする人間に対して表面化する有力な候補になります。  


> 追加クレジット (後で説明を補足): "Dialogs" フォルダーに OrderDialog クラスを作成します。  [FormFlow](https://docs.botframework.com/ja-jp/csharp/builder/sdkreference/forms.html) を使用して、ボットでプリントを注文するプロセスを作成します。  ボットで次の情報を収集する必要があります。写真サイズ (8x10、5x7、ウォレットなど)、プリント数、光沢ありまたは光沢なし仕上げ、ユーザーの電話番号、ユーザーの電子メール アドレス。



最後に、上記のサービスのいずれでも理解できなかった場合は、既定のハンドラーを追加します。  この ScorableGroup は、LuisIntent 属性または RegexPattern 属性 (MethodBind を含む) で装飾されていないため、明示的な MethodBind が必要です。

```csharp

        // 以前のグループのスコラブルはいずれも獲得しなかったため、ダイアログでヘルプ メッセージを送信します。
        [MethodBind]
        [ScorableGroup(2)]
        public async Task Default(IDialogContext context, IActivity activity)
        {
            await context.PostAsync("I'm sorry.I didn't understand you.");
            await context.PostAsync("You can tell me to find photos, tweet them, and order prints.  次に例を示します。\"find pictures of food\".");
        }

```

F5 キーを押してボットを実行し、Bot Emulator でテストします。  



### [4_Publish_and_Register](./4_Publish_and_Register.md) に進みましょう  
[README](./0_README.md) に戻る