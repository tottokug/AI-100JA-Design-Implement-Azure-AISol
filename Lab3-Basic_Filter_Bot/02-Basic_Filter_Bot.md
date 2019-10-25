---
lab:
    title: 'ラボ 3: 基本的なフィルタリング ボットを作成する'
    module: 'モジュール 2: ボットの作成'
---

# ラボ 3: 基本的なフィルタリング ボットを作成する

## ラボ 3.0 始める前に考える

新たに登場する技術はどれも、それがもたらす機会と同じぐらい多くの疑問点が伴います。そして、AI を活用する技術には、固有の考慮事項があります。
AI ツールを設計して実装するときは、次の AI 倫理の原則を常に念頭に置いてください。

1. *公平性*: 効率を最大化するために尊厳を傷つけてはならない
1. *説明責任*: AI はそのアルゴリズムに責任を持つ必要がある
1. *透明性*: 偏見や、人間の尊厳の破壊を許してはならない
1. *倫理的な応用*: AI は人類を支援するものであり、インテリジェントにプライバシーを守る設計でなければならない

インテリジェントなアプリを構築するときの倫理面に関する考慮事項について、[詳しい説明](https://ai-ethics.azurewebsites.net/)を読むことをお勧めします。

## ラボ 3.1: ボット開発のためのセットアップを行う

このボットの開発には、最新の .NET SDK (v4) を使用します。開始するには、Bot Framework Emulator をダウンロードして、Web アプリ ボットを作成してソース コードを取得する必要があります。

v4 SDK をダウンロードする手順は、[Lab1-Technical_Requirements.md](../Lab1-Technical_Requirements/02-Technical_Requirements.md) に記載されています。

> **[2018/09/29] 重要な注意事項**
> ボット フレームワークのための v4 SDK は最近 [GA](https://github.com/Microsoft/botbuilder-dotnet) となりました。このラボを、v4 SDK ではなく v3 SDK で実施する場合は[ここ](./other-resources/sdk_v3_labs)を参照してください。ただし、V3 のラボは今後保守されないことに注意してください。

#### Bot Framework Emulator をダウンロードする

[Lab1-Technical_Requirements.md](../Lab1-Technical_Requirements/02-Technical_Requirements.md) に記載されている手順に従って v4 Preview Bot Framework Emulator をダウンロードします。これは、ボットをローカルでテストできるようにするためです。

## Azure Web App Bot を作成する

Microsoft Bot Framework を使用して作成されたボットは、パブリック アクセス可能な任意の URL でホストできます。  このラボの目的のために、[Azure Bot Service](https://docs.microsoft.com/ja-jp/bot-framework/bot-service-overview-introduction) を使用してボットを登録します。

ポータルに移動します。このポータルで、「リソースを作成」をクリックして「bot」を検索します。「Web App Bot」を選択して「作成」をクリックします。名前用に、一意の識別子を作成する必要があります。PictureBot[i][n] のような規則にすることをお勧めします。[i] は自分のイニシャルで、[n] は数字です (例: PictureBotamt6)。自分に最も近いリージョンに配置します。
価格レベルについては、**F0** を選択します。このラボではこれで十分です。ボット テンプレートは、**SDK v4** の **Echo Bot** 用、**C#** のものを選択します。後で、このコードをこのラボの PictureBot に合わせて更新します。新しい App Service プランを構成します (ボットと同じ場所に配置する)。Application Insights をオンにするかどうかを選択できます。「アプリ ID とパスワードの自動作成」は変更もクリックも **しないでください**。これは後で設定します。「作成」をクリックします。

![Azure Bot Service を作成する](../images/CreateBot2.png)

デプロイされたら、Web アプリのリソースに移動します。ここまでの手順で、ごく単純なエコー ボットを、Echo Bot テンプレートを使用してデプロイしました。必要に応じて、デプロイ後に「Test in Web Chat」タブを選択すると、このボットで何ができるかを見ることができます。

作成しようとしているボットはこれで完成しているわけではないため、次にソース コードをダウンロードする必要があります。このコードを次のいくつかのラボで編集してから、このサービスにもう一度発行します。

ポータル内の「ビルド」タブに移動し、「ボットのソース コードをダウンロードする」を選択します。都合の良い場所に保存し、zip ファイルからすべてのファイルを抽出します。

Azure ポータルを開いたままにしているので、この機会に `botFilePath` と `botFileSecret` を見つけて保存しておきましょう。これらは、作成した Web App Bot サービスの **「App Service の設定」 > 「アプリケーションの設定」 > 「アプリケーションの設定」** セクションにあります。

> 将来自分でボットを作成するときは、ここで行っているようにポータルでボットを作成してソース コードをダウンロードすることも、Bot Builder テンプレートを使用することもできます。その手順については、次のリンク先を参照してください: [Lab1-Technical_Requirements.md](../Lab1-Technical_Requirements/02-Technical_Requirements.md)

## ラボ 3.2: 単純なボットを作成して実行する

>注記: このラボでは、ボットのコードとロジックをセットアップします。コード内のトピックの中には、たとえば LUIS のように、まだレッスンで取り上げていないものもあります。この機能については、後のレッスンで学習します。

作成した Web App Bot のソリューション ファイルの場所に移動して Visual Studio で開きます (将来自分でボットを作成するときは Visual Studio Code を使用できますが、このラボの目的では Visual Studio を使用してください)。少し時間を取って、どのようなものが Echo Bot テンプレートから組み込まれるかを調べてください。個々のファイルについて説明することはしませんが、**後で** 少し時間を取ってこのサンプル (ともう 1 つの Web App Bot サンプル - Basic Bot) を実際に試して精査することを **強くお勧めします** (まだの場合)。この中には、ボット開発に必要となる、重要で有用なシェルがあります。これとその他の有用なシェルやサンプルは、[ここ](https://github.com/Microsoft/BotBuilder-Samples)にあります。

このラボの目的に合わせて、テンプレートのさまざまな部分に変更を加えますが、これは実際の開発作業でも必要になります。

まず、ソリューションを右クリックして「ビルド」を選択します。これでパッケージが読み込まれます。次に、appsettings.json ファイルの中に次のコードを追加します。これは、ボット サービスの情報を追加するコードです。

```json
{
    "botFilePath": "自分のボット ファイル パス",
    "botFileSecret": "自分のボット ファイル シークレット"
}
```

>重要: 別のボット サービスに切り替える場合は、この情報を更新する必要があります

次に、必要な NuGet パッケージに焦点を当てます。ソリューション エクスプローラーでソリューションを右クリックして「ソリューションの NuGet パッケージの管理」を選択します。

#### Microsoft.AspNetCore.All または Microsoft.AspNetCore を更新しないでください - 開始

「インストール済み」タブで、次のパッケージを **この順に** `4.1.5` に更新します (これは既に行われていることもあります)。

>注記: このラボは、4.1.5 よりも新しいバージョンでのテストは行われていません。

* Microsoft.Bot.Configuration
* Microsoft.Bot.Schema
* Microsoft.Bot.Connector
* Microsoft.Bot.Builder
* Microsoft.Bot.Builder.Integration.AspNet.Core

#### Microsoft.AspNetCore.All または Microsoft.AspNetCore を更新しないでください - 完了

次に、「参照」タブをクリックし、以下のすべてのパッケージをインストールします。必ず「プレリリースを含める」チェック ボックスをオンにして、プレリリースが「参照」タブに表示されていることを確認してください。

>注記: バージョン 4.1.5 を使用していることを確認してください (可能な場合)

* Microsoft.Bot.Builder.Azure
* Microsoft.Bot.Builder.AI.Luis
* Microsoft.Bot.Builder.Dialogs
* Microsoft.Azure.Search (最新バージョンを使用)

最後に、ソリューション エクスプローラーで **「依存関係」 > 「NuGet」** に移動して次のパッケージを削除します。

* [AsyncUsageAnalyzers](https://www.nuget.org/packages/AsyncUsageAnalyzers/)
* [StyleCop.Analyzers](https://www.nuget.org/packages/StyleCop.Analyzers)

また、"EchoBotWithCounter.ruleset" もルート プロジェクト ディレクトリから削除してかまいません。このファイルは、これらのパッケージを使用して作成されるものであるからです。これらは、本稼働用のボットに役立つパッケージであり、コーディングとコメントのスタイルを確実に標準化するのに役立ちます。これらの内容についてはこのワークショップでは取り上げませんが、上記のリンク先で詳しい情報を参照できます。

インストールが完了すると、ソリューション エクスプローラーの **「依存関係」 > 「NuGet」** の下に次のパッケージが表示されます。

```csharp
* Microsoft.AspNetCore
* Microsoft.AspNetCore.All
* Microsoft.Azure.Search
* Microsoft.Bot.Builder
* Microsoft.Bot.Builder.AI.Luis
* Microsoft.Bot.Builder.Azure
* Microsoft.Bot.Builder.Dialogs
* Microsoft.Bot.Builder.Integration.AspNet.Core
* Microsoft.Bot.Configuration
* Microsoft.Bot.Connector
* Microsoft.Bot.Schema
* Microsoft.Extensions.Logging.AzureAppServices
```

ご存知とは思いますが、Visual Studio のソリューション/プロジェクトの名前を変更することは、きわめて注意を要する作業です。**慎重に** 次のタスクを行って、すべての名前に EchoBot の代わりに PictureBot が反映される状態にしてください。

> 注記: Visual Studio でファイルの名前を変更すると、すべての参照が解決されるまでに最大 15 秒かかることがあります。この時間が経過するまで待たなかった場合は、ビルドに失敗するため、リファクタリングされたオブジェクトを手動で解決する必要があります。あせらないでください。

1. ソリューション、プロジェクトの順に名前を "EchoBotWithCounter" から "PictureBot" に変更します。Visual Studio を閉じて再度開きます。
1. Program.cs を開いて「BotBuilderSamples」をハイライトし、右クリックして「名前の変更」を選択します。文字列とコメントの中のすべての出現箇所を変更することを指定するチェック ボックスをオンにします。名前を「PictureBot」に変更して「適用」を選択します。
1. 「プロパティ」 > 「launchSettings.json」を開いて「EchoBotWithCounter」を「PictureBot」に変更します。
1. 「wwwroot」 > 「default.htm」を開き、「Echo bot with counter sample」と「Echo with Counter Bot」の出現箇所をすべて「PictureBot」で置き換えます。
1. 名前を "CounterState.cs" から "PictureState.cs" に変更します

> 注記: これを行うには、ファイルを右クリックして「名前を変更」を選択します。ポップアップが表示されます。「はい」を選択します。**名前の変更を求められたすべてのファイルに対して、この操作を行います。**


1. PictureState.cs を開き、クラス名が「PictureState」であることを確認します。そうでない場合は、「CounterState」をハイライトし、右クリックして「名前の変更」を選択します。文字列とコメントの中のすべての出現箇所を変更することを指定するチェック ボックスをオンにします。「PictureState」に変更して「適用」を選択します。
1. 「EchoBotAccessors.cs」を「PictureBotAccessors.cs」に変更します。
1. 「EchoWithCounterBot.cs」を「PictureBot.cs」に変更します。
1. 「EchoBotWithCounter.deps.json」を「PictureBot.deps.json」に変更します。
1. 「EchoBotWithCounter.runtimeconfig.json」を「PictureBot.runtimeconfig.json」に変更します。
1. ソリューションをビルドします。

>**ヒント**:  モニターが 1 台だけの場合に手順の説明と Visual Studio を簡単に切り替えるために、手順ファイルを Visual Studio ソリューションに追加できるようになりました。それには、ソリューション エクスプローラーでプロジェクトを右クリックし、**「追加」 > 「既存の項目」** を選択します。「Lab2」に移動し、種類が "MD ファイル" のすべてのファイルを追加します。

#### Hello World ボットを作成する

ラボの今後の部分で使用する命名と NuGet パッケージをサポートするように基本シェルの更新が完了したので、カスタム コードの追加を開始する準備ができました。最初に、V4 SDK を使用するボット開発のウォームアップとしてシンプルな "Hello world" ボットを作成します。

重要な概念の 1 つが "ターン" です。これは、ユーザーへのメッセージとボットからの応答を表すのに使用されます。
たとえば、ユーザーが「ハロー、ボット」と言ってボットが「こんにちは、お元気ですか?」と答えると、これが **1 つの** ターンとなります。次の画像で、1 つの **ターン** がボット アプリケーションの複数のレイヤーをどのように通過するかを確認してください。

![ボットの概念](../images/bots-concepts-middleware.png)

ラボのこのセクションの目的のために、Startup.cs の ConfigureServices メソッドに移動して `CounterState = conversationState.CreateProperty<CounterState>(PictureBotAccessors.CounterStateName),` の行をコメントアウトします (`//` を使用)。状態とアクセサーについては、後のセクションで説明します。

"Hello world" を動作させるためにその他に更新する必要があるファイルは、PictureBot.cs だけです。このファイルを開いてコメントを確認してください。

コードとコメントをある程度理解したら、`OnTurnAsync` メソッドを下記のコードで置き換えてください。
このメソッドは、会話のすべてのターンで呼び出されます。このことが重要である理由は後で分かりますが、ここでは OnTurnAsync がすべてのターンで呼び出されることを覚えておいてください。

```csharp
        /// <summary>
        /// PictureBot のすべての会話ターンでこのメソッドを呼び出します。
        /// ダイアログは使用しません。これは "シングル ターン" 処理、つまりただ 1 つの
        /// 要求と応答であるからです。後でダイアログを追加するときに、このメソッドを見直す必要があります。
        /// </summary>
        /// <param name="turnContext"><see cref="ITurnContext"/> 型。この会話ターンの処理に
        /// 必要なデータすべてが格納されます。</param>
        /// <param name="cancellationToken">(省略可能) <see cref="CancellationToken"/> 型。他のオブジェクトまたはスレッドで
        /// キャンセルの通知を受け取るのに使用されます。</param>
        /// <returns><see cref="Task"/> 型。実行のためにキューに登録された作業を表します。</returns>
        /// <seealso cref="BotStateSet"/>
        /// <seealso cref="ConversationState"/>
        /// <seealso cref="IMiddleware"/>
        public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
        {
            // ユーザーからメッセージが送信された場合
            if (turnContext.Activity.Type is "message")
            {
                {
                    await turnContext.SendActivityAsync($"Hello world.");
                }
            }
        }
```

では、ボットを起動しましょう (デバッグはオンでもオフでもかまいません)。「PictureBot」という表示のある、再生ボタンのような形のボタンを押します (またはキーボードの F5 キーを押してください)。NuGet が自動的に、適切な依存関係をダウンロードします。ブレーク ポイントに遭遇した場合は、**ブレーク ポイントを解除** して「続行」を選択してください (再生ボタンを押します)。

次のことに注意してください。

* default.htm (wwwroot の下) ページがブラウザーに表示されます。
* Web ページのローカルホスト ポート番号に注目してください。これは、Emulator のエンドポイントと一致しているはずです (一致する必要があります)。
* コンソール ウィンドウも表示されます。もし、Visual Studio Code を使用するとすれば、これは `dotnet run` の実行後にターミナルの「出力」として表示されます。このラボの目的では、これを無視してかまいません。

>行き詰まってしまったときは? このラボのこの時点までのソリューションは、[resources/code/FinishedPictureBot-Part0](./code/FinishedPictureBot-Part0) にあります。このソリューション内の readme ファイルを開くと、ソリューションを実行するためにどのキーの追加が必要かがわかります。

#### Bot Framework Emulator を使用する

ボットと対話するには:

* Bot Framework Emulator を起動します (このラボでは v4 Emulator を使用します)。
* 「Welcome」ページの「Open bot」を選択し、プロジェクトのルートにある ".bot" で終わるファイルを選択します。画面の指示に従って、`botSecret` の値を入力します。
* これで、左側のメニューにあるメッセージ タブと、「ENDPOINT」の下にある「development」をクリックできるようになりました。また、次に説明する「production」エンドポイントも表示されます。
* この時点で、ボットと会話できる状態になっています。
* 「hello」と入力すると、ボットはすべてのメッセージに「Hello World」と応答します。
* 「Start Over」を選択すると、会話の履歴が消去されます。

![ボット エミュレーター](../images/botemulator3.png)

ログの中に、次のような行があるはずです。

![ngrok](../images/ngrok.png)

これは、ngrok がローカル アドレスに対してバイパスされることを示しています。このワークショップでは ngrok を使用しませんが、仮に一般公開版のボットに接続するとしたら、"production" エンドポイントを介して接続します。この "production" エンドポイントを開き、各環境でのボットの違いを観察してください。これは、開発版ボットをテストして本番ボットと比較するときに役立つ機能です。

Emulator の使い方の詳細については、[ここ](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-service-debug-emulator?view=azure-bot-service-4.0)を参照してください。
> 余談: なぜこのポート番号でしょうか?  これは、プロジェクトのプロパティとして設定されます。  ソリューション エクスプローラーで、**「プロパティ」 > 「デバッグ」** をダブルクリックして内容を調べてください。アプリの URL は、エミュレーターでの接続先と一致していますか?

サンプルのボット コードのファイルの内容を調べます。特に、次の点に注目してください。

* **Startup.cs** は、サービス/ミドルウェアを追加し、HTTP 要求パイプラインを構成するのに使用するファイルです。この中に多数存在するコメントは、何が行われるのかを理解するのに役立ちます。少し時間を取って読んでみてください。

* **PictureBot.cs** の中にある `OnTurnAsync` がエントリ ポイントであり、ここでユーザーからのメッセージを待ち受けます。`turnContext.Activity.Type is "message"` で、受信したメッセージへの対応を行い、それ以降のメッセージを待ち受けます。  `turnContext.SendActivityAsync` を使用して、ボットからユーザーにメッセージを返すことができます。

## ラボ 3.3:  状態とサービスの管理

もう一度 Startup クラスに移動します。このクラスの内容を確認します (まだの場合)。確認したら、`using` ステートメントのリストに次の行を追加します。

```csharp
using System.Text.RegularExpressions;
using Microsoft.Bot.Builder.AI.Luis;
```

上記の内容はまだ使用しませんが、いつ使うかを考えてみてください。

`ConfigureServices` メソッドに注目してください。これは、ボットにサービスを追加するのに使用されます。内容を精査して、何が自動的にビルドされるかを見つけてください。

> より深く理解するための追加の注意事項:
>
> * 依存関係挿入については、[こちらの詳しい説明](https://docs.microsoft.com/ja-jp/aspnet/web-api/overview/advanced/dependency-injection)を参考にしてください。
> * このラボとテストの目的では、ローカル メモリを使用できます。本番運用には、[状態データを管理する](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-builder-storage-concept?view=azure-bot-service-4.0)方法を実装する必要があります。`ConfigureServices` の中のまとまったコメント行に、このヒントがあります。
> * このメソッドの一番下で、状態アクセサーを作成して登録していることに気付いたでしょうか。状態の管理は、インテリジェントなボットを作成するための鍵です。[詳しくは、こちらを参照してください](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-builder-dialog-state?view=azure-bot-service-4.0)。

幸いなことに、このシェルはかなり包括的であり、追加する必要のあるアイテムはミドルウェアとカスタム状態アクセサーの 2 つだけです。

#### ミドルウェア

ミドルウェアは、単一のクラスまたはクラスの集合で、アダプターとボット ロジックの間に存在し、アダプターのミドルウェアのコレクションに初期化時に追加されます。SDK を使用して、開発者は独自のミドルウェアをプログラミングすることや、他者が作成したミドルウェアの再利用可能コンポーネントを追加することができます。ボットに出入りするすべてのアクティビティは、ミドルウェアを通って流れます。これについては、このラボで後ほど詳しく取り上げますが、現時点で理解しておいてほしいのは、どのアクティビティもミドルウェアを通って流れるということです。その理由は、ミドルウェアが `ConfigureServices` メソッドの中に存在し、このメソッドが実行時に呼び出されるからです (ユーザーと OnTurnAsync によって送信されるすべてのメッセージの間で実行されます)。次に示すコードを、`options.State.Add(conversationState);` の後の行に追加します。

```csharp

                var middleware = options.Middleware;
                // ミドルウェアをこの下に "middleware.Add(...." で追加する
                // この下に正規表現を追加する

```

#### カスタム状態アクセサー

必要なカスタム状態アクセサーについて説明する前に、背景情報が重要です。ダイアログ (次のセクションで説明します) は、マルチターン会話ロジックを実装するためのアプローチの 1 つです。この場合は、ユーザーが現在会話のどこにいるかを知るために、永続化された状態に依存する必要があります。このラボで作成するダイアログ ベースのボットでは、DialogSet を使用してさまざまなダイアログを保持します。この DialogSet を作成するには、"アクセサー" と呼ばれるオブジェクトへのハンドルを使用します。

アクセサーは、SDK の中にある `IStatePropertyAccessor` インターフェイスを実装します。これによって、状態に関する情報の取得、設定、削除ができるようになります。したがって、開発者はユーザーが会話の中のどのステップにいるかを追跡することができます。

作成するアクセサーのそれぞれについて、最初にプロパティ名を指定する必要があります。このラボでは、次のものを追跡します。

1. `PictureState`
    * 既にユーザーに挨拶したか?
        * 挨拶を 2 回以上する必要はありませんが、会話の最初に確実に挨拶する必要があります。
    * ユーザーは現在特定の単語を検索しているか? もしそうなら、それは何か?
        * ユーザーが何を探しているかをこちらに伝えてくれたかどうか、伝えてくれた場合はそれが何なのかを、追跡する必要があります。
2. `DialogState`
    * ユーザーは現在ダイアログの途中にいるか?
        * これは、ユーザーがあるダイアログまたは会話フローの中のどこにいるかを特定するのに使用します。ダイアログの知識がなくても、心配しないでください。この後すぐに説明します。

下記のコードでわかるように、EchoBot テンプレート (起点として使用したテンプレート) にはカスタム状態アクセサーがあり、これはカウンターの役割を果たします。つまり、1 つの会話の中でのターン数をこれで数えています ("CounterState")。

```csharp
                // カスタム状態アクセサーを作成します。
                // 状態アクセサーは、他のコンポーネントが状態の個々のプロパティの読み取りや書き込みを行うのに使用できます。
                var accessors = new PictureBotAccessors(conversationState)
                {
                    //CounterState = conversationState.CreateProperty<PictureState>(PictureBotAccessors.CounterStateName),
                };

                return accessors;
```

ターンを数えることはしませんが、類似のコンストラクトを使用して `PictureState` を追跡することができます。`CounterState` と同じ命名規則を使用して、`PictureState` をカスタム状態アクセサーのリストの中に追加します。

最後に、ダイアログの追跡を目的として組み込みの `DialogState` を使用しますが、そのために次の行をカスタム状態アクセサーのリストに追加します。

```csharp
                    DialogStateAccessor = conversationState.CreateProperty<DialogState>("DialogState"),
```

いくつかのアイテムの下にエラー (赤い波線) があります。しかし、これらを修正する前に疑問に思うかもしれません。なぜアクセサーを 2 つ作成する必要があったのか? なぜ 1 つでは不十分だったのか?

* `DialogState` は `Microsoft.Bot.Builder.Dialogs` ライブラリからのアクセサーです。メッセージが送信されると、Dialog サブシステムが `CreateContext` を `DialogSet` に対して呼び出します。このコンテキストを追跡するには、`DialogState` アクセサーが必要です。これは、適切なダイアログ状態 JSON を取得するためのアクセサーです。
* 一方、`PictureState` は、指定された特定の会話プロパティを会話全体で追跡するために使用されます (たとえば、既にユーザーに挨拶したかどうか)。

> ダイアログに関する専門用語を知らなくても、このプロセスは理解できるはずです。よく理解できなかった場合は、[状態のしくみの詳しい説明](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-builder-dialog-state?view=azure-bot-service-4.0)を参照してください。

では、先ほど見ていたエラーに戻りましょう。この情報を保存する必要がありますが、どこに、どのように保存するかはまだ指定していません。情報を保存してアクセスするには、"PictureState.cs" と "PictureBotAccessor.cs" を更新する必要があります。

"PictureState.cs" に移動します。ここに、アクティブな会話に関する情報を格納します。`TurnCount` という名前の整数を保存しようとしていることがわかります。EchoBotWithCounter ではこれが何のためのものであったか、わかりますか?

このラボではターンの追跡はしませんが、他のものを追跡するので、`TurnCount` を削除して、その場所に次の行を追加します。

```csharp
        public string Greeted { get; set; } = "not greeted";
        public string Search { get; set; } = "";
        public string Searching { get; set; } = "no";
```

文字列の目的を説明するコメントを自由に追加してください。これで、PictureState が適切に初期化されたので、"Startup.cs" で発生していたエラーを解消するように PictureBotAccessor を更新できます。

"PictureBotAccessors.cs" に移動し、`CounterStateName` と `CounterState` を見つけます。CounterState に対して指定されているものをテンプレートとして使用して、`PictureStateName` と `PictureState` に必要なものを実装します。

最後に、次のコードを追加します。これは、ダイアログ状態を通して `DialogSets` を使用できるようにするためです (すぐ後で説明します)。

```csharp
        /// <summary> DialogState アクセサーに使用する IStatePropertyAccessor{T} の名前を取得します。</summary>
        public static string DialogStateName { get; } = $"{nameof(PictureBotAccessors)}.DialogState";

        /// <summary> DialogState のための IStatePropertyAccessor{T} を取得または設定します。</summary>
        public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }
```

`using` ステートメントを "PictureBotAccessor.cs" と "Startup.cs" に追加する必要があります。どちらかわかりますか? このパッケージは、ダイアログを構築するための多くのツールが収録されているのと同じパッケージです。

これを正しく構成したかどうかが心配ですか? "Startup.cs" に戻り、カスタム状態アクセサーの作成に関するエラーが解決されていることを確認してください。

## ラボ 3.4: ボットのコードを整理する

ボットを開発する方法はさまざまなものがあり、好みもそれぞれ異なります。この SDK では、開発者の好みの方法でコードを整理することができます。このラボでは、会話を整理してさまざまなダイアログにまとめます。また、会話関連のコードを [MVVM スタイル](https://msdn.microsoft.com/ja-jp/library/hh848246.aspx)で整理する方法も考察します。

この PictureBot は、次のように整理します。

* **ダイアログ** - モデルを編集するためのビジネス ロジック
* **応答** - ユーザーへの出力を定義するクラス
* **モデル** - 変更対象のオブジェクト

作業に入る前に、プロジェクト内に 2 つのフォルダー "**Responses**" と "**Models**" を作成します。(ヒント: プロジェクトを右クリックして「追加」を選択します。)

#### ダイアログ

ダイアログとそのしくみについては、既にご存じかもしれません。そうでない場合は、続行する前に[こちらのダイアログに関するページ](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-builder-dialog-manage-conversation-flow?view=azure-bot-service-4.0&tabs=csharp)をお読みください。

1 つのボットで複数のタスクを実行できる場合は、複数のダイアログを持てるようにすると、ユーザーがさまざまな会話フローをたどることができるようになります。このラボの PictureBot では、ユーザーが最初のメニュー フロー (メイン ダイアログと呼ばれることもよくあります) を通過したら、複数のダイアログに分岐します。この分岐は、ユーザーがしようとしていることが写真の検索、写真の共有、写真の注文、ヘルプ参照のどれであるかに基づきます。これを簡単に行うには、ダイアログ コンテナーを使用します。このラボでは `DialogSet` を使用します。続行する前に、[モジュール型のボット ロジックと複雑なダイアログの作成に関する記事](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-builder-compositcontrol?view=azure-bot-service-4.0&tabs=csharp)をお読みください。

このラボの目的では、処理をごく単純なものにしますが、修了すると多数のダイアログから成るダイアログ セットを作成できるようになります。このラボの PictureBot には、次の 2 つの主要ダイアログがあります。

* **MainDialog** - ボット開始時の既定のダイアログ。このダイアログから、ユーザーが要求した他のダイアログが起動します。このダイアログはダイアログ セットのメイン ダイアログであるため、このダイアログでダイアログ コンテナーを作成し、必要に応じてユーザーを他のダイアログにリダイレクトします。
* **SearchDialog** - 検索要求を処理して結果をユーザーに返すことを管理するダイアログ。  *注記: このワークショップでは、この機能を呼び出しますが、検索は実装しません。*

ダイアログは 2 つしかないので、単純に両方とも PictureBot クラスに入れておくことができます。ただし、複雑なシナリオでは、これらを同じフォルダー内の別々のダイアログに分けることが必要になる可能性があります ("応答" と "モデル" を分けたのと同じように)。

PictureBot.cs に戻り、一連の `using` ステートメントを次のコードで置き換えます。

```csharp
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Extensions.Logging;
using System.Linq;
using PictureBot.Models;
using PictureBot.Responses;
using Microsoft.Bot.Builder.AI.Luis;
using Microsoft.Azure.Search;
using Microsoft.Azure.Search.Models;
using System;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
```

これで、モデル/応答へのアクセスと、サービス LUIS と Azure Search へのアクセスが追加されました。最後の Newtonsoft への参照は、LUIS からの応答を解析するのに役立ちます。これについては、後続のラボで学習します。

次に、`OnTurnAsync` の現在のメソッドを置き換える必要があります。具体的には、受け取ったメッセージを処理してからさまざまなダイアログを通してそのメッセージをルーティングするメソッドで置き換えます。

クラスの残りの部分を次のシェルで置き換えてください。

```csharp
namespace Microsoft.PictureBot
{
    /// <summary>
    /// 受け取ったアクティビティを処理するボットを表します。
    /// ユーザーとのやり取りのたびに、このクラスのインスタンスが 1 つ作成され、OnTurnAsync メソッドが呼び出されます。
    これは有効期間が一時的であるサービスです。  有効期間が一時的のサービスは、
    /// 要求されるたびに作成されます。受信したアクティビティごとに、このクラスの新しい
    /// インスタンスが作成されます。オブジェクトの構築にコストがかかる場合や、有効期間が
    /// ターン 1 つだけでない場合は、管理に注意が必要です。
    /// たとえば、<see cref="MemoryStorage"/> オブジェクトと、関連付けられた
    /// <see cref="IStatePropertyAccessor{T}"/> は、シングルトンの有効期間で作成されます。
    /// </summary>
    /// <seealso cref="https://docs.microsoft.com/ja-jp/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-2.1"/>
    /// <summary>この中に、写真ボットのためのダイアログとプロンプト一式があります。</summary>
    public class PictureBot : IBot
    {
        private readonly PictureBotAccessors _accessors;
        // LUIS 認識エンジンを初期化する

        private readonly ILogger _logger;
        private DialogSet _dialogs;

        /// <summary>
        /// PictureBot のすべての会話ターンでこのメソッドを呼び出します。
        /// ダイアログは使用しません。これは "シングル ターン" 処理、つまりただ 1 つの
        /// 要求と応答であるからです。後でダイアログを追加するときに、このメソッドを見直す必要があります。
        /// </summary>
        /// <param name="turnContext"><see cref="ITurnContext"/> 型。この会話ターンの処理に
        /// 必要なデータすべてが格納されます。</param>
        /// <param name="cancellationToken">(省略可能) <see cref="CancellationToken"/> 型。他のオブジェクトまたはスレッドで
        /// キャンセルの通知を受け取るのに使用されます。</param>
        /// <returns><see cref="Task"/> 型。実行のためにキューに登録された作業を表します。</returns>
        /// <seealso cref="BotStateSet"/>
        /// <seealso cref="ConversationState"/>
        /// <seealso cref="IMiddleware"/>
        public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
        {
            if (turnContext.Activity.Type is "message")
            {
                // ダイアログのコンテキストを会話の状態から確立します。
                var dc = await _dialogs.CreateContextAsync(turnContext);
                // 現在のダイアログがある場合は続行します。
                var results = await dc.ContinueDialogAsync(cancellationToken);

                // すべてのターンで応答を送信します。したがって、応答が何も送信されなかった場合は、
                // 現在アクティブなダイアログはありません。
                if (!turnContext.Responded)
                {
                    // メイン ダイアログを起動する
                    await dc.BeginDialogAsync("mainDialog", null, cancellationToken);
                }
            }
        }
        /// <summary>
        /// <see cref="PictureBot"/> クラスの新しいインスタンスを初期化します。
        /// </summary>
        /// <param name="accessors">状態の管理に使用される <see cref="IStatePropertyAccessor{T}"/> が格納されているクラス。</param>
        /// <param name="loggerFactory"><see cref="ILoggerFactory"/> 型。Azure アプリ サービス プロバイダーにフックされます。</param>
        /// <seealso cref="https://docs.microsoft.com/ja-jp/aspnet/core/fundamentals/logging/?view=aspnetcore-2.1#windows-eventlog-provider"/>
        public PictureBot(PictureBotAccessors accessors, ILoggerFactory loggerFactory /*, LuisRecognizer 認識エンジン*/)
        {
            if (loggerFactory == null)
            {
                throw new System.ArgumentNullException(nameof(loggerFactory));
            }

            // LUIS 認識エンジンのインスタンスを追加する

            _logger = loggerFactory.CreateLogger<PictureBot>();
            _logger.LogTrace("PictureBot ターン開始。");
            _accessors = accessors ?? throw new System.ArgumentNullException(nameof(accessors));

            // DialogSet には DialogState アクセサーが必要です。ターン コンテキストがあるときにこれを呼び出します。
            _dialogs = new DialogSet(_accessors.DialogStateAccessor);

            // この配列は、ウォーター フォールの実行方法を定義します。
            // さまざまなダイアログとそのステップをここで定義できます。
            // 必要であれば、オーバーラップも可能です。この場合は、非常に簡単です。
            // しかし、より複雑なシナリオでは、ダイアログをそれぞれ
            // 別のファイルに分けることが必要になります。
            var main_waterfallsteps = new WaterfallStep[]
            {
                GreetingAsync,
                MainMenuAsync,
            };
            var search_waterfallsteps = new WaterfallStep[]
            {
                // SearchDialog ウォーター フォールのステップを追加する

            };

            // 名前付きダイアログを DialogSet に追加します。これらの名前は、ダイアログ状態の中に保存されます。
            _dialogs.Add(new WaterfallDialog("mainDialog", main_waterfallsteps));
            _dialogs.Add(new WaterfallDialog("searchDialog", search_waterfallsteps));
            // 次の行で、プロンプトをダイアログ内で使用できるようにする
            _dialogs.Add(new TextPrompt("searchPrompt"));
        }
        // MainDialog 関連のタスクを追加する

        // SearchDialog ダイアログ関連のタスクを追加する

        // 検索関連のタスクを追加する

    }

}
```

少し時間を取ってこのシェルを精査し、ワークショップの他の参加者とディスカッションしてください。続行する前に、各行の目的を理解する必要があります。

これには、後ほどさらにコードを追加します。エラーは無視してかまいません (今のところは)。

#### 応答

ダイアログの内容を作成する前に、応答を準備しておく必要があります。既に説明したように、ダイアログと応答を分けます。このようにするとコードがすっきりするのと、ダイアログのロジックを追うのが簡単になるのが理由です。今は同意できなくても、すぐにできるようになります。

2 つのクラス "MainResponses.cs" と "SearchResponses.cs" を "Responses" フォルダーの中に作成します。既にお分かりかもしれませんが、Responses のファイルの内容はユーザーに送信するさまざまな出力だけであり、ロジックは含まれていません。

"MainResponses.cs" の中に、次の行を追加します。

```csharp
using System.Threading.Tasks;
using Microsoft.Bot.Builder;

namespace PictureBot.Responses
{
    public class MainResponses
    {
        public static async Task ReplyWithGreeting(ITurnContext context)
        {
            // 挨拶を追加する
        }
        public static async Task ReplyWithHelp(ITurnContext context)
        {
            await context.SendActivityAsync($"私は写真の検索、写真の共有、写真プリントの注文を行うことができます。");
        }
        public static async Task ReplyWithResumeTopic(ITurnContext context)
        {
            await context.SendActivityAsync($"どれをご希望ですか?");
        }
        public static async Task ReplyWithConfused(ITurnContext context)
        {
            // ユーザーへの応答を追加する (正規表現と LUIS のどちらも
            // ユーザーが伝えようとすることを理解しない場合)
        }
        public static async Task ReplyWithLuisScore(ITurnContext context, string key, double score)
        {
            await context.SendActivityAsync($"Intent: {key} ({score}).");
        }
        public static async Task ReplyWithShareConfirmation(ITurnContext context)
        {
            await context.SendActivityAsync($"あなたの写真を Twitter に投稿します...");
        }
        public static async Task ReplyWithOrderConfirmation(ITurnContext context)
        {
            await context.SendActivityAsync($"写真の標準プリントを注文します...");
        }
    }
}
```

値のない応答が 2 つあることに注目してください (ReplyWithGreeting と ReplyWithConfused)。適切だと思う値を入力してしてください。

"SearchResponses.cs" の中に、次の行を追加します。

```csharp
using Microsoft.Bot.Builder;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.Bot.Schema;

namespace PictureBot.Responses
{
    public class SearchResponses
    {
        // "ReplyWithSearchRequest" というタスクを追加する
        // これはコンテキストを受け取り、ユーザーに
        // 何を検索するかを尋ねます。



        public static async Task ReplyWithSearchConfirmation(ITurnContext context, string utterance)
        {
            await context.SendActivityAsync($"OK、{utterance} の写真を検索します");
        }
        public static async Task ReplyWithNoResults(ITurnContext context, string utterance)
        {
            await context.SendActivityAsync(" \"" + utterance + "\" の結果は見つかりませんでした。");
        }
    }
}
```

あるタスク全体が欠落していることに注目してください。適切だと思う内容を自分で入力してください。ただし、新しいタスクの名前は "ReplyWithSearchRequest" としてください。このとおりでない場合は、後で問題が発生する可能性があります。

#### モデル

時間に制限があるため、すべてのモデルの作成の説明はしません。これらのモデルは単純ですが、追加後に少し時間を取ってコードを精査することをお勧めします。"Models" フォルダーを右クリックして、**「追加」>「既存の項目」** を選択します。"code/Models" に移動し、3 つのファイルをすべて選択して「追加」を選択します。

この時点で、ソリューション エクスプローラーは次の画像のようになります。

![ボットのソリューション フォルダー ビュー](../../Linked_Image_Files/solutionExplorer.png)

不足しているものはありませんか? この機会に、すべてのファイルがそろっていることを確認してください。その他のエラーは無視してかまいません (今のところは)。

### ラボ 3.5: 正規表現とミドルウェア

ボットをさらに良いものにするためにできることは多数あります。何よりも、LUIS を単純な "写真を検索" メッセージ (ボットがかなり頻繁にユーザーから受け取る要求です) に使用したくはありません。  単純な正規表現でこれを行うことができ、時間の節約になり (ネットワークの待機時間)、費用も節約できます (LUIS サービスを呼び出すコスト)。

また、ボットの複雑さが増し、ユーザーの入力を受け取って複数のサービスを使用して解釈するようになると、そのフローを管理するプロセスが必要になります。  たとえば、最初に正規表現を試してみて、それで見つけられない場合は LUIS を呼び出し、その後は他のサービス、たとえば [QnA Maker](http://qnamaker.ai) や Azure Search を試します。これを管理するのに良い方法の 1 つは[ミドルウェア](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-builder-concept-middleware?view=azure-bot-service-4.0)を使用することであり、SDK はこれをサポートします。

ラボを続行する前に、ミドルウェアと Bot Framework SDK についてさらに学習してください。

1. [概要とアーキテクチャ](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-builder-basics?view=azure-bot-service-4.0)
1. [ミドルウェア](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-builder-concept-middleware?view=azure-bot-service-4.0)
1. [ミドルウェアの作成](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-builder-create-middleware?view=azure-bot-service-4.0&tabs=csaddmiddleware%2Ccsetagoverwrite%2Ccsmiddlewareshortcircuit%2Ccsfallback%2Ccsactivityhandler)

最終的には、ミドルウェアを使用して、ユーザーが言っていることの理解を試みます。最初に正規表現 (Regex) を使用し、理解できない場合は LUIS を呼び出します。それでも理解できない場合は、「おっしゃっていることの意味がわかりません」という応答、またはその他の開発者が "ReplyWithConfused" に対して指定したものを返します。

正規表現のミドルウェアを自分のソリューションに追加するには、新しいフォルダーを "Middleware" という名前で作成し、Middleware フォルダーの内容 (**resources > code** の下にあります) を自分のソリューションに追加します。**このミドルウェアは、将来の自分のプロジェクトでも使用できます!**

"Startup.cs" の、`ConfigureServices` の中にある「この下に正規表現を追加する」というコメントの下に、次の行を追加します。

```csharp
                middleware.Add(new RegExpRecognizerMiddleware()
                .AddIntent("search", new Regex("search picture(?:s)*(.*)|search pic(?:s)*(.*)", RegexOptions.IgnoreCase))
                .AddIntent("share", new Regex("share picture(?:s)*(.*)|share pic(?:s)*(.*)", RegexOptions.IgnoreCase))
                .AddIntent("order", new Regex("order picture(?:s)*(.*)|order print(?:s)*(.*)|order pic(?:s)*(.*)", RegexOptions.IgnoreCase))
                .AddIntent("help", new Regex("help(.*)", RegexOptions.IgnoreCase)));

```

> ここでは、正規表現の使い方のごく一部を示しています。興味がある場合は、[こちらの詳しい情報を参照してください](https://docs.microsoft.com/ja-jp/dotnet/standard/base-types/regular-expression-language-quick-reference)。

LUIS を追加していないので、このボットはいくつかのバリエーションを理解するだけですが、ユーザーがこのボットを使って写真を検索し、共有し、プリントを注文するときに、かなりのメッセージを理解するはずです。

> 余談: ボットができることについてのオプションを並べたメニューを受け取るためにユーザーが「help」と入力する必要はないだろうと主張する人もいるかもしれませんが、これはボットと最初に接触したときの既定の動作です。**見つけやすさ** はボットにとって最大の課題の 1 つです。このボットに何ができるかをユーザーに知ってもらう必要があります。  優れた[ボット設計の原則](https://docs.microsoft.com/ja-jp/bot-framework/bot-design-principles)が役立ちます。


## ラボ 3.6: ボットを実行する

#### MainDialog 再び

本題に戻りましょう。PictureBot.cs の中の MainDialog のコードを用意する必要があります。ユーザーがしたいことを言ったら、ボットがそれに反応できるようにするためです。

正規表現の結果に基づいて、会話を正しい方向に導く必要があります。コードをよく読んで、何をするかを確実に理解したら、次のコードを `// MainDialog 関連のタスクを追加する` の下に貼り付けてください。

```csharp
        // まだユーザーに挨拶していない場合は最初に挨拶しますが、会話の残りの
        // 部分では、挨拶済みであることを記憶しておく必要があります。
        private async Task<DialogTurnResult> GreetingAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
        {
            // 会話の現在のステップの状態を取得する
            var state = await _accessors.PictureState.GetAsync(stepContext.Context, () => new PictureState());

            // まだユーザーに挨拶していない場合
            if (state.Greeted == "not greeted")
            {
                // ユーザーに挨拶する
                await MainResponses.ReplyWithGreeting(stepContext.Context);
                // GreetedState を "挨拶済み" に更新する
                state.Greeted = "greeted";
                // 新しい "挨拶済み" 状態を会話の状態に保存する
                // これは以降のターンで再びユーザーに挨拶しないために行います。
                await _accessors.ConversationState.SaveChangesAsync(stepContext.Context);
                // 次に何をしたいかユーザーに尋ねる
                await MainResponses.ReplyWithHelp(stepContext.Context);
                // このステップでは明示的にユーザーにプロンプトを出さないので、ダイアログが終了します。
                // ユーザーが応答したときに、状態が維持されているので、else 句で
                // 次のウォーター フォール ステップに進ませます。
                return await stepContext.EndDialogAsync();
            }
            else // ユーザーに挨拶済み
            {
                // 次のウォーター フォール ステップ (MainMenuAsync) に移動する
                return await stepContext.NextAsync();
            }

        }

        // このステップでユーザーをさまざまなダイアログにルーティングする
        // この例では他のダイアログは 1 つだけのため単純ですが、
        // より複雑なシナリオでは、同様の他のダイアログに移動できます。
        public async Task<DialogTurnResult> MainMenuAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
        {
            // 現在ユーザーの検索を処理しているかどうかを調べる
            var state = await _accessors.PictureState.GetAsync(stepContext.Context);

            // 正規表現で何らかのことを理解した場合は、それを保存する
            var recognizedIntents = stepContext.Context.TurnState.Get<IRecognizedIntents>();
            // 認識された意図に基づいて、会話の方向を決める
            switch (recognizedIntents.TopIntent?.Name)
            {
                case "search":
                    // 検索ダイアログに切り替える
                    return await stepContext.BeginDialogAsync("searchDialog", null, cancellationToken);
                case "share":
                    // 写真を共有することを応答として返す
                    await MainResponses.ReplyWithShareConfirmation(stepContext.Context);
                    return await stepContext.EndDialogAsync();
                case "order":
                    // 注文することを応答として返す
                    await MainResponses.ReplyWithOrderConfirmation(stepContext.Context);
                    return await stepContext.EndDialogAsync();
                case "help":
                    // ヘルプを表示する
                    await MainResponses.ReplyWithHelp(stepContext.Context);
                    return await stepContext.EndDialogAsync();
                default:
                    {
                        await MainResponses.ReplyWithConfused(stepContext.Context);
                        return await stepContext.EndDialogAsync();
                    }
            }
        }
```

F5 キーを押してボットを実行します。テストするために、「help」、「share pics」、「order pics」、「search pics」などのコマンドを送信してテストします。期待どおりの結果を得られなかったのが "search pics" だけの場合は、すべては自分で構成したとおりに動作しています。"search pics" の失敗は、ラボのこの時点での予期される動作ですが、理由は分かりますか? 次に進む前に答えを考えてください。

>ヒント: ブレーク ポイントを使用して、case "search" への一致を、PictureBot.cs からトレースしてください。

>行き詰まってしまったときは? このラボのこの時点までのソリューションは、[resources/code/FinishedPictureBot-Part1](./code/FinishedPictureBot-Part1) にあります。このソリューション内の readme ファイルを開くと、ソリューションを実行するためにどのキーの追加が必要かがわかります。これは、ソリューションとして実行するのではなく、参照として使用することをお勧めしますが、実行する場合は、必要なキーを必ず追加してください (このセクションでは、Bot Service のシークレットのみのはずです)。

 ### さらに進む

 ボットの作成を練習するには、[.NET Bot Builder SDK チュートリアル](https://docs.microsoft.com/ja-jp/azure/bot-service/dotnet/bot-builder-dotnet-sdk-quickstart?view=azure-bot-service-4.0)を実行することと、[Bot Service のドキュメント](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-service-overview-introduction?view=azure-bot-service-4.0)を読むことをお勧めします。

ボットの構築ステップを復習するには、[Bot Builder Basics](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-builder-basics?view=azure-bot-service-4.0&tabs=cs) を確認してください。