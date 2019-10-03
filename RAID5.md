# Ejercicio: RAID 5

>A continuación queremos crear un raid 5 en una máquina de 2 GB, para ello vamos a utilizar discos virtuales de 1 GB. Modifica el fichero Vagrantfile del ejercicio anterior para crear una nueva máquina.

### Tarea 1: Crea una raid llamado md5 con los discos que hemos conectado a la máquina.

Discos:

~~~
vagrant@Almacenamiento:~$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0 19.8G  0 disk 
├─sda1   8:1    0 18.8G  0 part /
├─sda2   8:2    0    1K  0 part 
└─sda5   8:5    0 1021M  0 part [SWAP]
sdb      8:16   0    1G  0 disk 
sdc      8:32   0    1G  0 disk 
~~~

Creación del raid:

~~~
vagrant@Almacenamiento:~$ sudo mdadm --create /dev/md1 --level=5 --raid-devices=2 /dev/sdb /dev/sdc --spare-devices=1 /dev/sdd 
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md1 started.
~~~


### Tarea 2: Comprueba las características del RAID. Comprueba el estado del RAID. ¿Qué capacidad tiene el RAID que hemos creado?

~~~
vagrant@Almacenamiento:~$ cat /proc/mdstat 
Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10] 
md1 : active raid5 sdc[3] sdd[2](S) sdb[0]
      1046528 blocks super 1.2 level 5, 512k chunk, algorithm 2 [2/2] [UU]
      
unused devices: <none>
vagrant@Almacenamiento:~$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda      8:0    0 19.8G  0 disk  
├─sda1   8:1    0 18.8G  0 part  /
├─sda2   8:2    0    1K  0 part  
└─sda5   8:5    0 1021M  0 part  [SWAP]
sdb      8:16   0    1G  0 disk  
└─md1    9:1    0 1022M  0 raid5 
sdc      8:32   0    1G  0 disk  
└─md1    9:1    0 1022M  0 raid5 
sdd      8:48   0    1G  0 disk  
└─md1    9:1    0 1022M  0 raid5 
vagrant@Almacenamiento:~$ sudo mdadm --detail /dev/md1
/dev/md1:
           Version : 1.2
     Creation Time : Thu Oct  3 15:34:59 2019
        Raid Level : raid5
        Array Size : 1046528 (1022.00 MiB 1071.64 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 2
     Total Devices : 3
       Persistence : Superblock is persistent

       Update Time : Thu Oct  3 15:35:11 2019
             State : clean 
    Active Devices : 2
   Working Devices : 3
    Failed Devices : 0
     Spare Devices : 1

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : Almacenamiento:1  (local to host Almacenamiento)
              UUID : 3257002e:c399d4cf:acd05aab:66463050
            Events : 18

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       3       8       32        1      active sync   /dev/sdc

       2       8       48        -      spare   /dev/sdd
~~~

### Tarea 3: Crea un volumen lógico de 500Mb en el raid 5.

~~~
vagrant@Almacenamiento:~$ sudo fdisk /dev/md1 

Welcome to fdisk (util-linux 2.33.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0x9b12d38d.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 
First sector (2048-2093055, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-2093055, default 2093055): +500M

Created a new partition 1 of type 'Linux' and of size 500 MiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

vagrant@Almacenamiento:~$ lsblk
NAME      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda         8:0    0 19.8G  0 disk  
├─sda1      8:1    0 18.8G  0 part  /
├─sda2      8:2    0    1K  0 part  
└─sda5      8:5    0 1021M  0 part  [SWAP]
sdb         8:16   0    1G  0 disk  
└─md1       9:1    0 1022M  0 raid5 
  └─md1p1 259:1    0  500M  0 part  
sdc         8:32   0    1G  0 disk  
└─md1       9:1    0 1022M  0 raid5 
  └─md1p1 259:1    0  500M  0 part  
sdd         8:48   0    1G  0 disk  
└─md1       9:1    0 1022M  0 raid5 
  └─md1p1 259:1    0  500M  0 part  
~~~


### Tarea 4: Formatea ese volumen con un sistema de archivo xfs.
    
~~~
vagrant@Almacenamiento:~$ sudo mkfs.xfs /dev/md1p1 
meta-data=/dev/md1p1             isize=512    agcount=8, agsize=16000 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=0
data     =                       bsize=4096   blocks=128000, imaxpct=25
         =                       sunit=128    swidth=128 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=896, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
vagrant@Almacenamiento:~$ lsblk -f
NAME FSTYPE LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINT
sda                                                                   
├─sda1
│    ext4         b9ffc3d1-86b2-4a2c-a8be-f2b2f4aa4cb5   16.1G     7% /
├─sda2
│                                                                     
└─sda5
     swap         f8f6d279-1b63-4310-a668-cb468c9091d8                [SWAP]
sdb  linux_ Almacenamiento:1
│                 3257002e-c399-d4cf-acd0-5aab66463050                
└─md1
                                                                      
  └─md1p1
     xfs          b47b8f1b-7399-4f8d-a191-083945f9976c                
sdc  linux_ Almacenamiento:1
│                 3257002e-c399-d4cf-acd0-5aab66463050                
└─md1
                                                                      
  └─md1p1
     xfs          b47b8f1b-7399-4f8d-a191-083945f9976c                
sdd  linux_ Almacenamiento:1
│                 3257002e-c399-d4cf-acd0-5aab66463050                
└─md1
                                                                      
  └─md1p1
     xfs          b47b8f1b-7399-4f8d-a191-083945f9976c  
~~~

Si no se encuentra el comando mkfs.xfs se debe descargar el paquete **xfsprogs**


### Tarea 5: Monta el volumen en el directorio /mnt/raid5 y crea un fichero. ¿Qué tendríamos que hacer para que este punto de montaje sea permanente?

~~~
vagrant@Almacenamiento:~$ sudo mkdir /mnt/raid5
vagrant@Almacenamiento:~$ sudo mount /dev/md1p1 /mnt/raid5/
vagrant@Almacenamiento:~$ lsblk
NAME      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda         8:0    0 19.8G  0 disk  
├─sda1      8:1    0 18.8G  0 part  /
├─sda2      8:2    0    1K  0 part  
└─sda5      8:5    0 1021M  0 part  [SWAP]
sdb         8:16   0    1G  0 disk  
└─md1       9:1    0 1022M  0 raid5 
  └─md1p1 259:1    0  500M  0 part  /mnt/raid5
sdc         8:32   0    1G  0 disk  
└─md1       9:1    0 1022M  0 raid5 
  └─md1p1 259:1    0  500M  0 part  /mnt/raid5
sdd         8:48   0    1G  0 disk  
└─md1       9:1    0 1022M  0 raid5 
  └─md1p1 259:1    0  500M  0 part  /mnt/raid5
vagrant@Almacenamiento:~$ sudo touch /mnt/raid5/prueba
vagrant@Almacenamiento:~$ ls /mnt/raid5/prueba 
/mnt/raid5/prueba
~~~

Para montar el raid de forma permanente hay que editar el fichero fstab.


### Tarea 6: Marca un disco como estropeado. Muestra el estado del raid para comprobar que un disco falla. ¿Podemos acceder al fichero?

~~~
vagrant@Almacenamiento:~$ sudo mdadm /dev/md1 -f /dev/sdb1
mdadm: stat failed for /dev/sdb1: No such file or directory
vagrant@Almacenamiento:~$ sudo mdadm --detail /dev/md1
/dev/md1:
           Version : 1.2
     Creation Time : Thu Oct  3 15:34:59 2019
        Raid Level : raid5
        Array Size : 1046528 (1022.00 MiB 1071.64 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 2
     Total Devices : 3
       Persistence : Superblock is persistent

       Update Time : Thu Oct  3 15:55:37 2019
             State : clean 
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 1
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : Almacenamiento:1  (local to host Almacenamiento)
              UUID : 3257002e:c399d4cf:acd05aab:66463050
            Events : 37

    Number   Major   Minor   RaidDevice State
       2       8       48        0      active sync   /dev/sdd
       3       8       32        1      active sync   /dev/sdc

       0       8       16        -      faulty   /dev/sdb
vagrant@Almacenamiento:~$ ls /mnt/raid5/
prueba
~~~


### Tarea 7: Una vez marcado como estropeado, lo tenemos que retirar del raid.

~~~
vagrant@Almacenamiento:~$ sudo mdadm --manage /dev/md1 --remove /dev/sdb
mdadm: hot removed /dev/sdb from /dev/md1
vagrant@Almacenamiento:~$ sudo mdadm --detail /dev/md1
/dev/md1:
           Version : 1.2
     Creation Time : Thu Oct  3 15:34:59 2019
        Raid Level : raid5
        Array Size : 1046528 (1022.00 MiB 1071.64 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

       Update Time : Thu Oct  3 16:21:40 2019
             State : clean 
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : Almacenamiento:1  (local to host Almacenamiento)
              UUID : 3257002e:c399d4cf:acd05aab:66463050
            Events : 38

    Number   Major   Minor   RaidDevice State
       2       8       48        0      active sync   /dev/sdd
       3       8       32        1      active sync   /dev/sdc
~~~
   
 
### Tarea 8: Imaginemos que lo cambiamos por un nuevo disco nuevo (el dispositivo de bloque se llama igual), añádelo al array y comprueba como se sincroniza con el anterior.

~~~
vagrant@Almacenamiento:~$ sudo mdadm /dev/md1 --add /dev/sdb
mdadm: added /dev/sdb
vagrant@Almacenamiento:~$ sudo mdadm --detail /dev/md1
/dev/md1:
           Version : 1.2
     Creation Time : Thu Oct  3 15:34:59 2019
        Raid Level : raid5
        Array Size : 2093056 (2044.00 MiB 2143.29 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 3
     Total Devices : 3
       Persistence : Superblock is persistent

       Update Time : Thu Oct  3 16:38:24 2019
             State : clean, degraded, recovering 
    Active Devices : 2
   Working Devices : 3
    Failed Devices : 0
     Spare Devices : 1

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

    Rebuild Status : 27% complete

              Name : Almacenamiento:1  (local to host Almacenamiento)
              UUID : 3257002e:c399d4cf:acd05aab:66463050
            Events : 64

    Number   Major   Minor   RaidDevice State
       2       8       48        0      active sync   /dev/sdd
       3       8       32        1      active sync   /dev/sdc
       4       8       16        2      spare rebuilding   /dev/sdb
~~~


### Tarea 9: Añade otro disco como reserva. Vuelve a simular el fallo de un disco y comprueba como automática se realiza la sincronización con el disco de reserva.

Fallo del disco:

~~~
vagrant@Almacenamiento:~$ sudo mdadm /dev/md1 -f /dev/sdd
mdadm: set /dev/sdd faulty in /dev/md1
vagrant@Almacenamiento:~$ sudo mdadm --detail /dev/md1
/dev/md1:
           Version : 1.2
     Creation Time : Thu Oct  3 15:34:59 2019
        Raid Level : raid5
        Array Size : 2093056 (2044.00 MiB 2143.29 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 3
     Total Devices : 3
       Persistence : Superblock is persistent

       Update Time : Thu Oct  3 16:39:42 2019
             State : clean, degraded 
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 1
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : Almacenamiento:1  (local to host Almacenamiento)
              UUID : 3257002e:c399d4cf:acd05aab:66463050
            Events : 79

    Number   Major   Minor   RaidDevice State
       -       0        0        0      removed
       3       8       32        1      active sync   /dev/sdc
       4       8       16        2      active sync   /dev/sdb

       2       8       48        -      faulty   /dev/sdd
~~~

Eliminamos el disco y se sincroniza el de reserva:

~~~
vagrant@Almacenamiento:~$ sudo mdadm --detail /dev/md1
/dev/md1:
           Version : 1.2
     Creation Time : Thu Oct  3 15:34:59 2019
        Raid Level : raid5
        Array Size : 2093056 (2044.00 MiB 2143.29 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 3
     Total Devices : 2
       Persistence : Superblock is persistent

       Update Time : Thu Oct  3 16:40:38 2019
             State : clean, degraded 
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : Almacenamiento:1  (local to host Almacenamiento)
              UUID : 3257002e:c399d4cf:acd05aab:66463050
            Events : 80

    Number   Major   Minor   RaidDevice State
       -       0        0        0      removed
       3       8       32        1      active sync   /dev/sdc
       4       8       16        2      active sync   /dev/sdb
~~~

    
### Tarea 10: Redimensiona el volumen y el sistema de archivo de 500Mb al tamaño del raid.

~~~
vagrant@Almacenamiento:~$ sudo fdisk /dev/md1

Welcome to fdisk (util-linux 2.33.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): d
Selected partition 1
Partition 1 has been deleted.

Command (m for help): w
The partition table has been altered.
Failed to remove partition 1 from system: Device or resource busy

The kernel still uses the old partitions. The new table will be used at the next reboot. 
Syncing disks.

vagrant@Almacenamiento:~$ sudo fdisk /dev/md1

Welcome to fdisk (util-linux 2.33.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): 

Using default response p.
Partition number (1-4, default 1): 
First sector (2048-4186111, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-4186111, default 4186111): 

Created a new partition 1 of type 'Linux' and of size 2 GiB.
Partition #1 contains a xfs signature.

Do you want to remove the signature? [Y]es/[N]o: n

Command (m for help): w

The partition table has been altered.
Failed to add partition 1 to system: Device or resource busy

The kernel still uses the old partitions. The new table will be used at the next reboot. 
Syncing disks.

~~~




