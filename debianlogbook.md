# Comment ajouter un user à un ou plusieurs groups:
usermod -a -G nomgroup1,nomgroup2 nomuser

# Probleme de bad password avec wicd 

pour régler le probleme: 
```
# apt-get remove network-manager
#service wicd restart
```


# NFS V4
## Obtenir version de nfs
### Sur le client
- 1 `#nfsstat -m` ou `nfsstat` ca marche sur le client. 
- 2 `# mount` si on a type nfs4 alors version 4 si on a type nfs alors version 3

### Sur le serveur
rpcinfo -p | grep nfs
La version est dans la deuxieme colonne. 




On veut monter un répertoire réel :
/home/users
SUR LE SERVER
1/ Créer un export filesystem:
   mkdir exports
   mkdir /export/users
2/ Monter le répertoire réel avec --bind
   mount -bind /home/users /export/users
   que l'on peut mettre dans le /etc/fstab
   /home/users 	 /export/users 	none  bind 0 0
3/ Pour exporter notre répertoire sur un réseau local 192.168.1.0/24
   /export 	 192.168.1.0/24(rw,fsid=0,insecure,no_subtree_check,async)
   /export/users 192.168.1.0/24(rw,nohide,insecure,no_subtree_check,async)

4/ Redémarrer le service

   /etc/init.d/nfs-kernel-server restart
CLIENT
On peut monter l'arbre complet 
   nfs-server:/users	   /mnt	  nfs4	_netdev,auto	0 0
On peut monter une partie de l'arbre:
   nfs-server:/	   /mnt	  nfs4	_netdev,auto	0 0

avec automount : cd /net/prony (nom du serveur tel qu'il est dans /etc/hosts/) ou son adresse ip.

## maproot 

maproot -> user1
maproot -> group1 qd j'accede le freenas je l'accede en user1 et group1 sur le freenas.

## montage sur le freenas du cabinet
je vois que sur le client le repertoire est monté avec nobody:nogroup
je mets sur le freenas le share avec mapall user : nobody et mapall group avec nogroup

## montage sur le freenas @home
j'ai un repertoire du freenas qui se monte sur saphir. 
Mais a chaque fois qu'un fichier se crée il appartient à root.
touch blabla.txt appartient à root:<leuser>
# MAC address
## Comment trouver l'adresse MAC d'une interface

  `cat /sys/class/net/eth0/address`


# FONCTIONNEMENT DU SERVEUR DE MAIL

Les mails sont sur un serveur pop sur l'internet

user@domain_emetteur.fr envoit un mail à toto@domain_receveur.fr. 

- 1 Pour cela il utilise un MUA (mail user agent) pour écrire le message. 
- 2 le MUA envoie le message au serveur de mail du FAI par SMTP.
- 3 Le MTA (mail transfert agent) réceptionne le message et y applique un certains nombre de traitements.
- 4 Le MTA fait une requete DNS (enregistrement MX) pour la recherche du serveur de mail rattaché au domaine "domain_receveur.fr"
- 5 Le message est alors transmis via SMTP entre les deux MTA
- 6 Si l'adresse correspond à un utilsateur valide (login ou alias), le second MTA stocke le messge localement pour que l'utilisateur demande à le lire.

Réception d'un mail:
Dans la pluspart des cas 
 Donald <donald@domaine-recepteur.fr> essaye de lire le message envoyé par Toto <toto@domaine-emeteur.fr>

Soit le cas sans doutes le plus courant : il n'y a pas de serveur de
mail local. L'utilisateur utilise son MUA pour rechercher le courrier
sur son serveur de mail en utilisant l'un des protocoles supportés :
POP3 (très souvent), IMAP, POP2, KPOP, etc ...

Soit le cas le plus évolué (que nous étudierons ici) : on utilise un
serveur de mail local. L'utilisateur utilise un logiciel (type
Fetchmail) déclenché automatiquement, voir périodiquement pour
rechercher le courrier sur son serveur de mail en utilisant l'un des
protocoles supportés : POP3, IMAP, POP2, KPOP, etc ... ou
éventuellement en déclenchant la délivrance des messages par le MTA du
FAI sur le MTA local en utlisant le protocol ETRN. 

      Les messages sont stockés sur la machine de l'utilisateur de
    façon standart (le plus souvent dans /var/spool/mail).
      L'utilisateur peut alors lire ses messages avec son MUA en
      interrogeant le MTA local (hors connection Internet). Le
      protocol de récupération peut être n'importe lequel de ceux vu
      précédement, mais une préférence sera donné à IMAP qui permet de
      garder les messages sur le serveur local (dans /var/spool/mail
      et dans le home pour les autres répertoires IMAP).

On utilise sb_server.py comme proxy. On donne comme information le nom
de ce serveur (pop.monfai.fr) à sb_server et un port d'écoute.

On utilise pour cela 
User interface url is http://localhost:8880/

On configure fetchmail pour aller chercher les mails sur le proxy.

fetchmail a deux fichiers de configuration:
- /etc/fetchmailrc
- /etc/default/fetchmail
fetchmail est un client pop et imap.
fetchmail peut forwarder en utilisant smtp. 

pop.aliceadsl.fr --->> sb_server proxy --->> fetchmail qui change le
To: --->> postfix ---> maildrop

Fetchmail passe les mails à postfix (MTA) qui utilise maildrop pour
les filtrer et les rangers dans différents répertoires en fonction de
variables fournis par postfix et du fichier de conf /etc/maildrorc.

is "laurent@francois.org" here
is "sophie@griman.org" here

Je ne comprends ou on dit à fetchmail d'utiliser postfix?

Comment savoir si fetchmail tourne en daemon:
# ps -e | grep fetch
10735 ?        00:00:00 fetchmail

%%% FONCTIONNEMENT DE POSTFIX

Je lis sur le site de la doc officielle de postfix qu'il faut créer:

Un compte utilisateur "postfix" avec un user id et un group id qui ne
sont pas utilisés par un autre compte. Preferably, this is an account
that no-one can log into. The account does not need an executable
login shell, and needs no existing home directory. Exemple:

          /etc/passwd:
              postfix:*:12345:12345:postfix:/no/where:/no/shell

          /etc/group:
              postfix:*:12345:

      
Créer un groupe "postdrop" avec un group id qui n'est pas utilisé par
un autre compte utilisateur, ni par le compte utilisateur
"postfix". Exemple:

          /etc/group:
              postdrop:*:54321:
    
Mais sous debian c'est déjà fait:
/etc/passwd:
	postfix:x:116:120::/var/spool/postfix:/bin/false
/etc/group:
	postdrop:x:121:

Connaitre la version de postfix: postconf mail_version

hostname -f 
prony.venus.org



Création des files d'attentes. Postfix utilise une arborescence
beaucoup plus élaborée que celle de ses prédécesseurs pour placer les
messages en attente de délivrance. Il suffit ici d'en indiquer la
racine (/var/spool/postfix/, généralement) car tous les autres
sous-répertoires y seront créés lors du premier démarrage de Postfix.

# mkdir /var/spool/postfix
# chmod 0755 /var/spool/postfix

Réecritures d'adresses: il faut éfinir les correspondances entre les adresses locales de type lfs@prony.venus.org, sof@prony.venus.org qui n'ont aucune valeur pour les correspondants de l'extérieur et celles qu'ils connaissent de type unidentifiant@monfai.fr. 
Comment démarrer postfix: invokerc.d postfix start

postfix check : vérification elementaire

postfix reload relecture des fichiers de conf

Examen de la file d'attente: 
       - mailq
       - postqueue -p

Examen des paramêtres de configuration:
       - postconf

Examen des paramêtres par defaut:
       - postconf -d

Examen des paramêtres différents de ceux par défaut:
       - postconf -n
 
Deux fichiers de configuration:
     -/etc/postfix/main.cf
     -/etc/postfix/master.cf

Si on les modifie il faut faire: 
#postfix reload.

Test:
telnet localhost 25
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
220 prony.localdomain ESMTP Postfix (Debian/GNU)
ehlo localhost
250-dawa.venus.org
250-PIPELINING
250-SIZE 10240000
250-VRFY
250-ETRN
250-STARTTLS
250-ENHANCEDSTATUSCODES
250-8BITMIME
250 DSN
mail from:root@localhost
250 2.1.0 Ok
rcpt to: sof@localhost
250 2.1.5 Ok
data
354 End data with <CR><LF>.<CR><LF>
subject: premier mail sur postfix

C'est mon premier mail sur postfix
. <taper enter>
250 2.0.0 Ok: queued as B6E8D2FC76
quit

On peut se connecter à distance telnet prony 25

Pour se déloguer: quit

%%%% MAILDROP

Les mails sont classés par maildrop. 

On trouve des infos dans "man maildropfilter"

%%% MAILDIR MBOX

Traditionnelement unix utilise mailbox pour stocker les mails. 

Pour mbox, chaque dossier de votre boîte mail (Reçu, Envoyés, Archive,
Corbeille, etc) est representé par un fichier, ou tout les mails sont
enregistrés à la suite, séparés par une ligne vide.

Avec Maildir, chaque mail est un fichier, et chaque dossier de la
boîte mail, un répertoire. Attention, c’est un peu plus compliqué que
ça. Les mails ne sont pas directement enregistrés dans boîte/mail.

Avec maildir, on crée le maildir avec maildirmake. 

%%%% DOVECOT
Pour accéder aux mails mise en place de Dovecot imap server. 

telnet localhost 143
1 login unsuer unpasswd
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
* OK Dovecot ready.
1 login unuser unpasswd
1 OK Logged in.
2 select inbox
* FLAGS (\Answered \Flagged \Deleted \Seen \Draft)
* OK [PERMANENTFLAGS (\Answered \Flagged \Deleted \Seen \Draft \*)] Flags permitted.
* 0 EXISTS
* 0 RECENT
* OK [UIDVALIDITY 1200572030] UIDs valid
* OK [UIDNEXT 1] Predicted next UID
2 OK [READ-WRITE] Select completed.
4 list "" *
* LIST (\HasNoChildren) "." "INBOX"
4 OK List completed.
6 list "" *
* LIST (\HasNoChildren) "." "Popclub"
* LIST (\HasNoChildren) "." "LeMonde"
* LIST (\HasNoChildren) "." "Gnucash"
* LIST (\HasNoChildren) "." "Gulnc"
* LIST (\HasNoChildren) "." "WxPython"
* LIST (\HasNoChildren) "." "GNUmed"
* LIST (\HasNoChildren) "." "Trash"
* LIST (\HasNoChildren) "." "INBOX"
6 OK List completed.
7 logout

%%%% MAILDROP ET POSTFIX
2 possibilités:

  Direct sans utiliser le local delivery agent Postfix va livrer
  directement à maildrop sans utiliser l'agent de livraison
  'local'. Cela veut dire que l'on aura l'expansion des aliases local
  ou de $HOME/.forward. On utilise cela pour des domaines hosted avec
  des destinataires qui n'ont pas de repertoire home.

 /etc/postfix/main.cf:
 2     maildrop_destination_recipient_limit = 1
 3     virtual_mailbox_domains = some.domain someother.domain
 4     virtual_transport = maildrop
 5     virtual_mailbox_maps = hash:/etc/postfix/virtual_mailbox
 6     virtual_alias_maps = hash:/etc/postfix/virtual_alias
 7 
 8 /etc/postfix/virtual_mailbox:
 9     user1@some.domain        ...text here does not matter...
10     user2@some.domain        ...text here does not matter...
11     user3@someother.domain   ...text here does not matter...
12 
13 /etc/postfix/virtual_alias:
14     postmaster@some.domain           postmaster
15     postmaster@someother.domain      postmaster


/etc/postfix/master.cf:
    maildrop  unix  -       n       n       -       -       pipe
      flags=ODRhu user=vmail argv=/path/to/maildrop /path/to/maildroprc  ${recipient}

Indirect delivery via local delivery agent.
C'est moins efficace que directement mais permet l'expansion des local aliases et de $HOME/.forward. C'est utilisé pour des domaines listés dans "mydestination" et qui ont des user avec un home. 



# Group et user
##  Comment connaitre les utilisateurs sur un systeme
## Changer les informations d'un user
`chfn`
Cette commande permet d'indiquer dans le champ numéro 5 du fichier /etc/passwd différentes informations sur un utilisateur, son nom complet, son bureau, ses numeros de téléphone (séparées par des virgules). 

## Pour examiner les valeurs par défaut appliquées par useradd :
commande useradd -D ou éditer /etc/default/useradd 

## Connaitre ses groupes 
id
ou
groups nomutilisateur

## Comment se rajouter à un groupe 
usermod -a -G groupe user

# SKYPE 
Pas facile d'installer la version 64bits de skype
version 2.1.0.81
Probleme avec pulseaudio en installant j'ai reussi à regler mes probleme s de son que j'avais avec amarok. Mais par contre plus de son avec skype.
Pour réobtenir le son avec skype il faut faire un killall pulseaudio. Ensuite pour faire remarcher pulseaudio je ne sais pas comment faire et amarok ne marche plus. 
malgres killall pulseaudio :
lfs       4565  0.1  0.1 211728  5628 pts/0    Tl   07:01   0:00 pulseaudio


# Apache

httpd est en mode démon. Lorsqu'il est invoqué il lit le fichier de config /etc/apache2/httpd.conf. On peut alors utiliser un navigateur pour se connecter au serveur, et afficher la page de de test située dans le répertoire définie par la directive "DocumentRoot"

par exemple
 DocumentRoot /usr/web

un accès à http://www.my.host.com/index.html se réfère alors à /usr/web/index.html. Si chemin répertoire n'est pas un chemin absolu, il est considéré comme relatif au chemin défini par la directive ServerRoot.

Au démarrage apache, un port et une adresse lui sont associé sur l'hote local et le serveur attend les requetes. Il écoute sur toutes les adresses de l'hote. on peut lui préciser sur quelles adresses et quels ports écouter. 
pour écouter sur les ports 80 et 8000 sur toutes les interfaces :
Listen 80
Listen 8000

pour écouter sur 80 sur une IP et 8000 su une autre 
Listen 192.0.2.1:80
Listen 192.0.2.5:8000 

Comment tout cela fonctionne avec les hotes virtuels
Listen n'implémente pas les autes virtuels. S'il n'y a pas de directive VirtualHost alors le serveur se comporte de la meme facon pour toutes les requetes. Il faut une VirtualHost pour chaque couple adresse+port afin de définir le comportement de cet hote virtuel. Il faut s'assurer que le server écoute sur cette adresse et ce port. 

Les serveurs virtuels permet de faire fonctionner un ou plusieurs serveurs web comme www.example1.com et www.example2.com sur une meme machine. 
les serveurs virtuels peuvent être par IP (chaque serveur à son IP ) et par nom plusieurs noms de domaines se cotoient sur une meme adresse IP. Cette méthode est appelée host-based ou serveur virtuel non IP. C'et la méthode de choix

Comment on fait:
On désigne l'adresse IP sur laquelle le serveur va attendre les requetes. 
NameVirtualHost addr[:port]
j'ai *:80

Ensuite création d'une section <VirtualHost> pour chaque serveur à créer. L'argument de la directive <VirtualHost> doit etre le meme que l'argument de la directive NameVirtualHost. Dans chaque section <VirtualHost> il faut définir au minimum:
ServerName 
DocumentRoot précise l'emplacement sur le systeme de fichiers du contenu de ce serveur

Supposons que l'on héberge le domaine www.domain.tld et que l'on souhaite ajouter le serveur virtuel www.otherdomain.tld qui pointe sur la même adresse IP. Il suffit d'ajouter la configuration suivante à apache2.conf :

NameVirtualHost *:80
<VirtualHost *:80>
ServerName www.domain.tld
ServerAlias domain.tld *.domain.tld
DocumentRoot /www/domain
</VirtualHost>

<VirtualHost *:80>
ServerName www.otherdomain.tld
DocumentRoot /www/otherdomain
</VirtualHost>

ServerAlias indique aux utilisateurs les autres noms permis pour accéder au même site web.

Le premier serveur virtuel listé est le serveur virtuel par defaut. La directive DocumentRoot du serveur principal ne sera jamais utilisé si l'adresse IP correspond dans une directive NameVirtualHost. 

Le fichier de configuration principal
Apache est configuré en mettant des "directives". Le fichier de configuration principale est httpd.conf. D'autres fichiers de configuration peuvent ajoutés avec la directive Include. 

On peut vérifier l'absence d'erreur de syntaxe dans les fichiers de config avec la commande apachectl configtest.

Modules
si le serveur a été compilé de facon à utiliser les modules chargés dynamiquement, alors les modules peuvent être compilés séparément et chargés à n'importe quel moment à l'aide de la directive LoadModule. Dans le cas contraire, Apache doit être recompilé pour ajouter ou supprimer des modules.

Pour savoir quels mofules sont compilés ont peut utiliser en ligne de commande -l: apache2ctl -l 

Apache permet la gestion décentralisée de la configuration via des fichiers spéciaux placés dans l'arborescence du site web. Ces fichiers spéciaux se nomment en général .htaccess, mais tout autre nom peut être spécifié à l'aide de la directive AccessFileName. Les directives placées dans les fichiers .htaccess  s'appliquent au répertoire dans lequel vous avez placé le fichier, ainsi qu'à tous ses sous-répertoires. La syntaxe des fichiers .htaccess est la même que celle des fichiers de configuration principaux. Comme les fichiers .htaccess sont lus à chaque requête, les modifications de ces fichiers prennent effet immédiatement.

Les fichiers de configurations 
apache2.conf fichier principal

envvars 

conf.d/
les fichiers sous ce répertoire sont inclus par la ligne dans apache2.conf
Include /etc/apache2/conf.d

httpd.conf: vide

magic : vide

mods-available/
contient une série de fichiers .load et .conf 

mods-enable/
contient les symlink vers les .load et .conf dans le repertoire mods-avaible

ports.conf
directive de configuration pour les ports et adresses IP à écouter. 

sites-enabled/ 
contient des symlink vers les fichiers de sites-available/

a2enmod et a2dismod enabling et disabling modules: a2emod [module]

a2ensite et a2dissite enabling et disabling site: 

Directive DocumentRoot: 
DocumentRoot /usr/web (moi j'ai /var/www)
alors un accés à http://www.my.host.com/index.html se réfere à /usr/web/index.html. 

Fichiers situes en dehors du DocumentRoot
Il est parfois nécessaire de permettre un accés web à des parties du systeme de fichier qui n'est pas sous le DocumentRoot. Apache permet de faire cela de plusieurs facons.
On peut utiliser des liens symboliques. Pour des raisons de sécurité Apache va suivre les liens symboliques uniquement si Options pour le directory en question contient FollowSymlinks ou SymLinksIfOwnerMatch.
On peut utiliser la directive Alias va mapper n'importe qu'elle partie du systeme de fichier sur le web. par exemple avec Alias /docs /var/web, http://my.host.com/docs/dir/file.html sera servi à partir de /var/web/dir/file.html. 

# DRUPAL6

aptitude install php5-pgsql drupal6
http://localhost/drupal6/install.php) to populate the database. This is the preferred method in drupal 6.x and upper.

login: frouty
password : "comme d'habitude"

# GALLERY2

Créer un user séparé pour l'utilisation de gallery.

sudo adduser --system --group gallery

Créer un nouveau role (user), creer la database, mise en place d'un mot de passe, permettre que l'on se logue dans le role. 

sudo -u postgres psql
postgres=# create role gallery; # creation du role
postgres=# create database gallery2 owner gallery; # creation de la database gallery2
postgres=# alter role gallery encrypted password '123b654';
postgres=# alter role gallery login;
postgres=# \l+
postgres=# \q

permettre de se loguer dans le role gallery à partir d'un autre utilisateur sans avoir a lancer psql
nano /etc/postgresql/8.3/main/pg_hba.conf

local      database  user  auth-method  [auth-options]
local gallery2 gallery md5

admin user setup:
admin
password "comme d'habitude"

    *  The ImageMagick module was installed, but needs configuration to be activated
    * The Multiroot module was installed, but needs configuration to be activated
    * The Nokia Image Upload module was installed, but needs configuration to be activated
    * The URL Rewrite module was installed, but needs configuration to be activated

voir http://codex.gallery2.org/Gallery2:Security

# WORDPRESS

INSTALLATION DE MYSQL SERVER 

Using MySQL under NIS/YP requires a mysql user account to be added on the local system with:    adduser --system --group --home /var/lib/mysql mysql

You should also check the permissions and ownership of the /var/lib/mysql directory:
      /var/lib/mysql: drwxr-xr-x   mysql    mysql                     

While not mandatory, it is highly recommended that you set a password for the MySQL administrative "root" user. If this field is left blank, the password will not be changed. New password for the MySQL "root" user:   Je le laisse blank     

pour se connecter
mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 40
Server version: 5.1.49-3~bpo50+1 (Debian)

mysql> CREATE DATABASE wordpress;
Query OK, 1 row affected (0.00 sec)
mysql>grant all privileges on wordpress.* to wordpress@localhost identified by '123b654' with grant option;
Query OK, 0 rows affected (0.00 sec)
mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
mysql>exit

finalement je comprends rien alors:

nano /etc/apache2/site-avaible/wordpress

<VirtualHost *:80>
  ServerName kuendu.homelinux.org
  DocumentRoot /var/www/kuendu.homelinux.org/

  <Directory /var/www/keundu.homelinux.org/>
    AllowOverride All
    Order Deny,Allow
    Allow from all
  </Directory>
</VirtualHost>

ln -s /usr/share/wordpress /var/www/kuendu.homelinux.org
a2ensite wordpress && invoke-rc.d apache2 restart
cd /usr/share/doc/wordpress/examples
 
root@prony:/usr/share/doc/wordpress/examples# sh setup-mysql -n wordpress kuendu.homelinux.org

PING kuendu.homelinux.org (202.22.227.98) 56(84) bytes of data.
64 bytes from 202.22.227.98: icmp_seq=1 ttl=64 time=0.584 ms

--- kuendu.homelinux.org ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.584/0.584/0.584/0.000 ms
/etc/wordpress/config-kuendu.homelinux.org.php written
Trying to create upload directory: /srv/www/wp-uploads/kuendu.homelinux.org
Setting up permissions
Goto http://kuendu.homelinux.org to setup Wordpress


setup-mysql [-n NAME] [-h | -d | -b] FQDN

Comment dropper une table:
mysqladmin -u root -p drop wordpress

wordpress
sml_user
sml_password


# INSTALL OPENERP

sudo apt-get install openerp-server openerp-client python2.5 python2.5-dev python-profiler  
wget http://sourceforge.net/projects/pyxml/files/pyxml/0.8.4/PyXML-0.8.4.tar.gz
```
cd PyXML-0.8.4/   
sudo python2.5 setup.py install  
mkdir -p /usr/lib/python2.5/site-packages/oldxml/_xmlplus/utils/
sudo ln -s /usr/lib/python2.6/dist-packages/oldxml/_xmlplus/utils/boolean.so /usr/lib/python2.5/site-packages/oldxml/_xmlplus/utils/
 sudo su postgres   
# $ createuser openerp -P   
#   Enter password for new role: (enter password here)   
#   Enter it again: (enter password here)   
#   Shall the new role be a superuser? (y/n) n   
#   Shall the new role be allowed to create databases? (y/n) y   
#   Shall the new role be allowed to create more new roles? (y/n) n   
# $ exit      

   1. $ sudo cp /usr/bin/openerp-server /usr/bin/openerp-server.orig   
   2. $ sudo vi /usr/bin/openerp-server      

exec /usr/bin/python ./openerp-server.py $@  
to
exec /usr/bin/python2.5 ./openerp-server.py $@  

sudo vi /etc/openerp-server.conf     
  
   1. db_name = False   
   2. db_user = openerp   
   3. db_password = openerp 
      
sudo vi /etc/postgresql/8.3/main/pg_hba.conf

local all all ident sameuser to local all all trust
   
# sudo /etc/init.d/postgresql-8.3 restart  
# $ sudo /etc/init.d/openerp-server restart
```
On a donc crée un nouveau user openerp. On va se connecter avec 
psql -U openerp -W
$ psql -U openerp -W
psql: FATAL:  Ident authentication failed for user "openerp"

sudo vi /etc/postgresql/8.4/main/pg_hba.conf
local all all ident --> local all all trust

sudo /etc/init.d/postgresql-8.4 restart
psql -U openerp -W
psql: FATAL:  database "openerp" does not exist

C'est du au fait que postgres essaie de se connecter à une database qui a le meme nom que l'utilisateur si aucune database n'est spécifiée. On va essayer de se connecter à la database postgres qui est toujours installée. 

psql postgres-U openerp -W
Password for user openerp: 
psql: FATAL:  password authentication failed for user "openerp"
postgres@woodin:~$ psql postgres -U openerp -W
Password for user openerp: 
psql (8.4.5)
Type "help" for help.

postgres=# 

OK ca marche. 


# POSTGRESQL 
## Comment droper une database?

Drop database madatabase;

Si ca marche pas 
select * from pg_stat_activity where datname='madatabase';
et kill -9 les procpid.

puis Drop database madatabase;

# Conversion de wma en ogg 
ffmpeg -i 09\ Piste\ 9.wma -strict experimental -acodec vorbis -aq 100 09\ Piste\ 9.ogg


# INSTALLATION DE SYSLINUX sur 
une clef USB
```
# mkdir /tmp/cdrom
# mount -o loop,exec /path/to/systemrescuecd-x86-x.y.z.iso /tmp/cdrom
# cd /tmp/cdrom
# ./usb_inst.sh listdev
# ./usb_inst.sh writembr /dev/sdx
# ./usb_inst.sh format /dev/sdx1
# ./usb_inst.sh copyfiles /dev/sdx1
# ./usb_inst.sh syslinux /dev/sdx1
```
ne marche pas 

Marche en utilisant le LIVECD rescuecd avec 
```
# sysresccd-usbstick listdev to see which devices are seen as USB-sticks
# sysresccd-usbstick writembr /dev/sdX
# sysresccd-usbstick format /dev/sdXn
# sysresccd-usbstick copyfiles /dev/sdXn
# sysresccd-usbstick syslinux /dev/sdXn
```
# CREATION d'un LIVE CD DEBIAN sur une Clef USB

```
#mkdir /tmp/isohybrid
#cd /tmp/isohybrid
#lb config --distribution squeeze --mirror-bootstrap http://debian.nautile.nc/debian --mirror-chroot-security http://debian.nautile.nc/debian-security/ --language fr --packages-lists "kde-full rescue" --bootappend-live "locales=fr_FR.UTF-8 keyboard-layouts=fr"
#lb build
```
Marche pas avec testing le 170411

# COMMENT METTRE UN TIMESTAMP sur UN NOM DE FICHIER

touch tempo.$(date +%Y%m%d-%Hh%M)

# COMMENT CONNAITRE LE FSYSTEM D'UN DEVICE
fsarchiver probe simple
ou plus élégant :
blkid (du paquet util-linux
mnemotechnique : blockID
Donne aussi UUID

# Comment connaitre les UUID des disques
ls /dev/disk/by-uuid -lha

# Fonts
fc-list
fc-cache -fv – rebuilds cached list of fonts 
dpkg-reconfigure fontconfig-config
dpkg-reconfigure fontconfig


## FS_UUID
If you run "blkid" with no flags it should show you all block devices with details including LABEL (if any), type and uuid. 

# REPAIR GRUB2
J'ai un disque dur qui déconne. On peut le monter en disque dur externe USB mais on arrive pas à booter dessus en DD externe usb.
Comment réparer le grub.

installation du DD en USB externe avec le dock
démarrage sur une clef usb live
ATTENTION ca ne marche pas avec un installation existante sur un DD. Il faut utiliser le live
sudo fdisk -l pour trouver le nom des partitions du disque en panne disons /dev/sdx
sudo mount /dev/sdx2 (partition racine /) /mnt
sudo mount /dev/sdx1 (partition boot)  /mnt/boot
sudo mount --bind /dev /mnt/dev
sudo mount --bind /proc /mnt/proc
sudo chroot /
grub-install /dev/sdx 
grub-install recheck /dev/sdx
exit
sudo reboot

on enleve le USB key live 
et boot selection du DD externe et ca devrait marcher.  


# check md5sum
md5sum -c package.tgz package.md5


# INSTALL DE DEBIAN 
PARTITION MANUELLE
Si on utilise l'option guided use entire disk and set up encrypted LVM sur un DD de 1To la phase d'erasing les data est trop longue (plusieurs heures) donc on choisit manual. On crée une partition en ext3 bootable que l'on mount sur /boot. puis une nouvelle partition puis configure encrypted volume.
puis on crée un volume groupe puis les partition / /home /opt /usr /var /tmp swap comme d'habitued 

# Comment lister les paquets installé?
`aptitude search ~i `

# HOST DISCOVERY
`nmap -sP 192.168.2.1-254`
`nmap -sP 192.168.2.0/24`


# TP-LINK 8810
bridge.
pour communiquer avec le modem en bridge on peut le brancher rj45 en direct sur un pc et ping 192.168.1.1

firmware :3.06L.03-T1.0a-091210.A2pB023k.d17m

# FLASH PLAYER 
mise a jour : # update-flashplugin-nonfree --install
connaitre sa version de 
#strings /usr/lib/flashplugin-nonfree/libflashplayer.so|grep -i flashplayer_  

# NIKON
avant de faire les manips sur la carte 
`#dd if=/dev/sdX of=/tmp/r1 bs=512`

probleme avec les cartes compact flash qui ne veulent plus etre lue dans l'APN
J'essaie dos
mais la carte n'est tjs pas lu par l'APN qui demande a ce qu'elle soit formatée.
Sur une autre carte il y a bcp d'err et renommage auto de nombreux fichiers ca prend trois plombes.


# Télécharger les video de eyetube.net
login lau.francois@worldonline.fr
password habituel
recuperer le sources; trouver le code plugins: { 		
		controls: { url: '/assets_tube/flowplayer/flowplayer.controls-3.2.5.swf',
		
		bufferColor: '#CE151E', volumeColor: '#CE151E', timeColor: '#CE151E'},
		akamai: { url: '/assets_tube/flowplayer/flowplayer.akamai-3.2.0.swf'},
		rtmp: { url: '/assets_tube/flowplayer/flowplayer.rtmp-3.2.3.swf', netConnectionUrl: 'rtmp://video.bmctoday.net/ondemand/eyetube/streams/_definst_/'},
		bwcheck: { url: '/assets_tube/flowplayer/flowplayer.bwcheck-3.2.5.swf', 
		serverType: 'fms',
		dynamic: true}
	},
	clip: {  
		//scaling: 'fit',	
	
		urlResolvers: 'bwcheck',
		provider: 'rtmp',
		bufferLength: 4,
		autoPlay:true,
		bitrates: [
		
		{	url: "dumif_1000kbps", bitrate: 1000, normal:true, isDefault:true },
		{	url: "dumif_748kbps", bitrate: 748, normal:true },
		{	url: "dumif_512kbps", bitrate: 512, normal:true }
		]

et taper rtmpdump -r rtmp://video.bmctoday.net/ondemand/eyetube/streams/_definst_/dumif_1000kbps -o outputfile.flv

A faire:

*rtrtmpdump -r rtmp://video.bmctoday.net/ondemand/eyetube/streams/_definst_/hagaj_748kbps -o removal-canula-2.flv
*rtmpdump -r rtmp://video.bmctoday.net/ondemand/eyetube/streams/_definst_/hekoq_748kbps -o removal_canula.flv
*rtmpdump -r rtmp://video.bmctoday.net/ondemand/eyetube/streams/_definst_/deved_1000kbps -o chandelier_light.flv
*rtmpdump -r rtmp://video.bmctoday.net/ondemand/eyetube/streams/_definst_/gofob_1000kbps -o 9degree_insertion-23G.flv

*rtmp://video.bmctoday.net/ondemand/eyetube/streams/_definst_/sisire -o principles_sutureless_vitrectomy.flv
*rtmp://video.bmctoday.net/ondemand/eyetube/streams/_definst_/gibofo -o retrobulbar_hemorrhage.flv
*rtmp://video.bmctoday.net/ondemand/eyetube/streams/_definst_/bosuw_1000kbps -o reichel-chun-forceps.flv

*rtmp://video.bmctoday.net/ondemand/eyetube/streams/_definst_/pimoo_1000kbps
rtmp://video.bmctoday.net/ondemand/eyetube/streams/_definst_/forub_1000kbps

rtmp://video.bmctoday.net/ondemand/eyetube/streams/_definst_/locri_1000kbps


# COMMENT TELECHARGER SUR ARTE+7
aller sur la page de l'émission à telécharger 
view source
rechercher quelque chose du genre :videorefFileUrl=http%3A%2F%2Fvideos.arte.tv%2Ffr%2Fdo_delegate%2Fvideos%2Fle_dessous_des_cartes-6590276%2Cview%2CasPlayerXml.xml"
taper dans l'url cette adresse en convertissant %2F = / et %2C = ,
Dans cette nouvelle page récuper : <video lang="fr" ref="http://videos.arte.tv/fr/do_delegate/videos/le_dessous_des_cartes-6590280,view,asPlayerXml.xml"/>
A mettre dans l'url et récuperer enfin la bonne adresse. 
Mettre la bonne adresse entre ""

rtmpdump -o orgueil-prejuge-3.flv -r 'rtmp://artestras.fcod.llnwd.net/a3903/o35/mp4:geo/videothek/default/arteprod/A7_SGT_ENC_08_046395-003-A_PG_HQ_FR?h=8c32a538b827423589901e7acc2dd37f'

rtmpdump -o les-larmes-ameres-de-petra-van-kanpt.flv -o 'rtmp://artestras.fcod.llnwd.net/a3903/o35/mp4:geo/videothek/default/arteprod/A7_SGT_ENC_08_045604-000-A_PG_HQ_FR?h=e20dfeb9d6691b79c8c89fddfbf165ce'


*rtmp://artestras.fcod.llnwd.net/a3903/o35/mp4:geo/videothek/EUR_DE_FR/arteprod/A7_SGT_ENC_08_039191-000-A_PG_HQ_FR?h=6022a761fedf7414294de00d6502eab4

*rtmp://artestras.fcod.llnwd.net/a3903/o35/mp4:geo/videothek/ALL/arteprod/2011/11/10/ar-dz-20111105-obama_DE_149233_16by9_800_MP4?h=006311e1c700963745c9562a15283640

*rtmp://artestras.fcod.llnwd.net/a3903/o35/mp4:geo/videothek/default/arteprod/A7_SGT_ENC_08_046171-000-A_PG_HQ_FR?h=8793207dc1dc41fda58ff8f5e2aff9a9

*rtmp://artestras.fcod.llnwd.net/a3903/o35/mp4:geo/videothek/default/arteprod/A7_SGT_ENC_08_035710-006-A_PG_HQ_FR?h=68a55efc4481a8bf43e210242d439d71

rtmp://artestras.fcod.llnwd.net/a3903/o35/mp4:geo/videothek/ALL/arteprod/2010/04/03/ARTE3123758_FR_19217_16by9_800_MP4?h=be72da68905b6fafbcdeba0f8c09e5f5

*rtmp://artestras.fcod.llnwd.net/a3903/o35/mp4:geo/videothek/ALL/arteprod/2010/06/23/ar-dz-0024_fr_20100619-papouasie_35717_16by9_800_MP4?h=82ac29feefa8e4324c1d148d37586650



rtmpdump -o philosophie-vengeance.fvl -r 'rtmp://artestras.fcod.llnwd.net/a3903/o35/mp4:geo/videothek/EUR_DE_FR/arteprod/A7_SGT_ENC_08_046412-010-A_PG_HQ_FR?h=e5d705e18a5cfd121fdcd4eafd129516'



rtmpdump -o macular-surgery-7.flv -r rtmp://video.bmctoday.net/ondemand/eyetube/streams/_definst_/sigim_1000kbps


rtmpdump -o macular-surgery-12.flv -r rtmp://video.bmctoday.net/ondemand/eyetube/streams/_definst_/hekoq_748kbps
rtmpdump -o macular-surgery-13.flv -r rtmp://video.bmctoday.net/ondemand/eyetube/streams/_definst_/hagaj_748kbps


rtmpdmp -o phako-with-iris-coloboma.flv -r rtmp://video.bmctoday.net/ondemand/eyetube/streams/_definst_/mafed_1000kbps


rtmpdump -o choroidal-drainage.flv -r rtmp://video.bmctoday.net/ondemand/eyetube/streams/_definst_/mevof_1000kbps

rtmp -o reportage-baie-de-rio.flv -r "rtmp://artestras.fcod.llnwd.net/a3903/o35/mp4:geo/videothek/ALL/arteprod/A7_SGT_ENC_08_030273-373-A_PG_HQ_FR?h=0e08cfc933c07c58c81329addb153d96"

rtmpdump -o vers-union-fiscale-europenne.flv -r "rtmp://artestras.fcod.llnwd.net/a3903/o35/mp4:geo/videothek/ALL/arteprod/2012/06/15/AJ-20120614-FIS_FR_208164_16by9_800_MP4?h=74bcb8a0e4bdc2f3021a7b5c2ff9d8b1"


rtmpdump -o retour-au-chateau-1-7.flv -o 'rtmp://artestras.fcod.llnwd.net/a3903/o35/mp4:geo/videothek/EUR_DE_FR/arteprod/A7_SGT_ENC_08_046311-001-B_PG_HQ_FR?h=c289243c24f01a6874035a7d01488998'

# INSTALLATION DE JAVA
openjdk-7-jre 
et icedtea-plugin


# EMACS

Gestion des paquet avec ELPA
M-x package-list-packages
r pour update de la liste
i pour le marquer pour l'installation
x pour le downloader et l'installer. 


Comment supprimer le highlight des lignes trop longues. Je n'ai réussi qu'a augmenter la taille des lignes sans highlight avec :

M-x customize-group whitespace
puis whitespace Line.





# COMPRENDRE L'ALIGNEMENT
fdisk
/dev/sda
head 255
63 sectors/track
7783 cylinder
sector size 512 bytes

Heads = track per cylinder
Sectors = Sectors per track

Les partitions doivent etre alignées sur le SSD erase block size
Le erase block size est de 512kb sur les SSD en général.


Pour LVM:
En premier avec fdisk -l on cherche les partitions dont on dispose. 
sur un disque: fdisk /dev/sdX
add a new partition
on la choisit primary
First cylinder : default
Last cylinder ou taille : on choisit ce que l'on veut +25000M ie (25G)
on choisit le type Linux LVM: 8e
On écrit la table de partition : w
Cela crée une partition /dev/sdX1 de type Linux LVM
Ensuite on crée une partition pour LVM avec un regroupement de partition Linux LVM
pvcreate /dev/sdX1 /dev/sdY1 .....
pvdisplay
Sur ce physical volume on crée un volume groupe
vgcreate un_nom_de_groupe /dev/sdX1 /dev/SdY1 ....
Ensuite on crée un logical volume
lvcreat --name unnom_volume --size unetaille nom_de_groupe
Ensuite il faut faire les file systeme sur les logical volume
mkfs."typefs" /dev/nom_groupe/nom_volume
Apres il faut les monter. 

Sur mon disque ssd de 64g
je cree une partition primaire /boot de 100M ca devrait suffire
Je cree une autre partition primaire de 10G
Je cree un physical volume  sur cette sda2
Je crée un volume groupe sur ce physical volume
Je crée un logical volume de 5G que j'appelle root
Je crée un logical volume de 2G que j'appelle home

fdisk -l -u -c /dev/sdX
total sector le noter 125045424
combien j'ai de Gigabyte (Gigabyte formula):
125045424 / 2,097,152 = 59.6263 Gb
Combien j'ai de sector restant dans cette division
avec python
125045424 % 2 097 152 = 1313456
megabyte formula
1 312 456 / 2,048 = 641 Mb

Si je veux une partition de 26G:
2 097 152 * 26 = 54,525,952 sectors
Si je veux une partition de 256Mb
2 048 * 750 = 1,536,000 sectors
Si je veux une  partition de 26,750 (54 525,952 + 1,536,000) = 56,061,952 sectors

Alignement: Le starting sector doit etre divible par 512 pas de reste dans la division par 512.
--------------------d


Je veux une partition de 100Mb : 2 048 * 100 = 204 800
204 800 - 1 sector = 204 799

start 2048 end 204 799

Et une autre partition qui occupe tout le reste du SSD
fdisk -H 32 -S 32 -cu /dev/sdx

création des physical volume avec pvcreate --dataalignment 512k /dev/sdXY

Création des volume group:vgcreate fileserver /dev/sdb1 /dev/sdc1 /dev/sdd1 /dev/sde1

Création des logical volume:

Creation d'un systeme de fichier:
mkfs.ext4 -O extent -b 4096 -E stride=128,stripe-width=128 /dev/mapper/ssd-debian

# RIP RIPPER LES DVD

acidrip
dvdrip
handbrake

# PLAN de BATAILLE POUR AUGMENTER LE STOCKAGE SUR LES SERVEURS

L'OS est installé sous /dev/hdc
hdc1 /boot
hdc3 / 
hdc2 swap

pvscan : sda1;sda2;sda3;sda4 : 640G

hdd1: 320G. N'est pas monté. il n'a pas de données interesante. il est monté sur archive

pvscan : sda1;2;3;4

Il aurait ete interessant de faire un ll /etc >> droit.txt pour garder une trace des droits des fichiers car ils sont perdus avec fsarchiver.

Il faut récupérer les films du DD de la videobox qui n'ont pu etre récupéré avant l'augmentation de la taille du serveur.

Avant de pouvoir utiliser le hdd1 il faut récupérer le dir dvdrp-data


# CHROOT
problem avec le sysrescuecd
% chroot /host 
chroot: failed to run command `/bin/zsh': No such file or directory
cet erreur  est du à une mauvaise configuration de la variable shell
This type of error is due to the wrong setting of the SHELL variable.

root@sysresccd /root % echo $SHELL
/bin/zsh
root@sysresccd /root % SHELL=/bin/bash
root@sysresccd /root % echo $SHELL          
/bin/bash

Now you can chroot ! 

# TASK LIST OFF LINE

- remember the milk pas de off line depuis html5

- toodledo tres vite on depasse les possibilité de la version gratuite

- plancake ne marche plus en offline des que l'on ferme le browser

- todolist

- Google task la technique avec l'onglet dans TB ne marche pas offline.
pas moyen d'avoir l'adresse ics du calendrier associé.
C'est bien car pas besoin de due date mais pas de offline.

- Coolendar c'est pas mal se lie bien avec TB car il y a un lien ics qui permet d'avoir un calendrier qui fonctionne bien en cache. Le probleme c'est que c'est un calendrier et que c'est pas pratique pour des taches que je n'ai pas de date pour les terminer.en fait bodf ne se synchrsosie pas tres bien avec TB

Ne se lie pas bien avec google calendar. 



# MYSQL PASSWORD
New password pour Mysql 'root' user : 123b654


# AMPOULE 
6V
20WG4
 
# CHMOD 

## Recursively chmod directories only
June 19th, 2006

`find . -type d -exec chmod 755 {} \;`

This will recursively search your directory tree (starting at dir ‘dot’) and chmod 755 all directories only.

Similarly, the following will chmod all files only (and ignore the directories):

`find . -type f -exec chmod 644 {} \;`

# ASUS TRANSFORMER TFT300T
- idVendor: 0b05
- idProduct: 4c80

 ┌─────────────────────────────────────────────────────────────────────────────┤ Configuring backuppc ├─────────────────────────────────────────────────────────────────────────────┐                                
                               │ Web administration default user created
                               │ BackupPC can be managed through its web interface:                                       │  http://woodin.venus.org/backuppc/                                                                                                           
                               │ For that purpose, a web user named 'backuppc' with 'jUifU8ft' as password has been created. You can change this password by running 'htpasswd /etc/backuppc/htpasswd backuppc'.  │                                




# PROBLEME avec GRUB2
Cette methode ne marche pas si pas de connection internet
J'avais fait une mise à jour par aptitude safe-upgrade et je n'arrivais plus à booter sur le server. 

un cd live de debian. 


qui n'est pas dans le live.   	      
`blkid` pour avoir une idée des partitions et savoir ce qu'il faut monter

setxkbmap fr
mount /dev/sda3 /mnt
mount /dev/sda1 /mnt/boot
for i in /dev /dev/pts /proc /sys; do mount -B $i /mnt$i; done
chroot /mnt /bin/bash
grub-install --recheck /dev/sda
update-grub
exit 
halt
enlever l'usblive

# PROBLEME AVEC UNE INSTALLATION QUI EST CASSEE

boot sur un livecd

mkdir -p /mnt/lf/boot
mount /dev/volgroup/root /mnt/lf
mount /dev/sda1 /mnt/lf/boot
for i in /dev /dev/pts /proc /sys; do mount -B $i /mnt/lf$i; done
chroot /mnt/lf /bin/bash
source /etc/profile 
export PS1="(chroot) $PS1"
cp /mounts/proc /etc/mtab

aptitude ne marche pas utiliser apt-get update et upgrade
exit 
reboot

# Comment ajouter un user à un ou plusieurs groups ?
usermod -a -G nomgroup1,nomgroup2 nomuser


# Linux Mint
## problem d'impression recto-verso 

- 1 Menu / Administration / printers / Installable options / cocher duplex.
- 2 Relancer libreoffice

Par contre je n'ai pas eu ce probleme avec Okular.

# password generateur.
`xkcdpass` il y a un paquet pour cela.

# cron job
- anacrontab -> va lancer les scripts meme si la machine n'était pas allumée au moment ou ils auraient du etre joués.
- cron.d
- cron.daily
- cron.hourly
- cron.monthly
- crontab
- cron.weekly

On utilise crontab pour éditer un crontab
```
# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
```

## j'ai un problem avec le script 
- reboot  
c'est problement un probleme de variables d'environnement

### Comment obtenir les variables d'environnement de cron?
co- rajouter `* * * * *  env > ~/cronenv `
- cronenv 

```
LANGUAGE=fr_FR:fr
HOME=/root
LOGNAME=root
PATH=/usr/bin:/bin
LANG=fr_FR.UTF-8
SHELL=/bin/sh
PWD=/root
```
`which reboot` --> /sbin/reboot

Donc il y a bien un probleme de variable. 
Je rajoute dans le crontab -e : `30 6 * * 1-5 . $HOME/.zshrc; /sbin/reboot`
On verra si cela regle le probleme demain matin. 
Non cela ne marche pas.
https://stackoverflow.com/questions/22743548/cronjob-not-running

- 1 vérifier que cron tourne : `ps aux | grep cron`
- 2 `service cron start` `service cron restart`
- 3 cron fonctionne :` * * * * * /bin/echo "cron works" >> ~/file.txt`
- 4 `tail -f /var/log/messages` `tail -f /var/log/syslog`
- 5 need to have write access to the file you are redirecting the output.
- 6 do not rely on environment variables like PATH, as their value will likely not be the same under cron as under an interactive session
- dont suppress output while debugging
  - commonly used is this suppression: 30 1 * * * command > /dev/null 2>&1
  - suppress > /dev/null 2>&1
  - add ̀>> cront.out 2&1` will append standard output and standard error to cron.out in the invoking user's home directory.

## syntax
# Minute  Hour  Day of Month      Month         Day of Week    User Command    
# (0-59) (0-23)   (1-31)    (1-12 or Jan-Dec) (0-6 or Sun-Sat)  

    0       2       *             *                *          root /usr/bin/finduser field c'est juste pour root. Pas de user field pour les autres users. 


# Debian 10 
`ifconfig` deprecated   
à la place `ip addr`

## su 
`su` maintenant c'est `su -`


# Dual screen
## to be tested
```
sudo apt-get install --reinstall xserver-xorg-video-intel xserver-xorg-core
sudo dpkg-reconfigure xserver-xorg
```
Je suis passé de debian 9 à debian 10 et cela a reglé mon probleme de dual screen donc je n'ai pas testé.

# Comment changer de version de pythonChoisir sa version de python

## Comment connaitre sa version de python
```
python --version
Python 3.5.3
```
## liste des alternatives
```
# update-alternatives --list python
/usr/bin/python2.7
/usr/bin/python3.7
```

### List all options

```
$ ls /usr/bin/python*
/usr/bin/python  /usr/bin/python2  /usr/bin/python2.7  /usr/bin/python3  /usr/bin/python3.5  /usr/bin/python3.5m  /usr/bin/python3m
```

```
# update-alternatives --install /usr/bin/python python /usr/bin/python2.7 1
update-alternatives: using /usr/bin/python2.7 to provide /usr/bin/python (python) in auto mode
# update-alternatives --install /usr/bin/python python /usr/bin/python3.5 2
update-alternatives: using /usr/bin/python3.5 to provide /usr/bin/python (python) in auto mode
```
Plus l'entier est haut plus c'est prioritaire.



## Comment changer de version de python

```
# update-alternatives --config python
There are 2 choices for the alternative python (providing /usr/bin/python).
```

### Si on veut changer localement pour un user.
In case you need to only change a python version selectively on per user basis, you may try to edit user's .bashrc file. For example to change to python version 3.5 execute the following linux commands:
```
$ python --version
Python 2.7.13
$ echo 'alias python="/usr/bin/python3.5"' >> ~/.bashrc
$ . .bashrc 
$ python --version
Python 3.5.3
```

# Imprimante, hplip, hp-plugin, printing
j'ai eu un probleme pour installer l'imprimante. 
Des problemes de dépendance
dmesg avec des messages d'erreur venant de apparmor DENIED
Finalement j'ai fait :


- Install apparmor utils : `sudo apt-get install apparmor-utils`
- Run : `sudo aa-disable /usr/share/hplip/plugin.py`
- Run as normal user, not as root: `hp-plugin`
- Run `apt-get remove --purge apt-get remove hp--purge printer-driver-hpcups`
- Run `aptitude install hplip`
Et finalement c'est bon. 

- run :`xsane`le scanner est detecté, le scan s'effectue mais rien de n'affiche
- run cat /var/log/syslog
- j'ai fait un uninstall/install xsane et libsane et finalement c'est bon.
- mais en fait il faut d'abord faire : Acquire Preview puis Scan

## uninstall hplip

pour avoir la version minimale de hplip:
https://developers.hp.com/hp-linux-imaging-and-printing/supported_devices/index


# UID GID
## Lister les UID >1000
`awk -F: '($3 >= 1000) {printf "%s:%s\n",$1,$3}' /etc/passwd`
## Lister les UID de tous les user
`awk -F: '{printf "%s:%s\n",$1,$3}' /etc/passwd`
## Comment changer son user ID
`usermod -u [NEW_USER_ID] [USERNAME]`

# Network 
## Lister les interfaces 
`ip link`

# gestion des paquet - apt
## lister les dependances
### apt_rdepends
- apt-cache depends <nomdupaquet>
- apt-cache rdepends <nomdupaquet>
### debtree
## générer le source list
https://debgen.simplylinux.ch/index.php?generate

# installation de ohmyzsh
## install de zsh et changement de shell

``` 
apt install -y zsh git curl wget
chsh -s (which zsh)
```

On quitte et on relance le terminal

`echo $SHELL` qui affiche /usr/bin/zsh

## Installation des polices
```
wget https://github.com/ryanoasis/nerd-fonts/releases/download/v2.0.0/Hack.zip -O ~/hack.zip
unzip ~/hack.zip -d ~/hack && rm ~/hack.zip
find ~/hack/ -iname "*Windows*" -exec rm {} \; # suppression des fonts compatible Windows.
sudo mv hack /usr/share/fonts/
fc-cache -v # refresh le cache des polices.
fc-list | grep -i 'hack'# vérifier la présence et prise en charge de la police.
```

On install ohmyzsh et des plugins

```
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:=~/.oh-my-zsh/custom}/plugins/zsh-completions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```
On edite .zshrc:

plugins=(
zsh-autosuggestions
zsh-completions
zsh-syntax-highlighting)

source .zshrc

## Powerlevel10k
```
git clone https://github.com/romkatv/powerlevel10k.git $ZSH_CUSTOM/themes/powerlevel10k
```
changer le theme dans .zshrc : ZSH_THEME="powerlevel10k/powerlevel10k".

`source .zshrc`

### Configuration de powerlevel10k

`p10k configure`
On peut aussi editer `~/.p10k-zsh`

