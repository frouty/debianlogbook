snap
####
installation
************
.. code::

   sudo apt update
   sudo apt upgrade -y

   sudo apt install snapd
   sudo snap install core
   
   
   
start and enable snap
*********************

.. code::

   sudo systemctl start snapd
   sudo systemctl enable snapd
   sudo systemctl status snapd
   
Install snap store
******************

.. code::

   sudo snap install snap-store

Gestion des paquets
*******************

Liste  des paquets installés
----------------------------

.. code::

   snap list
   
Chercher un paquet
------------------

.. code::

   snap find nom_du
   
Uninstall a package
-------------------

.. code::

   sudo snap remove nom_du_paquet --purge
   
le snap-store ne marche pas bien. Il y a tres peu de logiciel qui sont proposés. Je vais le laisser tourner et voir s'il finit par downloader des informations. 