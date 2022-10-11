# dotfiles

## alias

```zsh
alias vi="vim"
alias cat="bat"

################################# exa #################################
alias e='exa --icons --git'
alias l=e
alias ls=e
alias la='exa -a --icons --git'
alias ll='exa -aahl --icons --git'
alias lt='exa -T -L 3 -a -I "node_modules|.git|.cache" --icons'
alias lta='exa -T -a -I "node_modules|.git|.cache" --color=always --icons | less -r'
alias l='clear && ls'

############################### AtCoder ###############################
alias ojt="oj t -c \"python3 main.py\" -d ./tests/"
alias py="python3 main.py"
alias submit="acc submit main.py"

################################# git #################################
alias g="git"
alias gs="git status"
alias gb="git branch"
alias gc="git checkout"
alias gct="git commit"
alias gg="git grep"
alias ga="git add"
alias gd="git diff"
alias gl="git log"
alias gcma="git checkout main"
alias gpo="git push origin"
alias gpom="git push origin main"
alias go="git open"

################################# man #################################
alias eman="env LANG=C man"
alias man="env LANG=ja_JP.UTF-8 man"

# Python2.x廃止の対策
alias brew='env PATH="${PATH//$(pyenv root)\/shims:/}" brew'
```

## plugin

starship, bat, exa に関しては brew でインストールしておく。

```zsh
CURRENT_DIR_PATH=${0:a:h}

# starship
zinit ice wait lucid as"program" from"gh-r" mv"starship* -> starship"
zinit light starship/starship

# syntax highlight
zinit ice wait lucid
zinit light zdharma-continuum/fast-syntax-highlighting

#auto suggest
zinit light zsh-users/zsh-autosuggestions
bindkey '^k' autosuggest-accept

# bat
zinit ice wait lucid as"program" from"gh-r" mv"bat* -> bat" pick"bat/bat"
zinit light sharkdp/bat

# git-open
zinit light paulirish/git-open

# k
zinit ice wait lucid
zinit light supercrabtree/k
```
