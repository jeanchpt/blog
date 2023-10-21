---
author: Jean Chaput
pubDatetime: 2023-10-21T7:22:00Z
title: Comment versionner ses fichiers de configuration
postSlug: dotfiles
featured: false
draft: false
tags:
  - configuration
description:
  Ma méthode pour facilement sauvegarder et répliquer des fichiers de configuration sous Linux.
---

# Pourquoi versionner ses fichiers de configuration ?

When you are switching from one computer to another all the time, it is not really convenient to copy manually your config files like your `.zshrc`, `vimrc` etc... There is plenty of solutions to automate this process but I adopted the one I saw in [this article](https://www.atlassian.com/git/tutorials/dotfiles).

It is a nice one because the only tool you need is `git` (with a Github repository for example).

For the following of this tutorial I will assume that you have `git` installed on a Unix-like operating system.

# Mise en place

First of all you nead a new git repository. I personally use Github for this but it is only a matter of choice.
Then, assuming that your repository is named `dotfiles` we will use the following commands :

```sh
git init --bare ~/.dotfiles
alias config='git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'
config config status.showUntrackedFiles no
config remote add origin git@github.com:<git-username>/dotfiles.git
```

Of course replace `<git-username>` with your own Github username. You can also put the second command into your `.bashrc` or `.zshrc` file for the persistence.

And that's all !

# Réplication

To replicate your configuration on another computer, simply use the following commands :

```sh
alias config='git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'
git clone --bare <git-repo-url> ~/.dotfiles
config checkout
config config status.showUntrackedFiles no
```

This time don't forget to replace `<git-repo-url>` with your repository url.

# Au quotidien

To store some dotfiles you can use the following commands :

```sh
config add <file>
config commit -m <message>
config push
```

To retrieve the modifications made on another computer, use :

```sh
config pull
```

In fact you can use every `git` command by replacing `git` with your `config` alias.