Cron
####

pour des raisons de sécruté Cron ytulise une environnement shell tres limité. Minimal PATH and HOME variable. 

Dans la plupart des cas il faut mettre le chemin absolu pour toutes les commandes. 


Comment vérifier la syntaxe d'un fichier cron
*********************************************

`ici <https://crontab.guru/>`_


.. code-block::console

sudo crontab -e
30 22 * * * root /usr/sbin/shutdown -h now

