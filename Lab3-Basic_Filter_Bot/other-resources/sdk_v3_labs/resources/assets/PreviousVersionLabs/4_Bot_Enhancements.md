## 4_ボットの拡張:
予想時間: 20-30 分

### ラボ: 正規表現とスコラブル グループ

ボットをさらに良いものにするためにできることは多数あります。何よりも、LUIS を単純な "hi" というあいさつ (ボットがかなり頻繁にユーザーから受け取る要求です) に使用したくはありません。  単純な正規表現でこれを行うことができ、時間の節約になり (ネットワークの待機時間)、費用も節約できます (LUIS サービスを呼び出すコスト)。  

また、ボットの複雑さが増し、ユーザーの入力を受け取って複数のサービスを使用して解釈するようになると、そのフローを管理するプロセスが必要になります。  たとえば、最初に正規表現を試してみて、見つからない場合は LUIS を呼び出し、その後は他のサービス、たとえば [QnA Maker](http://qnamaker.ai) や Azure Search を試します。  これを管理する優れた方法は、[ScorableGroups](https://blog.botframework.com/2017/07/06/Scorables/) です。  ScorableGroups では、これらのサービス呼び出しの順序を指定する属性が提供されます。  このコードでは、最初に正規表現と一致する順序を指定し、次に LUIS を呼び出して発話を解釈して、最後に最も優先度が低いものを、一般的な "I'm not sure what you mean" (おっしゃっていることの意味がわかりません) という応答にドロップダウンします。    

ScorableGroups を使用するには、LuisDialog の代わりに、DispatchDialog から RootDialog を継承する必要があります (ただし、クラスに LuisModel 属性を存在させることは可能)。  また、Microsoft.Bot.Builder.Scorables (およびその他) を参照することも必要です。  したがって、RootDialog.cs ファイルで以下を追加します。

```csharp

using Microsoft.Bot.Builder.Scorables;
using System.Collections.Generic;

```

and change your class derivation to:

```csharp

    public class RootDialog : DispatchDialog<object>

```

次に、ScorableGroup 0 の最優先事項として、正規表現と一致する新しいメソッドをいくつか追加してみましょう。  RootDialog クラスの先頭に次の値を追加します。

```csharp

        [RegexPattern("^hello")]
        [RegexPattern("^hi")]
        [ScorableGroup(0)]
        public async Task Hello(IDialogContext context, IActivity activity)
        {
            await context.PostAsync("Hello from RegEx!  I am a Photo Organization Bot.  I can search your photos, share your photos on Twitter, and order prints of your photos.  You can ask me things like 'find pictures of food'.");
        }

        [RegexPattern("^help")]
        [ScorableGroup(0)]
        public async Task Help(IDialogContext context, IActivity activity)
        {
            // ボタン メニューがあるヘルプ ダイアログを開きます  
            List<string> choices = new List<string>(new string[] { "Search Pictures", "Share Picture", "Order Prints" });
            PromptDialog.Choice<string>(context, ResumeAfterChoice, 
                new PromptOptions<string>("How can I help you?", options:choices));
        }

        private async Task ResumeAfterChoice(IDialogContext context, IAwaitable<string> result)
        {
            string choice = await result;
            
            switch (choice)
            {
                case "Search Pictures":
                    PromptDialog.Text(context, ResumeAfterSearchTopicClarification,
                        "What kind of picture do you want to search for?");
                    break;
                case "Share Picture":
                    await SharePic(context, null);
                    break;
                case "Order Prints":
                    await OrderPic(context, null);
                    break;
                default:
                    await context.PostAsync("I'm sorry.I didn't understand you.");
                    break;
            }
        }

```

このコードは、"hi"、"hello"、および "help" で始まるユーザーの式と一致します。  ユーザーが助けを求めると、ボットが実行できる 3 つの主要な操作 (写真の検索、写真の共有、プリントの注文) のボタンの簡単なメニューが表示されます。  

> 楽しい余談: ボットができることについてのオプションを並べたメニューを受け取るためにユーザーが「help」と入力する必要はないと主張する人もいるかもしれませんが、これはボットと最初に接触したときの既定の動作です。**見つけやすさ** はボットにとって最大の課題の 1 つです。このボットに何ができるかをユーザーに知ってもらう必要があります。  優れた[ボット設計の原則](https://docs.microsoft.com/ja-jp/bot-framework/bot-design-principles)が役立ちます。   

この設定により、Scorable Group 1 で正規表現と一致するものがない場合に、2 回目の試行として LUIS を呼び出すようになります。  

LUIS の "None" という意図は、発話が意図にマッピングされていないことを意味します。  このような状況では、次のレベルの ScorableGroup に落とし込みましょう。  次のように、RootDialog クラスに "None" メソッドを変更します。

```csharp

        [LuisIntent("")]
        [LuisIntent("None")]
        [ScorableGroup(1)]
        public async Task None(IDialogContext context, LuisResult result)
        {
            // Luis が勝利の意図として "None" と返したため、
            // 次のレベルの ScorableGroups にドロップダウンします。  
            ContinueWithNextGroup();
        }

```

"greeting" メソッドで、ScorableGroup 属性を追加し、"from LUIS" (LUIS から) を追加して区別します。  コードを実行するときは、"hi" と "hello" (正規表現の一致によって取得する必要がある) と言ってから、"greetings" または "hey there" (トレーニング方法に応じて、LUIS によって取得されることがある) と言ってみてください。  どのメソッドが応答するかに注意してください。  

```csharp

        [LuisIntent("Greeting")]
        [ScorableGroup(1)]
        public async Task Greeting(IDialogContext context, LuisResult result)
        {
            // SCorables について指導できるように、ロジックが重複しています。  
            await context.PostAsync("Hello from LUIS!  I am a Photo Organization Bot.  I can search your photos, share your photos on Twitter, and order prints of your photos.  You can ask me things like 'find pictures of food'.");
        }

```

次に、"SearchPics" メソッドと "OrderPic" メソッドに ScorableGroup 属性を追加します。  

```csharp

        [LuisIntent("SearchPics")]
        [ScorableGroup(1)]
        public async Task SearchPics(IDialogContext context, LuisResult result)
        {
            ...
        }

        [LuisIntent("OrderPic")]
        [ScorableGroup(1)]
        public async Task OrderPic(IDialogContext context, LuisResult result)
        {
            ...
        }

```

> 追加クレジット (後で説明を補足): "Dialogs" フォルダーに OrderDialog クラスを作成します。  [FormFlow](https://docs.botframework.com/ja-jp/csharp/builder/sdkreference/forms.html) を使用して、ボットでプリントを注文するプロセスを作成します。  ボットで次の情報を収集する必要があります。写真サイズ (8x10、5x7、ウォレットなど)、プリント数、光沢ありまたは光沢なし仕上げ、ユーザーの電話番号、ユーザーの電子メール アドレス。

"SharePic" メソッドも更新できます。  これには、はい/いいえの確認のためのプロンプトを表示する方法と、ScorableGroup を設定する方法を示す小さなコードが含まれています。  Twitter の開発者アカウントなどで全員を設定する時間を費やしたくないため、このコードでは実際にはツイートを投稿しませんが、必要に応じて実装しても構いません。  

```csharp

        [LuisIntent("SharePic")]
        [ScorableGroup(1)]
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

### ラボ: ボットを公開する

Microsoft Bot を使用して作成されたボットは、パブリック アクセス可能な任意の URL でホストできます。  このラボの目的のために Azure の Web サイト/App Service でボットをホストします。  

Visual Studio のソリューション エクスプローラーで、ボット アプリケーション プロジェクトを右クリックし、「公開」を選択します。  これにより、ボットを Azure に公開するために役立つウィザードが起動します。  

公開対象として「Microsoft Azure App Service」を選択します。  

![ボットを Azure App Service に公開する](./resources/assets/PublishBotAzureAppService.png) 

「App Service」画面で、適切なサブスクリプションを選択し、「新規」をクリックします。次に、API アプリ名、サブスクリプション、これまでに使用したのと同じリソース グループ、および  App Service プランを入力します。  

![App Service を作成する](./resources/assets/CreateAppService.jpg) 

最後に、Web 配置の設定が表示されたら、「公開」をクリックできます。  Visual Studio の出力ウィンドウに、配置プロセスが表示されます。  次に、ボットが http://testpicturebot.azurewebsites.net/ のような URL でホストされます。"testpicturebot" は App Service API アプリ名です。  

### ラボ: ボット コネクターでボットを登録する

Web ブラウザーを開き、[http://dev.botframework.com](http://dev.botframework.com) に移動します。  「[Register a bot](https://dev.botframework.com/bots/new)」をクリックします。ボットの名前、ハンドル、および説明を入力します。  メッセージング エンドポイントは、https://testpicturebot.azurewebsites.net/api/messages のように最後に "api/messages" が追加された Azure Web サイトの URL になります。  

![ボットの登録](./resources/assets/BotRegistration.jpg) 

次に、ボタンをクリックして Microsoft アプリ ID とパスワードを作成します。  これは、Web.config で必要になるボット アプリ ID とパスワードです。  ボット アプリの名前、アプリ ID、アプリのパスワードを安全な場所に保存してください!  パスワードに対して「OK」をクリックすると、パスワードに戻る方法はなくなります。  次に、「終了してボットのフレームワークに戻る」をクリックします。  

![ボットがアプリ名、ID、およびパスワードを生成する](./resources/assets/BotGenerateAppInfo.jpg) 

ボットの登録ページで、アプリ ID が自動的に入力されている必要があります。必要に応じて、ボットからログに記録するための AppInsights インストルメンテーション キーを追加できます。利用規約に同意する場合は、チェックボックスをオンにして、「登録」をクリックします。  

その後、ボットのダッシュボード ページが表示され、https://dev.botframework.com/bots?id=TestPictureBot のような URL が使用されますが、独自のボット名になります。ここで、さまざまなチャネルを有効にすることができます。  Skype と Web チャットの 2 つのチャネルが自動的に有効になります。  

最後に、ボットの登録情報を更新する必要があります。  Visual Studio に戻り、Web.config を開きます。  ボット ID をアプリ名で、MicrosoftAppId をアプリ ID で更新し、ボットの登録サイトから受け取ったアプリ パスワードを使用して MicrosoftAppPassword を更新します。  

```xml

    <add key="BotId" value="TestPictureBot" />
    <add key="MicrosoftAppId" value="95b76ae6-8643-4d94-b8a1-916d9f753a30" />
    <add key="MicrosoftAppPassword" value="kC200000000000000000000" />

```

プロジェクトを再構築し、ソリューション エクスプローラーでプロジェクトを右クリックし、「公開」をもう一度選択します。  設定は前回に記憶されたものであるはずなので、「公開」をクリックするだけです。 

> MicrosoftAppPassword に関するエラーが発生した場合、これは XML 内にあるため、キーに「&」、「<」、「>」、「'」、「"」が含まれている場合は、これらの記号をそれぞれの[エスケープ機能](https://en.wikipedia.org/wiki/XML#Characters_and_escaping)である「&amp;」、「&lt;」、「&gt;」、「&apos;」、「&quot;」に置き換える必要があります。 

ボットのダッシュボード (https://dev.botframework.com/bots?id=TestPictureBot など) に戻ります。  チャット ウィンドウで話してみてください。  Web チャットでは、カルーセルの外観がエミュレーターの場合とは異なる場合があります。  Channel Inspector と呼ばれる優れたツールがあり、https://docs.botframework.com/ja-jp/channel-inspector/channels/Skype/#navtitle の別のチャネルでさまざまなコントロールのユーザー エクスペリエンスを確認します。  
ボットのダッシュボードから、他のチャンネルを追加したり、Skype、Facebook Messenger、または Slack でボットを試したりできます。  ボットのダッシュボードのチャンネル名の右側にある「追加」ボタンをクリックし、指示に従います。


### [5_Challenge_and_Closing](./5_Challenge_and_Closing.md) に進みましょう

[README](./0_README.md) に戻る