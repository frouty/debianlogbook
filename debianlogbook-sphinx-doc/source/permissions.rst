Permissions
###########


+---+-------+----------------------------+-+-+-+-+-+-+-+
| 0 | - - - | aucun droit                | | | | | | | |
| 1 | - - x | execution                  | | | | | | | |
| 2 | - w - | écriture                   | | | | | | | |
| 3 | - w x | ecriture et execution      | | | | | | | |
| 4 | r - - | lecture seule              | | | | | | | |
| 5 | r - x | lecture et execution       | | | | | | | |
| 6 | r w - | lecture ecriture           | | | | | | | |
| 7 | r w x | lecture ecriture execution | | | | | | | |
+---+-------+----------------------------+-+-+-+-+-+-+-+

+---------+-------+-----------+------------------------------------------------------------------------------------------------------+
| command | octal |           | signification                                                                                        |
+=========+=======+===========+======================================================================================================+
| chmod   | 400   | file      | To protect a file against accidental overwriting.                                                    |
| chmod   | 500   | directory | To protect yourself from accidentally removing, renaming or moving files from this directory.        |
| chmod   | 600   | file      | A private file only changeable by the user who entered this command.                                 |
| chmod   | 644   | file      | A private file only changeable by the user who entered this command.                                 |
| chmod   | 660   | file      | Users belonging to your group can change this file, others don't have any access to it at all.       |
| chmod   | 700   | file      | Protects a file against any access from other users, while the issuing user still has full access.   |
| chmod   | 755   | directory | For files that should be readable and executable by others, but only changeable by the issuing user. |
| chmod   | 755   | file      | Standard file sharing mode for a group.                                                              |
| chmod   | 777   | file      | Everybody can do everything to this file.                                                            |
+---------+-------+-----------+------------------------------------------------------------------------------------------------------+

Changement de user
******************
chown unuser unfichier

Changement de group
*******************
chgrp ungroup unfichier

Changement group et user en meme temps
**************************************
chown unuser:ungroup unfichier

Les droits des repertoires
**************************
X est nécessaire pour que les repertoires puissent etre ouvert.

X est à éviter dans les fichiers html et txt.