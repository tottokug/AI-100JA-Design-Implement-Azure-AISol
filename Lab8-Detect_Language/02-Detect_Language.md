# ラボ 8 - 言語を検出する

このラボでは、Cognitive Services の言語検出機能をボットに統合します。

## ラボ 8.1: Cognitive Services の URL とキーを取得する

1. [Azure Portal](https://portal.azure.com) を開く

1. リソース グループに移動し、汎用的な Cognitive Services リソースを選択します (別名、すべてのエンド ポイントが含まれます)。

1. 「**リソース管理**」 の 「**クイック スタート**」  タブを選択し、Cognitive Services リソースの URL とキーを記録します。

## ラボ 8.2: ボットに言語サポートを追加する

1. まだ開いていない場合は、**PictureBot** ソリューションを開きます。

1. プロジェクトを右クリックし、**「Nuget パッケージの管理」** を選択します。

1. 「**参照**」を選択します

1. **Microsoft.Azure.CognitiveServices.Language.TextAnalytics** を検索し、「**インストール**」を選択し、「**同意する**」をクリックします。

1. **Startup.cs** ファイルを開き、次の using ステートメントを追加します。

```csharp
using Microsoft.Azure.CognitiveServices.Language.TextAnalytics;
using Microsoft.Azure.CognitiveServices.Language.TextAnalytics.Models;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime;
```

1. 次のコードを **ConfigureServices** メソッドに追加します。

```csharp
services.AddSingleton(sp =>
{
    string cogsBaseUrl = Configuration.GetSection("cogsBaseUrl")?.Value;
    string cogsKey = Configuration.GetSection("cogsKey")?.Value;

    var credentials = new ApiKeyServiceClientCredentials(cogsKey);
    TextAnalyticsClient client = new TextAnalyticsClient(credentials)
    {
        Endpoint = cogsBaseUrl
    };

    return client;
});
```

1. **PictureBot.cs** ファイルを開き、次の using ステートメントを追加します。

```csharp
using Microsoft.Azure.CognitiveServices.Language.TextAnalytics;
using Microsoft.Azure.CognitiveServices.Language.TextAnalytics.Models;
```

1. 次のクラス変数を追加します。

```csharp
private TextAnalyticsClient _textAnalyticsClient;
```

1. コンストラクタを変更して、新しい TextAnalyticsClient を含めます。

```csharp
Public PictureBot（PictureBotAccessors アクセサー、ILoggerFactory loggerFactory、LuisRecognizer 認識エンジン、TextAnalyticsClient analyticsClient）
```

1. コンストラクター内で、クラス変数を初期化します。

```csharp
_textAnalyticsClient = analyticsClient;
```

1. **OnTurnAsync** メソッドに移動し、次のコード行を見つけます。

```csharp
var utterance = turnContext.Activity.Text;
var state = await _accessors.PictureState.GetAsync(turnContext, () => new PictureState());
state.UtteranceList.Add(utterance);
await _accessors.ConversationState.SaveChangesAsync(turnContext);
```

1. その後、次のコード行を追加します

```csharp
//言語を確認する
var result = _textAnalyticsClient.DetectLanguage(turnContext.Activity.Text, "us");

switch (result.DetectedLanguages[0].Name)
{
    case "English":
        break;
    default:
        //エラーを発生する
        await turnContext.SendActivityAsync($"申し訳ありませんが英語しか理解できません。[{result.DetectedLanguages[0].Name}]");
        return;
        break;
}
```

1. **appsettings.json** ファイルを開き、Cognitive Services の設定が入力されていることを確認します。

```csharp
"cogsBaseUrl": "",
"cogsKey" :  ""
```

1. **F5** を押してボットを開始します

1. ボット エミュレーターを使用して、複数のフレーズを送信し、何が起こるかを確認します。

- Como Estes?
- Bon Jour!
- Привет
- Hello

## さらに進む

以前のラボで既に LUIS を紹介しているため、LUIS を使用して複数の言語をサポートするために必要な変更を検討してください。  役立つ記事:

- [LUIS の言語と地域のサポート](https://docs.microsoft.com/ja-jp/azure/cognitive-services/luis/luis-language-support)

## リソース

- [例: Text Analytics で言語を検出する](https://docs.microsoft.com/ja-jp/azure/cognitive-services/text-analytics/how-tos/text-analytics-how-to-language-detection)
- [クイック スタート: .NET 用 Text Analytics クライアント ライブラリ](https://docs.microsoft.com/ja-jp/azure/cognitive-services/text-analytics/quickstarts/csharp)

## 次のステップ

- [ラボ 09-01: DirectLine でボットをテストする](../Lab9-Test_Bots_DirectLine/01-Introduction.md)