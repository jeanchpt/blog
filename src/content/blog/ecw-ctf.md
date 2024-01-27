---
author: Jean Chaput
pubDatetime: 2023-10-30T10:53:00Z
title: ECW CTF Write-up
postSlug: ecw-ctf
featured: false
draft: false
tags:
  - ctf
  - write-up
description:
  Write-up for the ECW CTF (October 13 to 29, 2023)
---

The ECW CTF took place from October 13 to 29, 2023. I offer here a correction of two challenges. More corrections will be offered by a teammate on [this website](https://www.enzo-cadoni.fr). The late discovery of these qualifications in the middle of a university project did not allow me to do more.

## Table of contents

## Forensics

### DumpCyber

For this forensics challenge, we start with a disk image of an unknown system. To begin with, we can try to find out more about the system from which the image in question comes from :

```sh
sudo ~/Tools/volatility3-2.5.0/vol.py -f task.raw windows.info
```

We have the following result :

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

Since this is a Windows disk image, you can use the `windows.filescan.FileScan` module to list its files. For simplicity's sake, we'll redirect the output to a text file that can be browsed later :

```sh
sudo ~/Tools/volatility3-2.5.0/vol.py -f task.raw windows.filescan.FileScan > files.txt
```

Now that we have a list of the files contained in the filesystem, we want to look out to the user's directories. To do this, we list the user directory as follows :

```sh
cat files.txt | grep '\\Users'
```

We realize that there's only one user, `vboxuser`, and by browsing through his files, it seems that there are some promising things on his desktop. So we list its contents :

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

Now that we have a list of files and their addresses, we can proceed to retrieve them :

```sh
sudo ~/Tools/volatility3-2.5.0/vol.py -f task.raw -o ./ windows.dumpfiles.DumpFiles --physaddr 0x3fab94a0
sudo ~/Tools/volatility3-2.5.0/vol.py -f task.raw -o ./ windows.dumpfiles.DumpFiles --physaddr 0x3fa18070
sudo ~/Tools/volatility3-2.5.0/vol.py -f task.raw -o ./ windows.dumpfiles.DumpFiles --physaddr 0xef58430
```

Since these are `.rar` files, we proceed to extract them :

```sh
unrar x file.txt.rar
unrar x generator.rar
unrar x file.rar
```

The result is a file called `file.txt.enc` which appears to be an encrypted file. This is not very surprising, since we are asked to retrieve a key and an initialization vector. In addition, we have two executables: `generator.exe` and `encryptor.exe`.

At that point, there were several different ways of proceeding, but for my part, I simply ran the generator :

```sh
wine generator.exe
```

This produces two files : `dessktop.ini` and `deesktop.ini`. This is a very good lead, since we remember seeing these files on `vboxuser`'s desktop. We now proceed to extract these two files, one of which should be our key and the other our initialization vector :

```sh
sudo ~/Tools/volatility3-2.5.0/vol.py -f task.raw -o ./ windows.dumpfiles.DumpFiles --physaddr 0x3fcd2430
sudo ~/Tools/volatility3-2.5.0/vol.py -f task.raw -o ./ windows.dumpfiles.DumpFiles --physaddr 0x3fd737b0
```

To find out which is the key and which is the initialization vector, we could have decompiled the generator, or simply tried one after the other. Let's try AES decryption of the following cipher on [Cyberchef](https://cyberchef.org/) :

```md
e609874a80d1b27efd46cc0a131a4dcd01a866f5e31a99ae56ed91d8c517
5c8b88a0d8d8c6412889bac60d5999cf61517bfdbf7f9fc6e2fa530e4263
bd92af6a
```

Our key is `1b54ee420bd5b8396e15fc9fe01055f8` and the initialization vector is `e609874a80d1b27efd46cc0a131a4dcd`. Setting decryption to `CBC/NoPadding` yields the following plaintext :

```md 
=À;K	õ®AlñÐ 0flag{82a30fadcfc07d634fbed1bffe4a2aa1}
```

We have our flag : `flag{82a30fadcfc07d634fbed1bffe4a2aa1}`.

## Cryptography

### Random_key

This cryptographic challenge lies around two files.

The first one is a **Python** script :

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

We quickly realize that, to encrypt the flag, Alice has used `FEDCBA9876543210` as an initialization vector and a key generated by the function `generate_256bits_encryption_key(unsigned char *recipient_name)` with the parameter `Control_center`.

If we take a closer look at this function, we can see that the key creation process is as follows :
1. Initialization of the key with random values on the 32 bytes of `key`.
2. Compute the MD5 of `Control_center` and insert it into the first 16 bytes of `key`.
3. Insertion of the MD5 calculation time into the `key` index 8 cell (9th cell).

This procedure presents a problem. As the MD5 calculation is faster than one second, the `O` character will always be inserted in `key[8]`. Since the null byte is the **C** end-of-string character and the key is retrieved as such by the **Python** script, the key is therefore half the MD5 of `Control_center`.

```md
echo -n "Control_center" | md5sum
134abb7bd9d248a98d914daea81a8737  -
```
So we try to decrypt the flag on our favorite [Cyberchef](https://cyberchef.org/) with initialization vector `FEDCBA9876543210` and key `134ABB7BD9D248A9` entered in `UTF-8`. Applying the `CBC/NoPadding` algorithm gives us the following flag: `ECW{random_key_7AgmwlBXo1tDhyqR}`.

And that's it, challenge validated.