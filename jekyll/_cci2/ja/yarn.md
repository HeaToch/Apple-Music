---
layout: classic-docs
title: "CircleCI での Yarn (npm の代替) の使用"
short-title: "Yarn パッケージ マネージャー"
categories:
  - how-to
description: "CircleCI での Yarn パッケージ マネージャーの使用方法"
version:
  - Cloud
  - Server v2.x
---

[Yarn](https://yarnpkg.com/ja/) は、JavaScript 用のオープンソース パッケージ マネージャーです。 Yarn によってインストールされるパッケージはキャッシュできるため、 ビルドを高速化できるだけでなく、さらに重要なメリットとして、ネットワーク接続に関するエラーも低減できます。

## CircleCI での Yarn の使用方法

[`docker` Executor](https://circleci.com/ja/docs/2.0/executor-types/#docker-を使用する) を使用している場合は、ビルド環境に既に Yarn がインストールされている可能性があります。 [CircleCI が提供しているビルド済み Docker イメージ](https://circleci.com/ja/docs/2.0/circleci-images/)では、Node.js イメージ (`circleci/node`) に Yarn がプリインストールされています。 If you are using one of the other language images such as `circleci/python` or `circleci/ruby`, there are two [image variants](https://circleci.com/docs/2.0/circleci-images/#language-image-variants) that will include Yarn as well as NodeJS. `-node` と `-node-browsers` のイメージ バリアントです。 たとえば、Docker イメージ `circleci/python:3-node` を使用すると、Yarn と Node.js がインストールされた Python ビルド環境が提供されます。

独自の Docker イメージ ベース、または `macos`、`windows`、`machine` の Executor を使用している場合は、[Yarn の公式ドキュメント](https://yarnpkg.com/lang/ja/docs/install/)の手順に従って Yarn をインストールできます。 Yarn ドキュメントには、マシン環境別のインストール手順が記載されています。 たとえば Unix 系の環境にインストールする場合には、以下の curl コマンドを使用します。

```sh
curl -o- -L https://yarnpkg.com/install.sh | bash
```

## キャッシュ

Yarn packages can be cached to improve CI build times.

An example for Yarn 2:

{% raw %}
```yaml
#...

      - restore_cache:
          name: Restore Yarn Package Cache
          keys:
            - yarn-packages-{{ checksum "yarn.lock" }}
      - run:
          name: Install Dependencies
          command: yarn install --immutable
      - save_cache:
          name: Save Yarn Package Cache
          key: yarn-packages-{{ checksum "yarn.lock" }}
          paths:
            - ~/.cache/yarn
#...
```
{% endraw %}

An example for Yarn 1.x:

{% raw %}
```yaml
#...

      - restore_cache:
          name: Restore Yarn Package Cache
          keys:
            - yarn-packages-{{ checksum "yarn.lock" }}
      - run:
          name: Install Dependencies
          command: yarn install --frozen-lockfile --cache-folder ~/.cache/yarn
      - save_cache:
          name: Save Yarn Package Cache
          key: yarn-packages-{{ checksum "yarn.lock" }}
          paths:
            - ~/.cache/yarn
#...
```
{% endraw %}

## See also

[Caching Dependencies]({{ site.baseurl }}/2.0/caching/)