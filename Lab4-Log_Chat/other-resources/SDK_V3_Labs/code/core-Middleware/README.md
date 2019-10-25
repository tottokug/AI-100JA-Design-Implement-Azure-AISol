# ミドルウェア ボットのサンプル

メッセージをインターセプトしてログに記録する方法を示すサンプル ボット。

[![Deploy to Azure][Deploy Button]][Deploy CSharp/MiddlewareLogging]

[Deploy Button]: https://azuredeploy.net/deploybutton.png
[Deploy CSharp/MiddlewareLogging]: https://azuredeploy.net

### 前提条件

このサンプルを実行するための最小の前提条件は次のとおりです。
* Visual Studio 2015 の最新の更新プログラム。Community バージョンは[ここ](http://www.visualstudio.com)から無料でダウンロードできます。
* Bot Framework EmulatorBot Framework Emulator をインストールするには、[ここ](https://emulator.botframework.com/)からダウンロードしてください。Bot Framework Emulator の詳細については、[このドキュメントの記事](https://github.com/microsoft/botframework-emulator/wiki/Getting-Started)を参照してください。

### コードのハイライト

会話の履歴を処理するとき、最も一般的な操作の 1 つは、ボットとユーザーの間のメッセージ アクティビティをインターセプトしてログに記録することです。[`Microsoft.Bot.Builder.History`](https://github.com/Microsoft/BotBuilder/tree/master/CSharp/Library/Microsoft.Bot.Builder.History)名前空間には、これを行うためのインターフェイスとクラスが用意されています。特に、[`IActivityLogger`](https://github.com/Microsoft/BotBuilder/blob/master/CSharp/Library/Microsoft.Bot.Builder/Dialogs/IActivityLogger.cs)インターフェイスには、クラスでメッセージ アクティビティをログに記録するために実装する必要がある機能の定義が含まれています。

デバッグで実行しているときにのみトレース リスナーにメッセージ アクティビティを書き込む`IActivityLogger`インターフェイスの[`DebugActivityLogger`](DebugActivityLogger.cs)の実装を確認してください。

````C#
public class DebugActivityLogger : IActivityLogger
{
    public async Task LogAsync(IActivity activity)
    {
        Debug.WriteLine($"From:{activity.From.Id} - To:{activity.Recipient.Id} - Message:{activity.AsMessageActivity()?.Text}");
    }
}
````

既定では、BotBuilder ライブラリで、操作なしアクティビティ ロガーである会話コンテナーによって[`NullActivityLogger`](https://github.com/Microsoft/BotBuilder/blob/master/CSharp/Library/Microsoft.Bot.Builder/Dialogs/IActivityLogger.cs#L81)が登録されます。[`Global.asax.cs`](DebugActivityLogger.cs)内のサンプルの[`DebugActivityLogger`](Global.asax.cs#L11-L13)の Autofac コンテナーでの登録を確認してください。

````C#
var builder = new ContainerBuilder();
builder.RegisterType<DebugActivityLogger>().AsImplementedInterfaces().InstancePerDependency();
builder.Update(Conversation.Container);
````

### 結果

サンプル ソリューションを開いて実行するときに、[Visual Studio 出力ウィンドウの「Debug text」 (テキストのデバッグ) ペイン](https://blogs.msdn.microsoft.com/visualstudioalm/2015/02/09/the-output-window-while-debugging-with-visual-studio/)に次の結果が表示されます。

ボットが受信したメッセージ:
````
From:default-user - To:ii845fc9l02209hh6 - Message:Hello bot!
````

ユーザーに送信されたメッセージ:
````
From:ii845fc9l02209hh6 - To:default-user - Message:You sent Hello bot! which was 10 characters
````

### 詳細情報

.NET 用の Bot Builder および会話の履歴の使用方法の詳細については、次のリソースを参照してください。

* [.NET 用の Bot Builder](https://docs.microsoft.com/ja-jp/bot-framework/dotnet/)
* [メッセージをインターセプトする](https://docs.microsoft.com/ja-jp/bot-framework/dotnet/bot-builder-dotnet-middleware)
* [Microsoft.Bot.Builder.History 名前空間](https://docs.botframework.com/ja-jp/csharp/builder/sdkreference/dc/dc6/namespace_microsoft_1_1_bot_1_1_builder_1_1_history.html)
* [TableLogger (Azure Table Storage を使用したアクティビティ ロガー)](https://github.com/Microsoft/BotBuilder/blob/master/CSharp/Library/Microsoft.Bot.Builder.Azure/TableLogger.cs#L60)