## 1_AzureSearch:
予想時間: 20-30 分

## Azure Search 

[Azure Search](https://docs.microsoft.com/ja-jp/azure/search/search-what-is-azure-search) は、サービスとしての検索ソリューションです。開発者がインフラストラクチャを管理する必要や、検索のエキスパートになる必要がなく、優れた検索エクスペリエンスをアプリケーションに組み込むことが可能になります。

開発者は、アプリでより良い、より迅速な結果を達成するために、Azure で PaaS サービスを探します。検索は多くの種類のアプリケーションの鍵を握っていますが、Web 検索エンジンでは検索のハードルが高く設定されています。スペルを間違えた場合や、余分な単語を含めた場合でも、ユーザーは瞬時の結果、入力時のオートコンプリート、結果に表示される項目の強調表示、ランキング、検索している内容を理解する能力を期待しています。

検索は困難で、コアな専門分野はまれです。インフラストラクチャの観点からは、高可用性、耐久性、拡張性、および運用が必要です。機能の観点からは、ランク付け、言語サポート、および地理空間機能が必要です。

![検索要件の例](./resources/assets/AzureSearch-Example.png) 

上記の例では、ユーザーが検索エクスペリエンスで期待しているコンポーネントの一部を示しています。[Azure Search](https://docs.microsoft.com/ja-jp/azure/search/search-what-is-azure-search) では、これらのユーザー エクスペリエンス機能を実現し、[監視とレポート作成](https://docs.microsoft.com/ja-jp/azure/search/search-traffic-analytics)、[簡単なスコアリング](https://docs.microsoft.com/ja-jp/rest/api/searchservice/add-scoring-profiles-to-a-search-index)。および[プロトタイプ作成](https://docs.microsoft.com/ja-jp/azure/search/search-import-data-portal)と[検査](https://docs.microsoft.com/ja-jp/azure/search/search-explorer)のためのツールを提供できます。

一般的なワークフロー:
1. サービスをプロビジョニングする
	- [portal](https://docs.microsoft.com/ja-jp/azure/search/search-create-service-portal) または [PowerShell](https://docs.microsoft.com/ja-jp/azure/search/search-manage-powershell) から Azure Search サービスを作成またはプロビジョニングできます。
2. インデックスを作成する
	- [インデックス](https://docs.microsoft.com/ja-jp/azure/search/search-what-is-an-index)はデータのコンテナーであり、「テーブル」と考えてください。スキーマ、[CORS オプション](https://docs.microsoft.com/ja-jp/aspnet/core/security/cors)、検索オプションがあります。[Portal](https://docs.microsoft.com/ja-jp/azure/search/search-create-index-portal) で、または[アプリの初期化](https://docs.microsoft.com/ja-jp/azure/search/search-create-index-dotnet)中に作成できます。 
3. インデックス データ
	- [データにインデックスに設定](https://docs.microsoft.com/ja-jp/azure/search/search-what-is-data-import)する方法は 2 つあります。最初のオプションは、Azure Search [REST API](https://docs.microsoft.com/ja-jp/azure/search/search-import-data-rest-api) または [.NET SDK](https://docs.microsoft.com/ja-jp/azure/search/search-import-data-dotnet) を使用して、データをインデックスに手動でプッシュすることです。2 番目のオプションは、[サポートされているデータ ソース](https://docs.microsoft.com/ja-jp/azure/search/search-import-data-portal)をインデックスにポイントし、スケジュールに基づいて Azure Search でデータを自動的に取得できるようにする方法です。
4. インデックスを検索する
	- 検索要求を Azure Search に送信する場合は、簡単な検索オプションを使用して、[結果をフィルター処理](https://docs.microsoft.com/ja-jp/azure/search/search-filters)、[並べ替え](https://docs.microsoft.com/ja-jp/rest/api/searchservice/add-scoring-profiles-to-a-search-index)、[投影](https://docs.microsoft.com/ja-jp/azure/search/search-faceted-navigation)、および[ページ オーバー](https://docs.microsoft.com/ja-jp/azure/search/search-pagination-page-layout)できます。スペルミス、ふりがな、および正規表現に対応する機能があり、検索と[提案](https://docs.microsoft.com/ja-jp/rest/api/searchservice/suggesters)を操作するためのオプションもあります。これらのクエリ パラメーターを使用すると、[フルテキスト検索エクスペリエンス](https://docs.microsoft.com/ja-jp/azure/search/search-query-overview)を詳細に制御できます。


### ラボ: Azure Search サービスを作成する

Azure Portal で、**「リソースの作成」->「Web + モバイル」->「Azure Search」** をクリックします。

これをクリックしたら、適切だと思う内容を入力する必要があります。このラボでは、"F0" Free レベルで十分です。1 つのサブスクリプションで所有できる無料 Azure Search インスタンスは 1 つだけなので、サブスクリプションの他のメンバーが既にこれを行っている場合は、「基本」価格レベルを使用する必要があります。このワークショップのすべてのラボに 1 つのリソース グループを使用します。`lab01.1-pcl_and_cognitive_services`を完了している場合は、そのリソース グループを使用できます。

![新しい Azure Search サービスを作成する](./resources/assets/AzureSearch-CreateSearchService.png)

作成が完了したら、新しい Search Service のパネルを開きます。

### ラボ: Azure Search インデックスを作成する

インデックスは、Azure Search サービスで使用されるドキュメントやその他の構成要素の永続的なストアです。インデックスは、データを保持し、検索クエリを受け入れるデータベースのようなものです。データベース内のフィールドと同様に、検索するドキュメントの構造を反映するようにマッピングするインデックス スキーマを定義します。これらのフィールドには、フルテキスト検索が可能かどうかや、フィルター処理が可能かどうかを伝えるプロパティを含めることができます。  プログラムで[コンテンツをプッシュ](https://docs.microsoft.com/ja-jp/rest/api/searchservice/addupdate-or-delete-documents)するか、[Azure Search インデクサー](https://docs.microsoft.com/ja-jp/azure/search/search-indexer-overview) (データの一般的なデータ ストアをクロールできる) を使用して、コンテンツを Azure Search に入力できます。

このラボでは、[Cosmos DB 用の Azure Search インデクサー](https://docs.microsoft.com/ja-jp/azure/search/search-howto-index-documentdb)を使用して、Cosmos DB コンテナー内のデータをクロールします。 

![インポート ウィザード](./resources/assets/AzureSearch-ImportData.png) 

作成した Azure Search ブレード内で、**「データのインポート」->「データ ソース」->「ドキュメント DB」** をクリックします。

![DocDB 用のインポート ウィザード](./resources/assets/AzureSearch-DataSource.png) 

これをクリックしたら、Cosmos DB データ ソースの名前を選択します。前のラボ `lab01.1-pcl_and_cognitive_services`を完了している場合は、データが存在している Cosmos DB アカウントと、対応するコンテナーとコレクションを選択します。前のラボを完了していない場合は、「または接続文字列を入力」を選択し、`AccountEndpoint=https://timedcosmosdb.documents.azure.com:443/;AccountKey=0aRt6JVgbf9KafBxRVuDMNfAj9YoSBbmpICdJ41N5CwHcjuMcVk7jWDBcu4BxbTitLR1zteauQsnF1Tgqs1A3g==;`に貼り付けます。どちらの場合も、データベースは "イメージ" であり、コレクションは "メタデータ" である必要があります。

**「OK」** をクリックします。

この時点で、Azure Search は Cosmos DB コンテナーに接続されています。いくつかのドキュメントを分析して Azure Search インデックスの既定のスキーマを識別します。これが完了したら、アプリケーションで必要とされるフィールドのプロパティを設定できます。

>**注:** 「_ts」 フィールドが有効なフィールド名ではないという警告が表示されることがあります。このラボではこれを無視できますが、詳細については[ここ](https://docs.microsoft.com/azure/search/search-indexer-field-mappings)をご覧ください。

インデックス名を **images** に更新する

キーを **id** に更新する (各ドキュメントを一意に識別する)

すべてのフィールドを **「取得可能」** に設定します (クライアントが検索時にこれらのフィールドを取得できるようにする)

フィールド「タグ」、**「NumFaces」、および「顔」** を **「フィルター設定可能」** に設定します (クライアントがこれらの値に基づいて結果をフィルター処理できるようにする)

フィールド **「NumFaces」** を **「並べ替え可能」** に設定します (クライアントがイメージ内の顔の数に基づいて結果を並べ替えることができるようにする)

フィールド「タグ」、**「NumFaces」、および「顔」** を **「ファセット可能」** に設定します (クライアントが結果を数でグループ化できるようにする (たとえば、この検索結果の場合、"beach" というタグが付けられている 5 つの写真がある)

フィールド **「キャプション」、「タグ」、「顔」** を **「検索可能」** に設定します (クライアントがこれらのフィールドのテキストに対してフルテキスト検索を行えるようにする)

![Azure Search インデックスを構成する](./resources/assets/AzureSearch-ConfigureIndex.png) 

この時点で、Azure Search アナライザーを構成します。  大まかには、アナライザーは、ユーザーが入力した用語を受け取り、インデックス内で最も一致する用語を見つけるものです。  Azure Search には、56 の言語を深く理解している Bingや Office などのテクノロジで使用されるアナライザーが含まれています。  

**「アナライザー」** タブをクリックし、フィールド **「キャプション」、「タグ」、「顔」** を設定して、**英語 - Microsoft** [アナライザー](https://docs.microsoft.com/ja-jp/azure/search/search-analyzers)を使用します。

![言語アナライザー](./resources/assets/AzureSearch-Analyzer.png) 

最後のインデックス構成手順として、[**提案者**](https://docs.microsoft.com/ja-jp/rest/api/searchservice/suggesters) を作成して、事前に入力するために使用されるフィールドを設定し、ユーザーがこれらのフィールドで最も一致しているものを探す単語の一部を入力できるようにします。提案者の詳細と、ユーザーが単語のスペルを間違えた場合でも、近い一致に基づいて結果を得られるようにあいまい検索をサポートするように検索を拡張する方法については、[この例](https://docs.microsoft.com/ja-jp/azure/search/search-query-lucene-examples#fuzzy-search-example)を確認してください。


**「候補者」** タブをクリックし、「提案者名」 に **「sg」** を入力し、用語の候補を検索するフィールドとして **「タグ」と「顔」** を選択します。

![検索候補](./resources/assets/AzureSearch-Suggester.png) 

**「OK」** をクリックして、インデクサーの構成を完了します。  インデクサーが変更をチェックする頻度をスケジュールで設定できますが、このラボでは 1 回だけ実行します。  

**「詳細オプション」** をクリックし、**「Base-64 エンコード キー」** を選択して、「ID」フィールドで、「Azure Search key」 フィールドでサポートされている文字のみが使用されることを確認します。

**「OK」をクリック** し、Cosmos DB データベースからのデータのインポートを開始するインデクサー ジョブを開始します。

![インデクサーを構成する](./resources/assets/AzureSearch-ConfigureIndexer.png) 

***検索インデックスに対してクエリを実行する***

インデックス作成が開始されたことを示すメッセージがポップアップ表示されるはずです。  インデックスのステータスを確認する場合は、Azure Search のメイン ブレードで「インデックス」オプションを選択できます。

この時点で、インデックスを検索できます。  

**「検索エクスプローラー」** をクリックし、結果のブレードでインデックスがまだ選択されていない場合は、「インデックス」を選択します。

**「検索」をクリック** して、すべてのドキュメントを検索します。"water" または他の語句を検索し、**Ctrl キーを押しながらクリック** を使用して、URL を選択して表示します。結果は予想通りでしたか?

![検索エクスプローラー](./resources/assets/AzureSearch-SearchExplorer.png)

結果の json では、`@search.score`の後に番号が表示されます。スコアリングとは、検索結果に返されるすべての項目の検索スコアを計算することです。スコアは、現在の検索操作のコンテキストにおける項目の関連性を示します。スコアが高いほど、項目の関連性が高いことを意味します。検索結果では、各項目について計算された検索スコアに基づいて、スコアが高い項目から低い項目の順にランク付けされます。

Azure Search では、既定のスコアリングを使用して初期スコアを計算しますが、[スコアリング プロファイル](https://docs.microsoft.com/ja-jp/rest/api/searchservice/add-scoring-profiles-to-a-search-index)を使用して計算をカスタマイズすることもできます。このワークショップの最後には、[用語のブースト](https://docs.microsoft.com/ja-jp/rest/api/searchservice/Lucene-query-syntax-in-Azure-Search#bkmk_termboost)を使用するハンズオン実習を行うための追加のラボがあります。

**予定より早く終了した場合この追加のクレジット ラボをお試しください:**

[Postman](https://www.getpostman.com/) は、Azure Search REST API 呼び出しを簡単に実行できるようにする優れたツールであり、優れたデバッグ ツールです。  Azure Search エクスプローラーから任意のクエリを実行し、Postman 内で実行する Azure Search API キーを使用できます。

[Postman](https://www.getpostman.com/) ツールをダウンロードしてインストールします。 

インストールしたら、Azure Search エクスプローラーからクエリを実行して、Postman に貼り付け、要求の種類として「GET」を選択します。  

「ヘッダー」をクリックし、次のパラメーターを入力します。

+ Content Type: application/json
+ api-key: [「キー」セクションの下にある Azure Search ポータルから API キーを入力します]

「送信」を選択すると、JSON 形式で書式設定されたデータが表示されるはずです。

[これらの例](https://docs.microsoft.com/ja-jp/rest/api/searchservice/search-documents#a-namebkmkexamplesa-examples)を使用して、他の検索を実行してみてください。


### [2_LUIS](./2_LUIS.md) に進みましょう

[README](./0_README.md) に戻る