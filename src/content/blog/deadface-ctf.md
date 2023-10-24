---
author: Jean Chaput
pubDatetime: 2023-10-24T21:36:00Z
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

La Deadface CTF s'est déroulée les 20 et 21 octobre 2023 et je propose ici une correction de quelques épreuves. D'autres corrections seront proposées par un coéquipier sur [ce site web](https://www.enzo-cadoni.fr).

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

### Git rekt

Ce défi propose d'analyser une capture réseau pour essayer de retrouver le mot de passe de l'utilisateur `spookyboi` qui se serait fait avoir par une campagne de phishing. On commence par ouvrir la capture avec `wireshark` et on se rend compte que celle-ci contient énormément de paquets et de protocoles différents. 

Puisque le mot de passe recherché aurait été envoyé en réponse à une campagne de phishing, il est probable que cela se soit produit via une interface web alors on peut essayer de filtrer les paquets `HTTP` ainsi que la méthode `POST`. Cela se fait avec le filtre `http.request.method == "POST"`.

Il ne reste plus qu'un seul paquet dans la liste et quand on l'exporte au format texte  on a le résultat suivant :

```md
No.     Time           Source                Destination           Protocol Length Info
   3240 1593.789095    165.227.73.138        147.182.253.207       HTTP     1200   POST /session HTTP/1.1  (application/x-www-form-urlencoded)

Frame 3240: 1200 bytes on wire (9600 bits), 1200 bytes captured (9600 bits)
Ethernet II, Src: fe:00:00:00:01:01 (fe:00:00:00:01:01), Dst: 02:50:e8:ba:d7:30 (02:50:e8:ba:d7:30)
Internet Protocol Version 4, Src: 165.227.73.138, Dst: 147.182.253.207
Transmission Control Protocol, Src Port: 41336, Dst Port: 80, Seq: 1, Ack: 1, Len: 1134
Hypertext Transfer Protocol
HTML Form URL Encoded: application/x-www-form-urlencoded
    Form item: "commit" = "Sign in"
    Form item: "authenticity_token" = "itYs+HLxxadKOu/LcUSIlVEkCT0DBQ6EwYw2TO0D28Za9lQoiAGqbgjQ0p2IewNCvvtRkN0XcJrK5Me1ndYRvw=="
    Form item: "login" = "spookyboi@deadface.io"
    Form item: "password" = "SpectralSecrets#2023"
    Form item: "webauthn-conditional" = "undefined"
    Form item: "javascript-support" = "true"
    Form item: "webauthn-support" = "unsupported"
    Form item: "webauthn-iuvpaa-support" = "unsupported"
    Form item: "return_to" = "https://github.com/login"
    Form item: "allow_signup" = ""
    Form item: "client_id" = ""
    Form item: "integration" = ""
    Form item: "required_field_826d" = ""
    Form item: "timestamp" = "1696980020598"
    Form item: "timestamp_secret" = "701122f4b577941e1c787414ea0775e8cd9e974f8c5b46eceff028a721e9d713"
```

On voit ici clairement le mot de passe transmit et on a par conséquent le flag suivant : `flag{SpectralSecrets#2023}`

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

Pour cette épreuve nous disposons d'un acces SSH à `vim@gh0st404.deadface.io` avec le mot de passe `letmevim`. Lorsque l'on se connecte, il semble que l'on ne se retrouve pas dans un shell classique mais plûtot dans un `vim`. Si l'on essaye simplement d'en sortir avec la commande `:q`, on se déconnecte du SSH. 

Il se trouve que `vim` s'exécute dans un terminal ou il peut exécuter les commandes que l'utilisateur lui donne avec la commande `:!` suivi de n'importe quelle instruction. On peut donc par exemple taper `:!bash`.

Cela nous ouvre un shell bash qui se trouve être directement dans le répertoire de l'utilisateur `vim` ou l'on était censé chercher quelque chose. On liste donc le contenu du répertoire avec `ls` et on découvre qu'un fichier `hostbusters1.txt` s'y trouve. Le flag est donc `flag{esc4P3_fr0m_th3_V1M}`.

### Tin balloon

Cette fois-ci on récupère une archive contenant un fichier `Untitlednosubject.docx` protégé par mot de passe ainsi qu'un fichier MP3. En écoutant le MP3 pendant que l'on essaye de trouver une solution pour le mot de passe, on se rend compte qu'il y a des sons étranges autour de 3 minutes 15. 

On ouvre donc le fichier avec `audacity` et on affiche le spectrogramme avant de zoomer sur la partie qui nous intéresse. On découvre alors ceci :

![Spectrogramme](@assets/images/spectrogram.png)

Il s'agit évidemment du mot de passe du fichier ou se trouve le nom de l'exécutable. On en déduit le flag : `flag{Wh1t3_N01Z3.exe}`