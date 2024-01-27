---
author: Jean Chaput
pubDatetime: 2023-10-21T7:22:00Z
title: How to manage your dotfiles ?
postSlug: dotfiles
featured: false
draft: false
tags:
  - configuration
description:
  My method to save and replicate my config files on linux
---

When you change computer or distribution regularly, it's not very convenient to copy your configuration files by hand. There are lots of solutions to automate the process, but I decided to use the one I saw in [this article](https://www.atlassian.com/git/tutorials/dotfiles) as I find it rather simple and elegant. The only tool needed is `git` (with an associated repository).

## Table of contents

## Instructions

First of all, you need to have `git` installed and have created a repository on the platform of your choice (Github on my side).

Then run the commands :

```sh
git init --bare ~/.dotfiles
alias config='git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'
config config status.showUntrackedFiles no
config remote add origin <url-to-git-repository>
```

The second command can be inserted into the file where aliases are declared on our machine for persistence (probably `.bashrc` or `.zshrc`).

And that's all !

## Replication

To replicate your configuration :

```sh
alias config='git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'
git clone --bare <git-repo-url> ~/.dotfiles
config checkout
config config status.showUntrackedFiles no
```

## Manage files

To manage your dotfiles :

```sh
config status
config add <file>
config commit -m <message>
config push
```

To retrieve your modifications from another computer :

```sh
config pull
```