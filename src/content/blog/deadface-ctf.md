---
author: Jean Chaput
pubDatetime: 2023-10-21T7:22:00Z
title: Write-up Deadface CTF
postSlug: deadface-ctf
featured: true
draft: false
tags:
  - ctf
  - write-up
description:
  Write-up de la Deadface CTF du 20 et 21 octobre 2023
---

## Table des matières

## Reverse-engineering

### Cereal Killer 05

Pour cette épreuve nous avons affaire à un exécutable (`elf` ou `exe` au choix). On télécharge l'exécutable au format `elf` et on le rend exécutable :

```sh
chmod +x re05.bin
```

On peut maintenant le lancer voir ce qu'il se produit :

```sh
./re05.bin
```

Sans surprise on nous demande un mot de passe. On quitte l'exécution et on utilise la commande `strings` pour afficher les chaînes de caractères imprimables dans le fichier :

```sh
strings re05.bin
```

En remontant un peu dans le terminal on aperçoit le texte suivant :

```md
Dr. Geschichter, just because he is evil, doesn't mean he doesn't have a favorite cereal.
Please enter the passphrase, which is based off his favorite cereal and entity:
notf1aq{you-guessed-it---this-is-not-the-f1aq}
Xen0M0rphMell0wz
```

Il est probable que le mot de passe entré dans le terminal lors de l'exécution du programme soit comparé avec une chaîne de caractères et celle-ci pourrait bien être `Xen0M0rphMell0wz`. On relance donc le programme en entrant la chaîne précédemment trouvée lorsque le mot de passe est demandé. Bingo c'est le flag : `flag{XENO-DO-DO-DO-DO-DO-DOOOOO}`.

## Traffic analysis

### Sometimes it lets you down

### Git rekt

### Creepy crawling

Pour cette épreuve, on démarre avec la capture réseau `PCAP02.pcapng`. On peut simplement commencer par en afficher le contenu avec `tshark` :

```sh
tshark -r PCAP02.pcapng
```

On voit alors qu'il y a beaucoup de contenu et différents protocoles. Par curiosité on peut compter le nombre de lignes :

```sh
tshark -r PCAP02.pcapng | wc -l
```

Cela nous affiche 13631 lignes, ce qui fait effectivement beaucoup. Pour simplifier sachant que l'on recherche une version d'SSH on peut filtrer la sortie avec l'option `-Y` de `tshark` :

```sh
tshark -r PCAP02.pcapng -Y "ssh" | wc -l
```

Cela fait encore 991 paquets. De là on pourrait directement commencer à regarder à la main pour voir si quelque chose nous saute aux yeux mais puisque l'on sait que l'on recherche une version de protocole sous la forme `SSH-...`, on peut filtrer la sortie de la commande précédente avec `grep` :

```sh
tshark -r PCAP02.pcapng -Y "ssh" | grep "SSH-" | wc -l
```

Cette fois-ci, on tombe à 141 paquets. Regardons un peu ce qu'il y a dans la capture :

```md
...
 7360 1272.457814  10.10.10.80 → 10.10.10.50  SSH 78 Client: Protocol (SSH-2.0-libssh2_1.10.0)
 7362 1272.457835  10.10.10.80 → 10.10.10.50  SSH 78 Client: Protocol (SSH-2.0-libssh2_1.10.0)
 7363 1272.457835  10.10.10.80 → 10.10.10.50  SSH 78 Client: Protocol (SSH-2.0-libssh2_1.10.0)
 7430 1275.322495  10.10.10.50 → 10.10.10.80  SSH 111 Server: Protocol (SSH-2.0-9.29 FlowSsh: Bitvise SSH Server (WinSSHD) 9.29)
 7433 1275.370603  10.10.10.80 → 10.10.10.50  SSH 78 Client: Protocol (SSH-2.0-libssh2_1.10.0)
 7457 1275.421885  10.10.10.50 → 10.10.10.80  SSH 111 Server: Protocol (SSH-2.0-9.29 FlowSsh: Bitvise SSH Server (WinSSHD) 9.29)
 7459 1275.423011  10.10.10.50 → 10.10.10.80  SSH 111 Server: Protocol (SSH-2.0-9.29 FlowSsh: Bitvise SSH Server (WinSSHD) 9.29)
 7461 1275.423866  10.10.10.50 → 10.10.10.80  SSH 111 Server: Protocol (SSH-2.0-9.29 FlowSsh: Bitvise SSH Server (WinSSHD) 9.29)
 7463 1275.424656  10.10.10.50 → 10.10.10.80  SSH 111 Server: Protocol (SSH-2.0-9.29 FlowSsh: Bitvise SSH Server (WinSSHD) 9.29)
 7465 1275.425436  10.10.10.50 → 10.10.10.80  SSH 111 Server: Protocol (SSH-2.0-9.29 FlowSsh: Bitvise SSH Server (WinSSHD) 9.29)
 7467 1275.426277  10.10.10.50 → 10.10.10.80  SSH 111 Server: Protocol (SSH-2.0-9.29 FlowSsh: Bitvise SSH Server (WinSSHD) 9.29)
 7469 1275.427042  10.10.10.50 → 10.10.10.80  SSH 111 Server: Protocol (SSH-2.0-9.29 FlowSsh: Bitvise SSH Server (WinSSHD) 9.29)
 7480 1275.434033  10.10.10.50 → 10.10.10.80  SSH 111 Server: Protocol (SSH-2.0-9.29 FlowSsh: Bitvise SSH Server (WinSSHD) 9.29)
 7482 1275.434869  10.10.10.50 → 10.10.10.80  SSH 111 Server: Protocol (SSH-2.0-9.29 FlowSsh: Bitvise SSH Server (WinSSHD) 9.29)
 7484 1275.435663  10.10.10.50 → 10.10.10.80  SSH 111 Server: Protocol (SSH-2.0-9.29 FlowSsh: Bitvise SSH Server (WinSSHD) 9.29)
 7487 1275.482577  10.10.10.80 → 10.10.10.50  SSH 78 Client: Protocol (SSH-2.0-libssh2_1.10.0)
 7488 1275.482577  10.10.10.80 → 10.10.10.50  SSH 78 Client: Protocol (SSH-2.0-libssh2_1.10.0)
 7489 1275.482577  10.10.10.80 → 10.10.10.50  SSH 78 Client: Protocol (SSH-2.0-libssh2_1.10.0)
 7490 1275.482609  10.10.10.80 → 10.10.10.50  SSH 78 Client: Protocol (SSH-2.0-libssh2_1.10.0)
...
```

On reconnaît là le format que l'on était en train de rechercher et il se pourrait bien que `SSH-2.0-9.29 FlowSsh: Bitvise SSH Server (WinSSHD) 9.29` soit notre flag. À ce moment là on est sûr de rien mais on peut essayer `flag{SSH-2.0-9.29 FlowSsh: Bitvise SSH Server (WinSSHD) 9.29}`.

Et hop ! Défi validé.

## Forensics

### What's the wallet

Ce défi commence avec un fichier `Bitcoin.txt` où on nous demande de retrouver l'addresse du porte monnaie. Lorsque l'on parcourt le fichier, une fonction attire l'oeil :

```js
function Store-BtcWalletAddress {
  `$global:BtcWalletAddress = [System.Convert]::FromBase64String([System.Text.Encoding]::UTF8.GetBytes('bjMzaGE1bm96aXhlNnJyZzcxa2d3eWlubWt1c3gy'))
}
```

Il paraît évident à ce moment là que l'on récupère l'addresse du porte monnaie en la convertissant depuis la base 64. On met donc la chaîne de caractères `bjMzaGE1bm96aXhlNnJyZzcxa2d3eWlubWt1c3gy` dans un convertisseur de base 64 et on obtient le contenu du flag. On peut donc valider ce défi avec le flag : `flag{n33ha5nozixe6rrg71kgwyinmkusx2}`.

### Host busters 1

### Tin balloon
