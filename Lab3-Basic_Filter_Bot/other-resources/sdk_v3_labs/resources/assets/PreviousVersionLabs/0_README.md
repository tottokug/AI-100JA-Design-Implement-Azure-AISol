# LUIS と Azure Search を使用したインテリジェント アプリケーションの開発

このハンズオン ラボでは、Microsoft Bot Framework、Azure Search、およびマイクロソフトの Language Understanding Intelligent Service (LUIS) を使用して、エンド ツー エンドでインテリジェントなボットを作成する方法について説明します。


## 目的
このワークショップでは、次の操作を行います。
- Azure Search 機能を実装して、アプリケーション内で肯定的な検索エクスペリエンスを提供する方法を理解する
- フルテキスト検索、言語認識検索を有効にするためにデータを拡張するように Azure Search サービスを構成する
- ボットが効果的に通信できるように LUIS モデルを構築、トレーニング、公開する
- LUIS と Azure Search を活用する Microsoft Bot Framework を使用してインテリジェント ボットを構築する


ここでは LUIS と Azure Search に重点を置いていますが、次のテクノロジも活用します。

- Data Science Virtual Machine (DSVM)
- Windows 10 SDK (UWP)
- CosmosDB
- Azure Storage
- Visual Studio


## 前提条件

このワークショップは、Azure での AI 開発者を対象としています。これは短いワークショップですので、事前に必要なことがあります。

まず、Visual Studio での経験が必要です。ワークショップで構築するすべてに Visual Studio を使用するので、アプリケーションを作成するために[その使用方法](https://docs.microsoft.com/ja-jp/visualstudio/ide/visual-studio-ide)に精通している必要があります。また、これは、アプリケーションのコーディングや開発方法を教えるクラスではありません。C# でのコーディング方法を知っている ([ここ](https://mva.microsoft.com/ja-jp/training-courses/c-fundamentals-for-absolute-beginners-16169?l=Lvld4EQIC_2706218949)で学習できます) ものの、高度な検索および NLP (自然言語処理) ソリューションを実装する方法は知らないことを前提とします。

次に、マイクロソフトの Bot Framework を使用してボットを開発した経験が必要です。設計方法やダイアログのしくみについてディスカッションする時間はあまりありません。Bot Framework に慣れていない場合は、ワークショップに参加する前に、[この Microsoft Virtual Academy コース](https://mva.microsoft.com/ja-jp/training-courses/creating-bots-in-the-microsoft-bot-framework-using-c-17590#!)を受講する必要があります。

3 番目に、portal での経験があり、Azure でリソースを作成する (および費用をかける) ことが可能である必要があります。このワークショップでは、Azure Pass は提供しません。

>注記: このワークショップは、Visual Studio Community バージョン 15.4.0 を使用した Data Science Virtual Machine (DSVM) で開発およびテストされました。

## 概要

独自のイメージを取り込み、Cognitive Services を使用してイメージ内のオブジェクトや人物を検索して、それらの人物がどのように感じているかを把握し、そのデータをすべて NoSQL ストア (DocumentDB) に格納することが可能なエンド ツー エンドのシナリオを構築します。この NoSQL ストアを使用して Azure Search インデックスを設定し、LUIS を使用して Bot Framework ボットを構築して、簡単なターゲットを絞ったクエリを実行できるようにします。

> 注記: このラボは`lab01.1-pcl_and_cognitive_services`の続きです。Cognitive Services　を使用して写真を取り込んでそのイメージに関する情報を判断し、DocumentDB にデータを格納することはスキップします。そのラボで使用するのは、検索インデックスを入力する DocumentDB だけです。`lab01.1-pcl_and_cognitive_services`を完了している場合は、必要に応じて、指定された文字列の代わりに DocumentDB 接続文字列を使用できます。

## アーキテクチャ

`lab01.1-pcl_and_cognitive_services`では、ローカル ドライブから写真を取り込み、いくつかの Cognitive Services を呼び出して、それらのイメージに関するデータを取得できる単純な C# アプリケーションを構築しました。

- [Computer Vision](https://www.microsoft.com/cognitive-services/ja-jp/computer-vision-api): タグと説明を取得するために使用します
- [Face](https://www.microsoft.com/cognitive-services/ja-jp/face-api): 各イメージから顔とその詳細を取得するために使用します
- [Emotion](https://www.microsoft.com/cognitive-services/ja-jp/emotion-api): イメージ内の各顔から感情スコアを引き出すために使用します

このデータを取得した後でそれを処理し、[NoSQL](https://en.wikipedia.org/wiki/NoSQL) [PaaS](https://azure.microsoft.com/ja-jp/overview/what-is-paas/) オファリングである [DocumentDB](https://azure.microsoft.com/ja-jp/services/documentdb/) に必要なすべての情報を保存しました。

DocumentDB で、[Azure Search](https://azure.microsoft.com/ja-jp/services/search/) インデックスを作成します (Azure Search は、フォールト トレラントなファセット検索のための PaaS オファリングで、管理のオーバーヘッドがない Elastic Search とお考えください)。データのクエリを実行し、そのクエリを実行する [Bot Framework](https://dev.botframework.com/) ボットを構築する方法について説明します。最後に、[LUIS](https://www.microsoft.com/cognitive-services/ja-jp/language-understanding-intelligent-service-luis) を使用してこのボットを拡張し、クエリから意図を自動的に抽出し、それらを使用して検索をインテリジェントに指示します。

![アーキテクチャの図](./resources/assets/AI_Immersion_Arch.png)

> このラボは、この [Cognitive Services チュートリアル](https://github.com/noodlefrenzy/CognitiveServicesTutorial)を変更したものです。

## GitHub のナビゲート##

[リソース](./resources) フォルダーにはいくつかのディレクトリがあります:

- **assets**: ここには、ラボ マニュアルのすべてのイメージが含まれています。このフォルダーは無視できます。
- **code**: ここには、使用するいくつかのディレクトリがあります。
	- **LUIS**: ここでは、PictureBot の LUIS モデルを見つけることができます。独自の LUIS を作成しますが、進行が遅れた場合や、別の LUIS モデルをテストする場合は、.json ファイルを使用してこの LUIS アプリをインポートできます。
	- **Models**: これらのクラスは、PictureBot に検索を追加するときに使用されます。
	- **Finished-PictureBot**: ここでは、LUIS と検索インデックスを Bot Framework に統合する、ワークショップの後半のセクション用に完成した PictureBot.sln があります。進行が遅れた場合や、行き詰まってしまった場合は、これを使用できます。

> これらのラボを実行するには Visual Studio が必要ですが、ワークショップのいずれか用に Windows Data Science Virtual Machine を既にデプロイしている場合は、それを使用できます。

## キーの収集

このラボを通して、さまざまなキーを収集します。ワークショップ全体で簡単にアクセスできるように、それらのすべてのキーをテキスト ファイルに保存することをお勧めします。

>_Keys_
>- LUIS API:
>- Cosmos DB 接続文字列:
>- Azure Search 名:
>- Azure Search キー:
>- Bot Framework アプリ名:
>- Bot Framework アプリ ID:
>- Bot Framework アプリのパスワード:


## ラボのナビゲート

このワークショップは 5 つのセクションに分かれています。
- [1_AzureSearch](./1_AzureSearch.md): ここでは、Azure Search について学習し、インデックスを作成します
- [2_LUIS](./2_LUIS.md): LUIS モデルを構築して、ボットの言語理解を深めます (次のラボでビルドします)
- [3_Bot](./3_Bot.md): Bot Framework を使用してすべてをまとめる方法を学習します
- [4_Bot_Enhancements](./4_Bot_Enhancements.md): 正規表現でボットを拡張して完成させ、ボット コネクターを使用して公開します。
- [5_Challenge_and_Closing](./4_Challenge_and_Closing.md): すべてのラボを通して、この課題にチャレンジしましょう。また、学習した内容の概要と詳細情報を得られる場所も示します。



### [1_AzureSearch](./1_AzureSearch.md) に進みましょう


