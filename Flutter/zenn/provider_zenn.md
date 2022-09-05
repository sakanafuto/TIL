# はじめに

22新卒でエンジニア就職した者ですが、7月でやっとすべての研修が終了しました。
配属先はこれまで経験のないモバイルアプリチーム。会社の変革期ということもあり、運良く『モバイルアプリのFlutter一本化』のプロジェクトに参画させていただくことになりました。

バックエンド主体で開発をしてきたので``Flutter``はいろいろと慣れない部分が多く、情報の整理が必要と感じています。というわけで学習メモを残すことにしました。

基本的に公式サイトの翻訳になります。

https://docs.flutter.dev/development/data-and-backend/state-mgmt/intro

---

``Provider``を理解するための必要な要素は次の3つです。

- ``ChangeNotifier``
- ``ChangeNotifierProvider``
- ``Consumer``

## ChangeNotifier

``ChangeNotifier``は``Flutter SDK``に含まれるシンプルなクラスで、リスナーに変更を通知します。

プロバイダにおいて、``ChangeNotifier``はアプリケーションの状態をカプセル化する1つの方法になります。非常にシンプルなアプリの場合、1つの``ChangeNotifier``で十分です。複雑なアプリではいくつかのモデルを持つことになり、その結果いくつかの``ChangeNotifier``を持つことになります。

かんたんなショッピングアプリ（商品リストやカートに商品を追加・カートの中身をみる）のカートの状態を``ChangeNotifier``で管理した場合、以下のようになります。

```dart
/// ChangeNotifierを継承しています
class CartModel extends ChangeNotifier {
  final List<Item> _items = [];

  /// 外部からのカート内のアイテム変更を不可能に
  UnmodifiableListView<Item> get items => UnmodifiableListView(_items);

  /// すべての商品は42$とする
  int get totalPrice => _items.length * 42;

  void add(Item item) {
    _items.add(item);
    // このモデルをリッスンしているウィジェットに再びbuildするよう指示します
    notifyListeners();
  }

  void removeAll() {
    _items.clear();
    // このモデルをリッスンしているウィジェットに再びbuildするよう指示します
    notifyListeners();
  }
}
```

``ChangeNotifier``に固有のコードは``notifyListeners()``の呼び出しのみです。アプリのUIを変更する可能性のある方法でモデルが変更された時はいつでもこのメソッドを呼び出します。

``ChangeNotifier``はflutter:foundationの一部で``Flutter``の上位のクラスには依存せず、簡単にテストができます（ウィジェットテストすら必要ありません）。
以下は``CartModel``の簡単なユニットテストです。

```dart
test('adding item increases total cost', () {
  final cart = CartModel();
  final startingPrice = cart.totalPrice;
  cart.addListener(() {
    expect(cart.totalPrice, greaterThan(startingPrice));
  });
  cart.add(Item('Dash'));
});
```

## ChangeNotifierProvider

``ChangeNotifierProvider``は、``ChangeNotifier``のインスタンスを知らせるウィジェットです。
プロバイダパッケージから提供されます。

``ChangeNotifierProvider``をどこに置くかはすでに決まっています。アクセスする必要があるウィジェットの上です。``CartModel``の場合、それは``MyCart``と``MyCatalog``両方の上位です。

```dart
void main() {
  runApp(
    ChangeNotifierProvider(
      create: (context) => CartModel(),
      child: const MyApp(),
    ),
  );
}
```

``CartModel``の新しいインスタンスを作成するビルダーを定義していることに注意してください。``ChangeNotifierProvider``は賢いので、必要でない限り``CartModel``を再構築することはありません。
また、インスタンスが不要になると自動的に``CartModel``の``dispose()``を呼び出します。

もし、複数のクラスを提供したい場合は、``MultiProvider``を使用できます。

```dart
void main() {
  runApp(
    MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (context) => CartModel()),
        Provider(create: (context) => SomeOtherClass()),
      ],
      child: const MyApp(),
    ),
  );
}
```

## Consumer

冒頭の``ChangeNotifierProvider``により、アプリ内のウィジェットに``CartModel``が伝えられました。
早速使ってみましょう。

``Consumer``ウィジェットを通じて行われます。

```dart
return Consumer<CartModel>(
  builder: (context, cart, child) {
    return Text('Total price: ${cart.totalPrice}');
  },
);
```

アクセスしたいモデルのタイプを指定する必要があります。今回は``CartModel``が欲しいので、``Consumer<CartModel>``と書いています。
プロバイダは型をベースにしており、型がなければ何をしたいのかはわかりません。

``Consumer``ウィジェット唯一の必須引数は``builder``です。``Builder``は、``ChangeNotifier``が変更されるたびに呼び出される関数です。言い換えれば、モデル内で``notifyListeners()``を呼び出すと対応するすべての ``Consumer``ウィジェットの``builder``メソッドが呼び出されます。

ビルダーは3つの引数で呼び出されます。
最初の引数は``context``で、これはすべてのビルドメソッドで渡されます。

2番目引数は、``ChangeNotifier``のインスタンスです。
モデル内のデータを使って、任意の時点で``UI``がどのように見えるべきかを定義できます。

3番目の引数は``child``で、これは最適化のために存在します。
``Consumer``の下に、モデルが変わっても変化しない大きなウィジェットのサブツリーがある場合、それを一度構築してビルダーを通して取得できます。

```dart
return Consumer<CartModel>(
  builder: (context, cart, child) => Stack(
    children: [
      // ここでSomeExpensiveWidgetを使用すると、毎回再構築することなく使用できます。
      if (child != null) child,
      Text('Total price: ${cart.totalPrice}'),
    ],
  ),
  // expensiveウィジェットをここで構築する
  child: const SomeExpensiveWidget(),
);
```
	
``Consumer``ウィジェットはできるだけツリーの奥に配置するのがベストプラクティスです。
どこかが変わったからといって``UI``の大部分を作り直したくはないでしょう。

```dart 
// NG
return Consumer<CartModel>(
  builder: (context, cart, child) {
    return HumongousWidget(
      // ...
      child: AnotherMonstrousWidget(
        // ...
        child: Text('Total price: ${cart.totalPrice}'),
      ),
    );
  },
);
```
	
```dart
// GOOD
return HumongousWidget(
  // ...
  child: AnotherMonstrousWidget(
    // ...
    child: Consumer<CartModel>(
      builder: (context, cart, child) {
        return Text('Total price: ${cart.totalPrice}');
      },
    ),
  ),
);
```
	
## Provider.of

``UI``を変更するためにモデル内のデータを必要としないが、アクセスする必要がある場合はあります。
たとえば、``ClearCart``ボタンはユーザがカートからすべてを削除できるようにしたいとします。
カートの中身を表示する必要はなく、``clear()``メソッドを呼び出すだけでよいのです。

このために``Consumer<CartModel>``を使用することもますが、それはむだなことです。
再構築する必要のないウィジェットを再構築するようにフレームワークに要求することになります。

この使用例では、``listenパラメータ``を``false``に設定した``Provider.of``を使用できます。

```dart
Provider.of<CartModel>(context, listen: false).removeAll();
```

上記の行をビルドメソッドで使用すると、notifyListenersが呼ばれたときにこのウィジェットを再構築することはありません。

# まとめ

本記事では``Provider``を用いたかんたんな説明しか書いていませんが、もちろん[ドキュメント](https://docs.flutter.dev/development/data-and-backend/state-mgmt/simple)の前後のページには他の状態管理の方法についても詳しく書かれています。
	
この記事で取り上げたサンプルコードを置いておきます。
というかサンプルコード見ないとイメージつかないかもです。

https://github.com/sakanafuto/provider_shopper

（[公式](https://github.com/flutter/samples/tree/main/provider_shopper)のソースを和訳してちょっといじっただけです）
