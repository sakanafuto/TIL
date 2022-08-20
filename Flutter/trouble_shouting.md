# トラブルシューティング

### fvm flutter doctorで指摘

> Android toolchain ...

- Android Studio > Preferences > Appearance & Behavior > Android SDK > SDK Tools > Android SDK Command-line Tools (latest) にチェック。
- fvm flutter doctor --android-licenses → 全部Yes

> cocoapods not installed ...

- Homebrewでインストする
- simulator、**IDE再起動**
- reinstall
- brew unlink cocoapods && brew link cocoapods
