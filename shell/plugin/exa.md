# exa

Rustで書かれている。lsの代用です。
設定しているエイリアスは以下の通り。

```shell
   5   ｜ alias e='exa --icons --git'
   6   │ alias l=e
   7   │ alias ls=e
   8   │ alias la='exa -a --icons --git'
   9   │ alias ll='exa -aahl --icons --git'
  10   │ alias lt='exa -T -L 3 -a -I "node_modules|.git|.cache" --icons'
  11   │ alias lta='exa -T -a -I "node_modules|.git|.cache" --color=always --icons | less -r'
  12   │ alias l='clear && ls'p
```


- [github](https://github.com/ogham/exa)
