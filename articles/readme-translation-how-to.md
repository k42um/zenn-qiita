---
title: "README.md を自動で英訳したい"
emoji: "🔤"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["github", "translation", "DeepL", "初心者"]
published: true
---

## はじめに
GitHubにはSpecial Repositoryという機能があり、自分のユーザIDと同名のリポジトリにREADME.mdを作成し、自己紹介などを書き込むことで自身のプロフィール画面に表示することができます。

それを英語対応させたいと思ったため、専用のGitHub Actionsを作成しました。

ユーザは日本語のREADME.mdを作成するだけで、あとは自動で英語版を作成してくれます。今のところ英語だけにしていますが、必要であれば多言語対応も可能です。

## Special Repository 機能について
若干本題から外れてしまうので、分かりやすい記事へのリンクを貼っておきます。

https://zenn.dev/saitogo/articles/93dc53cce42936

## 使い方
本項では細かく使い方を説明していきますが、ある程度の知識を有していて、サクッと終わらせたい！という方はぜひリポジトリのREADMEを参考にして進めてください。　（Star等もらえますと、モチベになります！）

https://github.com/k42um/readme-translation?tab=readme-ov-file

※本記事は2026年4月に投稿しています。スクリーンショットや情報は若干変わっている可能性があります。

### 1. DeepL API で APIキーを取得する
以下のサイトにアクセスして、「無料で始める」をクリックしてください。

https://www.deepl.com/ja/pro-api

DeepL APIにアクセスすると、3つのプランがあります。今回のワークフローでは無料プランで問題ないので、「API Free」のプランを選択してください。（2026年4月現在、Freeプランでも500,000字まで翻訳できます。）

![プランの一覧](/images/readme-translation-how-to/deeplplans.png)
*DeepL APIのプラン*

「無料で登録する」を押すとアカウント登録の画面が出てくるので、DeepLのアカウントを持っている場合はログインを、持っていない場合はアカウント登録をしてください。

登録が完了したら、「DeepL翻訳」のトップページが表示されるので、右上のアカウントマークをクリックし、「アカウント」をクリックしてください。

その後上の「APIキーと制限」をクリックし、「キーを作成 +」をクリックしてください。

その後APIキーの名前を尋ねられますが、これは各々が識別するためのものなので、なんでも構いません。名前を入力し、「キーを作成」を押すと、APIキー（謎の英数字の羅列）が出てくるので、これをコピーしたらこのステップは完了です。

![APIキー](/images/readme-translation-how-to/apikey.png)
*APIキーが表示される*

### 2. GitHub に　APIキー を登録する
`readme-translation`を使いたいGitHubのリポジトリを開き、`Setting` → `Secrets and variables` → `Actions` → `New repository secret`と進むと以下ような画面が表示されます。

![シークレットの追加画面](/images/readme-translation-how-to/addsecret.png)
*シークレットを登録する場面*

ここでは`Name`に`DEEPL_API_KEY`を、`Secret`に先ほどコピーしたAPIキーを入力してください。そして、`Add secret`をクリックすると登録完了です。

### 3. ワークフローの登録
`readme-translation`を使いたいGitHubのリポジトリに、`.github/workflows/translate-readme.yml`を作成し、以下のコードをコピペしてください。

```yaml
name: Translate README
on:
  push:
    branches: main
    paths: README.md
  workflow_dispatch:

jobs:
  translate:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
      - uses: k42um/readme-translation@v1
        with:
          deepl_api_key: ${{ secrets.DEEPL_API_KEY }}
```

これを用いることで、**日本語**の`README.md`を自動で**英語**に翻訳することができます。`README.md`の上には自動で英語版へと飛ぶボタンが追加されるので、容易にアクセスすることができます。

### 4. カスタマイズする

`readme-translation`はオプションを変更することで、さまざまなカスタマイズを施すことができます。カスタマイズしたい場合は、`deepl_api_key: ${{ secrets.DEEPL_API_KEY }}`の下にオプションを指定するための行を追加してください。

詳しく知りたい方は`readme-translation`のREADMEをご確認ください。（以下のリンクからアクセスすることで、オプションの一覧へとアクセスできます！）

https://github.com/k42um/readme-translation?tab=readme-ov-file#%E3%82%AA%E3%83%97%E3%82%B7%E3%83%A7%E3%83%B3

### 5. README.md を更新する
あとは`README.md`を更新するだけで、自動的にワークフローが実行され、翻訳ファイルが生成されます。

:::message alert
本ワークフローはDeepL APIを用いた機械翻訳であり、翻訳の質や正確性に関しては一切保証しません。自動翻訳によって生成されたファイルが正しいかどうか、ご自身で確認していただくよう、お願いいたします。
:::

## さいごに
元はと言えば自分のために作成したワークフローでしたが、思いのほか便利だったのでGitHub Marketplaceへと公開しました。日本語のREADMEを作成するだけで、英語などの他言語へ対応させられる便利なワークフローとなっております。ぜひ、お気軽にご利用ください。

https://github.com/k42um/readme-translation?tab=readme-ov-file

また、何かエラーやバグを見つけた際は本記事のコメントやGitHubのissue等でご報告いただけますと幸いです。