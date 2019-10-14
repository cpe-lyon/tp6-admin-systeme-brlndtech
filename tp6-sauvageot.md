# Compte rendu TP6  <img src="https://image.flaticon.com/icons/svg/518/518713.svg" height="50" alt="Zozor" /> | Geoffrey Sauvageot-Berland 

## Exercice 1. Disques et partitions

### 2. Vérifiez que ce nouveau disque dur est bien détecté par le système

``` 
lsblk
``` 
``` 
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0    7:0    0 54,7M  1 loop /snap/lxd/12181
loop1    7:1    0 54,6M  1 loop /snap/lxd/11985
loop2    7:2    0 89,1M  1 loop /snap/core/7917
loop3    7:3    0   89M  1 loop /snap/core/7713
sda      8:0    0   20G  0 disk
├─sda1   8:1    0    1M  0 part
└─sda2   8:2    0   20G  0 part /
sdb      8:16   0    5G  0 disk
sr0     11:0    1 1024M  0 rom

``` 

### 3. Partitionnez ce disque en utilisant fdisk : créez une première partition de 2 Go de type Linux (n°83), et une seconde partition de 3 Go en NTFS (n°7)
``` 
fdisk /dev/sdb
p
d
n
p 
1
2048
+2G
q
write

/dev/sdb1          2048 4196351 4194304   2G 83 Linux
/dev/sdb2       4196352 8390655 4194304   2G 83 Linux


``` 

### 4. A ce stade, les partitions ont été créées, mais elles n’ont pas été formatées avec leur système de fichiers. A l’aide de la commande mkfs, formatez vos deux partitions
``` 
mkfs.ext4 /dev/sdb1
mkfs.ntfs /dev/sdb2 
``` 

### 5. Pourquoi la commande df -T, qui affiche le type de système de fichier des partitions, ne fonctionne-telle pas sur notre disque ?

Le disque n'est pas encore monté.

### 6. Faites en sorte que les deux partitions créées soient montées automatiquement au démarrage de la machine, respectivement dans les points de montage /data et /win 

```
nano /etc/fstab 
/dev/sdb1 /media/hddtpl auto defaults umask=0 0 0
/dev/sdb2 /media/hddtp2 auto defaults umask=0 0 0
```

### 7. Utilisez la commande mount puis redémarrez votre VM pour valider la configuration

### 8. Montez votre clé USB dans la VM
```
mkdir /media/usb_1
mount -t vfat /dev/sdb1 /media/USB -o 
```

## Exercice 2 

### 1.On va réutiliser le disque de 5 Gio de l’exercice précédent. Commencez par démonter les systèmes de fichiers montés dans /data et /win s’ils sont encore montés, et supprimez les lignes correspondantes du fichier /etc/fstab

```
umount /dev/sdb1
umount /dev/sdb2

``` 

### 2. Supprimez les deux partitions du disque, et créez une patition unique de type LVM

```
fdisk /dev/sdb
d // delete 
1 // numéro de la partition 
d // delete 
2 // numéro de la partition 
```

### 3. A l’aide de la commande pvcreate, créez un volume physique LVM. Validez qu’il est bien créé, en utilisant la commande pvdisplay
```
pvcreate /dev/sdb
pvdisplay 

  --- NEW Physical volume ---
  PV Name               /dev/sdb
  VG Name
  PV Size               5,00 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               aG24dF-mwFN-ZgjR-nAJF-RTeK-JklI-iKgdhR
```
### 4. A l’aide de la commande vgcreate, créez un groupe de volumes, qui pour l’instant ne contiendra que le volume physique créé à l’étape précédente. Vérifiez à l’aide de la commande vgdisplay.
```
vgcreate vg_name_01 /dev/sdb
Volume group "vg_name_01" successfully created
vgdisplay
```
```
  VG Name               vg_name_01
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <5,00 GiB
  PE Size               4,00 MiB
  Total PE              1279
  Alloc PE / Size       0 / 0
  Free  PE / Size       1279 / <5,00 GiB
  VG UUID               5h505n-uTM1-34no-XaZG-0rWq-6Wp2-xWb4em

  ```

### 5. Créez un volume logique appelé lvData occupant l’intégralité de l’espace disque disponible.

```
lvcreate -l 100%FREE  vg_name_01 -n 1vData

```

### 6.  Dans ce volume logique, créez une partition que vous formaterez en ext4, puis procédez comme dans l’exercice 1 pour qu’elle soit montée automatiquement, au démarrage de la machine, dans /data.

```
mkfs -text4 /dev/vg_name_01/lvData

```

### 7. Eteignez la VM pour ajouter un second disque (peu importe la taille pour cet exercice). Redémarrez la VM, vérifiez que le disque est bien présent. Puis, répétez les questions 2 et 3 sur ce nouveau disque

```
vgextend vg_name_01/dev/sdc
```

### 8. Utilisez la commande vgextend <nom_vg> <nom_pv> pour ajouter le nouveau disque au groupe de volumes
```
lvextend -l 100%FREE /dev/vg_name_01/lvData
```
## Exercice 3 Execution de commandes en différé : at et cron

### 1. Programmez une tâche qui affiche un rappel pour la réunion qui aura lieu dans 3 minutes. Vérifiez entre temps que la tâche est bien programmée.
```
echo "Rappel de la réunion" | at 17:35
```

### 2. Est-ce que le message s’est affiché ? Si la réponse est non, essayez de trouver la cause du problème (par exemple en vous aidant des logs, du manuel...)

Non, car on a pas de serveur smtp / imap, pour gérer les envois / réceptions des messages. 

### 3. Pour tester le fonctionnement de cron, commencez par programmer l’exécution d’une tâche simple, l’affichage de “Il faut réviser pour l’examen !”, toutes les 3 minutes.

```
*/3 * * * * echo "Il faut réviser l'examen"
```
### 4. Programmez l’exécution d’une commande tous les jours, toute l’année, tous les quarts d’heure

```
0 0 * * * echo "coudcoud"
0 0 1 1 * echo "coudcouuuuuuuud"
*/15 * * * echo "bezet"
```
