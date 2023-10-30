---
author: Jean Chaput
pubDatetime: 2023-10-30T10:53:00Z
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

La ECW CTF s'est déroulée du 13 au 29 octobre 2023 et je propose ici une correction de deux épreuves. Plus de corrections seront proposées par un coéquipier sur [ce site web](https://www.enzo-cadoni.fr). La découverte tardive du déroulé de ces qualifications en pleine période de projet universitaire ne m'aura pas permis d'en faire plus.

## Table des matières

## Forensics

### DumpCyber

Pour cette épreuve de forensics, on démarre avec une image disque d'un système inconnu. Pour commencer, on peut essayer d'en savoir plus sur le système d'où provient l'image en question :

```sh
sudo ~/Tools/volatility3-2.5.0/vol.py -f task.raw windows.info
```

Le résultat affiché est le suivant.

```md
Volatility 3 Framework 2.5.0
Progress:  100.00		PDB scanning finished                        
Variable	Value

Kernel Base	0xf8000260c000
DTB	0x187000
Is64Bit	True
IsPAE	False
layer_name	0 WindowsIntel32e
memory_layer	1 FileLayer
KdDebuggerDataBlock	0xf800027fd0a0
NTBuildLab	7601.17514.amd64fre.win7sp1_rtm.
CSDVersion	1
KdVersionBlock	0xf800027fd068
Major/Minor	15.7601
MachineType	34404
KeNumberProcessors	1
SystemTime	2023-08-17 16:20:26
NtSystemRoot	C:\Windows
NtProductType	NtProductWinNt
NtMajorVersion	6
NtMinorVersion	1
PE MajorOperatingSystemVersion	6
PE MinorOperatingSystemVersion	1
PE Machine	34404
PE TimeDateStamp	Sat Nov 20 09:30:02 2010
```

Étant donné qu'il s'agit d'une image disque windows, on peut utiliser le module `windows.filescan.FileScan` pour en lister les fichiers. Pour plus de simplicité, on redirige la sortie vers un fichier texte que l'on pourra parcourir par la suite :

```sh
sudo ~/Tools/volatility3-2.5.0/vol.py -f task.raw windows.filescan.FileScan > files.txt
```

Maintenant que l'on a une liste des fichiers contenus dans le système, on a envie d'aller voir du côté des répertoires utilisateurs car c'est souvent ici que l'on trouve des choses intéressantes. Pour cela, on liste le répertoire utilisateur comme suit :

```sh
cat files.txt | grep '\\Users'
```

On se rend compte qu'il y a un seul utilisateur `vboxuser` et en parcourant un peu ses fichiers il semble qu'il y ai des choses prometteuses sur son bureau. On liste donc le contenu de celui-ci :

```sh
cat files.txt | grep '\\Users\\vboxuser\\Desktop\\'
```

```md
0x2fae8c0	\Users\vboxuser\Desktop\desktop.ini		216
0xef58430	\Users\vboxuser\Desktop\file.rar		216
0x3fa144a0	\Users\vboxuser\Desktop\generator.rar	216
0x3fa18070	\Users\vboxuser\Desktop\file.txt.rar	216
0x3fab94a0	\Users\vboxuser\Desktop\generator.rar	216
0x3fabd070	\Users\vboxuser\Desktop\file.txt.rar	216
0x3fb5e4a0	\Users\vboxuser\Desktop\generator.rar	216
0x3fb62070	\Users\vboxuser\Desktop\file.txt.rar	216
0x3fcd2430	\Users\vboxuser\Desktop\dessktop.ini	216
0x3fd737b0	\Users\vboxuser\Desktop\deesktop.ini	216
```

Maintenant que l'on dispose d'une liste de fichiers et de leurs adresses, on peut procéder à la récupération de ceux-ci :

```sh
sudo ~/Tools/volatility3-2.5.0/vol.py -f task.raw -o ./ windows.dumpfiles.DumpFiles --physaddr 0x3fab94a0
sudo ~/Tools/volatility3-2.5.0/vol.py -f task.raw -o ./ windows.dumpfiles.DumpFiles --physaddr 0x3fa18070
sudo ~/Tools/volatility3-2.5.0/vol.py -f task.raw -o ./ windows.dumpfiles.DumpFiles --physaddr 0xef58430
```

Étant donné qu'il s'agit de fichiers `.rar`, on procède à leur extraction :

```sh
unrar x file.txt.rar
unrar x generator.rar
unrar x file.rar
```

On se retrouve donc avec un fichier `file.txt.enc` qui semble être un fichier chiffré. Rien d'étonnant puisque l'on nous demande de retrouver une clé et un vecteur d'initialisation. En plus de cela, on dispose de deux exécutables : `generator.exe` et `encryptor.exe`.

A ce moment là on aurait pu procéder de plusieurs manières différentes mais pour ma part, j'ai simplement exécuté le générateur :

```sh
wine generator.exe
```

Cela nous produit deux fichiers : `dessktop.ini` et `deesktop.ini`. C'est une très bonne piste puisqu'on se rappelle avoir vu ces fichiers sur le bureau de `vboxuser`. On passe donc à l'extraction de ces deux fichiers dont l'un doit-être notre clé et l'autre le vecteur d'initialiation :

```sh
sudo ~/Tools/volatility3-2.5.0/vol.py -f task.raw -o ./ windows.dumpfiles.DumpFiles --physaddr 0x3fcd2430
sudo ~/Tools/volatility3-2.5.0/vol.py -f task.raw -o ./ windows.dumpfiles.DumpFiles --physaddr 0x3fd737b0
```

Pour savoir lequel est la clé et lequel est le vecteur d'initialisation on aurait pû décompiler le générateur, ou bien simplement essayer à tour de rôle. On essaye donc le déchiffrement AES du chiffré suivant sur le [Cyberchef](https://cyberchef.org/) :

```md
e609874a80d1b27efd46cc0a131a4dcd01a866f5e31a99ae56ed91d8c517
5c8b88a0d8d8c6412889bac60d5999cf61517bfdbf7f9fc6e2fa530e4263
bd92af6a
```

Notre clé est `1b54ee420bd5b8396e15fc9fe01055f8` et le vecteur d'initialisation `e609874a80d1b27efd46cc0a131a4dcd`. En réglant le déchiffrement sur `CBC/NoPadding`, on obtient le texte clair suivant :

```md 
=À;K	õ®AlñÐ 0flag{82a30fadcfc07d634fbed1bffe4a2aa1}
```

On a donc notre flag : `flag{82a30fadcfc07d634fbed1bffe4a2aa1}`.

## Cryptographie

### Random_key

Cette épreuve de cryptographie tourne autour de deux fichiers.

Le premier est un script **Python** :

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

from ctypes import *
from Crypto.Cipher import AES

SO_FILE = './generate_key.so'
FLAG_FILE = '../../flag.txt'

with open(FLAG_FILE, 'r') as f:
	FLAG = f.read().encode()

def encrypt(plaintext, key):
	return AES.new(key, AES.MODE_CBC, b'FEDCBA9876543210').encrypt(plaintext)

if __name__ == '__main__':
	print("""Un message chiffré a été envoyé par l'IA ALICE à son centre de contrôle. Vous avez réussi à mettre la main sur certains extraits de code utilisés par ALICE pour chiffrer son message ainsi que sur le texte chiffré. Votre mission est de retrouver le message en clair.""")
	
	generate_256bits_encryption_key = CDLL(SO_FILE).generate_256bits_encryption_key
	generate_256bits_encryption_key.restype = c_char_p
	key = generate_256bits_encryption_key(b'Control_center').hex().encode()
	
	enc = encrypt(FLAG, key)
	print('Enc:', enc.hex())
	
	# Enc: 21952f9ced6c9109f8ce7c41cd3e0e6981c97a84745d5fdc75b2584e9a5a05e0
```

Le deuxième est un code en **C** :

```c
unsigned char key[32] = { 0 };

void md5(unsigned char *in, unsigned char *out)
{
	MD5_CTX ctx;
	MD5_Init(&ctx);
	MD5_Update(&ctx, in, strlen(in));
	MD5_Final(out, &ctx);
}

unsigned char *generate_256bits_encryption_key(unsigned char *recipient_name)
{
	int i = 0;
	FILE *f = NULL;
	time_t now1 = 0L;
	time_t now2 = 0L;
	time_t delta = 0L;
	
	now1 = time(NULL);
	
	f = fopen("/dev/urandom", "rb");
	fread(&key, 1, 32, f);
	fclose(f);
	
	md5(recipient_name, key);
	
	now2 = time(NULL);
	delta = now2 - now1;
	
	key[8] = delta;
	
	return key;
}
```

On se rend rapidement compte que pour chiffrer le flag, Alice a utilisé `FEDCBA9876543210` en vecteur d'initialisation ainsi qu'une clef générée par la fonction `generate_256bits_encryption_key(unsigned char *recipient_name)` avec comme paramètre : `Control_center`.

Quand on regarde d'un peu plus près cette fonction, on voit que le procédé de création de la clef est le suivant :
1. Initialisation de la clef avec des valeurs aléatoires sur les 32 octets de `key`
2. Calcul du MD5 de `Control_center` et insertion dans les 16 premiers octets de `key`
3. Insertion du temps de calcul du MD5 dans la case d'indice 8 de `key` (9ème case)

Ce procédé présente un problème. Le calcul du MD5 étant plus rapide qu'une seconde, le caractère `O` sera toujours inséré dans `key[8]`. Étant donné que l'octet nul est le caractère de fin de chaîne en **C** et que la clé est récupérée comme telle par le script **Python**, la clé est donc la moitié du MD5 de `Control_center`.

```md
echo -n "Control_center" | md5sum
134abb7bd9d248a98d914daea81a8737  -
```
On essaye donc le déchiffrement du flag sur notre [Cyberchef](https://cyberchef.org/) préféré avec comme vecteur d'initialisation `FEDCBA9876543210` et comme clef `134ABB7BD9D248A9` saisis en `UTF-8`. L'application de l'algorithme `CBC/NoPadding` nous donne le flag suivant : `ECW{random_key_7AgmwlBXo1tDhyqR}`.

Et voilà, challenge validé.