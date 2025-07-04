---
lab:
  title: ラボ環境のセットアップ
  module: Setup
---

# ローカル ラボ環境のセットアップ

これらのラボは、ホストされているラボ環境で実行することをお勧めします。 ご自分のコンピューターで実行する場合は、次のソフトウェアをインストールしてください。 独自の環境を使用すると、予期しないダイアログや動作が発生する場合があります。 さまざまなローカル構成が考えられるので、ご自分の環境で問題が発生しても、コース チームが問題をサポートすることはできません。

## Windows のインストール

> &#128221; 以下の手順は、Windows 10 コンピューター用です。 また、Linux または MacOS も使用できます。 選択した OS に対してラボの手順を調整する必要がある場合があります。

### Windows 10 (OS)

1. Windows 10 (*バージョン 2004 以降*) をインストールします。

1. すべての利用可能な更新プログラムを適用します。

### Edge

1. [microsoft.com/edge] から最新バージョンの Microsoft Edge をインストールします。

### .NET 8 SDK

1. [dotnet.microsoft.com/download/dotnet/8.0](https://dotnet.microsoft.com/en-us/download/dotnet/8.0) から SDK (ランタイムではない) をダウンロードしてインストールします。

### PowerShell 7

1. [github.com/powershell/powershell/releases] からダウンロードしてインストールします。

### Git

1. [git-scm.com/downloads] からダウンロードしてインストールします。

    - インストーラーで既定のオプションを使用します。

### Windows ターミナル

1. [github.com/microsoft/terminal/releases] からダウンロードしてインストールします。

1. **PowerShell** を既定のターミナルとして構成する

### Visual Studio Code (および拡張機能)

1. [code.visualstudio.com/download] からダウンロードしてインストールします。

    - インストーラーで既定のオプションを使用します。

1. インストール後、Visual Studio Code を起動します。

1. **[拡張機能]** メニューで、Microsoft の次の拡張機能を検索してインストールします。

    - [C#][marketplace.visualstudio.com/ms-dotnettools.csharp]

### Azure Cosmos DB Emulator

1. [docs.microsoft.com/azure/cosmos-db/local-emulator] からダウンロードしてインストールします。
    - インストーラーで既定のオプションを使用します。

[code.visualstudio.com/download]: https://code.visualstudio.com/download
[docs.microsoft.com/azure/cosmos-db/local-emulator]: https://docs.microsoft.com/azure/cosmos-db/local-emulator#download-the-emulator
[dotnet.microsoft.com/download/dotnet/6.0]: https://dotnet.microsoft.com/download/dotnet/6.0
[git-scm.com/downloads]: https://git-scm.com/downloads
[github.com/microsoft/terminal/releases]: https://github.com/microsoft/terminal/releases/latest
[github.com/powershell/powershell/releases]: https://github.com/powershell/powershell/releases/latest
[marketplace.visualstudio.com/ms-dotnettools.csharp]: https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp
[microsoft.com/edge]: https://microsoft.com/edge
