---
title: ラボ環境のセットアップ
lab:
  title: ラボ環境のセットアップ
  module: Setup
layout: default
nav_order: 2
parent: JavaScript SDK labs
---

# ローカル ラボ環境のセットアップ

これらのラボは、ホストされているラボ環境で実行することをお勧めします。 ご自分のコンピューターで実行する場合は、次のソフトウェアをインストールしてください。 独自の環境を使用すると、予期しないダイアログや動作が発生する場合があります。 さまざまなローカル構成が考えられるので、ご自分の環境で問題が発生しても、コース チームが問題をサポートすることはできません。

## Azure コマンドライン ツール

1. [Azure CLI](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest) または [Azure Cloud Shell](https://shell.azure.com): Azure portal ではなく CLI を使用してコマンドを実行する場合はインストールします。

## Node.js

1. [nodejs.org/en/download] から Node.js v18.0.0 以降をダウンロードしてインストールします。

1. [npmjs.com/get-npm] から NPM v10.2.3 以降をダウンロードしてインストールします。

最新バージョンの NPM と Node.js を Windows にインストールする推奨の方法:

- [github.com/coreybutler/nvm-windows] から NVM をインストールします
- nvm install latest を実行します
- nvm list を実行します (使用可能な NPM/Node.js のバージョンを確認する場合)
- nvm use latest を実行します (使用可能な最新バージョンを使用する場合)

### Git

1. [git-scm.com/downloads] からダウンロードしてインストールします。

    - インストーラーで既定のオプションを使用します。

### Visual Studio Code (および拡張機能)

1. [code.visualstudio.com/download] からダウンロードしてインストールします。

    - インストーラーで既定のオプションを使用します。

1. インストール後、Visual Studio Code を起動します。

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
[nodejs.org/en/download]: https://nodejs.org/en/download
[npmjs.com/get-npm]: https://npmjs.com/get-npm
[github.com/coreybutler/nvm-windows]: https://github.com/coreybutler/nvm-windows
