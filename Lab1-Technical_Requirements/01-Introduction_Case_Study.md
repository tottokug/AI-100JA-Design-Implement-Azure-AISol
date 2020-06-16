# ラボ 1: 技術的な要件

## 紹介

このラボでは、ワークショップのケース スタディを紹介し、Microsoft Cognitive Services スイート内でツールを構築できるように、ローカル ワークステーションと Azure インスタンスにツールをセットアップします。

## ワークショップのケース スタディ

あなたは、新しい顧客である Adventure Works LLC の担当になりました。この会社は自転車や自転車部品を販売しています。

Adventure Works Cycles は大規模な多国籍製造会社です。同社はインターネット チャネルとリセラーの流通ネットワークを通じて、自転車や自転車部品を北米、欧州、アジアの商業市場向けに製造および販売しています。同社の本拠はワシントン州カークランドで、社員数は 290人 です。また、市場ベース全体で複数の地域営業チームが配置されています。

営業年度の成功を収めた Adventure Works は、既存顧客への追加販売を目標にして、収益の増加を期待しています。昨年、マーケティング部門は、さまざまな展示会やレース イベントで Adventure Works 製品の製品評価データを手動で収集するイニシアチブに着手しました。このデータは現在、Microsoft Excel ファイルで保持されています。

マーケティング部門では、収集された情報によって、評価の結果の手動の分析から、既存のクライアント ベースに追加の製品を販売できることを証明できました。マーケティング チームは、Adventure Works の Web サイトに調査のオンライン バージョンを作成してこれを拡張したいと考えましたが、その作業を聞いた営業部門が、オンライン調査を作成しました。このプラットフォームで着手した調査が非常に貧弱で、フィールドが Excel ファイルで収集されたデータと一致していません。

マーケティング部門は、アップセルのビジネス チャンスとして、評価データを利用して、他の製品を推奨できると強く信じています。しかし、Web サイトに調査を置くことは、必要なデータを収集するためには不十分なチャンネルであることがわかってきており、この状況を改善する方法についてのアドバイスを求めています。さらに、チームは、顧客が増えるにつれて、分析を手動で行うことが非常に困難になることも認識しています。

 Adventure Works は、話す言語が多様な顧客からの大量の問い合わせを処理するために、シームレスにスケールアップすることを目指しています。さらに、スケーラブルなカスタマー サービス プラットフォームを作成し、お客様のニーズ、問題、製品評価に関するより多くの分析情報を得たいと考えています。

また、カスタマー サービス部門は、顧客サポート機能の一部を対話型プラットフォームに任せたいと考えています。目的は、スタッフの作業負荷を軽減し、一般的な質問に速やかに答えることで顧客満足度を高めることです。

### ソリューション

対話型プラットフォームは、次の機能で構成されるボットとして構想できます。

- 顧客の言語を検出する (現時点では英語のみがサポートされていると対応する)

- ユーザーのセンチメントを監視する

- 画像のアップロードを許可し、オブジェクトが自転車であるかどうかを判断する

- FAQ をチャットボットに統合する

- ボット チャットに入力されたテキストに基づいてユーザーの意図を判断する

- 後で確認するためにチャット ボット セッションをログに記録する

ローカル ドライブから写真を取り込み、[Computer Vision API](https://www.microsoft.com/cognitive-services/ja-jp/computer-vision-api) を呼び出して画像を分析し、タグと説明を取得できる簡単な C# アプリケーションをビルドします。

このラボの続きでは、顧客のテキストによる問い合わせに対応する [Bot Framework](https://dev.botframework.com/) ボットをビルドする方法について説明します。次に、[QnA Maker](https://docs.microsoft.com/ja-jp/azure/cognitive-services/qnamaker/overview/overview) によって、既存のナレッジ ベースと FAQ をボット フレームワークに統合するためのクイック ソリューションについて説明します。最後に、[LUIS](https://www.microsoft.com/cognitive-services/ja-jp/language-understanding-intelligent-service-luis) を使用してこのボットを拡張し、クエリから意図を自動的に抽出し、それらを使用して顧客のテキストによる問い合わせにインテリジェントに対応します。

Bing Search を使用して、顧客がボットとの対話中に他のデータにアクセスできるようにするコンテキストのみを提供しますが、ラボではこれらのシナリオを実装しません。参加者は、[Bing Web Search](https://azure.microsoft.com/ja-jp/services/cognitive-services/directory/search/)サービスに関する詳細を読むように指示されます。

このアーキテクチャはこのラボの範囲外ですが、[Blob Storage](https://docs.microsoft.com/ja-jp/azure/storage/storage-dotnet-how-to-use-blobs) と [Cosmos DB](https://azure.microsoft.com/ja-jp/services/cosmos-db/) によって、イメージとメタデータのストレージを管理する Azure データ ソリューションをこのアーキテクチャに統合します。

![アーキテクチャ図](../images/AI_Immersion_Arch.png)

### アーキテクチャ

あなたのチームでは最近、Adventure Works の承認を得た潜在的なアーキテクチャ (以下) を提示しました。

![アーキテクチャ](../images/AI_Immersion_Arch.png)

- [Computer Vision](https://azure.microsoft.com/ja-jp/services/cognitive-services/computer-vision/) により、画像をアップロードし、コンテンツを検出できる
- 静的なナレッジ ベースからのボットとの対話を容易にする [QnA Maker](https://azure.microsoft.com/ja-jp/services/cognitive-services/qna-maker/)
- [Text Analytics](https://azure.microsoft.com/ja-jp/services/cognitive-services/text-analytics/) により、言語検出が可能になる
- [LUIS](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/Home) (Language Understanding Intelligent Service)
により、テキストから意図とエンティティを抽出する
- チャットボット インターフェイスでアプリ インテリジェンスを利用できるようにする [Azure Bot Service](https://azure.microsoft.com/ja-jp/services/bot-service/) コネクタ サービス

## 次のステップ

- [ラボ 01-02: 技術的な要件](02-Technical_Requirements.md)