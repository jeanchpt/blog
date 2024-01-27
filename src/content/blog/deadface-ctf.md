---
author: Jean Chaput
pubDatetime: 2023-10-24T21:36:00Z
title: Deadface CTF Write-up
postSlug: deadface-ctf
featured: false
draft: false
tags:
  - ctf
  - write-up
description:
  Write-up for the Deadface CTF (October 20 and 21, 2023)
---

The Deadface CTF took place on October the 20 and 21, 2023. I propose here a correction for some challenges. Other one will be published by my team mate on [this website](https://www.enzo-cadoni.fr).

## Table of contents

## Reverse-engineering

### Cereal Killer 05

For this challenge, we have to deal with a binary file (we can choose between an `elf` or an `exe`). We download the binary in `elf` format and we make it executable :

```sh
chmod +x re05.bin
```

Now we can lauch it to see what it does :

```sh
./re05.bin
```

Without a surprise, we are asked for a password. So we left the execution and we try the `strings` command to show printable strings present in the file :

```sh
strings re05.bin
```

When we go up in the terminal, we can see the following text :

```md
Dr. Geschichter, just because he is evil, doesn't mean he doesn't have a favorite cereal.
Please enter the passphrase, which is based off his favorite cereal and entity:
notf1aq{you-guessed-it---this-is-not-the-f1aq}
Xen0M0rphMell0wz
```

It's likely that the password entered in the terminal when running the program will be compared with a character string, and this could well be `Xen0M0rphMell0wz`. So we restart the program, entering the previously found string when the password is requested. And here is the flag: `flag{XENO-DO-DO-DO-DO-DO-DOOOOO}`.

## Steganography

### You've been ransomwared

In this test, we're asked to find the pseudonym of the attacker who carried out a ransomware attack. For this, we have the following image available :

![Rançongiciel](@assets/images/deadface-ctf-23/ransomwared.png)

If you look closely at the image, you can make out some text at the very bottom, written in red on a red background. For greater legibility, we can open the image with `Gimp` and play with the color balance. This gives us the following image :

![Rançongiciel](@assets/images/deadface-ctf-23/filtered-ransomware.png)

By entering the highlighted text into a binary translator, we obtain :

```md
This ransomware brought to you by mirveal.
```

So we have the flag : `flag{mirveal}`

## Traffic analysis

### Git rekt

This challenge involves analyzing a network capture in an attempt to recover the password of the user `spookyboi`, who may have been tricked by a phishing campaign. We start by opening the capture with `wireshark` and realize that it contains a huge number of different packets and protocols. 

Since the password we're looking for was sent in response to a phishing campaign, it's likely to have been sent via a web interface, so we can try filtering the `HTTP` packets as well as the `POST` method. This is done with the filter `http.request.method == "POST"`.

Only one packet remains in the list, and when exported in text format we get the following result :

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

Here we can clearly see the password transmitted and therefore we have the flag : `flag{SpectralSecrets#2023}`

### Creepy crawling

For this challenge, we start with the network capture `PCAP02.pcapng`. We can simply start by displaying its contents with `tshark` :

```sh
tshark -r PCAP02.pcapng
```

You can see that there's a lot of content and different protocols. For the sake of curiosity, we can count the number of lines :

```sh
tshark -r PCAP02.pcapng | wc -l
```

This displays 13631 lines, which is indeed a lot. To simplify, knowing that we're looking for an SSH version, we can filter the output using the `-Y` option in `tshark` :

```sh
tshark -r PCAP02.pcapng -Y "ssh" | wc -l
```

That's another 991 packets. From here we could directly start looking by hand to see if anything jumps out at us, but since we know we're looking for a protocol version in the form `SSH-...`, we can filter the output of the previous command with `grep` :

```sh
tshark -r PCAP02.pcapng -Y "ssh" | grep "SSH-" | wc -l
```

This time, we're down to 141 packages. Let's take a look at what's in the capture :

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

We recognize the format we were looking for and it could well be that `SSH-2.0-9.29 FlowSsh: Bitvise SSH Server (WinSSHD) 9.29` is our flag. At this point we can't be sure, but we can try `flag{SSH-2.0-9.29 FlowSsh: Bitvise SSH Server (WinSSHD) 9.29}`.

It works ! Challenge validated.

## Forensics

### What's the wallet

This challenge starts with a `Bitcoin.txt` file where we're asked to find the wallet's address. As we go through the file, one function catches our eye :

```js
function Store-BtcWalletAddress {
  `$global:BtcWalletAddress = [System.Convert]::FromBase64String([System.Text.Encoding]::UTF8.GetBytes('bjMzaGE1bm96aXhlNnJyZzcxa2d3eWlubWt1c3gy'))
}
```

At this point, it seems obvious that the wallet's address should be recovered by converting it from base 64. So we put the string `bjMzaGE1bm96aXhlNnJyZzcxa2d3eWlubWt1c3gy` into a base-64 converter and obtain the contents of the flag. We can therefore validate this challenge with the flag: `flag{n33ha5nozixe6rrg71kgwyinmkusx2}`.

### Host busters 1

For this challenge, we have SSH access to `vim@gh0st404.deadface.io` with the password `letmevim`. When we connect, it seems that we're not in a classic shell, but rather in a `vim`. If you simply try to exit with the command `:q`, you'll disconnect from SSH. 

As it happens, `vim` runs in a terminal where it can execute commands given by the user with the command `:!` followed by any instruction. So, for example, you could type `:!bash`.

This opens a bash shell which happens to be directly in the user's `vim` directory, where we were supposed to be looking for something. So we list the content of the directory with `ls` and discover that a `hostbusters1.txt` file is there. The flag is therefore `flag{esc4P3_fr0m_th3_V1M}`.

### Tin balloon

This time we retrieve an archive containing a password-protected `Untitlednosubject.docx` file and an MP3 file. Listening to the MP3 while we try to find a solution for the password, we realize that there are some strange sounds around 3 minutes 15. 

So we open the file with `audacity` and display the spectrogram before zooming in on the part we're interested in. This is what we discover :

![Spectrogramme](@assets/images/deadface-ctf-23/spectrogram.png)

This is obviously the password for the file containing the executable's name. The flag is : `flag{Wh1t3_N01Z3.exe}`