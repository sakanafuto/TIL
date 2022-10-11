# はじめに

AppBar をカスタマイズしてみました。
以下の完成品を作る手順を解説します。

![](https://storage.googleapis.com/zenn-user-upload/7b75249e0092-20220806.gif)

---

## 準備

プロジェクトを開始したら`lib/main.dart`を以下のようにかんたんに書き換えます。

```dart:main.dart
import 'package:flutter/material.dart';
import 'package:zenn/search_bar.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(primarySwatch: Colors.blue),
      home: const Scaffold(
        appBar: SearchBar(),
        body: Center(
          child: Text('Hello'),
        ),
      ),
    );
  }
}
```

`Scaffold`中の`appBar`で、カスタマイズする`SearchBar`を生成しています。

## AppBar のカスタマイズ

`appBar`クラスは`PreferredSizeWidget`で宣言されているので、カスタマイズする際も`PreferredSizeWidget`実装する必要があります。

```dart:search_bar.dart
import 'package:flutter/material.dart';

class SearchBar extends StatefulWidget implements PreferredSizeWidget {
  const SearchBar({Key? key}) : super(key: key);

  @override
  State<SearchBar> createState() => SearchBarState();

  @override
  Size get preferredSize => const Size.fromHeight(kToolbarHeight);
}
```

といっても、やることはインターフェイスの実装と`preferredSize`のゲッターをオーバーライドしてあげるくらいです。

## 検索バーの実装

中身を書いていきます。検索バー自体は`AppBar`の`title`プロバティを書き換えて実装しています。

まずは要点だけ抜き出しています。

```dart:search_bar.dart
class SearchBarState extends State<SearchBar> {
  final _controller = TextEditingController();

  void _submission(text) {
    setState(() {
      _controller.clear();
      if (kDebugMode) {
        print(text);
      }
  });
}

  @override
  Widget build(BuildContext context) {
    return AppBar(
      title: SizedBox(
        ...
          child: TextField(
            controller: _controller,
            decoration: const InputDecoration(
              hintText: 'Search Text',
              prefixIcon: Icon(Icons.search),
              suffixIcon: Icon(Icons.clear),
              ...
            ),
            onSubmitted: (text) => _submission(text),
          ),
        ...
```

`TextField`は便利な`contoller`プロパティがあり、`TextEditingController`をいれてあげるだけでコントローラから入力値などを取得できます。

検索バーによくある両端の『🔎』や『❌』は`decoration`プロバティの`InputDecoration`で設定でき、それぞれ`prefixIcon`、`suffixIcon`になります。プレースホルダーは`hintText`プロバティで設定します。

ただ大抵の場合『❌』は入力値のクリアなどを割り当てている場合が多く、その際には別途`ButtonIcon`などで実装する必要がでてきます。

最後の`onSubmitted`は`TextField`に入力された値を処理するプロバティで、引数には入力した値が`String`型として渡されます。

今回は`TextField`入力値の削除と入力値のコンソールへの出力を`_submittion()`で実行しています。

`onSubmitted: (text) => _submission(text)`の書き方については、前回書いた記事でもほんのちょっとだけ触れています。

## 最終的なコード

体裁を整えた最終的なコードは以下のとおりです。

```dart:search_bar.dart
import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';

class SearchBar extends StatefulWidget implements PreferredSizeWidget {
  const SearchBar({Key? key}) : super(key: key);

  @override
  State<SearchBar> createState() => SearchBarState();

  @override
  Size get preferredSize => const Size.fromHeight(kToolbarHeight);
}

class SearchBarState extends State<SearchBar> {
  final _controller = TextEditingController();

  void _submission(text) {
    setState(() {
      _controller.clear();
      if (kDebugMode) {
        print(text);
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    return AppBar(
      title: SizedBox(
        height: 40,
        child: Container(
          decoration: BoxDecoration(
            color: Colors.white,
            border: Border.all(color: Colors.blue),
            borderRadius: BorderRadius.circular(8.0),
          ),
          child: Center(
            child: Container(
              width: 340,
              margin: const EdgeInsets.symmetric(vertical: 10.0),
              child: TextField(
                controller: _controller,
                decoration: const InputDecoration(
                  hintText: 'Search Text',
                  prefixIcon: Icon(Icons.search),
                  suffixIcon: Icon(Icons.clear),
                  contentPadding: EdgeInsets.only(left: 8.0),
                  enabledBorder: InputBorder.none,
                  focusedBorder: InputBorder.none,
                  isDense: true,
                ),
                onSubmitted: (text) => _submission(text),
              ),
            ),
          ),
        ),
      ),
    );
  }
}

```

この例はシンプルなものですが、入力値のクリアや、入力値をタグとしてテキストフィールド内に表示させる（横並びにする）といった機能を実装するとなるとちょっと面倒なことになります（`Row`や`Stack`、`Expanded`、`SingleChildScrollView`あたりをゴニョゴニョする）。

他にも`TextField`には`onChanged`などおもしろいプロバティもありますので調べてみるといいと思います。

# まとめ

- `PreferredSizeWidget`を実装して`AppBar`をカスタマイズする。
- `TextEditingController()`を用いることで`TextField`の入力値をかんたんに操作できる。
- `TextField`のプレフィックスアイコンやプレースホルダーはプロバティに用意されている。
