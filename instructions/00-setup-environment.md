---
lab:
  title: ラボ環境のセットアップ
  module: Setup
ms.openlocfilehash: 6beb401fed4863929ed8772e2e8b3dff4b61434b
ms.sourcegitcommit: b90234424e5cfa18d9873dac71fcd636c8ff1bef
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/12/2022
ms.locfileid: "138025064"
---
# <a name="setup-local-lab-environment"></a>ローカル ラボ環境のセットアップ

ホストされているラボ環境でこれらのラボを完了するのが理想的です。 自分のコンピューターで完了したい場合は、次のソフトウェアをインストールします。 独自の環境を使用すると、予期しないダイアログや動作が発生する可能性があります。 さまざまなローカル構成が考えられるので、コース チームは、独自の環境で発生する可能性がある問題をサポートできません。

## <a name="windows-installation"></a>Windows のインストール

> &#128221; 以下の手順は、Windows 10 コンピューター用です。 また、Linux または MacOS も使用できます。 選択した OS に対してラボの手順を調整する必要がある場合があります。

### <a name="windows-10-os"></a>Windows 10 (OS)

1. Windows 10 (*バージョン 2004 以降*) をインストールします。

1. すべての利用可能な更新プログラムを適用します。

### <a name="edge"></a>Edge

1. [microsoft.com/edge] から最新バージョンの Microsoft Edge をインストールします。

### <a name="net-6-sdk"></a>.NET 6 SDK

1. [dotnet.microsoft.com/download/dotnet/6.0] から SDK (ランタイムではない) をダウンロードしてインストールします。

### <a name="powershell-7"></a>PowerShell 7

1. [github.com/powershell/powershell/releases] からダウンロードしてインストールします。

### <a name="git"></a>Git

1. [git-scm.com/downloads] からダウンロードしてインストールします。

    - インストーラーで既定のオプションを使用します。

### <a name="windows-terminal"></a>Windows ターミナル

1. [github.com/microsoft/terminal/releases] からダウンロードしてインストールします。

1. **PowerShell** を既定のターミナルとして構成する

### <a name="visual-studio-code-and-extensions"></a>Visual Studio Code (および拡張機能)

1. [code.visualstudio.com/download] からダウンロードしてインストールします。

    - インストーラーで既定のオプションを使用します。

1. インストール後、Visual Studio Code を起動します。

1. **[拡張機能]** メニューで、Microsoft の次の拡張機能を検索してインストールします。

    - [C#][marketplace.visualstudio.com/ms-dotnettools.csharp]

### <a name="azure-cosmos-db-emulator"></a>Azure Cosmos DB Emulator

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
