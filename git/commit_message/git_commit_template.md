# git commit のテンプレートを作る

絵文字プレフィックスをつけてコミット内容をかんたんに理解できるようにしたい！
ただ毎回好きな絵文字を使っていてはごちゃついてしまい、何かしらルールを設けたい。

コミット時のテンプレートを活用することで以下

- 絵文字プレフィックスを用いることでひと目でコミットの内容がわかる
- 絵文字プレフィックスの意味・絵文字の文字列（:xxx:など）を確認できる
- テンプレートファイルを共有すればチームでも使える

## Getting started

プロジェクトルートにファイルを作成する。

```shell
# ファイル名は . で始まっていればいい
$ vi .git_commit_template 
```

`$ git commit`時に表示される内容を記述する。
ファイルは[こちら](.git_commit_template)

```shell

# ==================== Emojis ====================
# 🎉  :tada:             initial commit
# ✨  :sparkles:         feature
# 🐛  :bug:              fix
# ♻️   :recycle:          refactor
# ⚡️  :zap:              performans
# 📚  :books:            documents
# 🧪  :test_tube:        test
# 🚧  :construction:     wip

# ==================== Format ====================
# :emoji: Subject
```

プロジェクトルートと書いたが、私の場合特にプロジェクト毎に内容は変わらないためグローバルに設定している。そのため`.git_commit_template`はワークディレクトリに置いてある。

最後に`git`に適用する。

```shell
# プロジェクトに関わらず同一のテンプレートを使う場合
$ git config --global commit.template [ディレクトリ]/.git_commit_template

# プロジェクト毎に分ける場合
$ git config commit.template [ディレクトリ]/.git_commit_template
```

コンソールからコミットするので、メッセージを編集する際は`vim`エディタでそのままコンソール上で行いたい。`zsh`の場合、デフォルトのエディタが`nano`になっているかも。

```shell
$ git config --global core.editor vim
```

## How to Use

```shell
$ git commit

# 1. 先程のテンプレートが表示されるので、コミットメッセージを打ち込む
:sparkles: 〇〇の実装

# 2. エディタを閉じる
# esc -> ZZ
```

## References

- [Git commit templateを使用する](https://qiita.com/usuket/items/7fc3274474205d4715fb)
- [[備忘録]vimをgitのエディタのdefault設定に](https://qiita.com/Syunto07ka/items/f87c9a82dfacf1472ee5)
