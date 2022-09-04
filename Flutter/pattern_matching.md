# Pattern matching

## freezed

freezedでは`union/sealed class`をかんたんに定義できる。

```dart
@freezed
class Union with _$Union {
  const factory Union(int value) = Data;
  const factory Union.loading() = Loading;
  const factory Union.error([String? message]) = ErrorDetails;
}
```

### when

```dart
var union = Union(42);

print(
  union.when(
    (int value) => 'Data $value',
    loading: () => 'loading',
    error: (String? message) => 'Error: $message',
  ),
);
```

各コールバックで、コンストラクタと同じ名前、型で実装する必要がある。
またすべてのコールバックが必須であり、nullを許容したい場合はMaybeWhenを使う。

## maybeWhen

すべてのコールバックを指定する必要がない。その代わり、フォールバックのためにorElseパラメータが必須。

```dart
var union = Union(42);

print(
  union.maybeWhen(
    null, 
    loading: () => 'loading',
    orElse () => 'fallback',
  ),
);
```

これは以下と同等である。

```dart
var union = Union(42);

String label;

union is Loading ? label = 'loading' : label = 'fallback';
```

## map/maybeMap

when/maybeWhenと同等だが、破壊することがない。こちら推奨。

```dart
@freezed
class Model with _$Model {
  factory Model.first(String a) = First;
  factory Model.second(int b, bool c) = Secont;
}
```

*whenの場合*

```dart
var model = Model.first('42');

print(
  model.when(
    first: (String a) => 'first $a',
    second: (int b, bool c) => 'second $b $c',
  ),
);
```

*mapならこう書き換えられる*

```dart
var model = Model.first('42');

print(
  model.map(
    first: (First value) => 'first ${value.a}',
    second: (Second value) => 'second ${value.b} ${value.c}',
  ),
);
```

一見どちらでも良さそうに思えるが、copyWithやtoStringなどを実行する際にやくだつ。

```dart
var model = Model.second(42, false)
print(
  model.map(
    first: (value) => value,
    second: (value) => value.copyWith(c: true),
  )
);
```