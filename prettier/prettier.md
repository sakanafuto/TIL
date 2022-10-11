## prettier

1. **[こちら](/npm/npm.md)**から npm をインストールする。
1. prettier を適用させたいプロジェクトで `$ npm init -y`する。
1. .gitignore に`node_modules`を追記する。
1. `npm add -D prettier`
1. `touch .prettierrc.yaml .prettierignore`

```yaml
# .prettierrc.yaml

semi: false
singleQuote: true
tabWidth: 2
useTabs": false
```

```yaml
# .prettierignore

# flutterなら
.dart_tool
.fvm
build
ios
android
```

package.json の"scripts"に"prettier"を追記する。

```json
// package.json

...
"scripts": {
  ...
  "prettier": "prettier --write ."
},
...
```

`$ npm run prettier`で実行！

## おまけ

こちらも git commit 時に実行できたら便利なので lefthook.yaml に項目を追記しておきます！

```yaml
prettier:
  run: npm run prettier && git add {staged_files}
```

lefthook については[**こちら**](/lefthook/lefthook.md)を参照。
