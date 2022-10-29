# Starship

シェルのプロンプトを可愛くできるツール。Rust 製なので高速。

## 必要なもの

- [Nerd Fonts](https://www.nerdfonts.com/)がインストールされていて、ターミナルで有効になっていること。

Nerd Fonts とは、`Font Awesome`、`Devicons`、`Octicons`などの人気のある「アイコンフォント」を表示できるフォントのことである。
Starship はこれらのアイコンフォントを多用するので、好きな Nerd Fonts を入れてあげる。

おすすめは日本語にも適した白源（[HackGen](https://github.com/yuru7/HackGen)）。
↓HackGen フォントをとりまインストールしたい場合 ↓

```shell
$ brew tap homebrew/cask-fonts
$ brew install font-hackgen
$ brew install font-hackgen-nerd
```

## Getting Started

brew からインストールする。

```shell
$ brew install starship
```

~/.zshrc に追記する。

```shell
eval "$(starship init zsh)"
```

設定は`~/.config/starship.toml`から。

```shell
$ mkdir -p ~/.config && touch ~/.config/starship.toml
```

[設定](https://starship.rs/ja-JP/config/)からお好きなようにカスタマイズ。
たしか Starship はリロード（`$ source ~/.zshrc`）しなくてもすぐ反映されたはず。

7ofu の設定はこちら。紆余曲折を経てシンプルにしてます。
iTerm2 に [Dracula](/shell/dracula.md) のカラバリを設定した上での設定です。

[**starship.toml**](/shell/starship/starship.toml)

![](/assets/starship.png)

_Shell_

## 参考

- [starship.rs](https://starship.rs/ja-jp/)
- [HackGen](https://github.com/yuru7/HackGen)
