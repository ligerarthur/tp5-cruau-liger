## Compte-Rendu avec mise en page XXXXXXXXXXXX

# Administration Système
# TP 5 - Services Réseau


Objectif :  mettre en place différents services réseau (serveur DHCP, serveur DNS, serveur web…) sur notre serveur.



## Exercice 1. Adressage IPv4

#### 1. Commencez par créer ensuite deux groupes ```groupe1``` et ```groupe2```

```bash
addgroup groupe1
addgroup groupe2
```

Puis pour vérifier : ```cat /etc/group | grep groupe```

#### 2. Créez ensuite 4 utilisateurs ```u1```,```u2```,```u3```,```u4``` avec la commande ```useradd```, en demandant la création de leur dossier personnel et avec ```bash``` pour shell

```bash
useradd -s /bin/bash -m u1
useradd -s /bin/bash -m u2
useradd -s /bin/bash -m u3
useradd -s /bin/bash -m u4
```

* ```-s``` sert à préciser le shell par défaut de l'utilisateur
* ```-m``` permet de créer son répertoire personnel

#### 3. Placez les utilisateurs dans les groupes :

* ```u1```, ```u2```, ```u4``` dans ```groupe1```
* ```u2```, ```u3```, ```u4``` dans ```groupe2```

```bash
usermod -a -G groupe1 u1
usermod -a -G groupe1 u2
usermod -a -G groupe1 u4
```

* ```-a, --append``` sert à ajouter
* ```-G``` précise que le champs suivant est un groupe 

OU (pour ajouter tous les utilisateurs d'un coup)

```bash
gpasswd -M u2,u3,u4 groupe2
```

* ```-M``` précise que les champs suivant sont les utilisateurs à ajouter au groupe

#### 4. Donnez deux moyens d’afficher les membres de ```groupe2```

```cat /etc/group | grep groupe1 | cut -f 4 -d :```

* ```-f``` pour choisir la partie du ```cut``` à afficher
* ```-d``` pour choisir le delimiter du ```cut```

  ou
  
```awk -F':' '/groupe1/{print $4}' /etc/group```


#### 5. Faites de ```groupe1``` le groupe propriétaire de ```/home/u1``` et ```/home/u2``` et de ```groupe2``` le groupe propriétaire de ```/home/u3``` et ```/home/u4```

```bash
chgrp groupe1 /home/u1
chgrp groupe1 /home/u2
 
chgrp groupe2 /home/u3
chgrp groupe2 /home/u4
```

#### 6. Remplacez le groupe primaire des utilisateurs :

* ```groupe1``` pour ```u1``` et ```u2```
* ```groupe2``` pour ```u3``` et ```u4```


```bash
 usermod -g groupe1 u1
 usermod -g groupe1 u2
 
 usermod -g groupe2 u3
 usermod -g groupe2 u4
 ```
 
 Pour vérifier que l'utilisateur à bien le bon groupe primaire :

```cut -d: -f1-4 /etc/passwd```

Le groupe primaire est le premier groupe affiché pour l'utilisateur
 
#### 7. Créez deux répertoires ```/home/groupe1``` et ```/home/groupe2``` pour le contenu commun aux groupes, et mettez en place les permissions permettant aux membres de chaque groupe d’écrire dans le dossier associé.

```bash
mkdir /home/groupe{1,2}
chmod g+w /home/groupe1
chmod g+w /home/groupe2 # Ajout des droits
```

#### 8. Comment faire pour que, dans ces dossiers, seul le propriétaire d’un fichier ait le droit de renommer ou supprimer ce fichier?

Pour retirer les droits aux autres ormis le propriétaire du fichier nous tapons la commande : ```sudo chmod go-rwx groupe1```.

#### 9. Pouvez-vous vous connecter en tant que ```u1```? Pourquoi?

Non on ne peut pas ce connecter car nous n'avons pas encore définit de mot de passe avec la commande ```passwd```
Avec l'utilisateur root, on va quand même pouvoir changer de session car pas de mot de passe est demandé
Mais avec un utilisateur standard, nous ne pouvons pas nous connecter/changer de session car un password est demandé

#### 10. Activez le compte de l’utilisateur ```u1``` et vérifiez que vous pouvez désormais vous connecter avec son compte.

```bash
passwd u1
Enter new UNIX password: u1
Retype new UNIX password: u1
passwd: password updated successfully
```
pour tester :
```bash
bob@server:~$ su u1
Password: u1
u1@server:/home/bob$

```

#### 11. Quels sont l’uid et le gid de ```u1```?

pour avoir l'ID de l'utilisateur : ```id -u u1```
```> 1001```
pour avoir l'ID du groupe primaire : ```id -g u1```
```> 1001```

Remarque : pour connaitre pour les id lié à l'utilsateur : ```id u1```

#### 12. Quel utilisateur a pour uid ```1003```?

```bash
getent passwd "1003" | cut -d: -f1
> u3
```

#### 13. Quel est l’id du groupe ```groupe1```?

```bash
getent group "groupe1" | cut -d: -f3
> 1001
```

#### 14. Quel groupe a pour guid ```1002```?

```bash
getent group "1002" | cut -d: -f1
> groupe2
```

#### 15. Retirez l’utilisateur ```u3``` du groupe ```groupe2```. Que se passe-t-il? Expliquez.

```bash
gpasswd -d u3 groupe2
Removing user u3 from group groupe2
```
Le groupe primaire ne peut pas être supprimé d'un utilisateur. Il faut avant changer le groupe primaire de l'utilisateur :
```usermod -g groupe1 u3```


#### 16. Modifiez le compte de ```u4``` de sorte que :
* il expire au 1er juin 2019
* il faut changer de mot de passe avant 90 jours
* il faut attendre 5 jours pour modifier un mot de passe
* l’utilisateur est averti 14 jours avant l’expiration de son mot de passe
* le compte sera bloqué 30 jours après expiration du mot de passe

```bash
chage -l u4
Last password change                                    : mars 20, 2019
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : never
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99999
Number of days of warning before password expires       : 7
```
permet de lister les attributs d'un utilisateurs.

* ```chage -E 2019-06-01 u4``` pour changer la date d'expiration du compte
* ```chage -m 5 u4``` pour changer le nombre minimum de jour entre deux changements de mot de passe
* ```chage -M 90 u4``` pour changer le nombre maximum de jour entre deux changements de mot de passe
* ```chage -W 14 u4``` pour changer le nombre de jour après lequel il est prévenu qu'il doit changer son mdp
* ```chage -I 14 u4``` pour changer le nombre de jour après lequel un coompte sera bloqué si son mdp est expiré

doc de ```chage``` : https://linux.die.net/man/1/chage

#### 17. Quel est l’interpréteur de commandes (Shell) de l’utilisateur ```root```?

pour trouver le shell de l'utilisateur root :

```getent passwd | grep root | cut -d: -f7```

#### 18. à quoi correspond l’utilisateur ```nobody```?

Dans la plupart des variantes d'Unix, nobody (personne en anglais) est le nom conventionnel d'un compte d'utilisateur à qui aucun fichier n'appartient, qui n'est dans aucun groupe qui a des privilèges et dont les seules possibilités sont celles que tous les "autres utilisateurs" ont.

#### 19. Par défaut, combien de temps la commande ```sudo``` conserve-t-elle votre mot de passe en mémoire? Quelle commande permet de forcer ```sudo``` à oublier votre mot de passe?


le timeout par défaut de la commande ```sudo``` pour retenir le mot de passe est de 5min

* ```visudo``` pour ouvrir le fichier sudoers et modifier les paramètres de sudo
* ```  Defaults   timestamp_timeout=30``` : Pour modifier le timeout du sudo (en min)
* ```sudo -K``` permet de réinitialiser le timeout

## Exercice 2. Gestion des permissions

#### 1. Dans votre ```$HOME```, créez un dossier ```test```, et dans ce dossier un fichier ```fichier1``` contenant quelques lignes de texte. Quels sont les droits sur ```test``` et ```fichier1```?

```bash
mkdir test
touch fichier1
echo -e "coucou \n je suis la \n et la " > fichier1
```
```drwxrwxr-x 2 bob  bob  4096 mars  20 09:17 test```
d = directory
l'utiisateur bob et les même du groupe bob peuvent : afficher le contenu / ecire / supprimer du contenu / et rentrer dans le répertoire
les autres utilisateur peuvent seulement entrer dans le dossier et afficher le contenu

```-rw-rw-r-- 1 bob bob 25 mars  20 09:17 fichier1```

le fichier appartient à l'utilisateur bob et au groupe bob

bob peut donc lire/ecrire
le groupe bob peut également lire/écrire
et les autres utilisateur peut seulement lire le fichier

#### 2. Retirez tous les droits sur ce fichier (même pour vous), puis essayez de le modifier et de l’afficher en tant que ```root```. Conclusion?

```bash
chmod ugo-rwx test/fichier1
```

On peut ouvrir le fichier mais on ne voit aucun contenu, nous avons également un message avec ```Permission denied```
Mais heureusement avec l'utilsateur root, nous pouvons entrer dans le dossier et le fichier pour modifier ce que nous désirons.
 
#### 3. Redonnez vous les droits en écriture et exécution sur ```fichier1``` puis exécutez la commande ```echo "echo Hello" > fichier1```. On a vu lors des TP précédents que cette commande remplace le contenu d’un fichier s’il existe déjà. Que peut-on dire au sujet des droits?

```chmod u+wx test/fichier1``` Pour redonner les droits en écriture/exécution

```bash
cat fichier1
cat: fichier1: Permission denied
```

Nous avons pu modifier et enregistrer le contenu du fichier mais il nous est impossible de lire son contenu car nous n'en avons pas le droit.

#### 4. Essayez d’exécuter le fichier. Est-ce que cela fonctionne? Et avec ```sudo```? Expliquez.

```bash
./fichier1
bash: ./fichier1: Permission denied
```

Même avec la permission d'exécuter nous ne pouvons pas l'éxécuter, nous pouvons en conclure qu'il est nécessaire d'avoir la permission de lire pour exécuter. 
Par contre l'exécution avec sudo fonctionne normalement.

#### 5. Placez-vous dans le répertoire ```test```, et retirez-vous le droit en lecture pour ce répertoire. Listez le contenu du répertoire, puis exécutez ou affichez le contenu du fichier ```essai```. Qu’en déduisez-vous? Rétablissez le droit en lecture sur ```test```

```bash
chmod u-r ../test/
~/test$ ls -l
ls: cannot open directory '.': Permission denied
```

On ne peut pas lister le contenu du répertoire car nous n'avons plus les droits pour.
Il est quand même possible d'exécuter ou de lire un fichier si nous avons les droits.

```chmod u+r ../test``` Pour réatablir les droits de lecture sur le dossier test 

#### 6. Créez dans ```test``` un fichier ```nouveau``` ainsi qu’un répertoire ```sstest```. Retirez au fichier ```nouveau``` et au répertoire ```test``` le droit en écriture. Tentez de modifier le fichier ```nouveau```. Rétablissez ensuite le droit en écriture au répertoire ```test```. Tentez de modifier le fichier ```nouveau```, puis de le supprimer. Que pouvez-vous déduire de toutes ces manipulations?

```bash
touch nouveau
mkdir sstest
sudo chmod u-w nouveau
sudo chmod u-w test
```

On ne peut pas modifier le fichier ```nouveau``` car on peut seulement l'ouvrir en lecture seul

#### 7. Positionnez vous dans votre répertoire personnel, puis retirez le droit en exécution du répertoire ```test```.Tentez de créer, supprimer, ou modifier un fichier dans le répertoire ```test```, de vous y déplacer, d’en lister le contenu, etc...Qu’en déduisez vous quant au sens du droit en exécution pour les répertoires ?

```bash
rm nouveau
rm: remove write-protected regular empty file 'nouveau'? yes
ls
fichier1  sstest
```
Retirer le droit en écriture enlève la permission de supprimer des fichiers du répertoire _sans validation_.


#### 8. Rétablissez le droit en exécution du répertoire ```test```. Positionnez vous dans ce répertoire et retirez lui à nouveau le droit d’exécution. Essayez de créer, supprimer et modifier un fichier dans le répertoire ```test```, de vous déplacer dans ```sstest```, de lister son contenu. Qu’en concluez-vous quant à l’influence des droits que l’on possède sur le répertoire courant? Peut-on retourner dans le répertoire parent avec ```cd ..```? Pouvez-vous donner une explication?

```bash
chmod u+x test/
cd test/
chmod u-x ../test
cd test/
bash: cd: test/: Permission denied
touch coucou test/hello
touch: cannot touch 'test/hello': Permission denied
rm test/
fichier1  sstest
rm test/sstest
rm: cannot remove 'test/sstest': Permission denied
ls test/
ls: cannot access 'test/sstest': Permission denied
ls: cannot access 'test/fichier1': Permission denied
fichier1  sstest
```

Nous pouvons ainsi plus nous déplacer ni lister le contenu de ce répertoire. 
Le droit en éxecution d'un répertoire permet donc d'accéder aux fichiers présent dans ce répertoire mais aussi de les modifiers/créer/supprimer.

#### 9. Rétablissez le droit en exécution du répertoire ```test```. Attribuez au fichier ```essai``` les droits suffisants pour qu’une autre personne de votre groupe puisse y accéder en lecture, mais pas en écriture

```
chmod u+x test/
touch essai
chmod g=r essai
```

Les membres du groupe ont la permission seulement en lecture : r--

#### 10. Définissez un umask très restrictif qui interdit à quiconque à part vous l’accès en lecture ou en écriture,ainsi que la traversée de vos répertoires. Testez sur un nouveau fichier et un nouveau répertoire.

```umask 077``` est très restrictif, il interdit (à part à nous) la lecture ou l'écriture et l'éxécution (traversée de répertoires). 

Test des droits :
```
umask 077
touch testumask
ls -l testumask
-rw------- 1 bob bob 0 mars  20 09:54 testumask
```

#### 11. Définissez un umask très permissif qui autorise tout le monde à lire vos fichiers et traverser vos répertoires, mais n’autorise que vous à écrire. Testez sur un nouveau fichier et un nouveau répertoire.

La commande ```umask 022``` est très permissif, elle permet à tout le monde de lire les fichiers, de traverser le répertoire en n'autorisant que nous en écriture 
```bash
rwxrw-rw-
```

#### 12. Définissez un umask équilibré qui vous autorise un accès complet et autorise un accès en lecture auxmembres de votre groupe. Testez sur un nouveau fichier et un nouveau répertoire.

La commande ```umask 037``` est équilibré permet d'autoriser l'accès complet et un accès en lecture aux membres du groupe. 

```bash
rwxr-----
```

#### 13. Transcrivez les commandes suivantes de la notation classique à la notation octale ou vice-versa (vous pourrez vous aider de la commande ```stat``` pour valider vos réponses) :
* ```chmod u=rx,g=wx,o=r fic```


 ```bash
 chmod 534 fic
 ```
 
On vérifie : ```stat -c %A test```

Droits initaux : ```--------- ``` (000)

Droits finaux : ```r-x-wxr--``` (534)

* ```chmod uo+w,g-rx fic``` en sachant que les droits initiaux de ```fic``` sont ```r--r-x---```

 ```bash
chmod 602 fic
```

Droits Initaux : ```r--r-x---``` (450)

Droits finaux : ```rw-----w-``` (602)


* ```chmod 653 fic``` en sachant que les droits initiaux de ```fic``` sont ```711```

```bash
chmod u-x,g+r,o+w fic
```

Droits Initaux : ```rwx--x--x``` (711)

Droits finaux : ```rw-r-x-wx``` (653)

* ```chmod u+x,g=w,o-r fic``` en sachant que les droits initiaux de ```fic``` sont ```r--r-x---```


```bash
chmod 520 fic
```

Droits initaux : ```r--r-x---``` (450)

Droits finaux : ```r-x-w----``` (520)


#### 14. Affichez les droits sur le programme ```passwd```. Que remarquez-vous? En affichant les droits du fichier ```/etc/passwd```, pouvez-vous justifier les permissions sur le programme ```passwd```?

```stat -c %A /usr/bin/passwd```

```-rwsr-xr-x```

En regardant les droits sur passwd, on s’aperçoit que ce fichier est setuidé. Setuid est un moyen de transférer des droits à l'utilisateur propriétaire : ici root
Le programme est donc éxecuté avec root même si c'est bob qui le lance
C’est-à-dire que lorsqu’un utilisateur lance la commande passwd, elle est lancée avec les droits du superutilisateur, ainsi l’écriture pourra se faire dans le fichier /etc/passwd et l’utilisateur aura changé son mot de passe sans être root. Sans le setuid, l’utilisateur n’aurait pas pu écrire dans le fichier /etc/passwd.
La notion de setuid n’existe pas pour les répertoires.


#### 15. Access Control Lists (ACL): suivez le tutoriel de cette page : https://doc.ubuntu-fr.org/acl.
##### Activation & Mise en pace
Pour utiliser les ACL, il faut d'abord vérifier que notre noyau prend bien en charge celle-ci :
```sh
bob@server:~$ grep ACL /boot/config-$(uname -r)
CONFIG_EXT4_FS_POSIX_ACL=y
CONFIG_REISERFS_FS_POSIX_ACL=y
CONFIG_JFS_POSIX_ACL=y
CONFIG_XFS_POSIX_ACL=y
CONFIG_BTRFS_FS_POSIX_ACL=y
CONFIG_F2FS_FS_POSIX_ACL=y
CONFIG_FS_POSIX_ACL=y
CONFIG_TMPFS_POSIX_ACL=y
CONFIG_HFSPLUS_FS_POSIX_ACL=y
CONFIG_JFFS2_FS_POSIX_ACL=y
CONFIG_NFS_V3_ACL=y
CONFIG_NFSD_V2_ACL=y
CONFIG_NFSD_V3_ACL=y
CONFIG_NFS_ACL_SUPPORT=m
CONFIG_CEPH_FS_POSIX_ACL=y
CONFIG_CIFS_ACL=y
CONFIG_9P_FS_POSIX_ACL=y
```
Dans notre cas, nous pouvons bien utiliser les ACL.

Ensuite pour pouvoir les utiliser, il faut préciser lors du montage de la partition que l'on veut utiliser les ACL :
* ajout d'un nouveau disque à la vm
* création d'un partition ext3 : fdisk /dev/sdb
* création d'un système de fichier : mkfs.ext3 /dev/sdb1
* création d'un dossier pour monter le dossier : mkdir /media
* mountage de la partition : mount -t ext3 -o defaults,acl /dev/sdb1/ /mount/folder/
Vérification :
```sh
root@server:/media# df -h | grep /dev/sdb1
/dev/sdb1       2,0G  3,2M  1,9G   1% /media
```



#### 16. Quotas disques: suivez le tutoriel de cette page :https://doc.ubuntu-fr.org/quota



