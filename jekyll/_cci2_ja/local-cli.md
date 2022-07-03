---
layout: classic-docs
title: "CircleCI のローカル CLI の使用"
short-title: "CircleCI のローカル CLI の使用"
description: "CLI を使用したローカル ジョブの実行方法"
categories:
  - troubleshooting
order: 10
---

## 概要

CircleCI CLI は、CircleCI の高度で便利なツールの多くを、使い慣れた端末から利用できるコマンドライン インターフェイスです。 CircleCI CLI を使用すると、以下のような作業が行えます。

- CI の設定ファイルのデバッグとバリデーション 
- ローカルでのジョブの実行 
- CircleCI API のクエリ
- Orbs の作成、パブリッシュ、表示、管理

このドキュメントでは、CLI ツールのインストールと使用方法について説明します。 **メモ:** 現在、最新の CLI は、サーバー上にインストールした CircleCI では利用できません。 旧バージョンの CLI であればサーバー上でも動作するため、インストールが可能です。

- 目次
{:toc}

## インストール

CLI のインストールには複数の方法があります。

### クイック インストール

**メモ:** 2018 年 10 月以前に CLI をインストールしている場合は、新しい CLI に切り替えるために追加で作業を行う必要があります。 以下の[更新手順に関するセクション](#旧バージョンの-cli-の更新)を参照してください。

通常は以下のコマンドを実行すれば、CircleCI CLI を使用できるようになります。

**Mac と Linux の場合**

```sh
curl -fLSs https://circle.ci/cli | bash
```

CircleCI の CLI ツールは、デフォルトで `/usr/local/bin` ディレクトリにインストールされます。 `/usr/local/bin` への書き込みアクセス権を持っていない場合は、上記のコマンドを `sudo` で実行する必要があります。 または、bash の実行時に `DESTDIR` 環境変数を定義して、別の場所にインストールすることも可能です。

```sh
curl -fLSs https://circle.ci/cli | DESTDIR=/opt/bin bash
```

### その他のインストール方法

CircleCI CLI は、これ以外の方法でもインストールできます。 インストール内容をカスタマイズする必要がある場合や、上記のクイック インストールを実行して問題が発生した場合は、以降に紹介する方法をお試しください。

#### Snap を使用したインストール
{:.no_toc}

以下のコマンドを実行すると、CircleCI CLI、Docker と共に、[Snap パッケージ](https://snapcraft.io/)と付属のセキュリティおよび自動更新機能をインストールできます。

```sh
sudo snap install docker circleci
sudo snap connect circleci:docker docker
```

**メモ:** Snap パッケージを使用して CLI をインストールする場合、この docker コマンドでは、以前にインストールした Docker のバージョンではなく、Docker Snap が使用されます。 セキュリティ上の理由から、Snap パッケージは $HOME 内でしかファイルを読み書きできません。

#### Homebrew を使用したインストール
{:.no_toc}

macOS で [Homebrew](https://brew.sh/index_ja) を使用している場合は、以下のコマンドを使用して CLI をインストールできます。

```sh
brew install circleci
```

**メモ:** Mac 版の Docker を既にインストールしている場合は、`brew install --ignore-dependencies circleci` を使用してください。

### 手動でのダウンロード

CLI を手動でダウンロードしてインストールする場合は、[GitHub 上のリリース ページ](https://github.com/CircleCI-Public/circleci-cli/releases)をご確認ください。 システム上の特定のパスに CLI をインストールしたいときには、この方法が最適です。

## CLI の更新

`circleci update` コマンドを使用して、CLI を最新のバージョンに更新できます。 なお、更新の有無を手動で確認するだけでインストールを行わない場合は、`circleci update check` コマンドで確認できます。

### 旧バージョンの CLI の更新
{:.no_toc}

CLI の最新バージョンは [CircleCI パブリック オープン ソース プロジェクト](https://github.com/CircleCI-Public/circleci-cli)です。 [旧バージョンの CLI をインストールしている](https://github.com/circleci/local-cli)場合は、以下のコマンドを実行して更新を行い、最新バージョンの CLI に切り替えてください。

```sh
circleci update 
circleci switch
```

インストール ディレクトリ `/usr/local/bin` への書き込みアクセス権が付与されていないユーザーがこのコマンドを実行すると、`sudo` を使用するように求められます。

## CLI の構成

CLI を使用する前に、[[Personal API Token (パーソナル API トークン)] タブ](https://circleci.com/account/api)から CircleCI の API トークンを生成する必要があります。 トークンを取得したら、以下を実行して CLI を構成します。

```sh
circleci setup
```

setup を実行すると、構成を行うように求められます。 circleci.com で CLI を使用している場合は、デフォルトの CircleCI Host を使用します。 独自のサーバーまたはプライベート クラウドにインストールした CircleCI を使用している場合は、インストール アドレスの値を変更します (例: circleci.your-org.com)。

## CircleCI の設定ファイルのバリデーション

CLI を使用して設定ファイルをローカルでバリデーションすると、config.yml をテストするたびに追加のコミットをプッシュする必要がなくなります。

設定ファイルをバリデーションするには、`.circleci/config.yml` ファイルがあるディレクトリに移動し、以下を実行します。

```sh
circleci config validate
# 設定ファイル .circleci/config.yml が有効かどうかチェックします
```

[Orbs](https://circleci.com/orbs/) を使用している場合は、それもバリデーションできます。

```sh
circleci orb validate /tmp/my_orb.yml
```

上記のコマンドは、コマンドを実行したディレクトリの `/tmp` フォルダーから `my_orb.yml` という Orb を検索します。

## 設定ファイルのパッケージ化

CLI の `pack` コマンドを使用すると、複数のファイルをまとめて 1 つの YAML ファイルを作成できます。 `pack` コマンドには、ディレクトリ ツリー内の複数ファイルにまたがる YAML ドキュメントを解析する [FYAML](https://github.com/CircleCI-Public/fyaml) が実装されています。 これは、容量の大きな Orbs のソース コードを分割している場合に特に利便性が高く、Orbs の YAML 構成のカスタム編成を行うことができます。 `circleci config pack` は、ディレクトリ構造とファイルの内容に基づいて、ファイル システム ツリーを 1 つの YAML ファイルに変換します。 `pack` コマンドを使用するときのファイルの**名前**や**編成**に応じて、最終的にどのような `orb.yml` が出力されるかが決まります。 以下のフォルダー構造を例に考えます。

```sh
$ tree
.
└── your-orb-source
    ├── @orb.yml
    ├── commands
    │   └── foo.yml
    └── jobs
        └── bar.yml

3 directories, 3 files
```

Unix の `tree` コマンドは、フォルダー構造の出力にたいへん便利です。 上記のツリー構造の例の場合、`pack` コマンドは、フォルダー名とファイル名を **YAML キー** にマップし、ファイルの内容を**値**として対応するキーにマップします。 上記の例のフォルダーを `pack` してみましょう。


{% raw %}
```sh
$ circleci config pack your-orb-source
```

```yaml
# ここに @orb.yml の内容が表示されます
commands:
  foo:
    # ここに foo.yml の内容が表示されます
jobs:
  bar:
    # ここに bar.yml の内容が表示されます
```
{% endraw %}

### その他の設定ファイル パッケージ化機能
{:.no_toc}

`@` で始まるファイルの内容は、その親フォルダーのレベルにマージされます。 この機能は、汎用的な `orb.yml` にメタデータを格納したいものの、`orb` のキー・値のペアにはマップしたくない場合に、トップレベルの Orb で使用すると便利です。

たとえば、以下のような場合

{% raw %}
```sh
$ cat foo/bar/@baz.yml
{baz: qux}
```
{% endraw %}

次のようにマップされます。

```yaml
bar: 
  baz: qux
```

### パッケージ化された Config.yml の例
{:.no_toc}

複数の YAML ソース ファイルを使用して記述した Orb の例については、[GitHub の CircleCI Orbs トピック タグ](https://github.com/search?q=topic%3Acircleci-orbs+org%3ACircleCI-Public&type=Repositories)を参照してください。 `circleci config pack` は通常、Orb ソース コードをパブリッシュできるように準備するときに、プロジェクトの CI/CD ワークフローの一部として実行します。

## 設定ファイルの処理

`circleci config process` を実行すると設定ファイルがバリデーションされますが、同時に、展開されたソースが元の設定ファイルの内容と共に表示されます (Orb を使用している場合に便利)。

`hello-build` Orb を使用する設定ファイルを例に考えます。

    version: 2.1
    
    orbs:
        hello: circleci/hello-build@0.0.5
    
    workflows:
        "Hello Workflow":
            jobs:
    
              - hello/hello-build
    

`circleci config process .circleci/config.yml` を実行すると、以下のように出力されます (これは、展開されたソースとコメントアウトされた元の設定ファイルから成ります)。

{% raw %}
```sh
# 'circleci/hello-build@0.0.5' に解決された Orb 'circleci/hello-build@0.0.5'
version: 2
jobs:
  hello/hello-build:
    docker:

    - image: circleci/buildpack-deps:curl-browsers
    steps:
    - run:
        command: echo "Hello ${CIRCLE_USERNAME}"
    - run:
        command: |-
          echo "TRIGGERER: ${CIRCLE_USERNAME}"
          echo "BUILD_NUMBER: ${CIRCLE_BUILD_NUM}"
          echo "BUILD_URL: ${CIRCLE_BUILD_URL}"
          echo "BRANCH: ${CIRCLE_BRANCH}"
          echo "RUNNING JOB: ${CIRCLE_JOB}"
          echo "JOB PARALLELISM: ${CIRCLE_NODE_TOTAL}"
          echo "CIRCLE_REPOSITORY_URL: ${CIRCLE_REPOSITORY_URL}"
        name: 一部の CircleCI ランタイム環境変数の表示
    - run:
        command: |-
          echo "uname:" $(uname -a)
          echo "arch: " $(arch)
        name: システム情報の表示
workflows:
  Hello Workflow:
    jobs:
    - hello/hello-build
  version: 2

# Original config.yml file:
# version: 2.1
# 
# orbs:
#     hello: circleci/hello-build@0.0.5
# 
# workflows:
#     \"Hello Workflow\":
#         jobs:
#           - hello/hello-build

```
{% endraw %}

## マシン上のコンテナ内でのジョブの実行

### 概要
{:.no_toc}

CLI を使用すると、Docker から設定ファイル内のジョブを実行できます。 そうすれば、テストを実行した後で、設定ファイルの変更をプッシュしたり、ビルド キューに影響を与えずにビルド プロセスをデバッグできたりするため、便利です。

### 前提条件
{:.no_toc}

システムに [Docker](https://www.docker.com/products/docker-desktop) と CLI ツールの最新バージョンをインストールしている必要があります。 また、有効な `.circleci/config.yml` ファイルを持つプロジェクトが必要です。

### ジョブの実行
{:.no_toc}

CLI では、Docker を使用してデスクトップ上の CircleCI から単一のジョブを実行できます。

```sh
$ circleci local execute --job JOB_NAME
```

CircleCI の設定ファイルをバージョン 2.1 以上に設定している場合、ジョブを実行するには、まず設定ファイルを `process.yml` にエクスポートしてから、そのファイルを指定する必要があります。

```sh
circleci config process .circleci/config.yml > process.yml
circleci local execute -c process.yml --job JOB_NAME
```

CircleCI のデモ アプリケーションを使って、ローカルのマシンからビルドのサンプルを実行してみましょう。

```sh
git clone https://github.com/CircleCI-Public/circleci-demo-go.git
cd circleci-demo-go
circleci local execute --job build
```

上記のコマンドは、*build* ジョブ全体を実行します (ローカルではジョブのみを実行でき、ワークフローは実行できません)。 CLI は、Docker を使用してビルドの要件をプル ダウンしてから、CI ステップをローカルで実行します。 この例では、Golang および Postgres の Docker イメージをプル ダウンして、ビルド中に依存関係のインストール、単体テストの実行、サービスの実行テストなどを行えるようにしています。

### ローカルでのジョブ実行時の制限事項
{:.no_toc}

`circleci` を使用してジョブをローカルで実行できるのは非常に便利ですが、いくつかの制限事項があります。

**Machine Executor**

ローカル ジョブでは Machine Executor を使用できません。 Machine Executor でジョブを実行するには、別の VM が必要になるためです。

**SSH キーの追加**

現時点では、CLI コマンド `add_ssh_keys` を使用して SSH キーを追加することはできません。

**ワークフロー**

CLI ツールでは、ワークフローの実行がサポートされていません。 基本的にワークフローは、複数のマシンで行うビルドの並列実行を活用することによって、高速で複雑なビルドを可能にします。 CLI はユーザーのマシンでのみ動作するため、単一の **jobs** (ワークフローを構成する一要素) しか実行できません。

**キャッシュとオンライン限定コマンド**

現在、ローカル ジョブではキャッシュがサポートされていません。 設定ファイルに `save_cache` ステップまたは `restore_cache` ステップが含まれている場合、`circleci` では該当のステップがスキップされ、警告が表示されます。

また、オンラインでは機能しても、ローカル マシンでは機能しないコマンドもあります。 たとえば、上記の Golang ビルドの例では `store_artifacts` ステップを実行していますが、ローカルでビルドした場合、アーティファクトはアップロードされません。 ローカルのビルドで利用できないステップがあった場合は、コンソールにエラーが表示されます。

**環境変数**

セキュリティ上の理由から、UI で構成した暗号化環境変数は、ローカルのビルドにはインポートされません。 代わりに、`-e` フラグを使用して CLI に環境変数を指定できます。 詳細については、`circleci help build` の出力結果を参照してください。 なお、環境変数を複数指定する場合は、このフラグを変数ごとに使用する必要があります (例: `circleci build -e VAR1=FOO -e VAR2=BAR`)。

## テストの分割

CircleCI CLI は、ジョブ実行中の一部の高度な機能、たとえばビルド時間最適化のための[テストの分割](https://circleci.com/ja/docs/2.0/parallelism-faster-jobs/#circleci-cli-を使用したテストの分割)にも使用できます。

## CircleCI Server での CLI の使用

現在、サーバーにインストールした CircleCI 上で実行できるのは、旧バージョンの CircleCI CLI のみです。 macOS や他の Linux ディストリビューションに旧バージョンの CLI をインストールする場合は、以下の手順を実施します。

1. [Docker のインストール手順](https://docs.docker.com/install/)に従って、Docker をインストールし、構成します。
2. 以下のコマンドを実行して、CLI をインストールします。

`$ curl -o /usr/local/bin/circleci https://circle-downloads.s3.amazonaws.com/releases/build_agent_wrapper/circleci && chmod +x /usr/local/bin/circleci`

CLI (`circleci`) は `/usr/local/bin` ディレクトリにダウンロードされます。 `/usr/local/bin` への書き込みアクセス権を持っていない場合は、上記のコマンドを `sudo` で実行する必要があります。 CLI は更新の有無を自動的に確認し、更新があった場合はメッセージが表示されます。

## アンインストール

CircleCI CLI のアンインストールに使用するコマンドは、インストール方法によって異なります。

- **curl インストール コマンドを使用した場合**: `usr/local/bin` から `circleci` 実行可能ファイルを削除します。
- **Mac で Homebrew を使用してインストールした場合**: `brew uninstall circleci` を実行します。
- **Linux で Snap を使用してインストールした場合**: `sudo snap remove circleci` を実行します。