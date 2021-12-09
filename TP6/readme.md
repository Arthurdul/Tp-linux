# TP6 Compte-rendu:

---

# Partie 1 : Préparation de la machine `backup.tp6.linux`


## I. Ajout de disque

**Ajouter un disque dur de 5Go à la VM `backup.tp6.linux`**

```bash
[arthur@backup ~]$ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda           8:0    0    8G  0 disk
├─sda1        8:1    0    1G  0 part /boot
└─sda2        8:2    0    7G  0 part
  ├─rl-root 253:0    0  6.2G  0 lvm  /
  └─rl-swap 253:1    0  820M  0 lvm  [SWAP]
sdb           8:16   0    5G  0 disk
sr0          11:0    1 1024M  0 rom
```
## II. Partitioning


**Partitionner le disque à l'aide de LVM**
```bash
[arthur@backup ~]$ sudo pvcreate /dev/sdb
[sudo] password for arthur:
  Physical volume "/dev/sdb" successfully created.
[arthur@backup ~]$ sudo pvs
  PV         VG Fmt  Attr PSize  PFree
  /dev/sda2  rl lvm2 a--  <7.00g    0
  /dev/sdb      lvm2 ---   5.00g 5.00g
  
[arthur@backup ~]$ sudo vgcreate backup /dev/sdb
Volume group "backup" successfully created
[arthur@backup ~]$ sudo vgs
  VG     #PV #LV #SN Attr   VSize  VFree
  backup   1   0   0 wz--n- <5.00g <5.00g
  rl       1   2   0 wz--n- <7.00g     0
  
[arthur@backup ~]$ sudo lvcreate -l 100%FREE backup -n backup
  Logical volume "backup" created.
[arthur@backup ~]$ sudo lvs
  LV     VG     Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  backup backup -wi-a-----  <5.00g
  root   rl     -wi-ao----  <6.20g
  swap   rl     -wi-ao---- 820.00m
```
**Formater la partition**
```bash
[arthur@backup ~]$ sudo mkfs -t ext4 /dev/backup/backup
[sudo] password for arthur:
mke2fs 1.45.6 (20-Mar-2020)
Creating filesystem with 1309696 4k blocks and 327680 inodes
Filesystem UUID: ee6c10c5-3fb8-4ca8-b608-c1406defbe26
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736
Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done
```

**Monter la partition**
```bash
[arthur@backup ~]$ sudo mkdir /mnt/backup
[arthur@backup ~]$ sudo mount /dev/backup/backup /mnt/backup/
[arthur@backup ~]$ df -h
Filesystem                 Size  Used Avail Use% Mounted on
[...]
/dev/mapper/rl-root        6.2G  1.8G  4.5G  29% /
/dev/sda1                 1014M  182M  833M  18% /boot
tmpfs                       81M     0   81M   0% /run/user/1000
/dev/mapper/backup-backup  4.9G   20M  4.6G   1% /mnt/backup
[arthur@backup ~]$ sudo cat /etc/fstab
[...]
/dev/mapper/rl-root     /                       xfs     defaults        0 0
UUID=cb9bc277-6931-4a4f-b84f-ec7f530acd42 /boot                   xfs     defaults        0 0
/dev/mapper/rl-swap     none                    swap    defaults        0 0
/dev/backup/backup /mnt/backup ext4 defaults 0 0
[arthur@backup ~]$ sudo umount /mnt/backup
[arthur@backup ~]$ sudo mount -av
/                        : ignored
/boot                    : already mounted
none                     : ignored
mount: /mnt/backup does not contain SELinux labels.
       You just mounted an file system that supports labels which does not
       contain labels, onto an SELinux box. It is likely that confined
       applications will generate AVC messages and not be allowed access to
       this file system.  For more details see restorecon(8) and mount(8).
/mnt/backup              : successfully mounted
```
---

# Partie 2 : Setup du serveur NFS sur `backup.tp6.linux`

**Préparer les dossiers à partager**
```bash
[arthur@backup backup]$ ls
backup  db.tp6.linux  web.tp6.linux
```

**Install du serveur NFS**
```bash
[arthur@backup backup]$ sudo dnf install nfs-utils
[...]
Complete !
```
**Conf du serveur NFS**



```bash
[arthur@backup /]$ sudo cat /etc/idmapd.conf  | grep Domain
Domain = tp6.linux
```



```bash
[arthur@backup /]$ cat /etc/exports
/backup/web.tp6.linux 10.5.1.11(rw,no_root_squash)
/backup/db.tp6.linux 10.5.1.12(rw,no_root_squash)
```


**Démarrez le service**
```bash
[arthur@backup /]$ sudo systemctl start nfs-server
[arthur@backup /]$ sudo systemctl status nfs-server
● nfs-server.service - NFS server and services
   Loaded: loaded (/usr/lib/systemd/system/nfs-server.service; disabled; vendor preset: disabled)
   Active: active (exited) since Fri 2021-12-03 09:02:30 CET; 8s ago
  Process: 5274 ExecStart=/bin/sh -c if systemctl -q is-active gssproxy; then systemctl reload gssproxy >
  Process: 5262 ExecStart=/usr/sbin/rpc.nfsd (code=exited, status=0/SUCCESS)
  Process: 5261 ExecStartPre=/usr/sbin/exportfs -r (code=exited, status=1/FAILURE)
 Main PID: 5274 (code=exited, status=0/SUCCESS)
Dec 03 09:02:29 backup.tp6.linux systemd[1]: Starting NFS server and services...
Dec 03 09:02:29 backup.tp6.linux exportfs[5261]: exportfs: Failed to stat /backup/db.tp6.linux: No such >
Dec 03 09:02:29 backup.tp6.linux exportfs[5261]: exportfs: Failed to stat /backup/web.tp6.linux: No such>
Dec 03 09:02:30 backup.tp6.linux systemd[1]: Started NFS server and services.
[arthur@backup /]$ sudo systemctl enable nfs-server
Created symlink /etc/systemd/system/multi-user.target.wants/nfs-server.service → /usr/lib/systemd/system/nfs-server.service.
```
**Firewall**
```bash
[arthur@backup /]$ sudo ss -lntp | grep 2049
LISTEN 0      64           0.0.0.0:2049       0.0.0.0:*
LISTEN 0      64              [::]:2049          [::]:*
---
```
# Partie 3 : Setup des clients NFS : `web.tp6.linux` et `db.tp6.linux`
---
**Install'**
```bash
[arthur@web ~]$ sudo dnf install nfs-utils
Last metadata expiration check: 0:02:29 ago on Fri 03 Dec 2021 09:12:57 AM CET.
[...]
```
**Conf'**
```bash
[arthur@web ~]$ sudo mkdir /srv/backup
[arthur@web ~]$ [arthur@web ~]$ cat /etc/idmapd.conf | grep Domain
Domain = tp6.linux
```
---
**Montage !**
```bash
[arthur@web srv]$ sudo mount -t nfs 10.5.1.13:/backup/web.tp6.linux /srv/backup
[arthur@web srv]$ df -h | grep backup
10.5.1.13:/backup/web.tp6.linux  6.2G  1.8G  4.4G  29% /srv/backup
[arthur@web srv]$ ls -l
total 0
drwxr-xr-x. 2 arthur arthur 6 Dec  3 10:11 backup
[arthur@web srv]$ sudo umount /srv/backup
[arthur@web srv]$ sudo mount -av
/                        : ignored
/boot                    : already mounted
none                     : ignored
mount.nfs: timeout set for Sat Nov 27 17:50:31 2021
mount.nfs: trying text-based options 'vers=4.2,addr=10.5.1.13,clientaddr=10.5.1.11'
/srv/backup              : successfully mounted
[arthur@web ~]$ cat /etc/fstab | grep backup
10.5.1.13:/backup/web.tp6.linux  /srv/backup nfs defaults 0 0
```
---
**Répéter les opérations sur `db.tp6.linux`**
```bash
[arthur@db /]$ sudo mount -t nfs 10.5.1.13:/backup/db.tp6.linux /srv/backup
[arthur@db /]$ df -h | grep backup
10.5.1.13:/backup/db.tp6.linux  6.2G  1.8G  4.4G  29% /srv/backup
[arthur@db /]$ sudo umount /srv/backup
[arthur@db /]$ sudo mount -av
/                        : ignored
/boot                    : already mounted
none                     : ignored
mount.nfs: timeout set for Fri Dec  3 10:34:33 2021
mount.nfs: trying text-based options 'vers=4.2,addr=10.5.1.13,clientaddr=10.5.1.12'
/srv/backup              : successfully mounted
[arthur@db /]$ cat /etc/fstab | grep backup
10.5.1.13:/backup/db.tp6.linux  /srv/backup nfs defaults 0 0
```
# Partie 4 : Scripts de sauvegarde
## I. Sauvegarde Web

---
**Ecrire un script qui sauvegarde les données de NextCloud**
```bash
[arthur@web /]$ sudo mkdir /var/log/backup
[arthur@web /]$ sudo nano /var/log/backup/backup.log
[arthur@web /]$ sudo nano /var/log/backup.sh`
```
**Créer un service et son timer**
```bash
[arthur@web system]$ sudo nano backup.service
[arthur@web /]$ sudo nano backup.timer
[arthur@web /]$ sudo systemctl daemon-reload
[arthur@web /]$ sudo systemctl start backup.timer
[arthur@web /]$ sudo systemctl enable backup.timer
Created symlink /etc/systemd/system/timers.target.wants/backup.timer → /etc/systemd/system/backup.timer.
[arthur@web /]$ sudo systemctl list-timers
NEXT                         LEFT          LAST                         PASSED      UNIT                         ACTIVATES
Tue 2021-12-05 22:00:00 CET  4min 58s left n/a                          n/a         backup.timer                 backup.service
[...]
```
**Vérifier que vous êtes capables de restaurer les données**
```bash
[arthur@web /]$ ls /srv/backup
nextcloud_211207_211249.tar.gz
[arthur@web /]$ sudo tar -xf /srv/backup/nextcloud_211207_211249.tar.gz
[arthur@web /]$ ls nextcloud/html/
3rdparty  AUTHORS  console.php  core      data        index.php  occ           ocs           public.php  resources   status.php  updater
apps      config   COPYING      cron.php  index.html  lib        ocm-provider  ocs-provider  remote.php  robots.txt  themes      version.php
[arthur@web /]$ cat /var/log/backup/backup.log
[21/12/07 22:12:20] Backup /srv/backup/nextcloud_211207_211249.tar.gz created successfully.
```
## II. Sauvegarde base de données

---
**Ecrire un script qui sauvegarde les données de la base de données MariaDB**
```bash
[arthur@db ~]$ sudo mkdir /var/log/backup
[arthur@db ~]$ sudo nano /var/log/backup/backup.log
[arthur@db /]$ sudo nano /srv/backup_db.sh
```
**Créer un service**
```bash
[arthur@db /]$ sudo nano /etc/systemd/system/backup_db.service
[arthur@db /]$ sudo systemctl daemon-reload
```
**Créer un `timer`**
```bash
[arthur@db /]$ sudo nano /etc/systemd/system/backup_db.timer
[arthur@db /]$ sudo systemctl daemon-reload
[arthur@db /]$ sudo systemctl start backup_db.timer
[arthur@db /]$ sudo systemctl enable backup_db.timer
Created symlink /etc/systemd/system/timers.target.wants/backup_db.timer → /etc/systemd/system/backup_db.timer.
[arthur@db /]$ sudo systemctl list-timers
NEXT                         LEFT          LAST                         PASSED    UNIT                         ACTIVATES
Tue 2021-12-07 23:00:00 CET  40min left    n/a                          n/a       backup_db.timer              backup_db.service
```
