SSH et échange de clefs
#######################

Par défaut la connection ssh se fait avec un mot de passe.
Cette méthode d'authentification n'est pas la plus sure. Il
est conseillé de se connecter en utilisant
l'authentification par échange de clefs.

- Sur le l'ordinateur qui va lancer la commande SSH:
    - ~/.ssh/id_xxx la clef privée
    - ~/.ssh/id_xxx.pub la clef publique.
- Sur l'ordinateur distant, le server:
  -  ~/.ssh/authorized_keys : liste des clefs publiques autorisées pour se connecter à l'utilisateur. C'est là que l'on va mettre les clefs publiques.

Génerer la paire de clef sur le pc local
****************************************
- ssh-keygen -t ed25519 ou
- ssh-keygen -t rsa -b 4096
- entrer une passphrase.

Transferer la clef publique sur le serveur distant
**************************************************
  ssh-copy-id <username>@ip_du_serveur.

La clef publique est copiée dans le fichier
~/.ssh/authorized_keys sur le serveur distant

