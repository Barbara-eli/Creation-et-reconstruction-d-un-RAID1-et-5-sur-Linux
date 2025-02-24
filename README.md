# Creation-et-reconstruction-d-un-RAID1-et-5-sur-Linux

## Création et Reconstitution d'un RAID 1
### Étape 1: Identification des Disques
Identifiez vos disques. La commande lsblk est un bon point de départ.

**lsblk**

Exemple de sortie:

>NAME MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
>
>sda 8:0 0 20G 0 disk
>
>├─sda1 8:1 0 512M 0 part /boot/efi
>
>├─sda2 8:2 0 1G 0 part /boot
>
>└─sda3 8:3 0 18.5G 0 part /
>
>sdb 8:16 0 10G 0 disk
>
>sdc 8:32 0 10G 0 disk
>
>sdd 8:48 0 10G 0 disk
>

Dans cet exemple, sdb et sdc (puis sdd pour le RAID 5) seront utilisés.

### Étape 2: Création du RAID 1
**sudo mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc**

* sudo mdadm --create /dev/md0: Crée un nouveau RAID nommé /dev/md0.

* --verbose: Affiche des informations détaillées pendant la création.

* --level=1: Spécifie le niveau RAID 1 (mirroring).

* --raid-devices=2: Indique que le RAID comportera 2 disques.

* /dev/sdb /dev/sdc: Les périphériques qui seront utilisés dans le RAID.

### Étape 3: Surveiller la Création du RAID
La création du RAID prendra un certain temps. On peut surveiller sa progression avec:

**sudo mdadm --detail /dev/md0**

ou

**cat /proc/mdstat**

On verra un pourcentage qui indique l'avancement de la synchronisation.

### Étape 4: Création d'un Système de Fichiers et Montage du RAID
Une fois la synchronisation terminée, créez un système de fichiers sur le RAID et montez-le.

**sudo mkfs.ext4 /dev/md0**
**sudo mkdir /mnt/raid1**
**sudo mount /dev/md0 /mnt/raid1**

### Étape 5: Rendre le Montage Permanent
Pour que le RAID soit monté automatiquement au démarrage, ajoutez une entrée dans /etc/fstab.

Récupérer l'UUID du RAID :

**sudo blkid /dev/md0**

La sortie ressemblera à :

> /dev/md0: UUID="votre-uuid" TYPE="ext4"

Éditer /etc/fstab :

**sudo nano /etc/fstab**

Ajoutez la ligne suivante:

**UUID=votre-uuid /mnt/raid1 ext4 defaults 0 2**

Tester le montage :

**sudo mount -a**

Si aucune erreur n'apparaît, le montage automatique fonctionne.

## Étape 6: Simulation d'une Panne
Pour simuler une panne, on retire un des disques du RAID ou on le marquecomme défaillant.

**sudo mdadm --fail /dev/md0 /dev/sdb**

ou

**sudo mdadm --remove /dev/md0 /dev/sdb**

Vérifier l'état du RAID:

**sudo mdadm --detail /dev/md0**

On devrait voir que le RAID est en état dégradé (degraded).

### Étape 7: Reconstruction du RAID
Pour reconstruire le RAID, il faut ajouter un nouveau disque (ou le même disque après l'avoir re-partitionné et formaté). On suppose qu'on réutilise /dev/sdb.

Ajouter le disque au RAID :

**sudo mdadm --add /dev/md0 /dev/sdb**

Surveiller la reconstruction :

**sudo mdadm --detail /dev/md0**

ou

**cat /proc/mdstat**

La reconstruction prendra un certain temps. Une fois terminée, le RAID sera de nouveau en état optimal.

## Création et Reconstitution d'un RAID 5

### Étape 1: Identification des Disques

Identifier les disques: /dev/sdb, /dev/sdc et /dev/sdd.

### Étape 2: Arrêt et Suppression de l'ancien RAID 1 (si existant)

Avant de créer le RAID 5, il faut s'assurer que l'ancien RAID 1 est arrêté et supprimé.

**sudo umount /mnt/raid1**
**sudo mdadm --stop /dev/md0**
**sudo mdadm --remove /dev/md0**

### Étape 3: Création du RAID 5

**sudo mdadm --create --verbose /dev/md0 --level=5 --raid-devices=3 /dev/sdb /dev/sdc /dev/sdd**

* --level=5: Spécifie le niveau RAID 5.
* --raid-devices=3: Indique que le RAID comportera 3 disques.
* /dev/sdb /dev/sdc /dev/sdd: Les périphériques qui seront utilisés dans le RAID.

### Étape 4: Surveiller la Création du RAID

**sudo mdadm --detail /dev/md0**

ou

**cat /proc/mdstat**

### Étape 5: Création d'un Système de Fichiers et Montage du RAID

Une fois la synchronisation terminée, créer un système de fichiers sur le RAID et montez-le.

**sudo mkfs.ext4 /dev/md0**
**sudo mkdir /mnt/raid5**
**sudo mount /dev/md0 /mnt/raid5**

##à Étape 6: Rendre le Montage Permanent

Modifier /etc/fstab pour monter automatiquement le RAID 5 au démarrage.

Récupérer l'UUID de /dev/md0 avec sudo blkid /dev/md0 et ajouter une ligne similaire à la suivante à /etc/fstab (en remplaçant l'UUID et le point de montage) :

**UUID=votre-uuid /mnt/raid5 ext4 defaults 0 2**

Tester avec **sudo mount -a**.

### Étape 7: Simulation d'une Panne (RAID 5)

**sudo mdadm --fail /dev/md0 /dev/sdb**
**sudo mdadm --remove /dev/md0 /dev/sdb**

Vérifier l'état du RAID:

**sudo mdadm --detail /dev/md0**

Le RAID sera en mode dégradé.

### Étape 8: Reconstruction du RAID 5

Remplacer le disque défaillant (/dev/sdb dans cet exemple) en ajoutant un nouveau disque (ou le même disque reformaté) au RAID.

**sudo mdadm --add /dev/md0 /dev/sdb**

Surveiller la reconstruction:

**sudo mdadm --detail /dev/md0**

ou

**cat /proc/mdstat**

La reconstruction du RAID 5 prendra plus de temps que celle du RAID 1.

### Historique des Commandes
>lsblk
>
>sudo mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
>
>sudo mdadm --detail /dev/md0
>
cat /proc/mdstat
sudo mkfs.ext4 /dev/md0
sudo mkdir /mnt/raid1
sudo mount /dev/md0 /mnt/raid1
sudo blkid /dev/md0
sudo nano /etc/fstab
sudo mount -a
sudo mdadm --fail /dev/md0 /dev/sdb
sudo mdadm --remove /dev/md0 /dev/sdb
sudo mdadm --detail /dev/md0
sudo mdadm --add /dev/md0 /dev/sdb
sudo mdadm --detail /dev/md0
cat /proc/mdstat
sudo umount /mnt/raid1
sudo mdadm --stop /dev/md0
sudo mdadm --remove /dev/md0
sudo mdadm --create --verbose /dev/md0 --level=5 --raid-devices=3 /dev/sdb /dev/sdc /dev/sdd
sudo mdadm --detail /dev/md0
cat /proc/mdstat
sudo mkfs.ext4 /dev/md0
sudo mkdir /mnt/raid5
sudo mount /dev/md0 /mnt/raid5
sudo blkid /dev/md0
sudo nano /etc/fstab
sudo mount -a
sudo mdadm --fail /dev/md0 /dev/sdb
sudo mdadm --remove /dev/md0 /dev/sdb
sudo mdadm --detail /dev/md0
sudo mdadm --add /dev/md0 /dev/sdb
sudo mdadm --detail /dev/md0
cat /proc/mdstat
