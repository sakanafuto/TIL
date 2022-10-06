# Lefthook で色々と自動化！

> 以下は Flutter での例

```shell
$ brew install lefthook
```

lefthook.yml というファイルを作成。

```yaml
# lefthook.yaml

pre-commit:
  parallel: true
  commands:
    linter:
      run: dart fix --apply lib && git add {staged_files}

    formatter:
      glob: "*.dart"
      run: dart format {staged_files} && git add {staged_files}
```

**pre-commit**
commit の直前という意味になります。
$ git commit と打った瞬間に実行され、処理が終了されるとファイルがコミットされます。
pre-push とすることで$ git push の直前に実行できます。

**parallel**
並列処理をするかどうか選択します。
今回は確実に formatter が最後に動いてほしかったので false としていますが、速度を求めるなら true のほうがいいと思います。

[lefthook](https://github.com/evilmartians/lefthook)

**command**
実行してほしい処理を記述します。

run: 実行するコマンド
glob: ファイルのフィルタリング
他にもいろんなオプションがありますので気になる方はこちらの GitHub をご覧ください。

また Lefthook には便利な変数のようなものが用意されています。

- {staged_files}: ステージング環境にあるファイル
- {all_files}: git によって追跡されているすべてのファイル

$ dart fix が結構時間かかるので、コミット完了まで 40 秒くらい掛かりました。
