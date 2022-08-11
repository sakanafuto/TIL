# StateProvider

StateProviderはプロバイダの値を取得する以外に、外部からプロバイダのステート（値）の変更が可能。
`count++`レベル以上の値の変更をする場合、`StateNotifierWidget`を使ったほうがいい見たい。

```dart
// 定義
final プロバイダ名 = StateProvider<型>((ref) => 値);
```

`ref.watch()`でプロバイダの状態を取得。
プロバイダの値が更新されると、`counter`に新しい値が再代入される・

```dart
// Widgetで使用可能に
final counter = ref.watch(counterProvider);
```

`ref.watch()`でプロバイダの状態を取得。
プロバイダの値が更新されると、`counter`に新しい値が再代入される・

```dart
// 外部から値を更新
ref.watch(counterProvider.notifier).state = 値;

// サンプル
// TODO まとめる
```

## 参考
- [【Flutter×Riverpod】StateProviderの使い方｜プロバイダで状態管理](https://flutternyumon.com/riverpod-how-to-use-stateprovider/)