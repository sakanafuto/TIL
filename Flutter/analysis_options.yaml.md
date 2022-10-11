# オススメ設定

Linter はきつくしよう！

```yaml
# mono0926 / pedantic_mono: https://github.com/mono0926/pedantic_mono
include: package:pedantic_mono/analysis_options.yaml

analyzer:
  # Dart Analysis の対象外にするファイル。
  # ここで指定したファイルはコンパイルエラーも報告しなくなるみたいなので注意。
  exclude:
    - lib/**/*.g.dart
    - lib/**/*.gql.dart
    - lib/**/*.freezed.dart
    - lib/**/dio_headerfixed_adapter.dart

  errors:
    # import_sorter の設定を優先するため除外
    directives_ordering: ignore

linter:
  rules:
    # pubspec.yaml の 依存を A-Z でソートしろというもの。
    # flutter などもソートされて気持ち悪かったし、
    # 役割で分けたほうがよさそうなので false にした。
    sort_pub_dependencies: false
```

最終更新：2022/09/28
