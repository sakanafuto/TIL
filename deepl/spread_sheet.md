# スプレッドシート

スプシの翻訳機能があまりにもしょぼいので DeepL の力を借りてわかりやすい日本語に翻訳する。

API 利用も無料で 50 万文字/月なのでとりあえずは大丈夫そう。

## Getting Started

1. [DeepL](https://www.deepl.com/pro#developer)アカウントを作る。Developer の無料枠で OK。
1. スプシ > 拡張機能 > Apps Script > 以下の関数をコピペして登録した際に発行された auth key を query に入れる。
1. 保存してスプシに戻る。
1. セルで`=deepl('hello')`とか`=deepl(A2:A3)（範囲）`で翻訳可能。

この関数は en -> ja なので、適宜 target_lang と source_lang を変えてあげる。

## 関数

```js
function deepl(text) {
  var response = UrlFetchApp.fetch(
    'https://api-free.deepl.com/v2/translate?auth_key=<my auth key>&text=' +
      text +
      '&target_lang=ja&source_lang=en'
  )
  var json = response.getContentText()
  var data = JSON.parse(json)
  var result = data.translations[0].text
  return result
}
```

## 参考

- https://qiita.com/chinotippy/items/d041627493cefb2f361d
