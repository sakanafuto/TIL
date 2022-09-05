## Factory Constructor in Dart

[freezed](https://pub.dev/packages/freezed)という便利なパッケージの中で`factory`という言葉が頻出していて、理解せずに先に進むことが難しそうだったので調べてまとめました。[参考](#参考)にある`Saravanan M`さんの記事がとても参考になりました。

ファクトリーデザインパターンの定義は次のとおりです。

> ロジックをクライアントに公開することなくオブジェクトを作成し、共通のインターフェイスを用いて新しく作成されたオブジェクトを参照する。

（？）。言葉では分かりづらいのでコードで表現します。

適当な`Drink class`を定義して、そのサブクラスに水（Water）とコーヒー（Coffee）クラスを定義します。

```dart
class Drink {
  int ammount;
  Drink(this.ammount);
}

class Water extends Drink {
  Water(int ammount) : super(ammount);
}

class Coffee extends Drink {
  Coffee(int ammount) : super(ammount);
}
```

ユーザーが休憩を望んでいる場合、そのお供としてコーヒーを提供し、そうでない場合は水を提供します。

```dart
void main() {
  Drink drink;

  int ammount = 250;
  bool isBreak = true;

  isBreak ? drink = Coffee(ammount) : drink = Water(ammount);
  // Instance of 'Coffee'
}
```

これでも問題ありませんが、複数の箇所でこのロジックを使う場合を考えると`if-else`を多用することになり、現実的ではありません。
どうすればこれを簡単にできるでしょうか？その答えが`Factory Constructor`です。

```dart
class Drink{
  int ammount;
  Drink(this.ammount);

  factory Drink.createDrink({required int ammount, bool breakTime = false}) 
      => breakTime ? Coffee(ammount) : Water(ammount);
}

void main() {
  Drink myBreakDrink = Drink.createDrink(ammount: 200, breakTime: true);  
  // Instance of 'Coffee'

  Drink mySportDrink = Drink.createDrink(ammount: 500);
  // Instance of 'Water'
}
```

ここで再びファクトリーデザインパターンの定義を見てみます。

> オブジェクトの作成ロジックをクライアントに公開することなく、オブジェクトを作成する。

オブジェクト生成ロジックをクライアント（=main()）に公開していません。ロジックは親クラス`Drink class`で定義されていて、クライアントは必要なオブジェクトを取得するために必要なパラメータを指定してファクトリーのコンストラクタを呼び出す必要があります。

> 新たに生成されたオブジェクトは、共通のインターフェイスを使って参照する。

Coffee（or Water）のインスタンスをDrink型で返しているのであって、Coffee（or Water）として返しているわけではありません。親クラスは両方のサブクラスと互換性があるため、ここでは共通のインターフェイスとなっています。

## Named Constructorとの違い

`Named Constructor`とはその名の通り名前の付いたコンストラクタです。

JavaやC++では引数が異なる限り、複数のコンストラクタを持つクラスを書くことができます（オーバーロード）。Dartにはそのような機能はなく、明示的に名前をつけたコンストラクタを定義する必要があります。

```dart
class Drink {
  int ammount;
  Drink(this.ammount);

  // Named Constructor
  Drink.fromJson(...){
    ...
  }

  // Factory Constructor
  factory Drink.createDrink(...){
    ...
  }
}
```

`Named Constructor`の場合、役割がより明確になり読みやすくなります（fromJson()であればJsonからインスタンスを作成しているんだなと推測できる）。

両者の違いは以下の通りで、必要に応じて使い分ければよさげですね。

**インスタンスのメンバーへのアクセス**

- `Named Constructor`は任意のメンバ変数およびメソッドにアクセスできます。
- `Factory Constructor`は静的なので、このキーワードにアクセスすることはできません。

**returnの有無**

- `Named Constructor`は通常のコンストラクタと同様に動作し、明示的にインスタンスを返す必要はありません。
- `Factory Constructor`は明示的にインスタンスを返す必要があります。

**生成されるインスタンスの型**

- `Named Constructor`は、実行するクラスのインスタンスしか生成できません。
- `Factory Constructor`は実行時にどのインスタンスを返すかを決めることができ、サブクラスのインスタンスも返すことができます。

## 参考

- [Factory Constructor in Dart — Part 1](https://medium.com/nerd-for-tech/factory-constructor-in-dart-part-1-1bbdf0d0f7f0)
- [Factory Constructor in Dart — Part 2](https://medium.com/nerd-for-tech/factory-constructor-in-dart-part-2-7db2a5981ac3)
- [Named Constructor vs Factory Constructor in Dart](https://medium.com/nerd-for-tech/named-constructor-vs-factory-constructor-in-dart-ba28250b2747)