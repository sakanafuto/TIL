[https://github.com/sakanafuto/npm_training](https://github.com/sakanafuto/npm_training)

[そもそも npm からわからない](https://zenn.dev/antez/articles/a9d9d12178b7b2)

[【nvm】node のバージョン管理を nodebrew から nvm に移行する&使い方](https://offlo.in/blog/node-version-nvm.html)

---

## やったこと

1. Node 系全部 Uninstall（Node, Nodebrew, npm など、詳細は[参考](notion://www.notion.so/10tofu01/Develop-6e8622d4897b4cf0860600faace001a5?v=13317267dbc14c74a08f050922222149&p=26b7f01e982b4a02a7803fc0e54cf8cf#%E5%8F%82%E8%80%83)）
2. 公式サイトでも Homebrew でもなく、curl で nvm のインストール

   1. `curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash`
   2. export path

      ```
      export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
      [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
      ```

   3. `source ~/.zshrc`

3. lts (long term support)版の node.js をインストール
   1. ✗ `~~nvm install stable --latest-npm~~`
   2. `nvm install --lts --latest-npm`
4. lts 版をデフォルトに
   1. `nvm alias default 'lts/*'`

## 確認

```
$ which node // /Users/***/.nvm/versions/node/v16.15.1/bin/node
$ brew list // nodeがないか確認

$ nvm ls
->     v16.15.1
default -> lts/* (-> v16.15.1)
iojs -> N/A (default)
unstable -> N/A (default)
node -> stable (-> v16.15.1) (default)
stable -> 16.15 (-> v16.15.1) (default)
lts/* -> lts/gallium (-> v16.15.1)
lts/argon -> v4.9.1 (-> N/A)
lts/boron -> v6.17.1 (-> N/A)
lts/carbon -> v8.17.0 (-> N/A)
lts/dubnium -> v10.24.1 (-> N/A)
lts/erbium -> v12.22.12 (-> N/A)
lts/fermium -> v14.19.3 (-> N/A)
lts/gallium -> v16.15.1

$ cat ~/dotfiles/zsh/01_config.zsh
...
  18   │ # nvm
  19   │ export NVM_DIR="$HOME/.nvm"
  20   │ [ -s "$NVM_DIR/nvm.sh" ] && \\. "$NVM_DIR/nvm.sh"  # This loads nvm
  21   │ [ -s "$NVM_DIR/bash_completion" ] && \\. "$NVM_DIR/bash_completion"  # This loads nvm bash_comp
       │ letion
...
```

## 参考

- [https://zenn.dev/antez/articles/a9d9d12178b7b2](https://zenn.dev/antez/articles/a9d9d12178b7b2)
- [https://offlo.in/blog/node-version-nvm.html](https://offlo.in/blog/node-version-nvm.html)
