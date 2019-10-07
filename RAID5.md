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






