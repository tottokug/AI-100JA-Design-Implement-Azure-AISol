# lab02.3-luis_and_search - LUIS と Azure Cognitive Search を使用したインテリジェント アプリケーションの開発

このハンズオン ラボでは、Microsoft Bot Framework、Azure Cognitive Search、およびいくつかの Cognitive Services を使用して、エンド ツー エンドでインテリジェントなボットを作成する方法について説明します。 

このワークショップでは、次のことを行います。
- インテリジェント サービスをアプリケーションに織り込む方法を理解する
- Azure Cognitive Search 機能を実装して、アプリケーション内で最適な検索エクスペリエンスを提供する方法を理解する
- フルテキスト検索、言語認識検索を有効にするためにデータを拡張するように Azure Cognitive Search サービスを構成する
- ボットが効果的に通信できるように LUIS モデルを構築、トレーニング、公開する
- LUIS と Azure Cognitive Search を活用する Microsoft Bot Framework を使用してインテリジェント ボットを構築する
- .NET アプリケーションでさまざまな Cognitive Services APIs (特に Computer Vision、Face、Emotion、LUIS) を呼び出す

ここでは LUIS と Azure Cognitive Search に重点を置いていますが、次のテクノロジも活用します。

- Computer Vision API
- Face API
- Emotion API
- Data Science Virtual Machine (DSVM)
- Windows 10 SDK (UWP)
- CosmosDB
- Azure Storage
- Visual Studio

## 前提条件

このワークショップは、Azure での AI 開発者を対象としています。これは半日のワークショップですので、事前に必要なことがあります。

まず、Visual Studio での経験が必要です。ワークショップで構築するすべてのものに Visual Studio を使用するので、アプリケーションを作成するために[その使用方法](https://docs.microsoft.com/ja-jp/visualstudio/ide/visual-studio-ide)に精通している必要があります。さらに、これはアプリケーションのコーディングまたは開発方法を教えるクラスではありません。C# でのコーディング方法を知っている ([ここ](https://mva.microsoft.com/ja-jp/training-courses/c-fundamentals-for-absolute-beginners-16169?l=Lvld4EQIC_2706218949)で学習できます) ものの、高度な検索および NLP (自然言語処理) ソリューションを実装する方法は知らないことを前提とします。 

次に、マイクロソフトの Bot Framework を使用してボットを開発した経験が必要です。設計方法やダイアログのしくみについてディスカッションする時間はあまりありません。Bot Framework に慣れていない場合は、ワークショップに参加する前に、[この Microsoft Virtual Academy コース](https://mva.microsoft.com/ja-jp/training-courses/creating-bots-in-the-microsoft-bot-framework-using-c-17590#!)を受講する必要があります。

3 番目に、portal での経験があり、Azure でリソースを作成する (および費用をかける) ことが可能である必要があります。このワークショップでは、Azure Pass は提供しません。

## 紹介

独自のイメージを取り込み、Cognitive Services を使用してイメージ内のオブジェクトや人物を検索して、それらの人物がどのように感じているかを把握し、そのデータをすべて NoSQL ストア (DocumentDB) に格納することが可能なエンド ツー エンドのシナリオを構築します。この NoSQL ストアを使用して Azure Cognitive Search インデックスを設定し、LUIS を使用して Bot Framework ボットを構築し、簡単なターゲットを絞ったクエリを実行できるようにします。

## アーキテクチャ

ローカル ドライブから写真を取り込み、いくつかの Cognitive Services を呼び出して、それらのイメージに関するデータを取得できる単純な C# アプリケーションを構築しました。

- [Computer Vision](https://www.microsoft.com/cognitive-services/ja-jp/computer-vision-api): タグと説明を取得するために使用します
- [Face](https://www.microsoft.com/cognitive-services/ja-jp/face-api): 各イメージから顔とその詳細を取得するために使用します
- [Emotion](https://www.microsoft.com/cognitive-services/ja-jp/emotion-api): イメージ内の各顔から感情スコアを引き出すために使用します

このデータを取得したら、必要な詳細を引き出して、すべてのデータを、当社の[](https://azure.microsoft.com/ja-jp/services/documentdb/) [NoSQL](https://en.wikipedia.org/wiki/NoSQL) [PaaS](https://azure.microsoft.com/ja-jp/overview/what-is-paas/) オファリングである [DocumentDB](https://azure.microsoft.com/ja-jp/services/documentdb/) に格納します。

DocumentDB で、[Azure Cognitive Search](https://azure.microsoft.com/ja-jp/services/search/) インデックスを作成します (Azure Cognitive Search は、フォールト トレラントなファセット検索のための PaaS オファリングで、管理のオーバーヘッドがない Elastic Search とお考えください)。データのクエリを実行し、そのクエリを実行する [Bot Framework](https://dev.botframework.com/) ボットを構築する方法について説明します。最後に、[LUIS](https://www.microsoft.com/cognitive-services/ja-jp/language-understanding-intelligent-service-luis) を使用してこのボットを拡張し、クエリから意図を自動的に抽出し、それらを使用して検索をインテリジェントに指示します。 

![アーキテクチャ図](./resources/assets/AI_Immersion_Arch.png)


## GitHub ## のナビゲーション

[リソース](./resources) フォルダーにはいくつかのディレクトリがあります:

- **assets**: ここには、ラボ マニュアルのすべてのイメージが含まれています。このフォルダーは無視できます。
- **code**: ここには、使用するいくつかのディレクトリがあります。
	- **ImageProcessing**: このソリューション (.sln) には、ワークショップの最初の部分にいくつかの異なるプロジェクトが含まれています。
		- **ImageProcessingLibrary**: これは、Vision に関連するさまざまな Cognitive Services にアクセスするためのヘルパー クラスと、結果をカプセル化するためのいくつかの "Insights" クラスが含まれるポータブル クラス ライブラリ (PCL) です。
		- **ImageStorageLibrary**: Cosmos DB では UWP が (まだ) サポートされていないため、これは Blob Storage および Cosmos DB にアクセスするためのポータブル ライブラリではありません。
		- **TestApp**: イメージを読み込み、それらに対してさまざまな Cognitive Services を呼び出し、その結果を調べることを可能にする UWP アプリケーション。イメージの実験や探索に役立ちます。
		- **TestCLI**: さまざまな Cognitive Services を呼び出し、イメージとデータを Azure にアップロードできるコンソール アプリケーション。イメージ が Blob Storage にアップロードされ、さまざまなメタデータ (タグ、キャプション、顔) が Cosmos DB にアップロードされます。

		_TestApp_ と _TestCLI_ の両方に Cognitive Services と Azure へのアクセスに必要なさまざまなキーとエンドポイントを含めて、`settings.json`ファイルが含まれています。空白で開始するため、リソースをプロビジョニングしたら、サービスキーを取得し、ストレージ アカウントと Cosmos DB インスタンスを設定します。
		
	- **LUIS**: ここでは、PictureBot の LUIS モデルを見つけることができます。独自の LUIS を作成しますが、進行が遅れた場合や、別の LUIS モデルをテストする場合は、.json ファイルを使用してこの LUIS アプリをインポートできます。
	- **Models**: これらのクラスは、PictureBot に検索を追加する際に使用します。
	- **PictureBot**: ここでは、LUIS と検索インデックスを Bot Framework に統合する、ワークショップの後半のセクション用の PictureBot.sln があります。


### ラボ: Azure アカウントのセットアップ

Azure の無料試用版は、[https://azure.microsoft.com/ja-jp/free/](https://azure.microsoft.com/ja-jp/free/) でアクティブ化できます。  

このラボを実行するために Azure Pass が付与されている場合は、[http://www.microsoftazurepass.com/](http://www.microsoftazurepass.com/) に 移動してアクティブ化できます。  アクティベーション プロセスについて文書化されている [https://www.microsoftazurepass.com/howto](https://www.microsoftazurepass.com/howto) の手順に従ってください。  Microsoft アカウントでは、Azure で **1 つの無料試用版**とそれに関連付けられた 1 つの Azure Pass を使用できるため、Microsoft アカウントで Azure Pass を既に有効にしている場合は、その無料試用版を使用するか、別の Microsoft アカウントを使用する必要があります。

### ラボ: Data Science Virtual Machine のセットアップ

Azure アカウントを作成すると、[Azure portal](https://portal.azure.com) にアクセスできます。Azure portal から、このラボのリソース グループを作成します。Data Science Virtual Machine に関する詳細情報は[オンライン](https://docs.microsoft.com/ja-jp/azure/machine-learning/data-science-virtual-machine/overview)で見つけることができますが、ここではこのワークショップに必要なものについてのみ説明します。リソース グループで、DS3_V2 のサイズで、Windows (2016) 用 Data Science Virtual Machine をデプロイして接続します (その他はすべて既定値で問題ありません)。 

接続が完了したら、ワークショップの用 DSVM をセットアップするために必要な操作がいくつかあります。

1. Firefox でこのリポジトリに移動し、zip ファイルとしてダウンロードします。すべてのファイルを抽出し、このラボ用のフォルダーをデスクトップに移動します。
2. 「Resources」 > 「code」 > 「ImageProcessing」 の下にある "ImageProcessing.sln" を開きます。Visual Studio が最初に開かれるときには、しばらく時間がかかる場合があり、ログインする必要があります。
3. 開かれると、Windows 10 アプリ開発 (UWP) 用の SDK をインストールするように求められます。プロンプトに従ってインストールします。プロンプトが表示されない場合は、「TestApp」を右クリックして「プロジェクトの再読み込み」を選択すると、プロンプトが表示されます。
4. インストール中に実行できるタスクがいくつかあります。 
	- Cortana の検索バーで「For developers settings」 (開発者向け設定) と入力し、設定を「開発者モード」に変更します。
	- Cortana の検索バーで「gpedit.msc」と入力し、Enter キーを押します。次のポリシーを有効にします。「コンピューターの構成」 > 「Windowsの設定」 > 「セキュリティの設定」> 「ローカル ポリシー」 > 「セキュリティ オプション」 > 「ユーザー アカウント制御」: 組み込みの管理者アカウントの管理者承認モード
	- "キーの収集" ラボを開始します。 
5. インストールが完了したら、開発者の設定とユーザー アカウント制御ポリシーを変更し、DSVM を再起動します。 
> **注** ワークショップの後、DSVM をオフにして、課金されないようにしてください。


### ラボ: キーの収集

このラボを通して、Cognitive Services キーとストレージ キーを収集します。将来のラボで簡単にアクセスできるように、それらのすべてのキーをテキスト ファイルに保存する必要があります。

>_Cognitive Services キー_
>- Computer Vision API:
>- Face API:
>- Emotion API:
>- LUIS API:

>_ストレージ キー_
>- Azure Blob Storage 接続文字列:
>- Cosmos DB URI:
>- Cosmos DB キー:

**Cognitive Services API キーの取得**

Portal で、使用する Cognitive Services のキーを作成します。主に [Computer Vision](https://www.microsoft.com/cognitive-services/ja-jp/computer-vision-api) Cognitive Services でさまざまな API を使用するので、最初に API キーを作成しましょう。

Portal で**「新規」**をクリックし、検索ボックスに**「cognitive」**と入力して、**「Cognitive Services」**を選択します。

![Cognitive Services キーの作成](./resources/assets/new-cognitive-services.PNG)

作成する API エンドポイントの詳細を入力し、目的の API と、エンドポイントを配置する場所、目的の料金プランを選択します。チュートリアルに必要なスループットを確保するために、S1 を使用して、新しい_リソース グループ_を作成します。以下の同じリソース グループを Blob Storage と Cosmos DB に使用します。見つけやすいように、_ダッシュボードにピン留め_します。Computer Vision API では、将来の Cognitive Services Vision サービスを改善するためにマイクロソフト社内に (安全な方法で) イメージを保存するため、アカウントの作成を_有効にする_必要があります。これは、サブスクリプションの管理者のみが有効にする権限を持つため、企業の環境におけるユーザーにとっては障害となる可能性がありますが、Azure Pass ユーザーにとっては問題ではありません。

![Cognitive Services の詳細の選択](./resources/assets/cognitive-account-creation.PNG) 

新しい API サブスクリプションを作成したら、ブレードの適切なセクションからキーを取得し、キーの _TestApp_ と _TestCLI_ の `settings.json` ファイルに追加できます。

![Cognitive API キー](./resources/assets/cognitive-keys.PNG)

また、Computer Vision ファミリー内の他の API も使用するので、この機会に _Emotion_ API と _Face_ API の API キーも作成します。これらは上記と同じ方法で作成され、作成したのと同じリソース グループを再利用する必要があります。_ダッシュボードにピン留め_して、これらのキーを`settings.json`ファイルに追加します。

このチュートリアルの後の方で [LUIS](https://www.microsoft.com/cognitive-services/ja-jp/language-understanding-intelligent-service-luis) を使用するので、この機会に LUIS サブスクリプションもここで作成します。上記とまったく同じ方法で作成しますが、「API」ドロップダウンから「Language Understanding Intelligent Service」を選択して、上記で作成したのと同じリソース グループを再利用します。もう一度、_ダッシュボードにピン留め_しますので、チュートリアルのその段階に到達すると、簡単にアクセスできます。  

**ストレージの設定**

このプロジェクトでは、Azure の 2 種類のストア (1 つには生のイメージを格納し、もう 1 つには Cognitive Services の呼び出しの結果を格納する) を使用します。Azure BLOB Storage は、ファイル システムに似た形式で大量のデータを格納するために作成されており、イメージなどのデータを格納するのに最適です。Azure Cosmos DB はマイクロソフトの回復性の高い NoSQL PaaS ソリューションであり、イメージ メタデータの結果と同様に、緩やかに構造化されたデータを格納するのに非常に便利です。他にも選択肢 (Azure Table Storage、SQL Server) がありますが、Cosmos DB ではスキーマを自由に進化させ (新しいサービスへのデータの追加など)、簡単にクエリを実行し、Azure Cognitive Search にすばやく統合するための柔軟性がもたらされます。

_Azure Blob ストレージ_

「はじめに」の詳細な手順は[オンラインで見つける](https://docs.microsoft.com/ja-jp/azure/storage/storage-dotnet-how-to-use-blobs)ことができますが、ここではこのラボに必要なものだけを詳しく見てみましょう。

Azure portal で、**「新規」->「ストレージ」->「ストレージ アカウント」**をクリックします。

![新しい Azure Storage](./resources/assets/create-blob-storage.PNG)

クリックすると、上記のフィールドに入力するフィールドが表示されます。ストレージ アカウント名 (小文字と数字) を選択し、「アカウントの種類」__を "Blob Storage" に設定し、____「レプリケーション」を__ "ローカル冗長ストレージ (LRS)" に設定して (この目的はコストを節約することのみ)、上記と同じリソース グループを使用して、「国や地域」を "米国西部"____ に設定します  (各リージョンで利用可能な Azure サービスの一覧は、https://azure.microsoft.com/ja-jp/regions/services/ にあります)。見つけやすいように、_ダッシュボードにピン留め_します。

Azure Storage アカウントを持っているので、__接続文字列を取得し、_TestCLI_ と _TestApp_ の`settings.json`に追加してみましょう。

![Azure Blob キー](./resources/assets/blob-storage-keys.PNG)

_Cosmos DB_

「はじめに」の詳細な手順は[オンラインで見つける](https://docs.microsoft.com/ja-jp/azure/documentdb/documentdb-get-started)ことができますが、ここではこのプロジェクトに必要なものだけを詳しく見てみましょう。

Azure portal で、**「新規」->「データベース」->「Azure Cosmos DB」**をクリックします。

![新しい Cosmos DB](./resources/assets/create-cosmosdb-portal.png)

これをクリックしたら、適切だと思う内容を入力する必要があります。 

![Cosmos DB の作成フォーム](./resources/assets/create-cosmosdb-formfill.png)

ここでは、小文字、数字、またはダッシュにする必要があるという制約に従って、必要な ID を選択します。Mongo ではなく、ドキュメント DB の SDK を使用するので、NoSQL API としてドキュメント DB を選択します。前の手順と同じリソース グループ、同じ場所を使用して、「ダッシュボードにピン留め」を選択して、ダッシュボードを__追跡して簡単に取得したら、「作成」をクリックします。

作成が完了したら、新しいデータベースのパネルを開き、「キー」__サブパネルを選択します。

![Cosmos DB の「キー」サブパネル](./resources/assets/docdb-keys.png)

**TestCLI** の`settings.json`ファイルには **URI** と_プライマリ キー_が必要になりますので、それらをコピーしてください。これでイメージとデータをクラウドに保存する準備が整いました。


## Cognitive Services

このワークショップでは、すべての Cognitive Services API に焦点を当てるわけではありません。Azure Cognitive Search と LUIS だけに焦点を当てます。ただし、Cognitive Services の演習を増やしたい場合は、「-TODO」を確認することをお勧めします。ラボ 1 に、複数の Cognitive Services を使用してインテリジェント キオスクを構築するリファレンス ラボのリンクを追加します。 

このワークショップでは、Computer Vision API を使用して (タグと説明によって) イメージを理解し、Face API を使用してアップロードしたイメージ内のすべての顔を追跡し、Emotion API を使用して人々が持っている感情のスコアを取得します。単純に API を呼び出して情報を取得します。必要なトレーニングやテストはありません。 

結果として得られる情報には、タグ、説明、性別/年齢/感情 (人物の場合) があります。次のラボを完了したら、作成された ImageInsights.json ファイルを確認して、この情報の構造を把握します。

後で、ボットが言語をよりよく理解できるように、LUIS モデルの開発にも少し時間を費やします。このサービスでは、トレーニングして公開する前に、意図、エンティティ、発話などをモデルに追加します。その後でようやく、API 呼び出しからアクセスできるようになります。

### ラボ: Cognitive Services とイメージ処理ライブラリの探索

`ImageProcessing.sln`ソリューションを開くと、イメージを読み込み、それらに対してさまざまな Cognitive Services を呼び出し、その結果を調べることを可能にする UWP アプリケーションが表示されます。これはイメージの実験や探索に役立ちます。このアプリは、イメージを分析するために TestCLI プロジェクトでも使用される`ImageProcessingLibrary`プロジェクトを基にして構築されています。 

ソリューションを "ビルド" する必要があります (`ImageProcessing.sln`を右クリックして、「ビルド」を選択する)。また、TestApp プロジェクトを再読み込みする必要がある場合もあります。これは、このプロジェクトを右クリックし、「プロジェクトの再読み込み」を選択することによって実行できます。 

アプリを実行する前に、`TestApp`プロジェクトの`settings.json`ファイルに Cognitive Services API キーを入力してください。これを行ったら、アプリを実行し、(`フォルダーの選択`ボタンを使用して) イメージが保存されている任意のフォルダーをポイントすると (最初に`sample_images`を解凍する必要がある)、その結果として、処理されたすべてのイメージが表示され、さらに、イメージ コレクションのフィルターとしても機能する一意の顔、感情、タグなどに分割されて表示されるはずです。

![UWP テスト アプリ](./resources/assets/UWPTestApp.JPG)

アプリで特定のディレクトリが処理されると、同じフォルダーに`ImageInsights.json`ファイルに結果がキャッシュされ、さまざまな API を呼び出す必要なしに、そのフォルダーの結果を再び見ることができます。 

## Cosmos DB の探索

Cosmos DB はこのワークショップの焦点ではありませんが、ご興味がある方のために、使用するコードのハイライトの一部をご紹介します。
- `ImageStorageLibrary`の`DocumentDBHelper.cs`クラスに移動します。使用する実装の多くについては、[スタート アップガイド](https://docs.microsoft.com/ja-jp/azure/documentdb/documentdb-get-started)を参照してください。
- `TestCLI`'`Util.cs`に移動し、`ImageMetadata`クラスを確認します。ここで、Cognitive Services から取得した`ImageInsights`を適切なメタデータに変換し、Cosmos DB に格納します。
- 最後に、`Program.cs`と`ProcessDirectoryAsync`を見てみましょう。まず、イメージとメタデータが既にアップロードされていることを確認します。`DocumentDBHelper`を使用して ID でドキュメントを検索します。ドキュメントが存在しない場合は`null`が返されます。次に、`forceUpdate`を設定しているか、またはイメージがまだ処理されていない場合は、`ImageProcessingLibrary`から`ImageProcessor`を使用して Cognitive Services を呼び出し、現在の`ImageMetadata`に追加する`ImageInsights`を取得します。 
- すべてが完了すると、イメージを格納できます。まず、`BlobStorageHelper`インスタンスを使用して実際のイメージを BLOB Storage に格納し、次に`DocumentDBHelper`インスタンスを使用して`ImageMetadata`を Cosmos DB に格納します。(これまでに確認したように) ドキュメントが既に存在している場合、既存のドキュメントを更新する必要があります。存在していない場合は、新しいドキュメントを作成する必要があります。

### ラボ: TestCLI を使用したイメージの読み込み

イベント ループ、フォーム、その他の UX 関連の中断を心配することなくコードの処理に集中できるように、メインの処理コードとストレージ コードをコマンド ライン/コンソール アプリケーションとして実装します。後で独自の UX を自由に追加してください。

_TestCLI_ の`settings.json`で Cognitive Services API キー、Azure Blob Storage 接続文字列、および Cosmos DB エンドポイントの URI およびキーを設定したら、_TestCLI_ を実行できます。

_TestCLI_ を実行し、コマンド プロンプトを開いて、ImageProcessing\TestCLI フォルダーに移動します (ヒント: "cd" コマンドを使用してディレクトリを変更する)。次に、`.\bin\Debug\TestCLI.exe`と入力します。以下の結果が得られるはずです。

    > .\bin\Debug\TestCLI.exe

    Usage:  [options]

    Options:
    -force            ファイルが既に追加されている場合でも、更新を強制するために使用します。
    -Settings         設定ファイル (オプション。設定されていない場合、埋め込まれたリソースの settings.json を使用します)
    -Process          処理するディレクトリ
    -query            実行するクエリ
    -?  | -h | --Help  ヘルプ情報を表示します

既定では、設定は`settings.json`から読み込まれます (`.exe`にビルドされます) が、`-settings`フラグを使用して独自の設定を指定することもできます。イメージ (および Cognitive Services のメタデータ) をクラウド ストレージに読み込むには、次のようにイメージ ディレクトリに対して`-process`を実行するように _TestCLI_ に指示するだけです。

    > .\bin\Debug\TestCLI.exe -process c:\my\image\directory

処理が完了したら、次のように _TestCLI_ を使用して Cosmos DB に対して直接クエリを実行できます。

    > .\bin\Debug\TestCLI.exe -query "select * from images"

## Azure Cognitive Search

[Azure Cognitive Search](https://docs.microsoft.com/ja-jp/azure/search/search-what-is-azure-search) は、サービスとしての検索ソリューションです。開発者がインフラストラクチャを管理する必要や、検索のエキスパートになる必要がなく、優れた検索エクスペリエンスをアプリケーションに組み込むことが可能になります。

開発者は、アプリでより良い、より迅速な結果を達成するために、Azure で PaaS サービスを探します。検索は、アプリケーションの多くのカテゴリのの鍵を握っています。Web 検索エンジンでは検索のハードルが高く設定されているため、スペルを間違えた場合や、余分な単語を含めた場合でも、ユーザーは瞬時の結果、入力時のオートコンプリート、結果に表示される項目の強調表示、ランキング、検索している内容を理解する能力を期待しています。

検索は困難で、コアな専門分野はまれです。インフラストラクチャの観点からは、高可用性、耐久性、拡張性、および運用が必要です。機能の観点からは、ランク付け、言語サポート、および地理空間機能が必要です。

![検索要件の例](./resources/assets/AzureSearch-Example.png) 

上記の例では、ユーザーが検索エクスペリエンスで期待しているコンポーネントの一部を示しています。[Azure Cognitive Search](https://docs.microsoft.com/ja-jp/azure/search/search-what-is-azure-search) では、これらのユーザー エクスペリエンス機能を実現し、[監視とレポート作成](https://docs.microsoft.com/ja-jp/azure/search/search-traffic-analytics)、[簡単なスコアリング](https://docs.microsoft.com/ja-jp/rest/api/searchservice/add-scoring-profiles-to-a-search-index)。および[プロトタイプ作成](https://docs.microsoft.com/ja-jp/azure/search/search-import-data-portal)と[検査](https://docs.microsoft.com/ja-jp/azure/search/search-explorer)のためのツールを提供できます。

一般的なワークフロー:
1. サービスをプロビジョニングする
	- [portal](https://docs.microsoft.com/ja-jp/azure/search/search-create-service-portal) または [PowerShell](https://docs.microsoft.com/ja-jp/azure/search/search-manage-powershell) から Azure Cognitive Search サービスを作成またはプロビジョニングできます。
2. インデックスを作成する
	- [インデックス](https://docs.microsoft.com/ja-jp/azure/search/search-what-is-an-index)はデータのコンテナーであり、「テーブル」と考えてください。スキーマ、[CORS オプション](https://docs.microsoft.com/ja-jp/aspnet/core/security/cors)、検索オプションがあります。[Portal](https://docs.microsoft.com/ja-jp/azure/search/search-create-index-portal) で、または[アプリの初期化](https://docs.microsoft.com/ja-jp/azure/search/search-create-index-dotnet)中に作成できます。 
3. インデックス データ
	- [データにインデックスに設定](https://docs.microsoft.com/ja-jp/azure/search/search-what-is-data-import)する方法は 2 つあります。最初のオプションは、Azure Cognitive Search [REST API](https://docs.microsoft.com/ja-jp/azure/search/search-import-data-rest-api) または [.NET SDK](https://docs.microsoft.com/ja-jp/azure/search/search-import-data-dotnet) を使用して、データをインデックスに手動でプッシュすることです。2 番目のオプションは、[サポートされているデータ ソース](https://docs.microsoft.com/ja-jp/azure/search/search-indexer-overview)をインデックスにポイントし、スケジュールに基づいて Azure Cognitive Search でデータを自動的に取得できるようにする方法です。
4. インデックスを検索する
	- 検索要求を Azure Cognitive Search に送信する場合は、簡単な検索オプションを使用して、[結果をフィルター処理](https://docs.microsoft.com/ja-jp/azure/search/search-filters)、[並べ替え](https://docs.microsoft.com/ja-jp/rest/api/searchservice/add-scoring-profiles-to-a-search-index)、[投影](https://docs.microsoft.com/ja-jp/azure/search/search-faceted-navigation)、および[ページ オーバー](https://docs.microsoft.com/ja-jp/azure/search/search-pagination-page-layout)できます。スペルミス、ふりがな、および正規表現に対応する機能があり、検索と[提案](https://docs.microsoft.com/ja-jp/rest/api/searchservice/suggesters)を操作するためのオプションもあります。これらのクエリ パラメーターを使用すると、[フルテキスト検索エクスペリエンス](https://docs.microsoft.com/ja-jp/azure/search/search-query-overview)を詳細に制御できます。


### ラボ: Azure Cognitive Search サービスを作成する

Azure portal で、**「新規」->「Web + モバイル」->「Azure Cognitive Search」**をクリックします。

これをクリックしたら、適切だと思う内容を入力する必要があります。このラボでは、"Free" レベルで十分です。

![新しい Azure Cognitive Search サービスを作成する](./resources/assets/AzureSearch-CreateSearchService.png)

作成が完了したら、新しい Search Service のパネルを開きます。

### ラボ: Azure Cognitive Search インデックスを作成する

インデックスはデータのコンテナーであり、SQL Server テーブルと同様の概念です。  テーブルに行があるように、インデックスにはドキュメントがあります。  テーブルにフィールドがあるように、インデックスにもフィールドがあります。  これらのフィールドには、フルテキスト検索が可能かどうかや、フィルター処理が可能かどうかを伝えるプロパティを含めることができます。  プログラムで[コンテンツをプッシュ](https://docs.microsoft.com/ja-jp/rest/api/searchservice/addupdate-or-delete-documents)するか、[Azure Cognitive Search インデクサー](https://docs.microsoft.com/ja-jp/azure/search/search-indexer-overview) (データの一般的なデータ ストアをクロールできる) を使用して、コンテンツを Azure Cognitive Search に入力できます。

このラボでは、[Cosmos DB 用の Azure Cognitive Search インデクサー](https://docs.microsoft.com/ja-jp/azure/search/search-howto-index-documentdb)を使用して、Cosmos DB コンテナー内のデータをクロールします。 

![インポート ウィザード](./resources/assets/AzureSearch-ImportData.png) 

作成した Azure Cognitive Search ブレード内で、**「データのインポート」->「データ ソース」->「ドキュメント DB」**をクリックします。

![DocDB 用のインポート ウィザード](./resources/assets/AzureSearch-DataSource.png) 

これをクリックして、Cosmos DB データソースの名前を選択し、データが存在している Cosmos DB アカウントと、対応するコンテナーとコレクションを選択します。  

[**OK**] をクリックします。

この時点で、Azure Cognitive Search は Cosmos DB コンテナーに接続されています。いくつかのドキュメントを分析して Azure Cognitive Search インデックスの既定のスキーマを識別します。  これが完了したら、アプリケーションで必要とされるフィールドのプロパティを設定できます。

インデックス名を **images** に更新する

キーを **id** に更新する (各ドキュメントを一意に識別する)

すべてのフィールドを**「取得可能」**に設定します (クライアントが検索時にこれらのフィールドを取得できるようにする)

フィールド「タグ」、**「NumFaces」、および「顔」**を**「フィルター設定可能」**に設定します (クライアントがこれらの値に基づいて結果をフィルター処理できるようにする)

フィールド**「NumFaces」**を**「並べ替え可能」**に設定します (クライアントがイメージ内の顔の数に基づいて結果を並べ替えることができるようにする)

フィールド**「タグ」、「NumFaces」、および「顔」**を**「ファセット可能」**に設定します (クライアントが結果を数でグループ化できるようにする (たとえば、この検索結果の場合、"beach" というタグが付けられている 5 つの写真がある)

フィールド**「キャプション」、「タグ」、「顔」**を**「検索可能」**に設定します (クライアントがこれらのフィールドのテキストに対してフルテキスト検索を行えるようにする)

![Azure Cognitive Search インデックスを構成する](./resources/assets/AzureSearch-ConfigureIndex.png) 

この時点で、Azure Cognitive Search アナライザーを構成します。  大まかには、アナライザーは、ユーザーが入力した用語を受け取り、インデックス内で最も一致する用語を見つけるものです。  Azure Cognitive Search には、56 の言語を深く理解している Bingや Office などのテクノロジで使用されるアナライザーが含まれています。  

**「アナライザー」**タブをクリックし、フィールド**「キャプション」、「タグ」、「顔」**を設定して、**英語 - Microsoft** アナライザーを使用します

![言語アナライザー](./resources/assets/AzureSearch-Analyzer.png) 

最後のインデックス構成手順として、事前に入力するために使用されるフィールドを設定し、ユーザーがこれらのフィールドで最も一致しているものを探す単語の一部を入力できるようにします。

**「候補者」** タブをクリックし、「提案者名」に**「sg」**を入力し、用語の候補を検索するフィールドとして**「タグ」と「顔」**を選択します。

![検索候補](./resources/assets/AzureSearch-Suggester.png) 

**「OK」**をクリックして、インデクサーの構成を完了します。  インデクサーが変更をチェックする頻度をスケジュールで設定できますが、このラボでは 1 回だけ実行します。  

「**詳細オプション**」をクリックし、「**Base-64 エンコード キー**」を選択して、「ID」フィールドで、「Azure Cognitive Search key」フィールドでサポートされている文字のみが使用されることを確認します。

**「OK」をクリック**し、Cosmos DB データベースからのデータのインポートを開始するインデクサー ジョブを開始します。

![インデクサーを構成する](./resources/assets/AzureSearch-ConfigureIndexer.png) 

***検索インデックスに対してクエリを実行する***

インデックス作成が開始されたことを示すメッセージがポップアップ表示されるはずです。  インデックスのステータスを確認する場合は、Azure Cognitive Search のメイン ブレードで「インデックス」オプションを選択できます。

この時点で、インデックスを検索できます。  

**「検索エクスプローラー」**をクリックし、結果のブレードでインデックスがまだ選択されていない場合は、「インデックス」を選択します。

**「検索」**をクリックして、すべてのドキュメントを検索します。

![検索エクスプローラー](./resources/assets/AzureSearch-SearchExplorer.png)

**予定より早く終了した場合この追加のクレジット ラボをお試しください:**

[Postman](https://www.getpostman.com/) は、Azure Cognitive Search REST API 呼び出しを簡単に実行できるようにする優れたツールであり、優れたデバッグ ツールです。  Azure Cognitive Search エクスプローラーから任意のクエリを実行し、Postman 内で実行する Azure Cognitive Search API キーを使用できます。

[Postman](https://www.getpostman.com/) ツールをダウンロードしてインストールします。 

インストールしたら、Azure Cognitive Search エクスプローラーからクエリを実行して、Postman に貼り付け、要求の種類として「GET」を選択します。  

「ヘッダー」をクリックし、次のパラメーターを入力します。

+ Content Type: application/json
+ api-key: 「キー」セクションの下にある Azure Cognitive Search ポータルから API キーを入力します

「送信」を選択すると、JSON 形式で書式設定されたデータが表示されるはずです。

[これらの例](https://docs.microsoft.com/ja-jp/rest/api/searchservice/search-documents#a-namebkmkexamplesa-examples)を使用して、他の検索を実行してみてください。


## LUIS

まず、[Language Understanding Intelligent Service (LUIS) について学習](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/Home)しましょう。

LUIS の概要を把握したところで、LUIS アプリを計画します。検索に基づいてイメージを返すボットを作成し、共有または並べ替えを行います。ボットが実行できるさまざまなアクションをトリガーする意図を作成し、そのアクションの実行に必要なパラメーターをモデル化するエンティティを作成する必要があります。たとえば、PictureBot の意図が "SearchPics" であり、写真を検索するために検索サービスがトリガーされます。これには、検索対象を知るために "facet" エンティティが必要です。アプリを計画するためのその他の例については、[ここ](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/plan-your-app)を参照してください。

アプリについて考え抜いたら、[それを構築してトレーニングする](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/luis-get-started-create-app)準備が整いました。LUIS アプリケーションの作成時に一般的に実行する手順は次のとおりです。
  1. [意図を追加する](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/add-intents) 
  2. [発話を追加する](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/add-example-utterances)
  3. [エンティティを追加する](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/add-entities)
  4. [機能を使用してパフォーマンスを向上させる](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/add-features)
  5. [トレーニングとテストを行う](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/train-test)
  6. [アクティブ ラーニングを使用する](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/label-suggested-utterances)
  7. [公開する](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/publishapp)



### ラボ: LUIS を使用してアプリケーションにインテリジェンスを追加する

次のラボでは、PictureBot を作成します。まず、LUIS を使用して、自然言語機能を追加する方法を見てみましょう。LUIS を使用すると、自然言語の発話を意図にマッピングできます。  ここでは、写真の検索、写真の共有、写真のプリントの順序付けなど、いくつかの意図があります。  これらの事柄を尋ねる方法として、発話のいくつかの例をご紹介します。LUIS では、学習した内容に基づいて、それぞれの意図に追加の新しい発話がマッピングされます。  

[Https://www.luis.ai](https://www.luis.ai) に移動して、Microsoft アカウントを使用してサインインします  (これは、このラボの冒頭で Cognitive Services キーを作成したのと同じアカウントにする必要があります)。  [https://www.luis.ai/applications](https://www.luis.ai/applications)の LUIS アプリケーションの一覧にリダイレクトされるはずです。  ボットをサポートする新しい LUIS アプリを作成します。  

> 楽しい余談: [現在のページ](https://www.luis.ai/applications)の「New App」 (新しいアプリ) ボタンの横に「Import App」 (アプリのインポート) もあります。  LUIS アプリケーションを作成した後、アプリ全体を JSON としてエクスポートし、ソース管理にチェックインできます。  これは推奨されるベスト プラクティスであり、コードのバージョン管理に合わせて LUIS モデルのバージョンを管理できます。  エクスポートされた LUIS アプリは、「Import App」 (アプリのインポート) ボタンを使用して再インポートできます。  ラボの進行が遅れてしまい、ショートカットする場合は、「Import App」 (アプリのインポート) ボタンをクリックして [LUIS モデル](./resources/code/LUIS/PictureBotLuisModel.json)をインポートできます。  

[Https://www.luis.ai/applications](https://www.luis.ai/applications) から「New App」 (新しいアプリ) ボタンをクリックします。  名前を付けて ("PictureBotLuisModel" を選択)、「Culture」 (文化) を「English」 (英語) に設定します。  必要に応じて、説明を入力できます。  ドロップダウンをクリックして使用するエンドポイント キーを選択し、ワークショップの開始時に Azure portal で作成した LUIS キーが存在している場合は、そのキーを選択します (このオプションは、アプリを公開するまで表示されない場合があるため、表示されない場合は心配しないでください)。  次に、「Create」 (作成) をクリックします。  

![LUIS の新しいアプリ](./resources/assets/LuisNewApp.jpg) 

新しいアプリのダッシュボードが表示されます。  アプリ ID が表示されます。**LUIS アプリ ID** については、後で説明します。  次に、「Create an intent」 (意図の作成) をクリックします。  

![LUIS ダッシュボード](./resources/assets/LuisDashboard.jpg) 

ボットが次のことを実行できるようにしましょう。
+ 写真を検索する
+ ソーシャル メディアで写真を共有する
+ 写真のプリントを注文する
+ ユーザーに挨拶する (ただし、これは後で説明するように、他の方法でも可能)

これらのそれぞれを要求するユーザーの意図を作成してみましょう。  「Add intent」 (意図の追加) ボタンをクリックします。  

最初の意図に "Greeting" という名前を付け、「Save」 (保存) をクリックします。  次に、ボットに挨拶するときにユーザーが話す可能性のある言葉の例をいくつか示し、それぞれの後に Enter キーを押します。  発話を入力したら、「Save」 (保存) をクリックします。  

![LUIS Greeting の意図](./resources/assets/LuisGreetingIntent.jpg) 

エンティティを作成する方法を見てみましょう。  ユーザーが写真の検索を要求するとき、何を探すかを指定できます。  エンティティ内でキャプチャしてみましょう。  

左側の列の「Entities」 (エンティティ) をクリックし、「Add custom entity」 (新しいエンティティの追加) をクリックします。  エンティティ名 "facet" とエンティティの種類「シンプル」を指定します。  次に、「Save」 (保存) をクリックします。  

![ファセット エンティティを追加する](./resources/assets/AddFacetEntity.jpg) 

ここで、左側のサイドバーの「Intents」 (意図) をクリックし、黄色の「Add Intent」 (意図の追加) ボタンをクリックします。  "SearchPics" という意図名を付け、「Save」 (保存) をクリックします。  

ここで、いくつかのサンプル発話 (ボットと話すときにユーザーが話す可能性のある単語/フレーズ/文) を追加してみましょう。  写真を検索する方法は数多くあります。  以下の発話の一部を自由に使用して、ボットに写真を検索するように依頼する方法について独自の文章を追加してください。 

+ Find outdoor pics (屋外の写真を見つける)
+ Are there pictures of a train? (電車の写真はあるか)
+ Find pictures of food. (食べ物の写真を見つける)
+ Search for photos of a 6-month-old boy (6 か月の男の子の赤ちゃんの写真を検索する)
+ Please give me pics of 20-year-old women (20 歳の女性の写真を見せて)
+ Show me beach pics (ビーチの写真を見せて)
+ I want to find dog photos (犬の写真を見つけたい)
+ Search for pictures of men indoors (屋内の男性の写真を検索する)
+ Show me pictures of girls looking happy (幸せそうな女の子の写真を見せて)
+ I want to see pics of sad boys (悲しんでいる少女達の写真を見たい)
+ Show me happy baby pics (幸せな赤ちゃんの写真を見せて)

ここで、検索トピックを "facet" エンティティとして選択する方法を LUIS に教える必要があります。  キーワードにカーソルを合わせてクリックするか、単語のグループをドラッグして選択選択し、次に "facet" エンティティを選択します。  

![ラベル付けエンティティ](./resources/assets/LabellingEntity.jpg) 

次の発話のリスト...

![ファセット エンティティを追加する](./resources/assets/SearchPicsIntentBefore.jpg) 

...ファセットにラベルが付けられると、このようになる可能性があります。  

![ファセット エンティティを追加する](./resources/assets/SearchPicsIntentAfter.jpg) 

完了したら、「Save」 (保存) をクリックすることを忘れないでください!  

最後に、左側のサイドバーの「Intents」 (意図) をクリックして、さらに 2 つの意図を追加します。
+ 1つの意図に **"SharePic"** という名前を付けます。  これは "この写真を共有する"、"それをツイートできますか"、"Twitter に投稿" などの発話によって識別されます。  
+ **"OrderPic"** という名前の別の意図を作成します。  これは、"この写真をプリントする"、"プリントを注文したい"、"それの 8 x 10 を手に入れることができますか"、"ウォレットを注文して" などの発話でコミュニケーションできます。  
発話を選択するときには、質問、命令、"...したい..." 形式の組み合わせを使用すると便利です。  

また、"None" (なし) という意図が 1 つあることにも注意してください。  いずれの意図にもマッピングされないランダムな発話は、"None" (なし) にマッピングされる場合があります。  「Do you like peanut butter and jelly?」 (ピーナッツバターとジャムは好きですか?) などが表示されます。

これで、モデルをトレーニングする準備が整いました。  左側のサイドバーの「Train & Test」 (トレーニングとテスト) をクリックします。  次に、「train」 (トレーニング) ボタンをクリックします。  これにより、指定したトレーニング データを使用して「utterance」 (発話) --> 「intent mapping」 (意図のマッピング) を行うモデルが構築されます。  

次に、左側のサイドバーにある「Publish App」 (アプリの公開) をクリックします。  まだ設定していない場合は、以前に設定したエンドポイント キーを選択するか、リンク先に従って Azure アカウントで新しいキーを作成します。  エンドポイント スロットは "Production" (運用環境) のままにしておくことができます。  次に、「Publish」 (公開) をクリックします。  

![LUIS アプリを公開する](./resources/assets/PublishLuisApp.jpg) 

公開すると、LUIS モデルを呼び出すエンドポイントが作成されます。  URL が表示されます。  

左側のサイドバーの「Train & Test」 (トレーニングとテスト) をクリックします。  「Enable published model」 (公開されたモデルを有効にする) チェック ボックスをオンにすると、モデルを直接呼び出すのではなく、公開されたエンドポイントを介して呼び出します。  いくつかの発話を入力して、意図が返されることを確認してください。  

![LUIS をテストする](./resources/assets/TestLuis.jpg) 

**予定より早く終了した場合次の追加クレジット タスクをお試しください。**


「SearchPics」という意図によって活用できる追加のエンティティを作成します。たとえば、アプリで年齢が判断されることがわかっています。年齢に対して事前に構築されたエンティティを作成してみましょう。 

エンティティの種類が "List" (リスト) のカスタム エンティティを使用して、感情と性別を取得してみましょう。以下の感情の例を参照してください。 

![リスト付きのカスタム感情エンティティ](./resources/assets/CustomEmotionEntityWithList.jpg) 

> **注**: エンティティや機能を追加するときには、`Intents>Utterances` に移動し、追加したエンティティに対して発話を確認することや、さらに発話を追加することを忘れないでください。また、モデルを再トレーニングして公開する必要があります。

## ボットの構築

Bot Framework に触れた経験があることを前提としています。経験があれば、問題ありません。経験がない場合でも、心配しすぎないでください。このセクションで十分に学習します。[この Microsoft Virtual Academy コース](https://mva.microsoft.com/ja-jp/training-courses/creating-bots-in-the-microsoft-bot-framework-using-c-17590#!)を修了し、[ドキュメント](https://docs.microsoft.com/ja-jp/bot-framework/)を確認することをお勧めします。

### ラボ: ボット開発の設定

C# SDK を使用してボットを開発します。  開始するには、次の 2 つのことが必要です。
1. Bot Framework プロジェクトのテンプレートは、[ここ](https://aka.ms/bf-bc-vstemplate)でダウンロードできます。  このファイルは "Bot Application.zip" と呼ばれ、\Documents\Visual Studio 2019\Templates\ProjectTemplates\Visual C#\ ディレクトリに保存する必要があります。  ここに zip ファイル全体をドロップするだけです。解凍する必要はありません。  
2. ボットをローカルでテストするために、Bot フレームワーク エミュレーターを[ここ](https://emulator.botframework.com/)から Bot Framework Emulator をダウンロードしてください。  エミュレーターは、ブラウザーに応じて、`c:\Users\`_your-username_`\AppData\Local\botframework\app-3.5.27\botframework-emulator.exe` フォルダーにインストールされます。 

### ラボ: 単純なボットを作成して実行する

Visual Studio で、「ファイル」 --> 「新しいプロジェクト」に移動し、「PictureBot」という名前のボット アプリケーションを作成します。  

![新しいボット アプリケーション](./resources/assets/NewBotApplication.jpg) 

サンプルのボット コードのファイルの内容を調べます。これは、メッセージとその文字数を繰り返すエコー ボットです。  特に、 **注**
+ App_Start の下の **WebApiConfig.cs** にあるルート テンプレートは api/{controller}/{id} で、ID は省略可能です。  そのため、必ず最後に API/メッセージが追加されたボットのエンドポイントを呼び出します。  
+ Controllers の下の **MessagesController.cs** は、ボットへのエントリポイントです。ボットはさまざまな種類のアクティビティに対応でき、メッセージを送信すると RootDialog が呼び出されます。  
+ Dialogs の下の **RootDialog.cs** にある "StartAsync" はユーザーからのメッセージを待機しているエントリ ポイントであり、"MessageReceiveAsync" は受信したメッセージを処理し、さらにメッセージを待機するメソッドです。  "context.PostAsync" を使用して、ボットからのメッセージをユーザーに送信します。  

ソリューションを右クリックして**「ソリューションの NuGet パッケージの管理」**を選択します。インストールされている Microsoft.Bot.Builder を検索し、最新バージョンに更新します。

F5 キーを押して、サンプル コードを実行します。  NuGet によって、適切な依存関係がが自動的にダウンロードされます。  

http://localhost:3979/ のような URL で、既定の Web ブラウザーでコードを起動します。  

> 楽しい余談: なぜこのポート番号でしょうか?  これは、プロジェクトのプロパティとして設定されます。  ソリューション エクスプローラーで、「プロパティ」をダブルクリックし、「Web」タブを選択します。  プロジェクトの URL は「サーバー」セクションで設定されます。  

![ボット プロジェクトの URL](./resources/assets/BotProjectUrl.jpg) 

プロジェクトがまだ実行されていることを確認し (プロジェクトのプロパティを確認するために停止した場合は、F5 キーをもう一度押す)、Bot Framework Emulator を起動します  (インストールしたばかりの場合は、ローカル コンピューターでの検索時にインデックスが表示されない可能性があるため、c:\Users\your-username\AppData\Local\botframework\app-3.5.27\botframework-emulator.exe. にインストールされていることを確認してください)。  ボットの URL が、上記でコードを起動したポート番号と一致していて、最後に API /メッセージが追加されていることを確認します。  ボットと会話できる状態になっているはずです。  

![Bot Emulator](./resources/assets/BotEmulator.png) 

### ラボ: LUIS を使用するようにボットを更新する

ここで、LUIS を使用するために、ボットを更新する必要があります。  これを行うには、[LuisDialog クラス](https://docs.botframework.com/ja-jp/csharp/builder/sdkreference/d8/df9/class_microsoft_1_1_bot_1_1_builder_1_1_dialogs_1_1_luis_dialog.html)を使用します。  

**RootDialog.cs** ファイルで、次の名前空間への参照を追加します。

```csharp

using Microsoft.Bot.Builder.Luis;
using Microsoft.Bot.Builder.Luis.Models;

```

次に、RootDialog クラスを変更して、IDialog<object> ではなく、LuisDialog<object> から派生させます。  次に、LUIS アプリ ID と LUIS キーがある LuisModel 属性をクラスに付与します。  (ヒント: LUIS アプリ ID にはハイフンが含まれ、LUIS キーには含まれません。  これらの値が見つからない場合は、http://luis.ai に戻ります。  アプリケーションをクリックすると、アプリ ID が「ダッシュボード」ページと URL に表示されます。  次に、[上部のサイドバーで「My keys」 (マイ キー) をクリック](https://www.luis.ai/home/keys)して、一覧でエンドポイント キーを見つけます)。  

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

それでは、コードを実行しましょう。  F5 キーをクリックして Visual Studio で実行し、Bot Framework Emulator で新しい会話を開始します。  ボットとチャットして、期待どおりの応答を得られることを確認してみましょう。  予期しない結果が生じた場合は、それをメモして LUIS を修正します。  

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

ここで、Azure Cognitive Search を呼び出すようにボットを更新しましょう。  まず、「ツール」 --> 「NuGet パッケージ マネージャー」 --> 「ソリューションの NuGet パッケージの管理」を開きます。  検索ボックスに「Microsoft.Azure.Search」と入力します。  対応するライブラリを選択し、プロジェクトを示すチェックボックスをオンにしてインストールします。  他の依存関係もインストールされる可能性があります。インストールされているパッケージで、"Newtonsoft.Json" パッケージの更新が必要な場合もあります。

![Azure Cognitive Search NuGet](./resources/assets/AzureSearchNuGet.jpg) 

Visual Studio のソリューション エクスプローラーでプロジェクトを右クリックし、「追加」 --> 「新しいフォルダー」を選択します。  "Models" という名前のフォルダーを作成します。  次に、"Models" フォルダーを右クリックして、「追加」 > 「既存の項目」を選択します。  "Models" フォルダーにこれら 2 つのファイルを追加するには、この操作を 2 回行います (必要に応じて名前空間を調整してください)。
1. [ImageMapper.cs](./resources/code/Models/ImageMapper.cs)
2. [SearchHit.cs](./resources/code/Models/SearchHit.cs)

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
            else
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

### ラボ: 正規表現とスコラブル グループ

ボットをさらに良いものにするためにできることは多数あります。何よりも、LUIS を単純な "hi" というあいさつ (ボットがかなり頻繁にユーザーから受け取る要求です) に使用したくはありません。  単純な正規表現でこれを行うことができ、時間の節約になり (ネットワークの待機時間)、費用も節約できます (LUIS サービスを呼び出すコスト)。  

また、ボットの複雑さが増し、ユーザーの入力を受け取って複数のサービスを使用して解釈するようになると、そのフローを管理するプロセスが必要になります。  たとえば、最初に正規表現を試してみて、見つからない場合は LUIS を呼び出し、その後は他のサービス、たとえば [QnA Maker](http://qnamaker.ai) や Azure Cognitive Search を試します。  これを管理する優れた方法は、[ScorableGroups](https://blog.botframework.com/2017/07/06/Scorables/) です。  ScorableGroups では、これらのサービス呼び出しの順序を指定する属性が提供されます。  このコードでは、最初に正規表現と一致する順序を指定し、次に LUIS を呼び出して発話を解釈して、最後に最も優先度が低いものを、一般的な "I'm not sure what you mean" (おっしゃっていることの意味がわかりません) という応答にドロップダウンします。    

ScorableGroups を使用するには、LuisDialog の代わりに、DispatchDialog から RootDialog を継承する必要があります (ただし、クラスに LuisModel 属性を存在させることは可能)。  また、Microsoft.Bot.Builder.Scorables (およびその他) を参照することも必要です。  したがって、RootDialog.cs ファイルで以下を追加します。

```csharp

using Microsoft.Bot.Builder.Scorables;
using System.Collections.Generic;

```

and change your class derivation to:

```csharp

    public class RootDialog : DispatchDialog

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
                    await context.PostAsync("I'm sorry. I didn't understand you.");
                    break;
            }
        }

```

このコードは、"hi"、"hello"、および "help" で始まるユーザーの式と一致します。  ユーザーが助けを求めると、ボットが実行できる 3 つの主要な操作 (写真の検索、写真の共有、プリントの注文) のボタンの簡単なメニューが表示されます。  

> 楽しい余談: ボットができることについてのオプションを並べたメニューを受け取るためにユーザーが「help」と入力する必要はないと主張する人もいるかもしれませんが、これはボットと最初に接触したときの既定の動作です。**見つけやすさ**はボットにとって最大の課題の 1 つです。このボットに何ができるかをユーザーに知ってもらう必要があります。  優れた[ボット設計の原則](https://docs.microsoft.com/ja-jp/bot-framework/bot-design-principles)が役立ちます。   

ここで、Scorable Group 1 で正規表現と一致するものがない場合に、2 回目の試行として LUIS を呼び出します。  

LUIS の "None" (なし) という意図は、発話が意図にマッピングされていないことを意味します。  このような状況では、次のレベルの ScorableGroup に落とし込みましょう。  次のように、RootDialog クラスに "None" メソッドを変更します。

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
            await context.PostAsync("I'm sorry. I didn't understand you.");
            await context.PostAsync("You can tell me to find photos, tweet them, and order prints.  次に例を示します。\"find pictures of food\".");
        }

```

F5 キーを押してボットを実行し、Bot Emulator でテストします。  

### ラボ: ボットを公開する

Microsoft Bot を使用して作成されたボットは、パブリック アクセス可能な任意の URL でホストできます。  このラボの目的のために Azure の Web サイト/App Service でボットをホストします。  

Visual Studio のソリューション エクスプローラーで、ボット アプリケーション プロジェクトを右クリックし、「公開」を選択します。  これにより、ボットを Azure に公開するために役立つウィザードが起動します。  

公開対象として「Microsoft Azure App Service」を選択します。  

![ボットを Azure App Service に公開する](./resources/assets/PublishBotAzureAppService.jpg) 

「App Service」画面で、適切なサブスクリプションを選択し、「新規」をクリックします。次に、API アプリ名、サブスクリプション、これまでに使用したのと同じリソース グループ、および  App Service プランを入力します。  

![App Service を作成する](./resources/assets/CreateAppService.jpg) 

最後に、Web 配置の設定が表示されたら、「公開」をクリックできます。  Visual Studio の出力ウィンドウに、配置プロセスが表示されます。  次に、ボットが http://testpicturebot.azurewebsites.net/ のような URL でホストされます。"testpicturebot" は App Service API アプリ名です。  

### ラボ: ボット コネクターでボットを登録する

ここで、Web ブラウザーを開き、[http://dev.botframework.com](http://dev.botframework.com) に移動します。  「[Register a bot](https://dev.botframework.com/bots/new)」をクリックします。ボットの名前、ハンドル、および説明を入力します。  メッセージング エンドポイントは、https://testpicturebot.azurewebsites.net/api/messages のように最後に "api/messages" が追加された Azure Web サイトの URL になります。  

![ボットの登録](./resources/assets/BotRegistration.jpg) 

次に、ボタンをクリックして Microsoft アプリ ID とパスワードを作成します。  これは、Web.config で必要になるボット アプリ ID とパスワードです。  ボット アプリの名前、アプリ ID、アプリのパスワードを安全な場所に保存してください!  パスワードに対して「OK」をクリックすると、パスワードに戻る方法はなくなります。  次に、「終了してボットのフレームワークに戻る」をクリックします。  

![ボットがアプリ名、ID、およびパスワードを生成する](./resources/assets/BotGenerateAppInfo.jpg) 

ボットの登録ページで、アプリ ID が自動的に入力されている必要があります。必要に応じて、ボットからログに記録するための AppInsights インストルメンテーション キーを追加できます。利用規約に同意する場合は、チェックボックスをオンにして、「登録」をクリックします。  

その後、ボットのダッシュボード ページが表示され、https://dev.botframework.com/bots?id=TestPictureBot のような URL が使用されますが、独自のボット名になります。ここで、さまざまなチャネルを有効にすることができます。  Skype と Web チャットの 2 つのチャネルが自動的に有効になります。  

最後に、ボットの登録情報を更新する必要があります。  Visual Studio に戻り、Web.config を開きます。  ボット ID をアプリ名で、MicrosoftAppId をアプリ ID で更新し、ボットの登録サイトから受け取ったアプリ パスワードを使用して MicrosoftAppPassword を更新します。  

```xml

    <add key="BotId" value="TestPictureBot" />
    <add key="MicrosoftAppId" value="95b76ae6-8643-4d94-b8a1-916d9f753ab0" />
    <add key="MicrosoftAppPassword" value="kC200000000000000000000" />

```

プロジェクトを再構築し、ソリューション エクスプローラーでプロジェクトを右クリックし、「公開」をもう一度選択します。  設定は前回に記憶されたものであるはずなので、「公開」をクリックするだけです。 

> MicrosoftAppPassword に関するエラーが発生した場合、これは XML 内にあるため、キーに「&」、「<」、「>」、「'」、「"」が含まれている場合は、これらの記号をそれぞれの[エスケープ機能](https://en.wikipedia.org/wiki/XML#Characters_and_escaping)である「&amp;」、「&lt;」、「&gt;」、「&apos;」、「&quot;」に置き換える必要があります。 

ここで、ボットのダッシュボード (https://dev.botframework.com/bots?id=TestPictureBot など) に戻ることができます。  チャット ウィンドウで話してみてください。  Web チャットでは、カルーセルの外観がエミュレーターの場合とは異なる場合があります。  Channel Inspector と呼ばれる優れたツールがあり、https://docs.botframework.com/ja-jp/channel-inspector/channels/Skype/#navtitle の別のチャネルでさまざまなコントロールのユーザー エクスペリエンスを確認します。  
ボットのダッシュボードから、他のチャンネルを追加したり、Skype、Facebook Messenger、または Slack でボットを試したりできます。  ボットのダッシュボードのチャンネル名の右側にある「追加」ボタンをクリックし、指示に従います。

**予定より早く終了した場合この追加のクレジット ラボをお試しください:**

高度な Azure Cognitive Search クエリを試してみましょう。LUIS モデルを拡張して、_"find happy people"_ などのエンティティを認識し、"happy" を "happiness"(Cognitive Services から返される感情) にマッピングし、[用語のブースト](https://docs.microsoft.com/ja-jp/rest/api/searchservice/Lucene-query-syntax-in-Azure-Search#bkmk_termboost)を使用してそれらをブースト クエリに変換することで、用語のブーストを追加します。 

## ラボの完了

このラボでは、Microsoft Bot Framework、Azure Cognitive Search、および複数の Cognitive Services を使用して、エンド ツー エンドでインテリジェントなボットを作成する方法について説明しました。

学習した内容は次のとおりです。
- インテリジェント サービスをアプリケーションに織り込む方法
- Azure Cognitive Search 機能を実装して、アプリケーション内で肯定的な検索エクスペリエンスを提供する方法
- フルテキスト検索、言語認識検索を有効にするためにデータを拡張するように Azure Cognitive Search サービスを構成する方法
- ボットが効果的に通信できるように LUIS モデルを構築、トレーニング、公開する
- LUIS と Azure Cognitive Search を活用する Microsoft Bot Framework を使用してインテリジェント ボットを構築する方法
- .NET アプリケーションでさまざまな Cognitive Services APIs (特に Computer Vision、Face、Emotion、LUIS) を呼び出す方法

将来のプロジェクト/学習のためのリソース
- [Azure Bot Services のドキュメント](https://docs.microsoft.com/ja-jp/bot-framework/)
- [Azure Cognitive Search のドキュメント](https://docs.microsoft.com/ja-jp/azure/search/search-what-is-azure-search)
- [Azure Bot Builder のサンプル](https://github.com/Microsoft/BotBuilder-Samples)
- [Azure Cognitive Search のサンプル](https://github.com/Azure-Samples/search-dotnet-getting-started)
- [LUIS のドキュメント](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/Home)
- [LUIS のサンプル](https://github.com/Microsoft/BotBuilder-Samples/blob/master/CSharp/intelligence-LUIS/README.md)