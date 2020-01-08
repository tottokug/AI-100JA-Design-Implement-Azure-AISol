---
lab:
    title: 'ラボ 1: 技術的な要件を満たす'
    module: 'モジュール 1: Azure Cognitive Services の概要'
---

# ラボ 1: 技術的な要件を満たす

## ラボ 1.1: テクノロジ、環境、キーの設定

このラボは、Azure での人工知能 (AI) エンジニアまたは AI 開発者を対象としています。演習を実行する時間を確保するために、このコースのラボを開始するには、特定の要件を満たしている必要があります。

これまで Visual Studio を操作した経験があることが望まれます。ラボで構築するすべてのものに Visual Studio を使用するので、アプリケーションを作成するために[その使用方法](https://docs.microsoft.com/ja-jp/visualstudio/ide/visual-studio-ide)に精通している必要があります。また、これはコードや開発を教えるクラスではありません。C# に関する知識をある程度持っている (中級レベル - [ここ](https://mva.microsoft.com/ja-jp/training-courses/c-fundamentals-for-absolute-beginners-16169?l=Lvld4EQIC_2706218949)と[ここ](https://docs.microsoft.com/ja-jp/dotnet/csharp/quick-starts/)で学習できます) ものの、Cognitive Services を使用してソリューションを実装する方法は知らないことを想定しています。

### アカウントの設定

> **注**このラボを実行するために、さまざまな環境を使用できます。講師が環境を稼働させるために必要な手順を指導します。これは、教室でログインしているコンピューターを使うくらい簡単な場合も、仮想環境を設定するほど複雑な場合もあります。ラボは Azure 上の Azure Data Science Virtual Machine (DSVM) を使用して作成およびテストされているため、Azure アカウントを使用する必要があります。

#### Azure アカウントを設定します。

Azure の無料試用版は、[https://azure.microsoft.com/ja-jp/free/](https://azure.microsoft.com/ja-jp/free/) でアクティブ化できます。

このラボを実行するために Azure Pass が付与されている場合は、[http://www.microsoftazurepass.com/](http://www.microsoftazurepass.com/) に 移動してアクティブ化できます。アクティベーション プロセスについて文書化されている [https://www.microsoftazurepass.com/howto](https://www.microsoftazurepass.com/howto) の手順に従ってください。Microsoft アカウントでは、Azure で **1 つの無料試用版** とそれに関連付けられた 1 つの Azure Pass を使用できるため、Microsoft アカウントで Azure Pass を既に有効にしている場合は、その無料試用版を使用するか、別の Microsoft アカウントを使用する必要があります。

### 環境の設定

これらのラボは、[Visual Studio 2019 Community Edition](https://www.visualstudio.com/downloads/) を使用した .NET Framework で使用することを目的としています。元のワークショップは、Azure Data Science Virtual Machine (DSVM) で使用するように設計され、テストされました。Azure で実際に DSVM リソースを作成できるのは Premium Azure サブスクリプションのみですが、ラボは Visual Studio 2019 Community エディションを実行しているローカル コンピューターと、ラボの手順全体で示されている必要なソフトウェアのダウンロードで実行できます。

### 必要な URL とキー

このラボを通じて、さまざまな Cognitive Services キーとストレージ キーを収集します。将来のラボで簡単にアクセスできるように、それらのすべてのキーをテキスト ファイルに保存する必要があります。

>_Keys_
>
>- Cognitive Services API URL:
>- Cognitive Services API キー:
>- LUIS API エンドポイント:
>- LUIS API キー:
>- LUIS API アプリ ID:
>- ボット アプリ名:
>- ボット アプリ ID:
>- ボット アプリのパスワード:
>- Azure Storage 接続文字列:
>- Cosmos DB URI:
>- Cosmos DB キー:
>- DirectLineキー:

> **注**: このラボでは、これらのすべてが入力されるわけではありません。

### Azure セットアップ

次の手順では、次のラボの Azure 環境を構成します。

####    Cognitive Services

最初のラボでは[Computer Vision](https://www.microsoft.com/cognitive-services/ja-jp/computer-vision-api) Cognitive Servicesに焦点を当てていますが、Microsoft Azure では、すべてのサービスにまたがる Cognitive Services アカウントを作成することも、個々のサービスの Cognitive Services アカウントを作成することもできます。次の手順では、すべての Cognitive Services にまたがるアカウントを作成します。

1.  [Azure Portal](https://portal.azure.com) を開く

1.  [**+リソースの作成**] をクリックし、検索ボックスに **Cognitive Services** と入力します 

1.  利用可能なオプションから **Cognitive Services** を選択し、[**作成**] をクリックします。

> **注**: 特定のCognitive Servicesリソースを作成することも、すべてのエンドポイントを含む単一のリソースを作成することもできます。

1.  任意の名前を入力してください

1.  サブスクリプションとリソース グループを選択してください

1.  価格帯については、 **S0**を選択します

1.  確認チェックボックスをオンにします

1.  [**作成**] をクリックします。

1.  新しいリソースに移動し、 **クイック スタート**をクリックします

1.  **URL** と**エンドポイント**をメモ帳にコピーします

####    作成

1.  Azure Portal で、[**+ リソースの作成**]をクリックし、検索ボックスに**storage**と入力します 

1.  利用可能なオプションから **ストレージ アカウント** を選択し、[**作成**] をクリックします。

1.  サブスクリプションとリソース グループを選択してください

1.  アカウントの一意の名前を入力します。

1.  場所には、リソースグループと同じものを選択します

1.  パフォーマンスは **Standard** である必要があります

1.  アカウントの種類は **StorageV2（汎用v2）** である必要があります

1.  レプリケーションは[**ローカル冗長ストレージ（LRS）**] を選択します

1.  アクセス層(規定)は **ホット** を選択します。

1.  [**Review + Create**] (確認および作成) をクリックします。

1.  [**作成**] をクリックします。

1.  新しいリソースに移動し、[**アクセス キー**] をクリックします。

1.  **接続文字列**をメモ帳にコピーします

1.  [**概要**]を選択し、[**コンテナー**] をクリックします

1.  **+ コンテナー**を選択します

1.  名前に「 **images**」と入力します

1.  パブリックアクセスレベルに [**プライベート(匿名アクセスはありません)**] を選択します。

1.  [**OK**] を選択します。

####    Cosmos DB

1.  [Azure Portal](https://portal.azure.com) を開く

1.  [**+リソースの作成**] をクリックし、検索ボックスに **cosmos** と入力します 

1.  利用可能なオプションから **Azure Cosmos DB** を選択します。

1.  [**作成**] をクリックします。

1.  サブスクリプションとリソース グループを選択してください

1.  一意のアカウント名を入力してください

1. APIは[**コア(SQL)**]を選択し、Apache Sparkは[**None**]を選択します。

1. 場所はリソースグループと同じ場所を選択します

1. geo冗長性は[**無効**]を選択します。

1. マルチリージョン書き込みは[**無効**]を選択します。

1.  [**Review + Create**] (確認と作成) をクリックします。

1.  [**作成**] をクリックします。

1.  新しいリソースに移動し、**設定** で [**キー**] を選択します

1.  **URI** と**主キー**をメモ帳にコピーします

### Bot Builder SDK

このコースでは、C# の Bot Builder テンプレートを使用してボットを作成します。

#### Bot Builder SDK のダウンロード

1.  [ここから C# 用の Bot Builder SDK v4 テンプレート](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4)のブラウザー ウィンドウを開きます 

1.  [**ダウンロード**] をクリックします。

1.  ダウンロード フォルダの場所に移動し、ダブルクリックしてインストールを開始します。

1.  Visual Studio のすべてのバージョンが選択されていることを確認し、[**インストール**] をクリックします。プロンプトが出されたら、「 **タスクの終了** 」をクリックします。

1.  [**閉じる**] をクリックします。これで、Visual Studio テンプレートにボット テンプレートが追加されました。

### Bot Emulator

最新の .NET SDK (v4) を使用してボットを開発します。ローカル テストを行うには、Bot Framework Emulator をダウンロードする必要があります。

### Bot Framework Emulator をダウンロードする

ボットをローカルでテストするために、v4 Bot Framework Emulator をダウンロードできます。ラボの残りの部分の手順では、v4 Emulator をダウンロードしていることを前提とします。

1.  エミュレーターをダウンロードするには、[このページ](https://github.com/Microsoft/BotFramework-Emulator/releases)にアクセスし、"4.5.1" というタグが付いている最新バージョンのエミュレーターをダウンロードします (Windows を使用している場合は、"*-windows-setup.exe" ファイルを選択します)。

> **注:** エミュレーターは、 
`"C:\Users\_your-username\AppData\Local\Programs\@bfemulatormain\Bot Framework Emulator.exe"` にインストールされますが、「 **bot framework** 」を検索することで、スタートメニューからアクセスできます。

## クレジット

このシリーズのラボは、[Cognitive Services チュートリアル](https://github.com/noodlefrenzy/CognitiveServicesTutorial)と [AI 学習短期集中講座](https://github.com/Azure/LearnAI-Bootcamp)を再利用しています。

##  リソース

ここで説明するアーキテクチャの理解を深め、より広範なチームが AI ソリューションの開発に関与できるようにするために、次のリソースを確認することをお勧めします。

- [Cognitive Services](https://www.microsoft.com/cognitive-services)  -  AI エンジニア
- [Cosmos DB](https://docs.microsoft.com/ja-jp/azure/cosmos-db/)  -  データ エンジニア
- [Azure Search](https://azure.microsoft.com/ja-jp/services/search/)  -  検索エンジニア
- [ボット開発者ポータル](http://dev.botframework.com) - AI エンジニア

## 次のステップ

-   [ラボ 02-01: Computer Vision を実装する](../Lab2-Implement_Computer_Vision/01-Introduction.md)
