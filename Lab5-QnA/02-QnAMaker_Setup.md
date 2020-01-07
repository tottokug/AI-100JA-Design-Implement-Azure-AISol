---
lab:
    title: 'ラボ 5: カスタマイズされた QnA Maker ボットの作成'
    module: 'モジュール 3: QnA Maker でのボットの拡張'
---

# ラボ 5: カスタマイズされた QnA Maker ボットの作成

##  紹介

このラボでは、事前にトレーニングされたナレッジ ベースに接続するボットを作成するための QnA Maker について調査します。QnAMaker を使用すると、ドキュメントをアップロードしたり、Web ページをポイントしたり、よくある質問などの一般的な使用のために単純なボットをフィードできるナレッジ ベースを事前に入力したりできます。

## ラボ 5.1: QnA Maker のセットアップ

1.  [Azure Portal](https://portal.azure.com) を開く

1.  [**新しいリソースを追加**] をクリックします

1.  **QnAシリーズメーカー**を入力し、**QnA シリーズメーカー**を選択します。

1.  [**作成**] をクリックします。

1.  名前を入力します

1.  リソース価格設定層の **S0** 層を選択します。1 MB を超えるファイルを後でアップロードするため、無料利用枠は使用していません。

1.  リソース グループを選択します。

1.  検索価格帯については、 **F** 層を選択します

1.  アプリ名を入力してください。

1.  **作成** をクリックします。これにより、リソース グループに次のリソースが作成されます。

-   アプリのサービスプラン
-   アプリ サービス
-   Application Insights
-   検索サービス
-   タイプ QnAMaker の Cognitive Service インスタンス

## ラボ 5.2: KnowledgeBase を作成する

1.  [QnA Maker サイト](https://qnamaker.ai)を開く

1.  右上の [**サインイン**] をクリックします。新しい QnA Maker リソースの Azure 資格情報を使用してログインします

1.  上部のナビゲーション領域で、「**ナレッジベースの作成**」をクリックします。

1.  リソースを既に作成しているため、**ステップ 1** をスキップします。

1.  QnA Maker リソースに関連付けられた Azure AD とサブスクリプションを選択してから、新しく作成された QnA Maker リソースを選択します。

1.  名前に「**Microsoft FAQs***」と入力します

1.  ファイルについては、**ファイルの追加**をクリックして、 **code/surface-pro-4-user-guide-EN.pdf** ファイルを参照します。

1.  ファイルの場合は、[**ファイルの追加**] をクリックして、コードを参照し、 **Azure Blob Storage** ファイルを管理します。

> **注**: サポートされているファイルの種類とデータソースの詳細については、[こちらをご覧ください。](https://docs.microsoft.com/ja-jp/azure/cognitive-services/qnamaker/concepts/data-sources-supported)

1.  **Chit-chat **には、**Witty** を選択します。

1.  [**Create your KB**] (KB を作成する) をクリックします。

## ラボ 5.3: ナレッジ ベースを公開してテストする

1.  ナレッジ ベースの QnA ペアを確認します。フィードした2つのドキュメントに基づいて、最大 200 の異なるペアが表示されます。

1.  トップメニューで [**公開**] をクリックします。

1.  公開ページで、[**公開**] をクリックします。サービスが公開されたら、成功ページの [**ボットの作成**]ボタンをクリックします

1.  プロンプトが表示されたら、ラボ リソース グループに関連付けられたアカウントとしてログインします。

1.  ボットサービスの作成ページで、命名エラーを修正し、[**作成**] をクリックします。

> **注**: Azure の最近の変更では、一部のリソース名からダッシュ（ "-"）を削除する必要があります。

1.  ボットリソースが作成されたら、新しい **Web アプリ ボット**に移動し、[**Webチャットでテストする**] をクリックします

1.  Surface Pro 4 または Azure Blog Storage の管理に関連する質問をボットに問い合わせてください。

+ メモリを追加するには
+ バッテリーの充電にはどれくらい時間がかかるか
+ ハード リセットする方法
+ BLOB とは

1.  次のような不明な点に関して質問をしてください。

+ ボウリングでストライキを出すには

## ラボ 5.4: ボット ソース コードをダウンロードする

1.  **ボットの管理**の下で、[**ビルド**] タブを選択

1.  [**ボット ソース コードのダウンロード**]をクリックし、プロンプトが表示されたら [**はい**] をクリックします。

1.  Azure はソースコードをビルドします。完了したら、[**ボットソースコードのダウンロード**]をクリックし、プロンプトが表示されたら [**はい**] を選択します。

1.  zip ファイルをローカル コンピュータに展開する

1.  ソリューション ファイルを開くと、Visual Studio が開きます。

1.  **Startup.cs** ファイルを開くと、特別なものは何も追加されていないことが分かります。

1.  **Bots/{BotName}.cs** ファイルを開き、QnA Maker が初期化されている場所に注意してください。

```csharp
var qnaMaker = new QnAMaker(new QnAMakerEndpoint
{
    KnowledgeBaseId = _configuration["QnAKnowledgebaseId"],
    EndpointKey = _configuration["QnAAuthKey"],
    Host = GetHostname()
},
null,
httpClient);
```

その後、以下を呼び出します:

```csharp
var response = await qnaMaker.GetAnswersAsync(turnContext);
if (response != null && response.Length > 0)
{
    await turnContext.SendActivityAsync(MessageFactory.Text(response[0].Answer), cancellationToken);
}
else
{
    await turnContext.SendActivityAsync(MessageFactory.Text("No QnA Maker answers were found."), cancellationToken);
}
```

ご覧のとおり、わずか数行のコードで、生成された QnA Maker を独自のボットに追加するのは非常に簡単です。

## さらに進む

QnAMaker コードを画像ボットに統合するには

##  リソース

-   [QnA Maker サービスとは](https://docs.microsoft.com/ja-jp/azure/cognitive-services/qnamaker/overview/overview)

##  次のステップ

-   [ラボ 06-01: LUIS を実装する](../Lab6-Implement_LUIS/01-Introduction.md)