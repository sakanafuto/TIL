# Lefthookで色々と自動化！

以下はFlutterでの例

LefthookはGit hooksをファイルで管理するためのツールで、Goで動いているので高速です。いろいろな方法で導入できますが、macOSならHomebrewが無難かと思います。

```shell
$ brew install lefthook
```

pubspec.ymlと同じディレクトリにlefthook.ymlというファイルを作成し次のように編集します。

```yaml
# lefthook.yaml

pre-commit:
  parallel: false
  commands:
    linter: 
      run: dart fix --apply lib && git add {staged_files}
    sort-imports:
      glob: "*.dart"
      run: flutter pub run import_sorter:main {staged_files} && git add {staged_files}
    formatter:
      glob: "*.dart"
      run: dart format {staged_files} && git add {staged_files}
```

**pre-commit**
commitの直前という意味になります。
$ git commitと打った瞬間に実行され、処理が終了されるとファイルがコミットされます。
pre-pushとすることで$ git pushの直前に実行できます。

**parallel**
並列処理をするかどうか選択します。
今回は確実にformatterが最後に動いてほしかったのでfalseとしていますが、速度を求めるならtrueのほうがいいと思います。

[lefthook](https://github.com/evilmartians/lefthook)

**command**
実行してほしい処理を記述します。

run: 実行するコマンド
glob: ファイルのフィルタリング
他にもいろんなオプションがありますので気になる方はこちらのGitHubをご覧ください。


またLefthookには便利な変数のようなものが用意されています。

- {staged_files}: ステージング環境にあるファイル
- {all_files}: gitによって追跡されているすべてのファイル

$ dart fixが結構時間かかるので、コミット完了まで40秒くらい掛かりました。

> エディタのGUIのGitツールからもコミットできますが画像のようなログは出力されません。
