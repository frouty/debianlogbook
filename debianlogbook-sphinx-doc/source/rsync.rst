rsync
#####

.. code::

   rsync dir1/ dir2
   
Attention au / car si on ne le met pas on obtiendra /dir2/dir1/[files]

Dry run
*******

.. code::
   rsync -avn dir1/ dir2
   
rsync move
**********

.. code::
   rsync -zzav --progress --remove-source-files sourcedir/ destinationdir
   rsync -rtDvz --progress --remove-source-files sourcedir/ destinationdir