# Flutter 用設定

最終更新：2022/09/28

```yaml
pre-commit:
  parallel: false
  commands:
    linter:
      run: dart fix --apply lib && git add {staged_files}
    sort-imports:
      glob: '*.dart'
      run: flutter pub run import_sorter:main {staged_files} && git add {staged_files}
    formatter:
      glob: '*.dart'
      run: dart format {staged_files} && git add {staged_files}
    prettier:
      run: npm run prettier && git add {staged_files}
```
