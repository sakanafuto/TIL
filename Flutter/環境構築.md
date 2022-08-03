# 環境構築

update at 2022/08/03

## FVM

Flutterはアプデが頻繁にあり問題が生まれそうなので、バージョン管理ができる[FVM](https://fvm.app/)を用いる。Dartもできればアンインストールして、HomebrewからインストしたFVMでプロジェクトを始めたほうが良さげ。

## Xcode

AppStoreからからインストすればよし。

## Android Studio

[公式サイト](https://developer.android.com/studio)からインスト。安定版。

FlutterとDartのプラグインを入れる。
ちなみにAndroid StudioはIDEとしても使えるが、私はVS Code派。理由はずっと使ってたから。
VS Codeの場合、FVMで落としたFlutter SDKを使うように設定する必要がある。方法はいろいろあるが、公式推奨のAutomatic Switching方式で設定。

https://fvm.app/docs/getting_started/configuration#option-1---automatic-switching-recommended

## Project

ここまでできたらいったんプロジェクトはじめてみる。
基本的にFlutterとかDartコマンドの前に`fvm`とつけてあげればいい。

基本コマンド

```bash
# プロジェクトルートで
fvm use <stable, 3.0.1, ...etc>

# ただこれはプロジェクト内でしか打てない。新規はこちら
fvm use <stable, 3.0.1,...etc> --force
fvm flutter create <myapp>

# インストール
fvm install <stable, 3.0.1, ...etc>

# インスト済み
fvm list

# info
fvm doctor

# マシンにグローバル設定
fvm global <stable, 3.0.1, ...etc>


```