## 2_Luis:
予想時間: 20-30 分

## LUIS

まず、[マイクロソフトの Language Understanding Intelligent Service (LUIS) について学習](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/Home)しましょう。

LUIS の概要を把握したところで、LUIS アプリを計画します。検索に基づいてイメージを返すボットを作成し、共有または並べ替えを行います。ボットが実行できるさまざまなアクションをトリガーする意図を作成し、そのアクションの実行に必要なパラメーターをモデル化するエンティティを作成する必要があります。たとえば、PictureBot の意図が "SearchPics" であり、写真を検索するために検索サービスがトリガーされます。これには、検索対象を知るために "facet" エンティティが必要です。アプリを計画するためのその他の例については、[ここ](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/plan-your-app)を参照してください。

アプリについて考え抜いたら、[それを構築してトレーニングする](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/luis-get-started-create-app)準備が整いました。LUIS アプリケーションの作成時に一般的に実行する手順は次のとおりです。
  1. [意図を追加する](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/add-intents) 
  2. [発話を追加する](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/add-example-utterances)
  3. [エンティティを追加する](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/add-entities)
  4. [機能を使用してパフォーマンスを向上させる](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/add-features)
  5. [トレーニングとテストを行う](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/train-test)
  6. [アクティブ ラーニングを使用する](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/label-suggested-utterances)
  7. [公開する](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/publishapp)


### ラボ: Portal での LUIS サービスの作成

Portal で **「リソースを作成」** をクリックし、検索ボックスに **「LUIS」** と入力して、**「Language Understanding Intelligent Service」** を選択します。

作成する API エンドポイントの詳細を入力し、目的の API と、エンドポイントを配置する場所、目的の料金プランを選択します。このラボでは、Free レベルで十分です。LUIS では、将来の Cognitive Services Vision サービスを改善するためにマイクロソフト社内に (安全な方法で) イメージを保存するため、これを承諾することを確認するチェックボックスをオンにする必要があります。

新しい API サブスクリプションを作成したら、ブレードの適切なセクションからキーを取得して、キーのリストに追加できます。

![Cognitive API キー](./resources/assets/cognitive-keys.PNG)

### ラボ: LUIS を使用してアプリケーションにインテリジェンスを追加する

次のラボでは、PictureBot を作成します。まず、LUIS を使用して、自然言語機能を追加する方法を見てみましょう。LUIS を使用すると、自然言語の発話 (ボットと話すときにユーザーが話す単語/フレーズ/文) を意図 (ユーザーが実行するタスクやアクション) にマッピングできます。  ここでは、写真の検索、写真の共有、写真のプリントの順序付けなど、いくつかの意図があります。  これらの事柄を尋ねる方法として、発話のいくつかの例をご紹介します。LUIS では、学習した内容に基づいて、それぞれの意図に追加の新しい発話がマッピングされます。  

[Https://www.luis.ai](https://www.luis.ai) に移動して、Microsoft アカウントを使用してサインインします (これは、前のセクションで LUIS キーを作成したのと同じアカウントにする必要があります)。[https://www.luis.ai/applications](https://www.luis.ai/applications) の LUIS アプリケーションの一覧にリダイレクトされるはずです。  ボットをサポートする新しい LUIS アプリを作成します。  

> 楽しい余談: [現在のページ](https://www.luis.ai/applications)の「New App」 (新しいアプリ) ボタンの横に「Import App」 (アプリのインポート) もあります。  LUIS アプリケーションを作成した後、アプリ全体を JSON としてエクスポートし、ソース管理にチェックインできます。  これは推奨されるベスト プラクティスであり、コードのバージョン管理に合わせて LUIS モデルのバージョンを管理できます。  エクスポートされた LUIS アプリは、「Import App」 (アプリのインポート) ボタンを使用して再インポートできます。  ラボの進行が遅れてしまい、ショートカットする場合は、「Import App」 (アプリのインポート) ボタンをクリックして [LUIS モデル](./resources/code/LUIS/PictureBotLuisModel.json)をインポートできます。  

[Https://www.luis.ai/applications](https://www.luis.ai/applications) から「New App」 (新しいアプリ) ボタンをクリックします。  名前を付けて ("PictureBotLuisModel" を選択)、「Culture」 (文化) を「English」 (英語) に設定します。  必要に応じて、説明を入力できます。次に、「Create」 (作成) をクリックします。  

![LUIS の新しいアプリ](./resources/assets/LuisNewApp.png) 

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

次に、左側のサイドバーの「Intents」 (意図) をクリックし、黄色の「Add Intent」 (意図の追加) ボタンをクリックします。  "SearchPics" という意図名を付け、「Save」 (保存) をクリックします。  

"greetings" で行ったのと同じように、いくつかのサンプル発話 (ボットと話すときにユーザーが話す可能性のある単語/フレーズ/文) を追加してみましょう。  写真を検索する方法は数多くあります。  以下の発話の一部を自由に使用して、ボットに写真を検索するように依頼する方法について独自の文章を追加してください。 

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

いくつかの発話を用意したら、**検索トピック** を "facet" エンティティとして選択する方法を LUIS に教える必要があります。"facet" エンティティが選択するものは何でも検索されます。キーワードにカーソルを合わせてクリックするか、単語のグループをドラッグして選択選択し、次に "facet" エンティティを選択します。 

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

また、"None" (なし) という意図が 1 つあることにも注意してください。  いずれの意図にもマッピングされないランダムな発話は、"None" (なし) にマッピングされる場合があります。  

これで、モデルをトレーニングする準備が整いました。左側のサイドバーの「Train & Test」 (トレーニングとテスト) をクリックします。次に、「train」 (トレーニング) ボタンをクリックします。これにより、指定したトレーニング データを使用して「utterance」 (発話) -->「intent mapping」 (意図のマッピング) を行うモデルが構築されます。トレーニングはすぐに行われるとは限りません。場合によっては、キューに入れられ、数分かかることがあります。

次に、左側のサイドバーにある「Publish App」 (アプリの公開) をクリックします。  アプリを公開するときには、[詳細なエンドポイント応答や Bing スペル チェッカー](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/PublishApp)を有効にすることを含めて、いくつかのオプションがあります。まだ設定していない場合は、以前に設定したエンドポイント キーを選択するか、リンク先に従って Azure アカウントからキーを追加します。  エンドポイント スロットは "Production" (運用環境) のままにしておくことができます。  次に、「Publish」 (公開) をクリックします。  



![LUIS アプリを公開する](./resources/assets/PublishLuisApp.png) 

公開すると、LUIS モデルを呼び出すエンドポイントが作成されます。  URL が表示されます。これについては、後のラボで説明します。

左側のサイドバーの「Train & Test」 (トレーニングとテスト) をクリックします。  「Enable published model」 (公開されたモデルを有効にする) チェック ボックスをオンにすると、モデルを直接呼び出すのではなく、公開されたエンドポイントを介して呼び出します。いくつかの発話を入力して、意図が返されることを確認してください。  
>残念ながら「Enable published model」 (公開されたモデルを有効にする) を開く操作にはバグがあり、Chrome でのみ動作します。Chrome をダウンロードして試すことも、スキップすることもできます。ただし、スキップする場合でも、有効になっていないことを覚えておいてください。

![LUIS をテストする](./resources/assets/TestLuis.png) 

[公開されたエンドポイントをブラウザーでテストする](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/PublishApp#test-your-published-endpoint-in-a-browser)こともできます。URL をコピーし、`{YOUR-KEY-HERE}`を、使用するリソースの "Key String" (キー文字列) 列に一覧表示されているキーのいずれかに置き換えます。ブラウザーでこの URL を開くには、URL パラメーター`&q`をテスト クエリに設定します。たとえば、URL に`&q=Find pictures of dogs`を追加し、Enter キーを押します。ブラウザーには、HTTP エンドポイントの JSON 応答が表示されます。

**予定より早く終了した場合次の追加クレジット タスクをお試しください。**


「SearchPics」という意図によって活用できる追加のエンティティを作成します。たとえば、アプリで年齢が判断されることがわかっています。年齢に対して事前に構築されたエンティティを作成してみましょう。 

エンティティの種類が "List" (リスト) のカスタム エンティティを使用して、感情と性別を取得してみましょう。以下の感情の例を参照してください。 

![リスト付きのカスタム感情エンティティ](./resources/assets/CustomEmotionEntityWithList.jpg) 

> **注**: エンティティや機能を追加するときには、**「Intents」 (意図) > 「Utterances」 (発話)** に移動し、追加したエンティティに対して発話を確認することや、さらに発話を追加することを忘れないでください。また、モデルを再トレーニングして公開する必要があります。


### [3_Bot](./3_Bot.md) に進みましょう

[README](./0_README.md) に戻る