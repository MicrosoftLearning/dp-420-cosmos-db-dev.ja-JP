---
ラボ: タイトル: "ラボ環境のセットアップ" モジュール: "セットアップ"
---

# ローカル ラボ環境のセットアップ

これらのラボは、ホストされているラボ環境で実行することをお勧めします。 ご自分のコンピューターで実行する場合は、次のソフトウェアをインストールしてください。 独自の環境を使用すると、予期しないダイアログや動作が発生する場合があります。 さまざまなローカル構成が考えられるので、ご自分の環境で問題が発生しても、コース チームが問題をサポートすることはできません。

## Azure コマンドライン ツール

1. [Azure CLI](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest) または [Azure Cloud Shell](https://shell.azure.com): Azure portal ではなく CLI を使用してコマンドを実行する場合はインストールします。

## Python

1. \<install location\>\Python36 と \<install location>\Python36\Scripts を PATH に追加して、[python.org/downloads]から Python 3.11 以降をダウンロードしてインストールします。

    - インストーラーで既定のオプションを使用します。

## Git

1. [git-scm.com/downloads] からダウンロードしてインストールします。

    - インストーラーで既定のオプションを使用します。

## Visual Studio Code (および拡張機能)

1. [code.visualstudio.com/download] からダウンロードしてインストールします。

    - インストーラーで既定のオプションを使用します。

1. インストール後、Visual Studio Code を起動します。

1. **[拡張機能]** メニューで、Microsoft の次の拡張機能を検索してインストールします。

    - [Visual Studio Code 用の Python 拡張機能][marketplace.visualstudio.com/mms-python.python]

### Azure Cosmos DB Emulator

1. [docs.microsoft.com/azure/cosmos-db/local-emulator] からダウンロードしてインストールします。
    - インストーラーで既定のオプションを使用します。

### ラボ リポジトリをクローンする

このラボで作業している環境に「**Azure Cosmos DB を使用してコパイロットを構築する**」 のラボ コード リポジトリをまだクローンしていない場合は、次の手順に従ってクローンします。 それ以外の場合は、以前にクローンしたフォルダーを **Visual Studio Code** で開きます。

1. **Visual Studio Code** を起動します。

    > &#128221; Visual Studio Code インターフェイスについてまだよく理解していない場合は、[Visual Studio Code の入門ガイド][code.visualstudio.com/docs/getstarted]を参照してください

1. コマンド パレットを開き、**Git: Clone** を実行して、任意のローカル フォルダーに ``https://github.com/solliancenet/microsoft-learning-path-build-copilots-with-cosmos-db-labs`` GitHub リポジトリをクローンします。

    > &#128161; **Ctrl + Shift + P** キーボード ショートカットを使用してコマンド パレットを開くことができます。

1. リポジトリが複製されたら、**Visual Studio Code** で選択したローカル フォルダーを開きます。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/azure/cosmos-db/local-emulator]: https://docs.microsoft.com/azure/cosmos-db/local-emulator#download-the-emulator
[code.visualstudio.com/download]: https://code.visualstudio.com/download
[git-scm.com/downloads]: https://git-scm.com/downloads
[python.org/downloads]: https://www.python.org/downloads/
[marketplace.visualstudio.com/mms-python.python]: https://marketplace.visualstudio.com/items?itemName=ms-python.python#overview
