- [Xcode でデバッグしよう（入門：ブレークポイント、ウォッチポイント、LLDB、、、）](https://qiita.com/chan_naruwo/items/61da7ee0f8dfb8d8132c)

viewController <=> storyboard とつながっている。

## 設定 / ショートカット

ビルド成功・失敗時に音を鳴らす。

1. Preferences > Behaviors > Build
2. Succeeds と Fails の`Play sound`にチェックボックスをつけ、それぞれの音を設定する。

選択した行のフォーマット。

1. `control + i`
   - 複数行に渡る場合は、範囲選択しないとだめ。

文字の折返しをしないようにする。

1. Menu > Editor > `Wrap Lines`

## エラー対処法

```
attach failed (Not allowed to attach to process.  Look in the console messages (Console.app), near the debugserver entries, when the attach failed.  The subsystem that denied the attach permission will likely have logged an informative message about why it was denied.)
```

- Xcode 再起動
