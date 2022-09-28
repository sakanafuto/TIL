## Caching

### Loggerクラスの作成

```dart
class Logger {
  final String name;
  Logger(this.name) { 
    print("\"${this.name}\"");
  }
}
```

実行

```dart
class A {
  late final Logger _logger;
  A() {
    _logger = Logger('Foo');
  }
}

void main() {
  for (int i = 1; i <= 5; i++) {
    print("===$iつ目のインスタンス===");
    A a = A();
    print("Aというインスタンスを作成しました。");
  }
}
```

結果

```bash
===1つ目のインスタンス===
"Foo"
Aというインスタンスを作成しました。 

===2つ目のインスタンス===
"Foo"
Aというインスタンスを作成しました。 

===3つ目のインスタンス===
"Foo"
Aというインスタンスを作成しました。 

===4つ目のインスタンス===
"Foo"
Aというインスタンスを作成しました。 

===5つ目のインスタンス===
"Foo"
Aというインスタンスを作成しました。 
```

### キャッシュ可能なLoggerクラスの作成

```dart
class Logger {
  final String name;

  static final Map<String, Logger> _cache = <String, Logger>{};

  factory Logger(String name) => _cache.putIfAbsent(name, () => Logger._(name));

  // private constructor
  Logger._(this.name) {
    print("\"${this.name}\"");
  }
}
```

パブリックコンストラクタの代わりにプライベートコンストラクタを定義しています。これにより、Loggerクラスのインスタンスを直接作成できないようにしています。
そして、一度作成したインスタンスをクラス名をキーとして保存するために使用される、`Map _cache`も作成しています。

> _cacheをstaticとして定義しないと、ファクトリコンストラクタは_cacheにアクセスできない。

もう一度`main()`関数を実行。

```bash
===1つ目のインスタンス===
"Foo"
Aというインスタンスを作成しました。 

===2つ目のインスタンス===
Aというインスタンスを作成しました。 

===3つ目のインスタンス===
Aというインスタンスを作成しました。 

===4つ目のインスタンス===
Aというインスタンスを作成しました。 

===5つ目のインスタンス===
Aというインスタンスを作成しました。 
```

本来であれば`A a = A();`で毎回インスタンスを作成するところを、キャッシュされたインスタンスを利用するか、それがない場合に新しく作成し、キャッシュに保存するという動作を実現しています。

今回は単純なLoggerクラスなので恩恵はほぼありませんが、作成するインスタンスが複雑で作成に時間がかかる場合には有効です。