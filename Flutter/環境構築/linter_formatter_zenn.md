> [こちらの記事](https://zenn.dev/10_tofu_01/articles/flutter_formatter_linter_lefthook)を御覧ください。

# 【Flutter】Linter と Formatter、そして Lefthook

# はじめに

**インデントを揃えただけのムダなコミット**が増えてきて、プルリクやコミット履歴に関係ないファイルが多くなり「これはいかん」となったので、Flutter における Formatter や Linter について調べてみました。

Formatter や Linter を調べてたら、なんとその自動化の方法やちょっとした便利なパッケージを見つけましたのでそちらもご紹介いたします。

この記事で紹介したことを導入すると、**コードの整形や解析が自動で行われ**、**正しい書き方や文法が身につき**、**import 文が見やすく**なります（最後のはおまけ）。

### 個人的結論

- Formatter は Dart 標準搭載のもので充分（$ dart format）
- 静的解析は有名な設定を include した上でプロジェクトに沿った設定をする
- Lefthook を導入して$ git commit 時にこれらが走るようにする

静的解析は制約が強めな[pedantic_mono](https://github.com/mono0926/pedantic_mono/blob/main/example/analysis_options.yaml)を導入させていただいています。

## Flutter における Formatter / Linter

文法やスタイルの問題をチェックする Linter、コードをよしなに整形してくれる Formatter は多種多様な言語で利用されています。

Dart / Flutter においては、Formatter と Linter（静的解析ツール）の両者ともに Dart に標準搭載されています。コマンドでいうと$ dart format や$ dart analyze など。

これらは VS Code や Android Studio を使う際にまず導入するであろう Flutter / Dart プラグインに含まれていますので、エディタのコマンドや問題パネルなどで知らずのうちに使っているかもしれません。

[!VS Code の Flutter プラグイン](https://storage.googleapis.com/zenn-user-upload/768e1d6f0639-20220920.png =250x)
_VS Code 用のプラグイン_

![!Android StudioのDartプラグイン](https://storage.googleapis.com/zenn-user-upload/f28696bd2fad-20220920.png =250x)
_Android Studio 用のプラグイン_

Formatter は Dart チームが推奨するコーディングスタイルに則っていれば問題ないと思いますが、Linter に関しては標準搭載のものは制限が結構ゆるいです（`const`つけるべき箇所でもとくに何も指摘されなかったり）。標準で厳しすぎても初心者に優しくなく、面白くないという配慮からでしょうか？

なんにせよ、Linter に関してはコードの堅牢性やパフォーマンスを高めるためにも、多少指摘が煩わしくても厳しめなルールを定めた方がいいと私は思います。

## Formatter

コードのフォーマットは先述の通り Dart に標準搭載されているもので充分です。

`$ dart format [ファイル]`, `$ dart format lib`とコマンドを叩くだけですぐにコードを整形してくれます。

また VS Code であれば`⌘ + P`でコマンドパレットを開き、`Format Document`から実行できます。
こちらも VS Code の紹介のみで恐縮ですが、設定ファイルに次のような項目を追記することでファイル保存時やペースト時に実行するよう設定できます。Android Studio にも似たような設定は可能かと思います。

```json
"editor.formatOnSave": true,
"editor.formatOnType": true,
"editor.formatOnPaste": true,
```

## Linter

こちらも標準搭載のものが用意されており、`$ dart analyze`で一定のルールに基づいて解析を実行し、`$ dart fix --apply lib`で修正可能なものについては自動修正してくれます。

こちらもプラグインを導入することで各種エディタから実行可能です。

修正可能なものとはたとえば、不要な import 文の削除や type annotation を付け足すとかです。能動的に、複雑なコードの改変が必要になる場合等は自動修正はしてくれませんので注意しましょう。

さてこの『一定のルール』ですが、プロジェクトのルートディレクトリに`analysis_options.yaml`というファイルを追加することでカスタマイズできます。

公式の[flutter/flutter](https://github.com/flutter/flutter/blob/master/analysis_options.yaml)プロジェクトでも例がありますね。

:::details flutter/flutter の analysis_options.yaml

```dart
# Specify analysis options.
#
# For a list of lints, see: http://dart-lang.github.io/linter/lints/
# See the configuration guide for more
# https://github.com/dart-lang/sdk/tree/main/pkg/analyzer#configuring-the-analyzer
#
# There are other similar analysis options files in the flutter repos,
# which should be kept in sync with this file:
#
#   - analysis_options.yaml (this file)
#   - https://github.com/flutter/plugins/blob/master/analysis_options.yaml
#   - https://github.com/flutter/engine/blob/master/analysis_options.yaml
#   - https://github.com/flutter/packages/blob/master/analysis_options.yaml
#
# This file contains the analysis options used for code in the flutter/flutter
# repository.

analyzer:
  language:
    strict-casts: true
    strict-raw-types: true
  errors:
    # allow self-reference to deprecated members (we do this because otherwise we have
    # to annotate every member in every test, assert, etc, when we deprecate something)
    deprecated_member_use_from_same_package: ignore
    # Turned off until null-safe rollout is complete.
    unnecessary_null_comparison: ignore
  exclude:
    - "bin/cache/**"
      # Ignore protoc generated files
    - "dev/conductor/lib/proto/*"

linter:
  rules:
    # This list is derived from the list of all available lints located at
    # https://github.com/dart-lang/linter/blob/master/example/all.yaml
    - always_declare_return_types
    - always_put_control_body_on_new_line
    # - always_put_required_named_parameters_first # we prefer having parameters in the same order as fields https://github.com/flutter/flutter/issues/10219
    - always_require_non_null_named_parameters
    - always_specify_types
    # - always_use_package_imports # we do this commonly
    - annotate_overrides
    # - avoid_annotating_with_dynamic # conflicts with always_specify_types
    - avoid_bool_literals_in_conditional_expressions
    # - avoid_catches_without_on_clauses # blocked on https://github.com/dart-lang/linter/issues/3023
    # - avoid_catching_errors # blocked on https://github.com/dart-lang/linter/issues/3023
    - avoid_classes_with_only_static_members
    - avoid_double_and_int_checks
    - avoid_dynamic_calls
    - avoid_empty_else
    - avoid_equals_and_hash_code_on_mutable_classes
    - avoid_escaping_inner_quotes
    - avoid_field_initializers_in_const_classes
    # - avoid_final_parameters # incompatible with prefer_final_parameters
    - avoid_function_literals_in_foreach_calls
    - avoid_implementing_value_types
    - avoid_init_to_null
    - avoid_js_rounded_ints
    # - avoid_multiple_declarations_per_line # seems to be a stylistic choice we don't subscribe to
    - avoid_null_checks_in_equality_operators
    # - avoid_positional_boolean_parameters # would have been nice to enable this but by now there's too many places that break it
    - avoid_print
    # - avoid_private_typedef_functions # we prefer having typedef (discussion in https://github.com/flutter/flutter/pull/16356)
    - avoid_redundant_argument_values
    - avoid_relative_lib_imports
    - avoid_renaming_method_parameters
    - avoid_return_types_on_setters
    - avoid_returning_null
    - avoid_returning_null_for_future
    - avoid_returning_null_for_void
    # - avoid_returning_this # there are enough valid reasons to return `this` that this lint ends up with too many false positives
    - avoid_setters_without_getters
    - avoid_shadowing_type_parameters
    - avoid_single_cascade_in_expression_statements
    - avoid_slow_async_io
    - avoid_type_to_string
    - avoid_types_as_parameter_names
    # - avoid_types_on_closure_parameters # conflicts with always_specify_types
    - avoid_unnecessary_containers
    - avoid_unused_constructor_parameters
    - avoid_void_async
    # - avoid_web_libraries_in_flutter # we use web libraries in web-specific code, and our tests prevent us from using them elsewhere
    - await_only_futures
    - camel_case_extensions
    - camel_case_types
    - cancel_subscriptions
    # - cascade_invocations # doesn't match the typical style of this repo
    - cast_nullable_to_non_nullable
    # - close_sinks # not reliable enough
    - combinators_ordering
    # - comment_references # blocked on https://github.com/dart-lang/linter/issues/1142
    - conditional_uri_does_not_exist
    # - constant_identifier_names # needs an opt-out https://github.com/dart-lang/linter/issues/204
    - control_flow_in_finally
    - curly_braces_in_flow_control_structures
    - depend_on_referenced_packages
    - deprecated_consistency
    # - diagnostic_describe_all_properties # enabled only at the framework level (packages/flutter/lib)
    - directives_ordering
    # - discarded_futures # not yet tested
    # - do_not_use_environment # there are appropriate times to use the environment, especially in our tests and build logic
    - empty_catches
    - empty_constructor_bodies
    - empty_statements
    - eol_at_end_of_file
    - exhaustive_cases
    - file_names
    - flutter_style_todos
    - hash_and_equals
    - implementation_imports
    - iterable_contains_unrelated_type
    # - join_return_with_assignment # not required by flutter style
    - leading_newlines_in_multiline_strings
    - library_names
    - library_prefixes
    - library_private_types_in_public_api
    # - lines_longer_than_80_chars # not required by flutter style
    - list_remove_unrelated_type
    # - literal_only_boolean_expressions # too many false positives: https://github.com/dart-lang/linter/issues/453
    - missing_whitespace_between_adjacent_strings
    - no_adjacent_strings_in_list
    - no_default_cases
    - no_duplicate_case_values
    - no_leading_underscores_for_library_prefixes
    - no_leading_underscores_for_local_identifiers
    - no_logic_in_create_state
    # - no_runtimeType_toString # ok in tests; we enable this only in packages/
    - non_constant_identifier_names
    - noop_primitive_operations
    - null_check_on_nullable_type_parameter
    - null_closures
    # - omit_local_variable_types # opposite of always_specify_types
    # - one_member_abstracts # too many false positives
    - only_throw_errors # this does get disabled in a few places where we have legacy code that uses strings et al
    - overridden_fields
    - package_api_docs
    - package_names
    - package_prefixed_library_names
    # - parameter_assignments # we do this commonly
    - prefer_adjacent_string_concatenation
    - prefer_asserts_in_initializer_lists
    # - prefer_asserts_with_message # not required by flutter style
    - prefer_collection_literals
    - prefer_conditional_assignment
    - prefer_const_constructors
    - prefer_const_constructors_in_immutables
    - prefer_const_declarations
    - prefer_const_literals_to_create_immutables
    # - prefer_constructors_over_static_methods # far too many false positives
    - prefer_contains
    # - prefer_double_quotes # opposite of prefer_single_quotes
    - prefer_equal_for_default_values
    # - prefer_expression_function_bodies # conflicts with https://github.com/flutter/flutter/wiki/Style-guide-for-Flutter-repo#consider-using--for-short-functions-and-methods
    - prefer_final_fields
    - prefer_final_in_for_each
    - prefer_final_locals
    # - prefer_final_parameters # we should enable this one day when it can be auto-fixed (https://github.com/dart-lang/linter/issues/3104), see also parameter_assignments
    - prefer_for_elements_to_map_fromIterable
    - prefer_foreach
    - prefer_function_declarations_over_variables
    - prefer_generic_function_type_aliases
    - prefer_if_elements_to_conditional_expressions
    - prefer_if_null_operators
    - prefer_initializing_formals
    - prefer_inlined_adds
    # - prefer_int_literals # conflicts with https://github.com/flutter/flutter/wiki/Style-guide-for-Flutter-repo#use-double-literals-for-double-constants
    - prefer_interpolation_to_compose_strings
    - prefer_is_empty
    - prefer_is_not_empty
    - prefer_is_not_operator
    - prefer_iterable_whereType
    # - prefer_mixin # Has false positives, see https://github.com/dart-lang/linter/issues/3018
    # - prefer_null_aware_method_calls # "call()" is confusing to people new to the language since it's not documented anywhere
    - prefer_null_aware_operators
    - prefer_relative_imports
    - prefer_single_quotes
    - prefer_spread_collections
    - prefer_typing_uninitialized_variables
    - prefer_void_to_null
    - provide_deprecation_message
    # - public_member_api_docs # enabled on a case-by-case basis; see e.g. packages/analysis_options.yaml
    - recursive_getters
    # - require_trailing_commas # blocked on https://github.com/dart-lang/sdk/issues/47441
    - secure_pubspec_urls
    - sized_box_for_whitespace
    # - sized_box_shrink_expand # not yet tested
    - slash_for_doc_comments
    - sort_child_properties_last
    - sort_constructors_first
    # - sort_pub_dependencies # prevents separating pinned transitive dependencies
    - sort_unnamed_constructors_first
    - test_types_in_equals
    - throw_in_finally
    - tighten_type_of_initializing_formals
    # - type_annotate_public_apis # subset of always_specify_types
    - type_init_formals
    # - unawaited_futures # too many false positives, especially with the way AnimationController works
    - unnecessary_await_in_return
    - unnecessary_brace_in_string_interps
    - unnecessary_const
    - unnecessary_constructor_name
    # - unnecessary_final # conflicts with prefer_final_locals
    - unnecessary_getters_setters
    # - unnecessary_lambdas # has false positives: https://github.com/dart-lang/linter/issues/498
    - unnecessary_late
    - unnecessary_new
    - unnecessary_null_aware_assignments
    - unnecessary_null_aware_operator_on_extension_on_nullable
    - unnecessary_null_checks
    - unnecessary_null_in_if_null_operators
    - unnecessary_nullable_for_final_variable_declarations
    - unnecessary_overrides
    - unnecessary_parenthesis
    # - unnecessary_raw_strings # what's "necessary" is a matter of opinion; consistency across strings can help readability more than this lint
    - unnecessary_statements
    - unnecessary_string_escapes
    - unnecessary_string_interpolations
    - unnecessary_this
    - unnecessary_to_list_in_spreads
    - unrelated_type_equality_checks
    - unsafe_html
    - use_build_context_synchronously
    # - use_colored_box # not yet tested
    # - use_decorated_box # not yet tested
    # - use_enums # not yet tested
    - use_full_hex_values_for_flutter_colors
    - use_function_type_syntax_for_parameters
    - use_if_null_to_convert_nulls_to_bools
    - use_is_even_rather_than_modulo
    - use_key_in_widget_constructors
    - use_late_for_private_fields_and_variables
    - use_named_constants
    - use_raw_strings
    - use_rethrow_when_possible
    - use_setters_to_change_properties
    # - use_string_buffers # has false positives: https://github.com/dart-lang/sdk/issues/34182
    - use_super_parameters
    - use_test_throws_matchers
    # - use_to_and_as_if_applicable # has false positives, so we prefer to catch this by code-review
    - valid_regexps
    - void_checks
```

:::

どういった一貫性を持てばいいのか、どういった形式に沿って書けばいいのかということに関しては以下が参考になるかもしれません。とくに、Effective Dart は Dart を扱う開発者にとって必読と思います。

- [**Effective Dart**](https://dart.dev/guides/language/effective-dart)
- [Linter for Dart](https://dart-lang.github.io/linter/lints/)

### analysis_options.yaml の書き方

Dart の静的解析は Analyzer と Linter から構成されます。

- Analyzer: コードに言語仕様で指定されているエラーや警告などのバグがないかを解析する。
- Linter: コードがスタイルガイドラインに準拠しているかを解析する。

この 2 つに加えて、`include: [パッケージ]`と書くことでパッケージの設定項目を利用できます。
もちろん include した設定項目を上書きしたり一部を除外したりすることも可能です。

1 つ 1 つ設定するよりも、有名なパッケージを include して必要に応じて追記する方針でいいと思います。以下は選択肢になりそうなパッケージです。

厳しめな設定のパッケージ

- [pedantic_mono](https://github.com/mono0926/pedantic_mono/blob/main/example/analysis_options.yaml)（作者さまによる記事: [Dart/Flutter の静的解析強化のススメ](https://medium.com/flutter-jp/analysis-b8dbb19d3978)）

Dart チーム推奨のパッケージ

- [lint](https://pub.dev/packages/lint)

Flutter チーム推奨のパッケージ（旧`pedantic`、Flutter 2.5 から標準搭載！）

- [flutter_lints](https://pub.dev/packages/flutter_lints)

今の所 linter のルールは追記していませんが、analyzer に Freezed などの自動生成系のファイルを除外設定しています。これらのファイルは編集しませんし、変に改変されても嫌なので。

私の`analysis_options.yaml`はこんな感じです。

```yaml
...略...

include: package:pedantic_mono/analysis_options.yaml

analyzer:
  exclude:
    - lib/**/*.gql.dart
    - lib/**/*.freezed.dart

linter:
  rules:

...略...
```

その他の個別で設定してもいいなと思うのはこのあたりですかね。

```yaml
...略...

analyzer:
  exclude:
    ...略...

  strong-mode:
    # 暗黙的な型変換を禁止する
    implicit-casts: false

  errors:
    # 何らかのルールのレベルを変更できる
    # 下記は unused_import の各レベルにおける VS Code 上の挙動
    unused_import: info     # 画像1
    unused_import: warning  # 画像2
    unused_import: error    # 画像3

linter:
  rules:

...略...
```

:::details 設定した重要度ごとの結果
![画像1. "info" Linterルールはデフォルトでこちらです](https://storage.googleapis.com/zenn-user-upload/9f4e3b496c1f-20220920.png 250x)
_画像 1. "info" Linter ルールはデフォルトでこちらです_

![画像2. "warning"](https://storage.googleapis.com/zenn-user-upload/bf89dc0afc04-20220920.png 250x)
_画像 2. "warning"_

![画像3. "error" こちらはビルドすらできなくなります](https://storage.googleapis.com/zenn-user-upload/290602aaecb5-20220920.png 250x)
_画像 3. "error" こちらはビルドすらできなくなります_
:::

![](https://storage.googleapis.com/zenn-user-upload/422e65ca546b-20220920.png)

![](https://storage.googleapis.com/zenn-user-upload/4218552b76eb-20220920.png)

![](https://storage.googleapis.com/zenn-user-upload/433b5b61262d-20220920.png)

## 参考

- [Dart/Flutter の静的解析強化のススメ](https://medium.com/flutter-jp/analysis-b8dbb19d3978)
- [Flutter でのフォーマットと静的解析（Linting）の使い方](https://zenn.dev/riemonyamada/articles/f1ca080007d40a#%E3%83%95%E3%82%A9%E3%83%BC%E3%83%9E%E3%83%83%E3%83%88)
- [Effective Dart](https://dart.dev/guides/language/effective-dart)
- [Linter for Dart](https://dart-lang.github.io/linter/lints/)
- [import_sorter](https://pub.dev/packages/import_sorter)
