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

Lorsque l'on change d'ordinateur ou de distribution régulièrement, ce n'est pas très convénient de copier ses fichiers de configuration à la main. Il existe un tas de solutions proposant d'automatiser le procédé cependant j'ai décidé d'utiliser celle que j'ai vu dans [cet article](https://www.atlassian.com/git/tutorials/dotfiles) car  je la trouve plutôt simple et élégante. Le seul outil nécessaire est `git` (avec un dépôt associé).

## Table des matières

## Mise en place

Avant tout de chose il est nécessaire d'avoir `git` installé et d'avoir créé un dépôt sur la plateforme de son choix (Github de mon côté).

On exécute ensuite les commandes :

```sh
git init --bare ~/.dotfiles
alias config='git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'
config config status.showUntrackedFiles no
config remote add origin <url-to-git-repository>
```

La deuxième commande peut être insérée dans le fichier ou sont déclarés les alias sur notre machine pour la persistence (`.bashrc` ou `.zshrc` probablement).

Et c'est tout !

## Réplication

Pour dupliquer la configuration sur une nouvelle machine on exécute les commandes :

```sh
alias config='git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'
git clone --bare <url-to-git-repository> ~/.dotfiles
config checkout
config config status.showUntrackedFiles no
```

## Au quotidien

Les commandes `git` de base sont utilisées pour versionner les fichiers (ajout ou modification) mais en utilisant l'alias `config`  :

```sh
config add <file>
config commit -m <message>
config push
```

Pour récupérer les modifications effectuées sur un autre ordinateur, on exécute la commande :

```sh
config pull
```