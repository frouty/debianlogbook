J'ai plus de place dans mon systeme
###################################

::

   df -h
   Filesystem                      Size  Used Avail Use% Mounted on
   /dev/dm-0                       8.2G  7.7G     0 100% /
   udev                             10M     0   10M   0% /dev
   tmpfs                           743M   11M  733M   2% /run
   tmpfs                           1.9G  4.0K  1.9G   1% /dev/shm
   tmpfs                           5.0M  4.0K  5.0M   1% /run/lock
   tmpfs                           1.9G     0  1.9G   0% /sys/fs/cgroup
   /dev/mapper/openerp70--vg-var   102G  4.8G   93G   5% /var
   /dev/sda1                       236M   33M  191M  15% /boot
   /dev/mapper/openerp70--vg-tmp   360M  2.1M  335M   1% /tmp
   /dev/mapper/openerp70--vg-home  197G   18G  169G  10% /home
   tmpfs                           372M     0  372M   0% /run/user/1000

qui est dm-0? dm = device mapper?
*********************************
device mapper du kernel utilisé par LVM.

::

   $ dmsetup ls 
   openerp70--vg-tmp (254:3)
   openerp70--vg-swap_1 (254:1)
   openerp70--vg-var (254:2)
   openerp70--vg-home   (254:4)
   openerp70--vg-root   (254:0)

::

   lsblk -o +kname

NAME                     MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT KNAME
sda                        8:0    0 596.2G  0 disk            sda
├─sda1                     8:1    0   243M  0 part /boot      sda1
├─sda2                     8:2    0     1K  0 part            sda2
└─sda5                     8:5    0 465.5G  0 part            sda5
  ├─openerp70--vg-root   254:0    0   8.4G  0 lvm  /          dm-0
  ├─openerp70--vg-swap_1 254:1    0   7.5G  0 lvm  [SWAP]     dm-1
  ├─openerp70--vg-var    254:2    0 102.8G  0 lvm  /var       dm-2
  ├─openerp70--vg-tmp    254:3    0   380M  0 lvm  /tmp       dm-3
  └─openerp70--vg-home   254:4    0   200G  0 lvm  /home      dm-4
sr0                       11:0    1  1024M  0 rom             sr0


::

   ls /dev/mapper
   crw------- 1 root root 10, 236 Jun 12 14:35 control
   lrwxrwxrwx 1 root root       7 Jun 12 14:35 openerp70--vg-home -> ../dm-4
   lrwxrwxrwx 1 root root       7 Jun 12 14:35 openerp70--vg-root -> ../dm-0
   lrwxrwxrwx 1 root root       7 Jun 12 14:35 openerp70--vg-swap_1 -> ../dm-1
   lrwxrwxrwx 1 root root       7 Jun 12 14:35 openerp70--vg-tmp -> ../dm-3
   lrwxrwxrwx 1 root root       7 Jun 12 14:35 openerp70--vg-var -> ../dm-2

::

   $ ls -ahlp /dev/disk/by-id
   $ ls -ahlp /dev/mapper

Obtenir des détails sur un dm:
******************************
::

    $ dmsetup info /dev/dm-0
   Name:              openerp70--vg-root
   State:             ACTIVE
   Read Ahead:        256
   Tables present:    LIVE
   Open count:        1
   Event number:      0
   Major, minor:      254, 0
   Number of targets: 1
   UUID: LVM-s0cNp65awilF27D1gcfIGLoGhDLCGzLcpGQ3Op5PZjPDs6lP1WkiOoYmKUsKip00

Quel est le device physique qui supporte le LVM
***********************************************
::

   $ pvdisplay -m

  --- Physical volume ---
  PV Name               /dev/sda5
  VG Name               openerp70-vg
  PV Size               465.52 GiB / not usable 2.00 MiB
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              119173
  Free PE               37499
  Allocated PE          81674
  PV UUID               Hth2w4-qb5a-sDRd-PaDg-pvsP-zQCc-XAtm2O
   
  --- Physical Segments ---
  Physical extent 0 to 2144:
    Logical volume   /dev/openerp70-vg/root
    Logical extents  0 to 2144
  Physical extent 2145 to 2859:
    Logical volume   /dev/openerp70-vg/var
    Logical extents  0 to 714
  Physical extent 2860 to 4778:
    Logical volume   /dev/openerp70-vg/swap_1
    Logical extents  0 to 1918
  Physical extent 4779 to 4873:
    Logical volume   /dev/openerp70-vg/tmp
    Logical extents  0 to 94
  Physical extent 4874 to 56073:
    Logical volume   /dev/openerp70-vg/home
    Logical extents  0 to 51199
  Physical extent 56074 to 81673:
    Logical volume   /dev/openerp70-vg/var
    Logical extents  715 to 26314
  Physical extent 81674 to 119172:
    FREE
   
Comment agrandir vg-root?
########################

pvs pour savoir ce qui reste de disponible

::

    $ pvs
  PV         VG           Fmt  Attr PSize   PFree  
  /dev/sda5  openerp70-vg lvm2 a--  465.52g 146.48g

Je vois qu'il reste 146.48 G de libre

::

   $ vgdisplay 
  --- Volume group ---
  VG Name               openerp70-vg
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  8
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                5
  Open LV               5
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               465.52 GiB
  PE Size               4.00 MiB
  Total PE              119173
  Alloc PE / Size       81674 / 319.04 GiB
  Free  PE / Size       37499 / 146.48 GiB
  VG UUID               s0cNp6-5awi-lF27-D1gc-fIGL-oGhD-LCGzLc
 
 J'ai un volume group qui s'appelle openerp70-vg qui a 146.48 G de dispo.
 
 On va voir les logical volume ::
 
   $ lvdisplay 
  --- Logical volume ---
  LV Path                /dev/openerp70-vg/root
  LV Name                root
  VG Name                openerp70-vg
  LV UUID                pGQ3Op-5PZj-PDs6-lP1W-kiOo-YmKU-sKip00
  LV Write Access        read/write
  LV Creation host, time openerp70, 2015-07-15 23:31:56 +1100
  LV Status              available
  # open                 1
  LV Size                8.38 GiB
  Current LE             2145
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           254:0
   
  --- Logical volume ---
  LV Path                /dev/openerp70-vg/var
  LV Name                var
  VG Name                openerp70-vg
  LV UUID                STNDLE-msfI-fImk-cX4C-njvG-R5Cb-znoeI5
  LV Write Access        read/write
  LV Creation host, time openerp70, 2015-07-15 23:31:56 +1100
  LV Status              available
  # open                 1
  LV Size                102.79 GiB
  Current LE             26315
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           254:2
   
  --- Logical volume ---
  LV Path                /dev/openerp70-vg/swap_1
  LV Name                swap_1
  VG Name                openerp70-vg
  LV UUID                jN5zPz-LQL6-2uH7-Dh2Q-3TPZ-IcYL-oeADtc
  LV Write Access        read/write
  LV Creation host, time openerp70, 2015-07-15 23:31:57 +1100
  LV Status              available
  # open                 2
  LV Size                7.50 GiB
  Current LE             1919
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           254:1
   
  --- Logical volume ---
  LV Path                /dev/openerp70-vg/tmp
  LV Name                tmp
  VG Name                openerp70-vg
  LV UUID                0899Qy-Eq5u-gaZc-obE7-OjCr-Joc3-eH6KA6
  LV Write Access        read/write
  LV Creation host, time openerp70, 2015-07-15 23:31:57 +1100
  LV Status              available
  # open                 1
  LV Size                380.00 MiB
  Current LE             95
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           254:3
   
  --- Logical volume ---
  LV Path                /dev/openerp70-vg/home
  LV Name                home
  VG Name                openerp70-vg
  LV UUID                awfoxK-lf1o-Tz6V-SiwX-C7j3-1IEg-V3FYLl
  LV Write Access        read/write
  LV Creation host, time openerp70, 2015-07-15 23:31:57 +1100
  LV Status              available
  # open                 1
  LV Size                200.00 GiB
  Current LE             51200
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           254:4
   
On augmente la taille du logical volume ::

   lvextend -L +10G /dev/mapper/openerp70--vg-root  
   Size of logical volume openerp70-vg/root changed from 8.38 GiB (2145 extents) to 18.38 GiB (4705 extents).
   Logical volume root successfully resized

   resize2fs /dev/mapper/openerp70--vg-root
   
J'ai ajouté 10G 



      