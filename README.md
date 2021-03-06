# 水研職員のためのRパッケージ開発チュートリアル

# はじめに

## 誰のためか
- なぜパッケージにする必要があるのかわからない
- コードは入手したものの手が動かない
- 共同開発の作法がよくわからない

## 誰のためでないか

詳しい開発方法を知りたい方は
[Writing R Extensions](https://cran.r-project.org/doc/manuals/r-release/R-exts.html)を読んでください。

## 前提

GitHubがナレッジベースであることを理解していること

# なぜパッケージにする必要があるのか

品質管理と可搬性が保証された仕事の最小単位だから

言い換えると:

- 製品
- プロの仕事としての最低要件

> Q. どのくらいの規模からパッケージ化したほうがいい？

A. 10分以内に終わらない仕事

## スクリプトとの違い

- テストがあること
- 不要な細部を隠せること
- 可搬性があること

### テストとは？

言い換えると:

- 確かに仕事をしたことを保証してくれるもの
- 暗黙の期待がコードの形で表現されたもの
- 動く仕様書
- セーフティネット
- 同じミスを二度と繰り返さないためのしくみ

よい副次的効果:

- 思考がクリアにする
- 手続き的なプログラミングの割合を減らせる
- 要件に関する誤解を防げる
- 弱点が見えるようになる


### 不要な細部を遮蔽できる

ユーザーに提供する機能（ビジネス価値）だけ`export`する
-> ユーザーが明確になる。自分がなにをしようとしているかが明確になる

下回りのコードも、悪目立ちを恐れずしっかり関数化できる
-> 作業に名前をつけることができ、思考がクリアになる

### 可搬性があること

可搬性がないプログラムとは:

> 「まずこのスクリプトをこのdirに置いて...」

> 「次に`library("hoge")`をして...あ、入ってない？じゃ入れてください」

#### よい副次効果

- dir構造を我流にしなくてすむ
- 仕事内容に集中できる
- 他人が書いたパッケージを理解できる

## なぜGitHub上に置く必要があるのか

- 透明性
- 再現性

### マネージャーからの視点

- 製品は期待通りに動いているのかね？ -> ビルドステータス

- 期待は確かなのかね？ -> テスト

- 顧客の満足度はどうだね？ -> Issue

- 去年の会議ではどんな議論が出たんだったかな？ -> Pull request

- シナリオへの対応状況はどうだね？ -> Network

これらは結局、開発者自身のためにもなる

### ユーザー

- どうやって使うんだろう？ -> チュートリアルページ

- 「これができたら良いのに」 -> Fork後、自分で改造

### 開発者

- ナレッジベースとコミュニケーションの連携 -> チャットツールとの連携

- プロジェクトマネジメント -> Projects

- 管理するのはほんとうのソースだけ -> Actions

# ワークフロー

「手が動かない」という方への解説

（コードは入手している前提: cloneするなり、`usethis::create_package()`するなり）

概略:

- ちゃんと動くのか確認: `devtools::test()`
- 作業場を作る: `git checkout -b BRANCH_NAME`
- 課題を追加する: テスト追加 & `devtools::test()`
- 仕事をする: コミット
- 仕事ができているか確認する: `devtools::test()`
- 見られても恥ずかしくないように掃除する: リファクタリング
- 製品としての検査を受ける: `devtools::check()`
- 提出する: Push
- 変更依頼を送る: Pull request

### やってはいけないこと

- コンソールで`library("hoge")`すること: 再現性がなくなります（`devtools::check()`時に一応検出できます）
- コードを直接評価すること: Globalenv上に古い関数オブジェクトが残り続け、謎の動作に自分が苦しむことになります
- `devtools::load_all()`し、コンソールで動作確認 <- なぜそれをする必要があるのですか？テストを書いていないからではないですか？

## バグが見つかったら

すぐに修正しないこと！

なぜか？
バグとは: 既存のテストを通過してしまう好ましくない動作。
「好ましくない」というところに既に暗黙知がある。
なにを好ましくないと思っているのかをテストで表現すること。
バグを再現するためのテストを追加する -> ビルドが壊れたのを確認してからコードを修正
同じ原因によるバグは二度と再発しなくなる

# 共同開発において意識すること

- 環境の統一
- コーディング規約
- きれいに

## 環境正準化

「自分の環境では〇〇」無意味です。
以下を使いましょう:

- 継続的インテグレーション
- Docker

## コーディング規約

参考にしているスタイルのURLを貼っておきましょう。
疑問に思ったら、気軽に相談しましょう。
ただの規約は「動かない」ので実行力がありません。
`lintr`などを使って、動く規約にしておきましょう。

## 同じことを繰り返さない

DRY（Don't Repeat Yourself）原則を意識しましょう:

- バグ: テストを追加
- コーディング規約: `lintr`
- ビルド: 継続的インテグレーション
- 作業通知: チャットサービスとの連携
- マニュアル: 「書かず」に、vignetteを使って「実装」する

## きれいに保つ

### コードをきれいに
少し汚いところは、もっと汚くなる[割れ窓理論](https://en.wikipedia.org/wiki/Broken_windows_theory)。
きれいに保とう。

コメントは「動かないドキュメント」の代表格。
動かないので、コードやテストの内容と解離することもある。
間違ったドキュメントを読んでしまうくらいなら読まない方がまし。

- コメントアウトコードは全て削除せよ（`git log -S "some search words"`などでいつでも検索できる）
- 説明的なコードを書くこと: コードはまさに動くドキュメントである

### コミット履歴をきれいに
コミット履歴をきれいにしましょう:
- テストを追加した時点で一旦コミットする: `git commit -m ":bug: Fix bug on something" -m "Fix #123"`
- 適宜コミットしながら仕事
- リファクタリングまで終わったら: `git rebase -i`でコミットをまとめる

コミットが大きくなりすぎる？ -> Issueを分割すること

### ナレッジベースをきれいに

Issueに関係ないコミットはしないようにしましょう。

つい変更してしまったら: その変更はいったん`git stash`し、別IssueとしてGitHubに登録しておきましょう

