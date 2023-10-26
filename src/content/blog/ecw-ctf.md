---
author: Jean Chaput
pubDatetime: 2023-10-24T21:36:00Z
title: Write-up ECW CTF
postSlug: ecw-ctf
featured: true
draft: false
tags:
  - ctf
  - write-up
description:
  Write-up de la ECW CTF
---

## Table des matières

## Forensics

### DumpCyber

```sh
sudo vol -f disk_image.raw windows.filescan.FileScan > files.txt
cat files.txt | grep '\\Users'
cat files.txt | grep '\\Users\\vboxuser\\Desktop\\'
```

```md
0x2fae8c0	\Users\vboxuser\Desktop\desktop.ini	216
0xef58430	\Users\vboxuser\Desktop\file.rar	216
0x3fa144a0	\Users\vboxuser\Desktop\generator.rar	216
0x3fa18070	\Users\vboxuser\Desktop\file.txt.rar	216
0x3fab94a0	\Users\vboxuser\Desktop\generator.rar	216
0x3fabd070	\Users\vboxuser\Desktop\file.txt.rar	216
0x3fb5e4a0	\Users\vboxuser\Desktop\generator.rar	216
0x3fb62070	\Users\vboxuser\Desktop\file.txt.rar	216
0x3fcd2430	\Users\vboxuser\Desktop\dessktop.ini	216
0x3fd737b0	\Users\vboxuser\Desktop\deesktop.ini	216
```

```sh
sudo vol -f disk_image.raw -o ./ windows.dumpfiles.DumpFiles --physaddr 0x3fab94a0
sudo vol -f disk_image.raw -o ./ windows.dumpfiles.DumpFiles --physaddr 0x3fa18070
sudo vol -f disk_image.raw -o ./ windows.dumpfiles.DumpFiles --physaddr 0xef58430
```

unrar x file.txt.rar -> file.txt.enc
unrar x generator.rar -> generator.exe
unrar x file.rar -> encryptor.rar

```md
e609874a80d1b27efd46cc0a131a4dcd01a866f5e31a99ae56ed91d8c517
5c8b88a0d8d8c6412889bac60d5999cf61517bfdbf7f9fc6e2fa530e4263
bd92af6a
```

La clé :

```md
1b54ee420bd5b8396e15fc9fe01055f8
```

Le vecteur d'initialisation :

```md
e609874a80d1b27efd46cc0a131a4dcd
```
Déchiffré :

```md 
=À;K	õ®AlñÐ 0flag{82a30fadcfc07d634fbed1bffe4a2aa1}
```

Cyberchef mode CBC/NoPadding

## Cryptographie
### Random_key

Le Cyberchef

```md
echo -n "Control_center" | md5sum
134abb7bd9d248a98d914daea81a8737  -
```

Cipher : `21952f9ced6c9109f8ce7c41cd3e0e6981c97a84745d5fdc75b2584e9a5a05e0`

Key : `134ABB7BD9D248A9` en UTF-8
IV : `FEDCBA9876543210` en UTF-8
Mode CBC/NoPadding

```md
ECW{random_key_7AgmwlBXo1tDhyqR}
```