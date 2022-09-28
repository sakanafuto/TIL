# Auto increment & indices

Hiveが符号なし整数のキーをサポートしていることはすでに知っています。お望みであれば、自動インクリメントのキーも使えます。これは複数のオブジェクトを保存したりアクセスしたりするのに非常に便利です。Boxをリストのように使うことができます。

```dart
late Box friends;
Future<void> main() async {
  await Hive.initFlutter();
  runApp(const MyApp());

  friends = await Hive.openBox('friends');
  friends.clear();

  friends.add('Lisa'); // index 0, key 0
  friends.add('Dave'); // index 1, key 1
  friends.put(123, 'Marco'); // index 2, key 123
  friends.add('Paul'); // index 3, key 124

  if (kDebugMode) {
    print(friends.getAt(0)); // Lisa
    print(friends.get(0)); // Lisa

    print(friends.getAt(1)); // Dave
    print(friends.get(1)); // Dave

    print(friends.getAt(2)); // Marco
    print(friends.get(123)); // Marco

    print(friends.getAt(3)); // Paul
    print(friends.get(124)); // Paul
  }
}
```