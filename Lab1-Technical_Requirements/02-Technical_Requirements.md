---
lab:
    title: 'ラボ 1: 技術要件の対応'
    module: 'モジュール 1: Azure Cognitive Services の概要'
---

# ラボ 1: 技術要件の対応

## ラボ 1.1: テクノロジ、環境、キーの設定

このラボは、Azure での人工知能 (AI) エンジニアまたは AI 開発者を対象としています。演習を実行する時間を確保するために、このコースのラボを開始するには、特定の要件を満たしている必要があります。

これまで Visual Studio を操作した経験があることが望まれます。ラボで構築するすべてのものに Visual Studio を使用するので、アプリケーションを作成するために[その使用方法](https://docs.microsoft.com/ja-jp/visualstudio/ide/visual-studio-ide)に精通している必要があります。また、これはコードや開発を教えるクラスではありません。C# に関する知識をある程度持っている (中級レベル - [ここ](https://mva.microsoft.com/ja-jp/training-courses/c-fundamentals-for-absolute-beginners-16169?l=Lvld4EQIC_2706218949)と[ここ](https://docs.microsoft.com/ja-jp/dotnet/csharp/quick-starts/)で学習できます) ものの、Cognitive Services を使用してソリューションを実装する方法は知らないことを想定しています。

### アカウントの設定

> 注記: このラボを実行するために、さまざまな環境を使用できます。  講師が環境を稼働させるために必要な手順を指導します。   これは、教室でログインしているコンピューターを使うくらい簡単な場合も、仮想環境を設定するほど複雑な場合もあります。  ラボは Azure 上の Azure Data Science Virtual Machine (DSVM) を使用して作成およびテストされているため、Azure アカウントを使用する必要があります。

#### Azure アカウントを設定します。

Azure の無料試用版は、[https://azure.microsoft.com/ja-jp/free/](https://azure.microsoft.com/ja-jp/free/) でアクティブ化できます。

このラボを実行するために Azure Pass が付与されている場合は、[http://www.microsoftazurepass.com/](http://www.microsoftazurepass.com/) に 移動してアクティブ化できます。  アクティベーション プロセスについて文書化されている [https://www.microsoftazurepass.com/howto](https://www.microsoftazurepass.com/howto) の手順に従ってください。  Microsoft アカウントでは、Azure で **1 つの無料試用版** とそれに関連付けられた 1 つの Azure Pass を使用できるため、Microsoft アカウントで Azure Pass を既に有効にしている場合は、その無料試用版を使用するか、別の Microsoft アカウントを使用する必要があります。

### 環境の設定

これらのラボは、[Visual Studio 2017 Community Edition](https://www.visualstudio.com/downloads/) を使用した .NET Framework で使用することを目的としています。  元のワークショップは、Azure Data Science Virtual Machine (DSVM) で使用するように設計され、テストされました。  Azure で実際に DSVM リソースを作成できるのは Premium Azure サブスクリプションのみですが、ラボは Visual Studio 2017 Community エディションを実行しているローカル コンピューターと、ラボの手順全体で示されている必要なソフトウェアのダウンロードで実行できます。

### キーの設定

このラボを通じて、さまざまな Cognitive Services キーとストレージ キーを収集します。将来のラボで簡単にアクセスできるように、それらのすべてのキーをテキスト ファイルに保存する必要があります。

>_Keys_
>
>- Cognitive Services トレーニング キー:
>- Cognitive Services API キー:
>- LUIS API エンドポイント、キー、アプリ ID:
>- ボット アプリ名:
>- ボット アプリ ID:
>- ボット アプリのパスワード:


#### トレーニング キーの取得:

トレーニング API キーを使用して、プログラムで Custom Vision プロジェクトを作成、管理、およびトレーニングできます。トレーニング キーは、それらをサポートする AI サービスで見つけることができます。  そうでない場合は、作成するサービスごとに生成されるキーを使用できます。   ラボでは、該当するサービスのキーの場所について説明します。

> 注記: Internet Explorer はサポートされていません。Edge、Firefox、または Chrome を使用することをお勧めします。

#### Cognitive Services API キーの取得:

主に [Computer Vision](https://www.microsoft.com/cognitive-services/ja-jp/computer-vision-api) Cognitive Services を使用するため、そのサービスの API キーを作成してみましょう。

Portal で、「**+ 新規作成**」ボタン (その上にカーソルを合わせると、「**リソースの作成**」と表示される) をクリックし、検索ボックスに「**computer vision**」と入力し、「**Computer Vision API**」を選択します。

![Cognitive Services キーの作成](../../Linked_Image_Files/new-cognitive-services.PNG)

作成する API エンドポイントの詳細を入力し、目的の API と、エンドポイントを配置する場所 (**!!米国西部リージョンにしてください。そうしないと、動作しません!!**)、目的の料金プランを選択します。チュートリアルに必要なスループットを確保するために、**S1** を使用します。見つけやすいように、_ダッシュボードにピン留め_します。Computer Vision API では、将来の Cognitive Services Vision サービスを改善するためにマイクロソフト社内に (安全な方法で) 画像を保存するため、リソースを作成する前に、これを承諾することを示すチェックボックスをオンにする必要があります。

**米国西部に Computer Vision サービスを配置することを再確認する**

> 次のラボのコードでは、Computer Vision API を呼び出すために米国西部を使用するように設定されています。今後、[ここ](https://docs.microsoft.com/ja-jp/azure/cognitive-services/Computer-vision/Vision-API-How-to-Topics/HowToSubscribe)で他のリージョンを呼び出す方法を学習できます。


### Bot Builder SDK

このコースでは、C# の Bot Builder テンプレートを使用してボットを作成します。

#### Bot Builder SDK のダウンロード

[ここで C# 用の Bot Builder SDK v4 テンプレート](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4)をダウンロードし、「名前を付けて保存」をクリックして、Visual C# の Visual Studio ItemTemplates フォルダーに保存します。これは通常、`C:\Users\`_your-username_`\Documents\Visual Studio 2017\Templates\ItemTemplates\Visual C#` に配置されています。フォルダーの場所に移動し、インストールをダブルクリックし、「インストール」をクリックして Visual Studio テンプレートにテンプレートを追加します。お使いのブラウザーによっては、テンプレートをダウンロードするときに、それをダブルクリックして Visual Studio Community 2017 に直接インストールできます。それでもかまいません。

### ボット エミュレーター

最新の .NET SDK (v4) を使用してボットを開発します。  開始するには、Bot Framework Emulator をダウンロードして、Web アプリ ボットを作成してソース コードを取得する必要があります。

##### Bot Framework Emulator のダウンロード

ボットをローカルでテストするために、v4 プレビュー Bot Framework Emulator をダウンロードできます。ラボの残りの部分の手順では、v4 Emulatorをダウンロードしていることを前提とします (v3 Emulator ではなく)。エミュレーターをダウンロードするには、[このページ](https://github.com/Microsoft/BotFramework-Emulator/releases)にアクセスし、"4.1.0" というタグが付いている最新バージョンのエミュレーターをダウンロードします (Windows を使用している場合は、".exe" ファイルを選択します)。

エミュレーターは、ブラウザーに応じて、`c:\Users\`_your-username_`\AppData\Local\botframework\app-`_version_`\botframework-emulator.exe` または Downloads フォルダーにインストールされます。

## ラボ 1.2: アーキテクチャの概要

ローカル ドライブから写真を取り込み、[Computer Vision API](https://www.microsoft.com/cognitive-services/ja-jp/computer-vision-api) を呼び出して画像を分析し、タグと説明を取得できる簡単な C# アプリケーションをビルドします。

このラボの続きでは、顧客のテキストによる問い合わせに対応する [Bot Framework](https://dev.botframework.com/) ボットをビルドする方法について説明します。次に、[QnA Maker](https://docs.microsoft.com/ja-jp/azure/cognitive-services/qnamaker/overview/overview) によって、既存のナレッジ ベースと FAQ をボット フレームワークに統合するためのクイック ソリューションについて説明します。最後に、[LUIS](https://www.microsoft.com/cognitive-services/ja-jp/language-understanding-intelligent-service-luis) を使用してこのボットを拡張し、クエリから意図を自動的に抽出し、それらを使用して顧客のテキストによる問い合わせにインテリジェントに対応します。

Bing Search を使用して、顧客がボットとの対話中に他のデータにアクセスできるようにするコンテキストのみを提供しますが、ラボではこれらのシナリオを実装しません。参加者は、[Bing Web 検索](https://azure.microsoft.com/ja-jp/services/cognitive-services/directory/search/)サービスに関する詳細を読むように指示されます。

このアーキテクチャはこのラボの範囲外ですが、[Blob Storage]((https://docs.microsoft.com/ja-jp/azure/storage/storage-dotnet-how-to-use-blobs) と [Cosmos DB](https://azure.microsoft.com/ja-jp/services/cosmos-db/) によって、イメージとメタデータのストレージを管理する Azure データ ソリューションをこのアーキテクチャに統合します。

![アーキテクチャの図](../../Linked_Image_Files/AI_Immersion_Arch.png)

## ラボ 1.3: 将来のプロジェクト/学習のためのリソース

ここで説明するアーキテクチャの理解を深め、より広範なチームが AI ソリューションの開発に関与できるようにするために、次のリソースを確認することをお勧めします。

- [Cognitive Services](https://www.microsoft.com/cognitive-services)  -  AI エンジニア
- [Cosmos DB](https://docs.microsoft.com/ja-jp/azure/cosmos-db/)  -  データ エンジニア
- [Azure Search](https://azure.microsoft.com/ja-jp/services/search/)  -  検索エンジニア
- [ボット開発者ポータル](http://dev.botframework.com) - AI エンジニア


## クレジット

このシリーズのラボは、[Cognitive Services チュートリアル](https://github.com/noodlefrenzy/CognitiveServicesTutorial)と [AI 学習短期集中講座](https://github.com/Azure/LearnAI-Bootcamp)を再利用しています。
