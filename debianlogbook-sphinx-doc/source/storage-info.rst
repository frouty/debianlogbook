Obtenir des informations sur les disques installé sur ton pc
############################################################

::


   lsblk -o NAME,MOUNTPOINT,MODEL,ROTA

Pour savoir sur quel disque est installé ton OS
###############################################

::

   df /


Pour savoir quel est le type de disque dur sans ouvrir le PC
############################################################

::

   lsblk -d -o name,rota | awk 'NR>1' | grep -v loop | while read CC; do dd=$(echo $CC | awk '{print $2}'); if [ ${dd} -eq 0 ]; then echo $(echo $CC | awk '{print $1}') is a SSD drive; fi; done
