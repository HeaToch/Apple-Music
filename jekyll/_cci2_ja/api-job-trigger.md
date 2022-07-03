---
layout: classic-docs
title: "API を使用したジョブのトリガー"
short-title: "API を使用したジョブのトリガー"
description: "ビルド以外のジョブを定義およびトリガーする方法"
categories:
  - configuring-jobs
order: 80
---

CircleCI API を使用してジョブをトリガーする方法について説明します。

**メモ:** 現在のところ、API から 2.1 設定ファイルを使用するジョブをトリガーすることはできません。

- 目次
{:toc}

## 概要

[CircleCI API](https://circleci.com/docs/api/#trigger-a-new-job) を使用して、`.circleci/config.yml` で定義した[ジョブ]({{ site.baseurl }}/2.0/jobs-steps/#ジョブの概要)をトリガーします。

`curl` を使用して `deploy_docker` ジョブをトリガーする例を以下に示します。

```bash
curl -u ${CIRCLE_API_USER_TOKEN}: \
     -d build_parameters[CIRCLE_JOB]=deploy_docker \
     https://circleci.com/api/v1.1/project/<vcs-type>/<org>/<repo>/tree/<branch>
```

この例には以下の変数が使用されています。

- `CIRCLE_API_USER_TOKEN`: [パーソナル API トークン]({{ site.baseurl }}/2.0/managing-api-tokens/#パーソナル-api-トークンの作成).
- `<vcs-type>`: 選択された VCS (`github` または `bitbucket`) を示すプレースホルダー変数
- `<org>`: CircleCI 組織の名前を示すプレースホルダー変数
- `<repo>`: リポジトリの名前を示すプレースホルダー変数
- `<branch>`: ブランチの名前を示すプレースホルダー変数

API の関連情報については、[CircleCI API ドキュメント](https://circleci.com/docs/api/#section=reference)にまとめられています。

**API を通してジョブをトリガーする場合の重要な検討事項**

- API によってトリガーされるジョブに `workflows` セクションが含まれてもかまいません。
- ワークフローが、API によってトリガーされるジョブを参照する必要は**ありません**。
- API によってトリガーされたジョブは、特定の [CircleCI コンテキスト]({{ site.baseurl }}/2.0/contexts/)用に作成された環境変数にアクセス**できません**。
- 環境変数を使用する場合は、それらの環境変数が[プロジェクトレベル]({{ site.baseurl }}/2.0/env-vars/#プロジェクト内で環境変数を設定する)で定義されている必要があります。
- 現在のところ、CircleCI 2.1 とワークフローを使用する場合には、単一のジョブをトリガーすることができません。
- [プロジェクトのビルドをトリガーする](https://circleci.com/docs/api/#trigger-a-new-build-by-project-preview)エンドポイントを使用して、CircleCI API で[ワークフロー]({{ site.baseurl }}/2.0/workflows/)をトリガーできます。

## API を使用したジョブの条件付き実行

以下は、デプロイされるビルドに対してのみ `setup_remote_docker` で Docker イメージを構築する場合の設定ファイル例です。

```yaml
version: 2
jobs:
  build:
    docker:
      - image: ruby:2.4.0-jessie
        environment:
          LANG: C.UTF-8
    working_directory: /my-project
    parallelism: 2
    steps:
      - checkout

      - run: echo "run some tests"

      - deploy:
          name: デプロイ ジョブを条件付きで実行
          command: |
            # これをビルド・デプロイのチェックに置き換えます (すなわち、現在のブランチが "release")
            if [[ true ]]; then
              curl --user ${CIRCLE_API_USER_TOKEN}: \
                --data build_parameters[CIRCLE_JOB]=deploy_docker \
                --data revision=$CIRCLE_SHA1 \
                https://circleci.com/api/v1.1/project/github/$CIRCLE_PROJECT_USERNAME/$CIRCLE_PROJECT_REPONAME/tree/$CIRCLE_BRANCH
            fi

  deploy_docker:
    docker:

      - image: ruby:2.4.0-jessie
    working_directory: /
    steps:
      - setup_remote_docker
      - run: echo "deploy section running"
```

この例では以下の点にご留意ください。

- ビルド ジョブの `deploy` ステップを必ず使用してください。これを使用しないと、並列処理の値が N の場合に、N 回のビルドがトリガーされることがあります。
- API 呼び出しを `build_parameters[CIRCLE_JOB]=deploy_docker` で使用し、`deploy_docker` ジョブのみが実行されるようにします。

## 関連項目

[トリガー]({{ site.baseurl }}/2.0/triggers/)