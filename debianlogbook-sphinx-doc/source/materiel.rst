Comment obtenir des informations sur le matériel
##################################################

lshw
=====

::
   apt install hw
   lshw -short
   lshw -C disk marque, model, serie, taille ....
   

hwinfo
=======

::
   
   apt install hwinfo
   
   hwinfo --short
   hwinfo --log /path/to/file.txt

inXi
====

::

   apt install inxi
   inxi résumé du hardware
   inxi -F plus d'informations
   inxi -A audio
   inxi -C CPU
   inxi -D drives
   inxi -G carte graphique
   inxi -M carte mere bios
   inxi -P partitions du disque
   inxi -N Reseau
   inxi -R Raid
   inxi -S nom d'hote, noyau, distro, ....
   inxi -n informations de reseau avancées. 
   inxi -s temperature, vitesse de ventilateur

lspci lscpu lsmem lsusb lsblk lsscsi
=====================================

Ce sont des commandes de base. 

Infos sur le CPU
================

::

   cat /proc/cpuinfo
   lshw -C CPU (-C -class)
   lscpu
   cpuid 'apt install cpuid si necessaire'