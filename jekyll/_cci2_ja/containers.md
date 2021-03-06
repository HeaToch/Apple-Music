---
layout: classic-docs
title: コンテナを使用する
categories:
  - how-to
description: CircleCI コンテナの使用方法
version:
  - Cloud
---

また、お使いのプランで提供されるコンテナを活用してジョブやワークフローの実行を高速化する方法についても解説します。 なお、CircleCI には新たに[従量課金制プラン](https://circleci.com/ja/pricing/)が導入されており、コンテナベースのプランはまもなく廃止となりますのでご注意ください。 If your builds are queuing with a Container-based plan, consider signing up for a CircleCI usage-based plan to mitigate queuing and to enjoy other benefits.

新しい従量課金制プランの詳細については、[こちらのドキュメント]({{ site.baseurl }}/ja/2.0/credits/)を参照してください。

## 概要
{: #overview }

バージョン管理システムに変更がコミットされると、CircleCI はコードをチェック アウトし、独立した新しいコンテナをオンデマンドに用意して、その中でワークフローのジョブを実行します。 このとき、該当するプランでは以下の処理が可能です。

- **同時処理** - 複数のコンテナを使用して、複数のビルドを同時に実行できます。 同時処理を行うには、[ワークフローのドキュメント]({{ site.baseurl }}/ja/2.0/workflows/)を参考に開発ワークフローを構成し、[2.0 設定ファイルのサンプル]({{ site.baseurl }}/ja/2.0/sample-config/#並列ジョブの構成例)に示す方法でジョブを並列実行してください。

- **並列処理** - テストを複数のコンテナに分割することで、テスト全体のスピードを大幅に向上できます。 テストを並列で実行するには、[CircleCI の構成に関するドキュメント]({{ site.baseurl }}/ja/2.0/configuration-reference/#parallelism)で説明されているように `.circleci/config.yml` ファイルを変更します。 設定ファイルを変更してテストの分割と並列処理を行い、ビルド時間を短縮する方法については、[テストの並列実行に関するドキュメント]({{ site.baseurl }}/ja/2.0/parallelism-faster-jobs/)をご覧ください。

## はじめよう
{: #getting-started }

無料の Linux プランでは、同時に実行できるワークフローは 1 つのみで、並列処理は行えません。 追加のコンテナが必要な場合は、有料の Linux プランを購入してください。 登録後、必要に応じて CircleCI アプリケーションの [Settings (設定)] ページでプランの変更が可能です。

CircleCI のお客様の大半が、フルタイムの開発者 1 人あたり 2 〜 3 個のコンテナをお使いになっています。 チームの規模が拡大したときやワークフローが複雑化したときには、必要な並列処理や同時処理を行えるよう、コンテナを追加してください。

オープンソース プロジェクトにはさらに 3 つの無料コンテナが提供されるため、ジョブを並列で実行できます。

## アップグレード
{: #upgrading }

プランのアップグレード手順については、[アップグレードに関するよくあるご質問]({{ site.baseurl }}/ja/2.0/faq/#コンテナ数を増やしビルドの待機時間を解消するにはどのようにコンテナ-プランをアップグレードしたらよいですか)をご覧ください。
