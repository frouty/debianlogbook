Users and Groups
################

Comment lister que les users humains
************************************
Les users humains ont un UID >= 1000

.. code::

   cut -d: -f1,3 /etc/passwd | egrep ':[0-9]{4}$' | cut -d: -f1
   
User nobody
***********

Son id est traditionnelement : 65534

Il est utilisé par servers NFS qui ne peuvent pas faire confiance aux uid et gid fournis par les clients ou qd la l'option root-squash

Il est parfois recommnandé d'utiliser ce user pour des programmes de peu de confiance ou des manipulation de data de peu de confiance. Mais c'est un mauvais conseille. Chaque service doit avoir son propre user account dédié.

Nobody c'est uniquement pour NFS. 