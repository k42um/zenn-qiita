---
title: "Hugo Narrow で baseURL 周りのエラーが出た"
emoji: "🏥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Hugo", "HugoNarrow", "WEB"]
published: true
---

## はじめに
### やりたいこと
- ローカルサーバでは`localhost:1313`、リモートサーバでは`k42um.github.io/Portfolio`でHugoビルドのページを公開（テスト）したい
- `hugo.yaml`で`baseURL`を`k42um.github.io/Portfolio`に設定したい

### 症状
1. `hugo.yaml`で設定した`baseURL`がうまく反映されず、GitHub Pagesで公開したサイトのデザインが総崩れ（CSSが参照できない）
2. （症状1の改善後に）`hugo server`を用い、ローカルでテストすると、`baseURL`が`localhost:1313/Portfolio`になってしまい、開けない

## 解決策
1. `hugo new site`で作られるルートディレクトリの`Hugo.yaml`は使わずに、`./config/_default/Hugo.yaml`の`baseURL`を変更する
2. `./config/development/Hugo.yaml`を作成し、そこに`baseURL: http://localhost:1313/`と記述する

## 解決策（詳細）
### 解決策1について
Hugoはルートディレクトリに存在する`Hugo.yaml`よりも`./config/`に存在する`Hugo.yaml`を優先するようです。それゆえに`./config/_default/Hugo.yaml`に記述されていた初期設定の`baseURL`が優先され、うまく変更できていなかったというのが問題でした。

### 解決策2について
おそらくグローバルに`baseURL`を変更したことが原因だったようです。これを改善するためにローカル開発用の設定として、`./config/development/Hugo.yaml`を作成し、そこの`baseURL`をローカルに合わせたものに設定することで問題が解決しました。

# さいごに
私が問題に詰まった際、解決するために多少の時間を要したので、TipsとしてZennに記事を残しました。誰かが同じ問題に直面した時、これが解決への糸口となってくれると幸いです。

:::details 筆者の環境
- Hugo
- テーマ：Hugo Narrow
- OS： macOS Sequoia 15.7.4
:::