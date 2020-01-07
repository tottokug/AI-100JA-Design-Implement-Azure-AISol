# PictureBot

ボット フレームワーク v4 エコー ボットのサンプル。

このボットは、[Bot Framework](https://dev.botframework.com) を使用して作成されており、ユーザーからの入力を受け取ってエコー バックする単純なボットを作成する方法を示します。

## 前提条件

- [.NET Core SDK](https://dotnet.microsoft.com/download) バージョン 2.1

```bash
  # .NET バージョンの確認
  dotnet --version
```

## これサンプルを試します

- ターミナルで、`PictureBot` に移動します。

```bash
    # プロジェクトフォルダへの変更
    cd # PictureBot
```

- ターミナルまたは Visual Studio からボットを実行するには、オプション A または B を選択します。

  A) 端末から

```bash
  # ボットを実行します
  dotnet run
```

  B) または Visual Studio から

  - Visual Studio を起動する
  - ファイル -> 開く -> プロジェクト/ソリューション
  - `PictureBot` フォルダに移動します。
  - `PictureBot.csproj` ファイルを選択する
  - `F5` キーを押してプロジェクトを実行します。

## Bot Framework Emulator を使用したボットのテスト

[Bot Framework Emulator](https://github.com/microsoft/botframework-emulator) は、ローカル ホストまたはトンネルを介してリモートで実行しているボットをテストおよびデバッグできるようにするデスクトップ アプリケーションです。

- [ここ](https://github.com/Microsoft/BotFramework-Emulator/releases)をクリックして、Bot Framework Emulator バージョン 4.5.0 以降をインストールします。

### Bot Framework Emulator を使用してボットに接続する

- Bot Framework Emulator を起動する
- ファイル -> ボットを開く
- `http://localhost:3978/api/messages` のボット URL を入力する

## ボットを Azure にデプロイする

Azure へのボットのデプロイの詳細については、「[Azure へのボットのデプロイ](https://aka.ms/azuredeployment)」を参照して、デプロイ手順の完全な一覧を参照してください。

## その他の資料

- [Bot Framework のドキュメント](https://docs.botframework.com)
- [ボットの基本](https://docs.microsoft.com/azure/bot-service/bot-builder-basics?view=azure-bot-service-4.0)
- [アクティビティ処理](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-builder-concept-activity-processing?view=azure-bot-service-4.0)
- [Azure Bot Services の概要](https://docs.microsoft.com/azure/bot-service/bot-service-overview-introduction?view=azure-bot-service-4.0)
- [Azure Bot Services のドキュメント](https://docs.microsoft.com/azure/bot-service/?view=azure-bot-service-4.0)
- [.NET Core CLI ツール](https://docs.microsoft.com/ja-jp/dotnet/core/tools/?tabs=netcore2x)
- [Azure CLI](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest)
- [Azure Portal](https://portal.azure.com)
- [LUIS を使用した Language Understanding](https://docs.microsoft.com/ja-jp/azure/cognitive-services/luis/)
- [チャネルとボット コネクタ サービス](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-concepts?view=azure-bot-service-4.0)
