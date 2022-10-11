# List / Set の値を更新しても ref.watch で再ビルドされない

```dart
final polygonSetProvider = StateProvider<Set<Polygon>>((ref) => {});
...

@override
Widget build(BuildContext context) {
  final polygonSet = ref.watch(polygonSetProvider);
...

// これらの書き方では動かない or 再ビルドされない
ref.read(polygonSetProvider.state).state = state.add(Polygon(...));
ref.read(polygonSetProvider.state).state = polygonSet;

// 再代入的な方法じゃないとだめみたい
ref.read(polygonSetProvider.state).state = {...polygonSet};
```

## 参考

- https://zenn.dev/shu_illy/articles/d43e842ea9ad4e
