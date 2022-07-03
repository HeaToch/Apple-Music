---
layout: classic-docs
title: "Linux プロジェクトの 1.0 から 2.0 への移行"
short-title: "Linux プロジェクトの 1.0 から 2.0 への移行"
description: "CircleCI 1.0 から 2.0 に移行する理由と方法"
categories:
  - migration
order: 15
version:
  - Server v2.x
---

このドキュメントでは、CircleCI 1.0 を 2.0 に移行する際に最初に行う作業について説明します。移行作業ではまず、既存の 1.0 の設定ファイルをコピーして利用し、古いキーに対応する新しいキーがある場合はキーを置き換えます。

- 目次
{:toc}

ここで説明する手順だけでは移行プロセスは完了しないこともありますが、このドキュメントの目的は、大部分のキーを同等のネスト構文に置き換えてから新しい機能を追加できるよう支援することです。

`circle.yml` ファイルがない場合は、最初から [2.0 `config.yml` のサンプル ファイル]({{ site.baseurl }}/2.0/sample-config)を参考にしてください。

## 概要
{:.no_toc}

CircleCI では、[`.circleci/config.yml`]({{ site.baseurl }}/2.0/configuration-reference) を作成して新しい必須キーを追加し、それらのキーに値を定義する必要があります。 **メモ:** 並列処理は `.circleci/config.yml` ファイルでのみ設定できます。CircleCI アプリケーションでの並列処理設定は無視されます。

既に `circle.yml` ファイルがある場合は、以降の各セクションの手順に従って、既存のファイルをコピーし、新しい必須キーを記述し、1.0 のキーを検索して 2.0 のキーに置き換えます。

### Using the 1.0 to 2.0 `config-translation` endpoint
{:.no_toc}

`config-translation` エンドポイントを使用すると、1.0 の設定ファイルから 2.0 の設定ファイルへの変換をすぐに始めることができます。詳細については、「[1.0 から 2.0 への config-translation エンドポイントを使用する]({{ site.baseurl }}/2.0/config-translation)」を参照してください。

## Steps to configure required keys

1. 既存の `circle.yml` ファイルをコピーして、プロジェクト リポジトリのルートにある新しい `.circleci` ディレクトリに置きます。

2. `.circleci/circle.yml` の名前を `.circleci/config.yml` に変更します。

3. `.circleci/config.yml` ファイルの先頭に、`version: 2` を記述します。

4. `config.yml` ファイルのバージョンを指定する行の後に、以下の 2 行を追加します。 既存の構成内容に `machine:` を記述していた場合は、`machine:` を以下の 2 行に置き換え、古い設定ファイルのすべてのセクションを `build` の下にネストします。

     ```
     jobs:
       build:
     ```

5. `docker:` キーと `- image:` キーを記述するか、`machine: true` を設定するか、`macos` を指定して、プライマリ コンテナを実行するときの言語とバージョンを追加します。 以下の `ruby:` の例のように、構成に言語とバージョンが含まれている場合は、修正が必要です。

     ```
       ruby:
         version: 2.3
     ```

     Replace with the following lines:
    

     ```
         docker:
           - image: circleci/ruby:2.3-jessie
             auth:
               username: mydockerhub-user
               password: $DOCKERHUB_PASSWORD  # context / project UI env-var reference
     ```

     最初に記述したイメージのインスタンスがプライマリ コンテナになります。 ジョブのコマンドはこのコンテナ内で実行されます。ジョブごとにコマンドを宣言します。 Docker コンテナを初めて使用する場合は、「[Docker 入門](https://docs.docker.com/get-started/#docker-concepts)」を参照してください。 
    

     ```yaml
         machine: true
     ```

     使用可能な VM イメージの詳細については、「Executor タイプを選択する」の「[Machine の使用](https://circleci.com/ja/docs/2.0/executor-types/#machine-の使用)」を参照してください。
    

     ```yaml
         macos: 
           xcode: 11.3.0
     ```

6. ソース ファイルに対してジョブを実行するには、`checkout:` ステップが必要です。 `steps:` の下に `checkout:` をネストして各ジョブを記述します。それには、以下のコードを検索します。

     ```
     checkout:
       post:
     ```

     上記を以下のように置き換えます。
    

     ```
         steps:
           - checkout
           - run:
     ```

     以下に例を示します。
    

     ```
     checkout:
      post:

        - mkdir -p /tmp/test-data
        - echo "foo" > /tmp/test-data/foo
     ```

     上の例は次のようになります。
    

     ```
         steps:
           - checkout
           - run: mkdir -p /tmp/test-data
           - run: echo "foo" > /tmp/test-data/foo
     ```

     `checkout` ステップを使わない場合でも、このステップを `config.yml` ファイルに追加する必要があります。
    

7. (オプション) ビルドへの SSH 接続を有効化するには、`add_ssh_keys` ステップとフィンガープリントを追加します。詳細については、「[CircleCI を設定する]({{ site.baseurl }}/2.0/configuration-reference/#add_ssh_keys)」を参照してください。

8. <http://codebeautify.org/yaml-validator> で YAML をバリデーションして、変更をチェックします。

## Environment variables

In CircleCI 2.0, all defined environment variables are treated literally. It is possible to interpolate variables within a command by setting it for the current shell.

For more information, refer to the CircleCI 2.0 document [Using Environment Variables]({{ site.baseurl }}/2.0/env-vars/).

## Steps to configure workflows

To increase the speed of your software development through faster feedback, shorter re-runs, and more efficient use of resources, configure workflows using the following instructions:

1. ワークフロー機能を使用するには、ビルド ジョブを複数のジョブに分割し、それぞれに一意の名前を付けます。 デプロイのみが失敗したときにビルド全体を再実行しなくても済むように、最初はデプロイ ジョブのみを分割するだけでもよいでしょう。

2. ベスト プラクティスとしては、`workflows:`、`version: 2`、`<ワークフロー名>` の各行をマスター `.circleci/config.yml` ファイルの*末尾*に追加します。`<ワークフロー名>` は、実際のワークフローの一意の名前に置き換えてください。 **メモ:** `config.yml` ファイルのワークフロー セクションは、構成の中にネストしません。ワークフロー セクションの `version: 2` は `config.yml` ファイルの先頭にある `version:` キーとは別に設定するものであるため、ワークフロー セクションはファイルの末尾に置くのが最善です。

     ```
     workflows:
       version: 2
       <ワークフロー名>:
     ```

3. `<ワークフロー名>` の下に `jobs:` キーの行を追加し、オーケストレーションするジョブ名をすべて記述します。 In this example, `build` and `test` will run concurrently.

     ```
     workflows:
       version: 2
       <ワークフロー名>:
           jobs:
             - build
             - test
     ```

4. 別のジョブが成功したかどうかに応じて順次実行する必要があるジョブについては、`requires:` キーを追加し、そのジョブを開始するために成功する必要があるジョブをその下にネストします。 `curl` コマンドを使用してジョブを開始していた場合、ワークフロー セクションでは、そのコマンドを削除して `requires:` キーでジョブを開始できます。

     ```
      - <ジョブ名>:
          requires:
            - <ジョブ名>
     ```

5. 特定のブランチでジョブを実行する必要がある場合は、`filters:` キーを追加し、その下に `branches` と `only` キーをネストします。 ジョブを特定のブランチで実行してはいけない場合は、`filters:` キーを追加し、その下に `branches` と `ignore` キーをネストします。 **メモ:** ワークフローは、ジョブ セクションに記述してあるブランチ指定を無視します。したがって、ジョブでブランチを指定していて、後からワークフローを追加する場合は、ジョブ セクションのブランチ指定を削除し、`config.yml` のワークフロー セクションで以下のようにブランチを宣言する必要があります。

     ```
     - <ジョブ名>:
         filters:
           branches:
             only: master
     - <ジョブ名>:
         filters:
           branches:
             ignore: master
     ```

6. <http://codebeautify.org/yaml-validator> で再度 YAML をバリデーションして、形式が正しいことをチェックします。

## Search and replace deprecated 2.0 keys

- 構成にタイムゾーンが含まれる場合は、`timezone: America/Los_Angeles` を検索して、以下の 2 行に置き換えます。

```yaml
    environment:
      TZ: "America/Los_Angeles"
```

- $PATH を変更している場合は、`.bashrc` ファイルにパスを追加し、以下の部分を 

```yaml
    environment:
      PATH: "/path/to/foo/bin:$PATH"
```

With the following to load it into your shell (the file $BASH_ENV already exists and has a random name in /tmp):

```yaml
    steps:
      - run: echo 'export PATH=/path/to/foo/bin:$PATH' >> $BASH_ENV 
      - run: some_program_inside_bin
```

- `hosts:` キーを検索し、たとえば以下のような部分を

```yaml
hosts:
    circlehost: 127.0.0.1
```

With an appropriate `run` Step, for example:

```yaml
    steps:
      - run: echo 127.0.0.1 circlehost | sudo tee -a /etc/hosts
```

- `dependencies:`、`database`、または `test` と `override:` の行を検索し、たとえば以下のような部分は

```yaml
dependencies:
  override:
    - <インストール済み依存関係>
```

Is replaced with:

```yaml
      - run:
          name: <名前>
          command: <インストール済み依存関係>
```

- `cache_directories:` キーを検索し、

```yaml
  cache_directories:
    - "vendor/bundle"
```

With the following, nested under `steps:` and customizing for your application as appropriate:

```yaml
     - save_cache:
        key: dependency-cache
        paths:
          - vendor/bundle
```

- `restore_cache:` キーを `steps:` の下にネストして追加します。

- `deployment:` を検索し、次のように置き換えて `steps` の下にネストします。

```yaml
     - deploy:
```

## YAML のバリデーション

When you have all the sections in `.circleci/config.yml` we recommend that you check that your YAML syntax is well-formed using a tool such as <http://codebeautify.org/yaml-validator>. Then, use the `circleci` CLI to validate that the new configuration is correct with regard to the CircleCI 2.0 schema. See the [Using the CircleCI Command Line Interface (CLI)]({{ site.baseurl }}/2.0/local-jobs/) document for instructions. Fix up any issues and commit the updated `.circleci/config.yml` file. When you push a commit the job will start automatically and you can monitor it in the CircleCI app.

## 次のステップ
{:.no_toc}

- 「[2.0 への移行のヒント]({{ site.baseurl }}/2.0/migration/)」を参照してください。
- デプロイの構成例については、「[デプロイの構成]({{ site.baseurl }}/2.0/deployment-integrations/)」を参照してください。
- CircleCI 2.0 の Docker イメージおよび machine イメージの詳細については、「[Executor タイプを選択する]({{ site.baseurl }}/2.0/executor-types/)」を参照してください。
- CircleCI 2.0 の `jobs` と `steps` の正確な構文と使用可能なすべてのオプションについては、「[CircleCI を設定する]({{ site.baseurl }}/2.0/configuration-reference/)」を参照してください。