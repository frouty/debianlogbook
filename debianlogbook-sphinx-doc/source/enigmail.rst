
Enigmail
########

Gestion des clefs sous debian bulleye gnome
*******************************************

help
====

`ici pour trouver de l'aide <https://help.gnome.org/users/seahorse/stable/index.html.en\#pgp-keys>`_

Les principes
=============

- 1 création de la clef publique et privée GPG
- 2 Configurer Evolution pour qu'il utilise cette paire de clef:
    Clique Droit sur le nom du compte / Security
    On recupere le key ID dans seahorse.
    On a un tout un tas de parametres à configurer
- 3 dans evolution dans la barre d'outils on a des icones pour parametre le pgg de chaque mail et le menu option pour choisir l'encryption.

Gestions des clefs sur plusieurs pc's
=====================================
Il faut que j'ai les memes clefs sur différents pc's sinon cela ne va pas marcher

`voir ici <https://askubuntu.com/questions/32438/how-to-share-one-pgp-key-on-multiple-machines>`_

Comment supprimer un compte dans evolution
==========================================
Dans Edit / Preferences
ou keyboard shortcut Ctl-Shift-S
Peut etre il faut se mettre offline (en bas dans le coin à gauche)

Ajout d'accounts dans evolution
===============================

J'ai utilise les fonctions de Settings / Online Account de gnome. Du coup il n'y a rien à faire dans evolution

Key Management
--------------

`ici document tres détaillé sur la gestion des clefs avec enigmail <https://www.enigmail.net/index.php/en/user-manual/key-management>`_

Où sont stockés les clefs ?
---------------------------

Qd on fait un backup il faut tenir compte du fait que le format de stockage peut changer au cours du temps. Seul le format d'export est stable. Donc il faut backuper les keyring files (pubring, secring, trustdv) pour rendre les restaurations plus faciles.

Je n'ai pas trouvé les clefs que j'avais mises en place avec manjaro 


Comment faire des clefs?
------------------------
`gpg --gen-key` 
Je vois dans ~/.gnupg :


/home/lfs/.gnupg
├── gpg-agent.conf
├── openpgp-revocs.d
│   └── 675764A7074570180EBE220AE50C7650F7CAEA0B.rev
├── private-keys-v1.d
│   ├── 020326FB896B4BF880879B2EA49BD798CF634B96.key
│   └── D86305F6D9180E8FEBE0465702149C935FFB3A3F.key
├── pubring.kbx
├── pubring.kbx~
└── trustdb.gpg

Lister les clefs publiques
--------------------------

`gpg --list-keys`

Pour partager une clef il faut d'abord l'exporter: 

`gpg --output unnomdefichier --armor --export any_part_of_the_user_ID_to_identify_the_key`
 
Je vais dans l'application seahorse et je vois que mes clefs sont bien listés dans gGnuPG keys. 

Gestion des clefs avec enigmail
-------------------------------
`ici <https://help.gnome.org/users/seahorse/stable/index.html.en#pgp-keys>`__

Backup des clefs
----------------

Avec l'application seahorse j'ai exporté les clefs dans le repertory keys-backup. Pour savoir ce que sont les fichiers dedans on peut faire `file ~/key-backup/nom_du_fichier`

