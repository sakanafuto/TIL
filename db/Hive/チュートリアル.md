# Hive Tutorial

## Read

読み取りはめっちゃかんたん。

```dart
var box = Hive.box('myBox');

String name = box.get('name');

DateTime birthday = box.get('birthday');
```

キーが存在しなかったら Null を返す。defaultValue をによって、キーが存在しない場合に返す値を指定できる。

```dart
double height = box.get('randomKey', defaultValue: 17.5);
```

get()が返すのは常に List<dynamic>型（Map<dynamic, dynamic>型のマップ）です。特定の方にキャストする場合は list.cast<Some Type>()を使用する。

## Write

box への書き込みはほとんどマップへの書き込みのようなものです。すべてのキーは、最大 255 文字の ASCII 文字列か、符号なし 32 ビット整数でなければなりません。

```dart
var box = Hive.box('myBox');

box.put('name', 'Paul');

box.put('friends', ['Dave', 'Simon', 'Lisa']);

box.put(123, 'test');

box.putAll({'key1': 'value1', 42: 'life'});
```

非同期コードなしで書き込めるのか不思議に思うかもしれませんが、これは Hive の主な強みの 1 つです。
変更はバックグラウンドでできるだけ早くディスクに書き込まれますが、すべてのリスナーにはすぐに通知されます。もし非同期操作が失敗したら（失敗してはいけないのですが）、すべてのリスナーに古い値で再度通知されます。
書き込みが成功したことを確認したいのであれば、その Future を待つだけでよいのです。

これは、遅延ボックスの場合は異なる動作をします。put()が返した Future が終了しない限り、get()は古い値（存在しない場合は null）を返します。

以下のコードはその違いを示しています。

```dart
var box = await Hive.openBox('box');

box.put('key', 'value');
print(box.get('key')); // value

var lazyBox = await Hive.openLazyBox('lazyBox');

var future = lazyBox.put('key', 'value');
print(lazyBox.get('key')); // null

await future;
print(lazyBox.get('key')); // value
```

## Delete

既存の値を変更したい場合は、put()などでオーバーライドするか、削除します。また Null を書き込んだからといって値を削除したことにはなりません。

```dart
late Box box;
Future<void> main() async {
  await Hive.initFlutter();
  runApp(const MyApp());
  box = await Hive.openBox('writeNullBox');

  box.put('key', 'value');

  box.put('key', null);
  print(box.containsKey('key')); // true

  box.delete('key');
  print(box.containsKey('key')); // false
}
```

## 参考

- [公式ドキュメント](https://docs.hivedb.dev/#/basics/read_write)
