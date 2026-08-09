Git needs a `.git` folder inside the project folder. A bare repo lets you put the `.git` folder somewhere else (like `~/.dotfiles`) while treating your entire home directory as the working tree. That way you don't need to move or symlink anything — your configs stay exactly where they are.

```
git init --bare $HOME/.dotfiles
```

```
alias dotfiles='git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'
```

```
dotfiles config --local status.showUntrackedFiles no
```

This line stops Git from showing every untracked file in your home directory, you only want to see files you explicitly add.