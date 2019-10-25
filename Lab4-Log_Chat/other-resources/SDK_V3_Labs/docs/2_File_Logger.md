# ファイル ロガー

## 1.	目的

この演習の目的は、ミドルウェアを使用してチャットの会話をファイルに記録することです。入出力 (I/O) 操作が多すぎると、ボットからの応答が遅くなる可能性があります。このラボでは、グローバル イベントを利用して、ファイルへのチャット会話の効率的なログ記録を実行します。このアクティビティは、前のラボの概念の拡張です。

Visual Studio の code\file-core-Middleware からプロジェクトをインポートします。

## 2. 非効率的なログの記録方法

チャットの会話をファイルに記録する非常に簡単な方法は、以下のコード スニペットに示すように File.AppendAllLines を使用することです。File.AppendAllLines でファイルを開き、指定した文字列をファイルに追加して、ファイルを閉じます。ただし、ユーザー/ボットからのすべてのメッセージのファイルを開いて閉じるので、これは非常に非効率なログ記録方法です。
tu
````C#
public class DebugActivityLogger : IActivityLogger
{
    public async Task LogAsync(IActivity activity)
    {
        File.AppendAllLines("C:\\Users\\username\\log.txt", new[] { $"From:{activity.From.Id} - To:{activity.Recipient.Id} - Message:{activity.AsMessageActivity().Text}" });
    }
}
````

## 3.	効率的なログの記録方法

より効率的なファイルへの記録方法は、Global.asax ファイルを使用して、ボットのライフサイクルに関連するイベントを使用することです。以下のコード スニペットに示すように、global.asax を拡張します。次の行を、環境内に存在しているパスに変更します。

````C# 
tw = new StreamWriter("C:\\Users\\username\\log.txt", true);
````

Application_Start で、特定のエンコーディングのストリームに文字を書き込むストリーム ラッパーを実装する StreamWriter を開きます。これはボットのライフサイクルを維持し、各メッセージのファイルを開いたり閉じたりせずに、ボットに書き込むことができます。また、StreamWriter がパラメータとして DebugActivityLogger に送信されることにも注目してください。Application_End でログ ファイルを閉じることができます。アプリケーションがアンロードされる前に、アプリケーションの有効期間ごとに 1 回呼び出されます。

````C#
using System.Web.Http;
using Autofac;
using Microsoft.Bot.Builder.Dialogs;
using System.IO;
using System.Diagnostics;
using System;


namespace MiddlewareBot
{
    public class WebApiApplication : System.Web.HttpApplication
    {
        static TextWriter tw = null;
        protected void Application_Start()
        {
            tw = new StreamWriter("C:\\Users\\username\\log.txt", true);
            Conversation.UpdateContainer(builder =>
            {
                builder.RegisterType<DebugActivityLogger>().AsImplementedInterfaces().InstancePerDependency().WithParameter("inputFile", tw);
            });

            GlobalConfiguration.Configure(WebApiConfig.Register);
        }

        protected void Application_End()
        {
            tw.Close();
        }
    }
}
````

DebugActivityLogger コンストラクターでファイル パラメーターを受け取り、LogAsync を更新し、次の行を追加してログ ファイルに書き込みます。

````C#
tw.WriteLine($"From:{activity.From.Id} - To:{activity.Recipient.Id} - Message:{activity.AsMessageActivity().Text}", true);
tw.Flush();
````

DebugActivityLogger コード全体は次のようになります。

````C#
using System.Diagnostics;
using System.Threading.Tasks;
using Microsoft.Bot.Builder.History;
using Microsoft.Bot.Connector;
using System.IO;

namespace MiddlewareBot
{    
    public class DebugActivityLogger : IActivityLogger
    {
        TextWriter tw;
        public DebugActivityLogger(TextWriter inputFile)
        {
            this.tw = inputFile;
        }


        public async Task LogAsync(IActivity activity)
        {
            tw.WriteLine($"From:{activity.From.Id} - To:{activity.Recipient.Id} - Message:{activity.AsMessageActivity().Text}", true);
            tw.Flush();
        }
    }

}
````

## 4. ログ ファイル

エミュレーターでボット アプリケーションを実行し、メッセージを使用してテストします。行で指定されているログ ファイルを調べます

````C# 
tw = new StreamWriter("C:\\Users\\username\\log.txt", true);
````

ユーザーとボットからログ メッセージを表示するには、メッセージがログ ファイルに次のように表示されます。

````From:2c1c7fa3 - To:56800324 - Message:a log message````

````From:56800324 - To:2c1c7fa3 - Message:You sent a log message which was 13 characters````

## 5. 選択的ログ

特定のシナリオでは、選択的にメッセージに対してモデリングを実行するか、選択的にログに記録することが望ましいでしょう。このような例は多数あります: a) 非常にアクティブなコミュニティ ボットは、大量のチャット メッセージを非常に迅速にキャプチャできますが、チャット メッセージの多くはあまり有用ではない可能性があります。b) 特定のユーザー/ボットまたは特定のトレンド製品のカテゴリ/トピックに関連するメッセージに焦点を当てるために、選択的にログに記録する必要があることもあります。

常にすべてのチャット メッセージをキャプチャし、選択的にメッセージをマイニングするためにフィルタリングを実行できます。ただし、選択的にログに記録する柔軟性があることが有益な場合があり、Microsoft Bot Framework がこれを可能にします。ユーザーからのメッセージを選択的にログに記録するには、アクティビティの json を調べて、"name" でフィルター処理します。たとえば、次の json では、メッセージが "Bot1" によって送信されました。

```json
{
  "type": "message",
  "timestamp": "2017-10-27T11:19:15.2025869Z",
  "localTimestamp": "2017-10-27T07:19:15.2140891-04:00",
  "serviceUrl": "http://localhost:9000/",
  "channelId": "emulator",
  "from": {
    "id": "56800324",
    "name": "Bot1"
  },
  "conversation": {
    "isGroup": false,
    "id": "8a684db8",
    "name": "Conv1"
  },
  "recipient": {
    "id": "2c1c7fa3",
    "name": "User1"
  },
  "membersAdded": [],
  "membersRemoved": [],
  "text": "You sent hi which was 2 characters",
  "attachments": [],
  "entities": [],
  "replyToId": "09df56eecd28457b87bba3e67f173b84"
}
```
json を調べた後、DebugActivityLogger の LogAsync メソッドを変更して、````activity.From.Name````プロパティをチェックする簡単な条件を組み込むことによって、ユーザー メッセージをフィルター処理できます。

````C#
public class DebugActivityLogger : IActivityLogger
{
        public async Task LogAsync(IActivity activity)
        {
           if (!activity.From.Name.Contains("Bot"))
            {
               Debug.WriteLine($"From:{activity.From.Id} - To:{activity.Recipient.Id} - Message:{activity.AsMessageActivity()?.Text}");
            }
        }
}
````


### [3_SQL_Logger](3_SQL_Logger.md) に進みましょう

[0_README](../0_README.md) に戻る