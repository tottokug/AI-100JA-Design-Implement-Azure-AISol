---
lab:
    title: 'ラボ 7: LUIS をボット ダイアログに統合する'
    module: 'モジュール 5: LUIS を使用したボットの拡張'
---

# ラボ 7: LUIS をボット ダイアログに統合する

> 前提条件: このラボは、[ラボ 2](../Lab2-Basic_Filter_Bot/02-Basic_Filter_Bot.md) に基づいて作成されます。
このラボで説明するようにログを実装するには、このラボを実行することをお勧めします。そうでない場合、ニーズによっては、すべての演習を注意深く読み、コードの一部を調るか、独自のアプリケーションで使用するだけで十分です。

このボットは、ユーザーの入力を受け取り、ユーザーの入力に基づいて応答できるようになりました。残念ながら、このボットのコミュニケーション スキルは不安定です。1 つのタイプミス、または単語の言い換えをボットは理解しません。これにより、ユーザーの不満を招くことがあります。[ラボ 4](../Lab4-Implement_LUIS/02-Implement_LUIS.md) で構築した LUIS モデルでボットが自然言語を理解できるようにすることで、ボットの会話能力を大幅に向上させることができます。

LUIS を使用するには、ボットを更新する必要があります。  "Startup.cs" と "PictureBot.c." を変更することで、これを行うことができます。

## ラボ 7.1: 自然言語の理解の追加

### LUIS の Startup.cs への追加

"Startup.cs" を開き、`ConfigureServices`メソッドを見つけます。ここで、状態アクセサーを作成して登録した後、LUIS 用の追加サービスを追加して、LUIS を追加します。

以下のように行います。
```csharp
            services.AddSingleton((Func<IServiceProvider, PictureBotAccessors>)(sp =>
            {
                .
                .
                .
                return accessors;
            });
```

以下を追加します。
```csharp
            // LUIS 認識エンジンを作成して登録します。
            services.AddSingleton(sp =>
            {
                // LUIS 情報を取得します
                var luisApp = new LuisApplication( "YourLuisAppId", "YourLuisKey", "YourLuisEndpoint");

                // LUIS オプションを指定します。これらはボットによって異なる場合があります。
                var luisPredictionOptions = new LuisPredictionOptions
                {
                    IncludeAllIntents = true,
                };

                // 認識エンジンを作成します
                var recognizer = new LuisRecognizer(luisApp, luisPredictionOptions, true,null);
                return recognizer;
            });
```

`luisApp`を LUIS モデルのアプリ ID、サブスクリプション キー、および基本 URI で更新します。基本 URI は "https://region.api.cognitive.microsoft.com/" になります (最後にスラッシュと URL プロトコル仕様を含める)。リージョンは、使用している鍵に関連付けられたリージョンです。リージョンの例としては、`westus`、`westcentralus`、`eastus2`、`southeastasia`などが挙げられます。

基本 URL を見つけるには、www.luis.ai にログインし、**「Publish」** タブに移動し、**「Resources and Keys」** (リソースと鍵) の **「Endpoint」** (エンドポイント) 列を確認します。基本 URL は、**「luis」** と他のパラメーターの前にある **エンドポイント URL** の一部です。

**ヒント**: LUIS アプリ ID にはハイフンが含まれ、LUIS サブスクリプション キーには含まれません。

## ラボ 7.2: LUIS の PictureBot の MainDialog への追加

"PictureBot.cs" を開きます。最初に行う必要があるのは、`PictureBotAccessors`で行ったのと同様に、LUIS 認識エンジンを初期化することです。コメント行`// LUIS 認識エンジンを初期化する`の下に、次の操作を追加します。
```csharp
private LuisRecognizer _recognizer { get; } = null;
```
次に、PictureBot クラスの新しいインスタンスを初期化する場所に移動します。コードの最初の行は次のようになります。
```csharp
public PictureBot(PictureBotAccessors accessors, ILoggerFactory loggerFactory /*, LuisRecognizer recognizer*/)
```

ここで、以前のラボでこのようにコメント アウトしたことにお気付きでしょうか。これまで LUIS を呼び出していなかったので、LUIS 認識エンジンで PictureBot に入力する必要がなかったため、ここでコメント アウトしました。ここで、認識エンジンを使用します。

 入力要件 (パラメーター`LuisRecognizer 認識エンジン`) のコメントを解除し、`// LUIS 認識エンジンを初期化する`の下に次の行を追加します。


```csharp
_recognizer = recognizer ?? throw new ArgumentNullException(nameof(recognizer));
```
繰り返しますが、これは`_accessors`のインスタンスを初期化した方法と非常によく似ています。

`MainDialog`を更新する限り、ユーザーの入力に関係なく、会話の開始時にユーザーが挨拶されるので、最初の`GreetingAsync`の手順で何かを追加する必要はありません。

`MainMenuAsync`では、正規表現を試してみることから始めるため、そのほとんどは残しておきます。ただし、正規表現で意図を見つけられない場合は、`default`アクションを変更します。次に、LUIS を呼び出します。

`MainMenuAsync`スイッチ ブロック内で、以下を置き換えます。
```csharp
                default:
                    {
                        await MainResponses.ReplyWithConfused(stepContext.Context);
                        return await stepContext.EndDialogAsync();
                    }
```
以下に置き換えます。
```csharp
                default:
                    {
                        // LUIS 認識エンジンを呼び出する
                        var result = await _recognizer.RecognizeAsync(stepContext.Context, cancellationToken);
                        // 結果から一番上の意図を取得する
                        var topIntent = result?.GetTopScoringIntent();
                        // 意図に基づいて、上記の正規表現と似た概念の会話を切り替えます
                        switch ((topIntent != null) ? topIntent.Value.intent : null)
                        {
                            case null:
                                // 結果がない場合は、アプリ ロジックを追加します。
                                await MainResponses.ReplyWithConfused(stepContext.Context);
                                break;
                            case "None":
                                await MainResponses.ReplyWithConfused(stepContext.Context);
                                // 各ステートメントで、単にテストを目的として LuisScore を追加し、LUIS が呼び出されたかどうかを確認します。
                                await MainResponses.ReplyWithLuisScore(stepContext.Context, topIntent.Value.intent, topIntent.Value.score);
                                break;
                            case "Greeting":
                                await MainResponses.ReplyWithGreeting(stepContext.Context);
                                await MainResponses.ReplyWithHelp(stepContext.Context);
                                await MainResponses.ReplyWithLuisScore(stepContext.Context, topIntent.Value.intent, topIntent.Value.score);
                                break;
                            case "OrderPic":
                                await MainResponses.ReplyWithOrderConfirmation(stepContext.Context);
                                await MainResponses.ReplyWithLuisScore(stepContext.Context, topIntent.Value.intent, topIntent.Value.score);
                                break;
                            case "SharePic":
                                await MainResponses.ReplyWithShareConfirmation(stepContext.Context);
                                await MainResponses.ReplyWithLuisScore(stepContext.Context, topIntent.Value.intent, topIntent.Value.score);
                                break;
                            default:
                                await MainResponses.ReplyWithConfused(stepContext.Context);
                                break;
                        }
                        return await stepContext.EndDialogAsync();
                    }
```
新しいコードの追加で行うことを簡単に説明します。まず、理解していないと答える代わりに、LUIS を呼び出します。そのため、LUIS 認識エンジンを使用して LUIS を呼び出し、一番上の意図を変数に格納します。次に、`switch`を使用して、ピックアップされた意図に応じて、さまざまな方法で応答します。これは、正規表現で行ったこととほぼ同じです。

> 注記: LUIS で、意図に[ラボ 4](../Lab4-Implement_LUIS/02-Implement_LUIS.md) で指示したのとは異なる名前を付けた場合は、それに応じて`case`ステートメントを変更する必要があります。

もう 1 つの注意点は、LUIS を呼び出したすべての応答の後に、LUIS の意図の値とスコアを追加することです。その理由は、正規表現とは対照的に、LUIS がいつ呼び出されたかだけを示すためです (最終製品からはこれらの応答を削除しますが、ボットをテストする際には良い指標です)。

## ラボ 7.3: 自然な音声フレーズのテスト
F5 キーを押してアプリを実行します。Bot Emulator で、さまざまな方法でイメージを検索してボットに送信してみてください。"send me pictures of water" (水の写真を送って) や "show me dog pics" (犬の写真を見せて) と言うと、どうなりますか? 他の方法で写真を要求したり、共有したり、並べ替えたりしてみてください。

時間が余った場合は、LUIS で予期しないものがピックアップされているかどうかを確認します。おそらく、この時点が luis.ai に移動し、[エンドポイントの発話を確認](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/label-suggested-utterances)し、モデルを再トレーニング/再公開する良いタイミングです。

> 楽しい余談: エンドポイントの発話を確認することは非常に有益です。  LUIS は、どの発話を表面化するかについてスマートに決定します。  人間参加型 (human-in-the-loop) で手動でラベル付けすると、改善するために最大限に役立つものを選びます。  たとえば、LUIS モデルで、特定の発話が 47% の信頼度で Intent1 にマッピングされ、48% の信頼度で Intent2 にマッピングされると予測された場合、このモデルが 2 つの意図の間に非常に近いため、手動でマッピングする人間に対して表面化する有力な候補になります。

## さらに進む
LUIS の実装のカスタマイズで問題が発生した場合は、[ここ](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-builder-howto-v4-luis?view=azure-bot-service-4.0&tabs=cs)でボットの LUIS への追加に関するドキュメント ガイダンスを参照してください。

>行き詰まってしまったときは? このラボのこの時点までのソリューションは、`code/FinishedPictureBot-Part1`(./code/FinishedPictureBot-Part1) にあります。Azure Bot Service のキーを`appsettings.json`ファイルに挿入する必要があります。このコードは、ソリューションとして実行するのではなく、参照として使用することをお勧めしますが、実行する場合は、必要なキーを必ず追加してください (このセクションでは、必ずしも必要はありません)。

**追加クレジット**

Azure Search を含めて、以前の補足的な LUIS model-with-search [training] (https://github.com/Azure/LearnAI-Bootcamp/tree/master/lab01.5-luis) で構築した LUIS ボットの統合を試みる場合は、以下のトレーニングを行ってください。[Azure Search](https://github.com/Azure/LearnAI-Bootcamp/tree/master/lab02.1-azure_search), と [Azure Search ボット](https://github.com/Azure/LearnAI-Bootcamp/blob/master/lab02.2-building_bots/2_Azure_Search.md).