## Debian

Il suffit de telecharger l'iso et de creer une machine virtuelle. Il faut egalement configurer la partie reseau de la vm afin de pouvoir se connecter en ssh.

docs:
- https://www.brianlinkletter.com/2012/10/installing-debian-linux-in-a-virtualbox-virtual-machine/

## LVM

Il est preferable de configurer manuellement les partitions, car par defaut tout le stockage est utilise pour les partitions, ce qui empeche de faire des snapshots.
LVM dispose de 4 partitions physiques (sda1, sda2, etc..) qui sont indispensables pour pouvoir booter et allumer son serveur.
Les partitions logiques sont infinies et commencent a sda5.

SWAP est une rallonge de la memoire vive (RAM) sur le disque dur qui stocke les programmes en pause ou ouvert en premier afin de liberer de la ram pour les nouveaux programmes.

docs:
- https://askubuntu.com/questions/950307/why-guided-partitioning-create-a-sda2-of-1-kb
- https://askubuntu.com/questions/508870/what-is-a-swap-area?answertab=votes#tab-top
- https://unix.stackexchange.com/questions/156719/ext2-filesystem-for-boot-partition#:~:text=ext2%20is%20simple%2C%20robust%20and,a%20good%20choice%20for%20%2Fboot.
- https://superuser.com/questions/778686/linux-lsblk-output
- https://www.youtube.com/watch?v=GEl2S5MI-WU
- https://www.youtube.com/watch?v=MeltFN-bXrQ

## Pare-feu UFW

Installation: 

```
sudo apt update
sudo apt install ufw
```

Politique par defaut:

```
sudo ufw default deny incoming
sudo ufw default allow outgoing
```
Etat:

```
sudo ufw status verbose
```

Par defaut, toute les connexions entrantes sont bloques a l'inverse des connexions sortantes.
On peut ouvrir un port pour autoriser une connexion ssh:

```
sudo ufw allow 4242
sudo ufw enable # on active ufw
```

Pour voir les ports ouverts:

```
sudo netstat -ntlp
# sudo apt install net-tools
```

Autoriser port 80 pour le serveur web:
```
sudo ufw allow http
```

docs:
- https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-20-04-fr
- https://linuxize.com/post/how-to-setup-a-firewall-with-ufw-on-debian-10/

# SSH

Pour voir l'etat du service ssh:

```
systemctl service sshd
```

Le fichier de configuration `etc/ssh/sshd_confid` permet de parametrer ssh:

```
Port 4242
PermitRootLogin no
```

Redemarrage du service ssh:

```
systemctl restart ssh
```

Connaitre l'ip de la vm:

```
curl ifconfig.me
```

Se connecter a la vm en ssh:
```
ssh arguilla@127.0.0.1 -p 4242
```

Pour virtualbox:

![alt text](images/vm-settings-1.png "Title")
![alt text](images/vm-settings-2.png "Title")
![alt text](images/vm-settings-3.png "Title")

docs:
- https://www.memoinfo.fr/tutoriels-linux/configurer-serveur-ssh/
- https://nsrc.org/workshops/2014/btnog/raw-attachment/wiki/Track2Agenda/ex-virtualbox-portforward-ssh.htm
- https://medium.com/nycdev/how-to-ssh-from-a-host-to-a-guest-vm-on-your-local-machine-6cb4c91acc2e
- https://www.youtube.com/watch?v=oNK1fXhxQHs

## Droits

### Sudo

Su permet de se connecter a n'importe quel compte, il demande le mot de passe du compte root pour acceder a root contrairement a sudo qui demande le mot de passe de l'utilisateur courant.

Installation:
```
su -l
apt install sudo
adduser arguilla sudo # ajout d'un utilisateur dans le groupe sudo
```

docs:
- https://blog.seboss666.info/2014/05/installer-et-utiliser-sudo-sur-debian/

### Apparmor


Installation:
```
apt install apparmor apparmor-profiles apparmor-utils
```
Etat:
```
aa-status
```

docs:
- https://debian-handbook.info/browse/fr-FR/stable/sect.apparmor.html
- https://blog.spyzone.fr/2017/06/comprendre-apparmor/
## Apt vs Aptitude

Aptitude est un regroupement plus homogene des fonctions d'apt (ex: aptitue regroupe les fonctions de apt-cache et apt en meme temps) avec une interface plus visuelle.
A savoir que apt est une version ameliore de apt-get. Apt se base sur une bibliotheque qui contient apt-get.
apt est plus visuel que apt-get, il est preferable d'utiliser apt-get pour les scripts.


docs:
- https://buzut.net/apt-get-vs-aptitude-preferez-le-second/
- https://debian-handbook.info/browse/fr-FR/stable/sect.apt-get.html
- https://unix.stackexchange.com/questions/767/what-is-the-real-difference-between-apt-get-and-aptitude-how-about-wajig

## Regles mot de passe sudo

Pour changer les regles des mot de passe sudo, il faut installer le paquet libpam-pwquality:

```
sudo apt install libpam-pwquality
```

Il faut ensuite editer le fichier `/etc/pam.d/common-password` et suivre le tuto ci-dessus en adaptant a notre cas:
- https://linuxhint.com/secure_password_policies_ubuntu/

Le tuto ci-dessus nous montre egalement comment creer des regles d'expiration du mot de passe via la commande `chage` qui modifie les regles pour les utilisateurs courants:

```
sudo chage -M 30 arguilla # defini la date d'expiration du mdp dans 3o jours
sudo chage -m 2 arguilla # defini le temps minimal requis entre chaque modification de mdp
```

Pour les futurs utilisateurs, il faut directement mettre les valeurs dans le fichier `/etc/login.defs`:

```
# ligne 160
PASS_MAX_DAYS 30
PASS_MIN_DAYS 2
PASS_WARN_AGE 7
```

Pour voir l'etat des parametres de configuration d'un utilisateur:

```
sudo chage -l arguilla
```

Pour configurer sudo, il existe le fichier `/etc/sudoers` manipulable avec la commande `sudo visudo` qui ouvrira un editeur de texte et qui previendra en cas d'erreur de syntaxe, qui rendrait unitilisable sudo en cas de sauvegarde.

Dans ce fichier, pour pouvoir cree un systeme de log, personnaliser les messages d'erreurs, restreindre sudo au mode tty, definir des paths securises, voici ma configuration:

```
#
# This file MUST be edited with the 'visudo' command as root.
#
# Please consider adding local content in /etc/sudoers.d/ instead of
# directly modifying this file.
#
# See the man page for details on how to write a sudoers file.
#
Defaults	env_reset
Defaults	mail_badpass
Defaults	secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
# Message d'erreur insultants
Defaults	insults
# fichier de logs
Defaults	log_host, log_year, logfile="/var/log/sudo/sudo.log"
# Dossiers pour les logs de commandes Input/Output
Defaults	iolog_dir="/var/log/sudo/"
# Permet de capturer les commandes pour les logs (Input/Output)
Defaults	log_input,log_output
# Permet d'autoriser sudo uniquement en mode TTY (exclus donc sudo pour les scripts cron ou cgi-bin)
Defaults	requiretty
# Host alias specification

# User alias specification

# Cmnd alias specification

# User privilege specification
root	ALL=(ALL:ALL) ALL

# Allow members of group sudo to execute any command
%sudo	ALL=(ALL:ALL) ALL

# See sudoers(5) for more information on "#include" directives:

#includedir /etc/sudoers.d
```

docs:
- https://www.techrepublic.com/article/how-to-manage-linux-password-expiry-with-the-chage-command/
- https://www.cyberciti.biz/faq/securing-passwords-libpam-cracklib-on-debian-ubuntu-linux/
- https://www.networkworld.com/article/2726217/how-to-enforce-password-complexity-on-linux.html (listes des parametres de configuration pour le fichier `/etc/pam.d/common-password`)
- https://www.thegeekstuff.com/2009/04/chage-linux-password-expiration-and-aging/
- https://www.howtogeek.com/428174/what-is-a-tty-on-linux-and-how-to-use-the-tty-command/ (comprendre les tty)
- https://unix.stackexchange.com/questions/108577/how-to-log-commands-within-a-sudo-su
- https://www.tecmint.com/sudoers-configurations-for-setting-sudo-in-linux/
- https://www.tecmint.com/sudo-insult-when-enter-wrong-password/
- https://www.digitalocean.com/community/tutorials/how-to-edit-the-sudoers-file-fr

## Infos complementaires

Si la commande `wall` affiche plusieurs fois le texte, c'est que l'utilisateur courant a plusieurs shell ouverts (sudo su ouvre un tty par exemple).

port forwading virtualbox pour port 80:
- https://www.it-connect.fr/configurer-le-port-forwarding-sur-une-vm-virtualbox%EF%BB%BF/
- https://www.osradar.com/install-wordpress-with-lighttpd-debian-10/

Configurer lighttpd, wordpress, php, maria-db:
- https://www.how2shout.com/linux/install-wordpress-on-lighttpd-web-server-ubuntu/

Identifiant bdd (pma) http://localhost:8080/phpmyadmin :
```
id: arguilla
mdp: 
```
Identifiant wordpress http://localhost:8080/site :
```
id: arguilla
mdp: 
```
Mot de passe encryption dd:
```

```
Mot de passe sudo:
```

```

# correction

verifier pare feu:
```
sudo ufw status verbose
```
verifier ports:
```
sudo netstat -ntlp
```
verifier le nombre de tty:
```
tty
```
voir tout les users:
```
awk -F: '{ print $1}' /etc/passwd
```

verifier les infos d'expiration du password d'un user:
```
sudo chage -l user
```

Creer un utilisateur et set le password:
```
sudo adduser user
sudo passwd user
```
getent group evulation

usermod -G evaluation jyeo 
