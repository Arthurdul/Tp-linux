# Compte rendu TP5
---
## Setup DB :

➜ **1. Install MariaDB :**
Install
```
[arthur@db ~]$ sudo dnf install mariadb-server
[...]
Complete!
[arthur@db ~]$ sudo systemctl start mariadb
[arthur@db ~]$ systemctl status mariadb
● mariadb.service - MariaDB 10.3 database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; disabled; vendor preset: disabled)
   Active: active (running) since Thu 2021-11-29 13:59:37 CET; 7s ago
     Docs: man:mysqld(8)
           https://mariadb.com/kb/en/library/systemd/
  Process: 4956 ExecStartPost=/usr/libexec/mysql-check-upgrade (code=exited, status=0/SUCCESS)
  Process: 4886 ExecStartPre=/usr/libexec/mysql-prepare-db-dir mariadb.service (code=exited, status=0/SUCCESS)
  Process: 4862 ExecStartPre=/usr/libexec/mysql-check-socket (code=exited, status=0/SUCCESS)
 Main PID: 4924 (mysqld)
   Status: "Taking your SQL requests now..."
    Tasks: 30 (limit: 11407)
   Memory: 67.4M
   CGroup: /system.slice/mariadb.service
           └─4924 /usr/libexec/mysqld --basedir=/usr
Nov 29 13:59:37 db.tp5.linux systemd[1]: Starting MariaDB 10.3 database server...
Nov 29 13:59:37 db.tp5.linux mysql-prepare-db-dir[4886]: Database MariaDB is probably initialized in /var/lib/mysql already, nothing is done.
Nov 29 13:59:37 db.tp5.linux mysql-prepare-db-dir[4886]: If this is not the case, make sure the /var/lib/mysql is empty before running mysql-prepare-db-dir.
Nov 29 13:59:37 db.tp5.linux mysqld[4924]: 2021-11-29 13:59:37 0 [Note] /usr/libexec/mysqld (mysqld 10.3.28-MariaDB) starting as process 4924 ...
Nov 29 13:59:37 db.tp5.linux systemd[1]: Started MariaDB 10.3 database server.
```
Le service
```bash 
[arthur@db ~]$ sudo systemctl enable mariadb
Created symlink /etc/systemd/system/mysql.service → /usr/lib/systemd/system/mariadb.service.
Created symlink /etc/systemd/system/mysqld.service → /usr/lib/systemd/system/mariadb.service.
Created symlink /etc/systemd/system/multi-user.target.wants/mariadb.service → /usr/lib/systemd/system/mariadb.service.
[arthur@db ~]$ sudo ss -ltnp | grep mysql
LISTEN 0      80                 *:3306            *:*    users:(("mysqld",pid=4924,fd=21))
[arthur@db ~]$ ps -aux  | grep mysql
mysql       4924  0.0  4.7 1296832 88752 ?       Ssl  10:59   0:00 /usr/libexec/mysqld --basedir=/usr
```
Firewall
```bash 
[arthur@db ~]$ sudo firewall-cmd --add-port=3306/tcp --permanent
success
[arthur@db ~]$ sudo firewall-cmd --reload
success
```
➜ **2. Conf MariaDB :**
```bash 
[arthur@db ~]$ mysql_secure_installation
[...]
Enter current password for root (enter for none): 
OK, successfully used password, moving on...
```
1. Faire entrer car aucun mot de passe défini pour le root.
```
Set root password? [Y/n] Y
New password: 
Re-enter new password: 
Password updated successfully!
Reloading privilege tables..
 ... Success!
 ```
2. Mettre un mot de passe à root.
 ```
Remove anonymous users? [Y/n] Y
 ... Success!
```
3. Enlever l'utilisateur anonymous, afin que personne autre que nous ne puissent y accéder.
```
Disallow root login remotely? [Y/n] Y
 ... Success!
```
4. Désactiver la base de donnée test pour que personne n'essaye de se connecter à root à distance.
```
Remove test database and access to it? [Y/n] Y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!
```
5. On l'enlève, elle ne nous sert à rien.
```
Reload privilege tables now? [Y/n] Y
 ... Success!
Cleaning up...
All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.
Thanks for using MariaDB!
```
6. Pour finir, accepter que nos changements sur les tables prennent directement effet.

➜ **Préparation de la base en vue de l'utilisation par NextCloud :**
```
[arthur@db ~]$ sudo mysql -u root -p 
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 20
Server version: 10.3.28-MariaDB MariaDB Server
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
MariaDB [(none)]> CREATE USER 'nextcloud'@'10.5.1.11' IDENTIFIED BY '***';
Query OK, 0 rows affected (0.001 sec)
MariaDB [(none)]> CREATE DATABASE IF NOT EXISTS nextcloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
Query OK, 1 row affected (0.001 sec)
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'10.5.1.11';
Query OK, 0 rows affected (0.001 sec)
MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.001 sec)
```
➜ **3. Test :**
```bash
[arthur@web ~]$ dnf provides mysql
Rocky Linux 8 - AppStream                                                                                                                                                                                   5.5 MB/s | 8.2 MB     00:01    
Rocky Linux 8 - BaseOS                                                                                                                                                                                      4.6 MB/s | 3.5 MB     00:00    
Rocky Linux 8 - Extras                                                                                                                                                                                       20 kB/s |  10 kB     00:00    
mysql-8.0.26-1.module+el8.4.0+652+6de068a7.x86_64 : MySQL client programs and shared libraries
Repo        : appstream
Matched from:
Provide    : mysql = 8.0.26-1.module+el8.4.0+652+6de068a7
[arthur@web ~]$ sudo dnf install mysql
[...]
Complete!
```
Tester la connexion : 
```bash 
[arthur@web ~]$ mysql -h 10.5.1.12 -P 3306 -u nextcloud -p -D nextcloud
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 5.5.5-10.3.28-MariaDB MariaDB Server
Copyright (c) 2000, 2021, Oracle and/or its affiliates.
Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql> SHOW TABLES;
Empty set (0.00 sec)
```
## II. Setup Web : 
```
[arthur@web ~]$ sudo dnf install httpd
[...]
Complete!
```
```
[arthur@web ~]$ sudo systemctl start httpd
[arthur@web ~]$ sudo systemctl status httpd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
   Active: active (running) since Mon 2021-11-29 14:18:13 CET; 3s ago
     Docs: man:httpd.service(8)
 Main PID: 2486 (httpd)
   Status: "Started, listening on: port 80"
    Tasks: 213 (limit: 11407)
   Memory: 24.8M
   CGroup: /system.slice/httpd.service
           ├─2486 /usr/sbin/httpd -DFOREGROUND
           ├─2487 /usr/sbin/httpd -DFOREGROUND
           ├─2488 /usr/sbin/httpd -DFOREGROUND
           ├─2489 /usr/sbin/httpd -DFOREGROUND
           └─2490 /usr/sbin/httpd -DFOREGROUND
Nov 29 14:18:13 web.tp5.linux systemd[1]: Starting The Apache HTTP Server...
Nov 29 14:18:13 web.tp5.linux systemd[1]: Started The Apache HTTP Server.
Nov 29 14:18:14 web.tp5.linux httpd[2486]: Server configured, listening on: port 80
[arthur@web ~]$ sudo systemctl enable httpd
Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.
[arthur@web ~]$ ps -aux | grep httpd
root        2486  0.0  0.5 280220 11112 ?        Ss   14:18   0:00 /usr/sbin/httpd -DFOREGROUND
apache      2487  0.0  0.4 294104  8280 ?        S    14:18   0:00 /usr/sbin/httpd -DFOREGROUND
apache      2488  0.0  0.7 1483020 14060 ?       Sl   14:18   0:00 /usr/sbin/httpd -DFOREGROUND
apache      2489  0.0  0.6 1351892 12012 ?       Sl   14:18   0:00 /usr/sbin/httpd -DFOREGROUND
apache      2490  0.0  0.6 1351892 12012 ?       Sl   14:18   0:00 /usr/sbin/httpd -DFOREGROUND
[arthur@web ~]$ sudo ss -ltnp | grep httpd
LISTEN 0      128                *:80              *:*    users:(("httpd",pid=2490,fd=4),("httpd",pid=2489,fd=4),("httpd",pid=2488,fd=4),("httpd",pid=2486,fd=4))
```
L'utilisareur est Apache.
Un premier test : 
```bash 
[arthur@web ~]$ sudo firewall-cmd --add-port=80/tcp --permanent
success
[arthur@web ~]$ sudo firewall-cmd --reload
success
❯ curl http://10.5.1.11
<!doctype html>
<html>
[...]
```
(La version web marche aussi)
**B. PHP :**
```bash 
[arthur@web ~]$ sudo dnf install epel-release
[...]
Complete!
[arthur@web ~]$ sudo dnf update
[...]
Complete!
[arthur@web ~]$ sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-8.rpm                                                                 
[...]
Complete!
[arthur@web ~]$ sudo dnf module enable php:remi-7.4
[...]
Complete!
[arthur@web ~]$ sudo dnf install zip unzip libxml2 openssl php74-php php74-php-ctype php74-php-curl php74-php-gd php74-php-iconv php74-php-json php74-php-libxml php74-php-mbstring php74-php-openssl php74-php-posix php74-php-session php74-php-xml php74-php-zip php74-php-zlib php74-php-pdo php74-php-mysqlnd php74-php-intl php74-php-bcmath php74-php-gmp                         
[...]
Complete!
```
➜ **2. Conf Apache :**
```bash 
[arthur@dweb ~]$ cat /etc/httpd/conf/httpd.conf | grep conf.d
# Load config files in the "/etc/httpd/conf.d" directory, if any.
IncludeOptional conf.d/*.conf
```
**Créer un VirtualHost qui accueillera NextCloud :**
```bash
[arthur@web conf.d]$ cat website.conf 
<VirtualHost *:80>
  DocumentRoot /var/www/nextcloud/html/  
  ServerName  web.tp5.linux  
  <Directory /var/www/nextcloud/html/>
    Require all granted
    AllowOverride All
    Options FollowSymLinks MultiViews
    <IfModule mod_dav.c>
      Dav off
    </IfModule>
  </Directory>
</VirtualHost>
[arthur@web conf.d]$ sudo systemctl restart httpd
```
Configurer la racine web : 
```bash 
[arthur@web ~]$ sudo mkdir -p /var/www/nextcloud/html
[arthur@web ~]$ cd /var/www
[arthur@web www]$ sudo chown -R apache:apache nextcloud/
[arthur@web www]$ ls -la
[...]
drwxr-xr-x.  3 apache apache   18 Nov 29 16:34 nextcloud
```
Configurer PHP : 
```bash 
[arthur@web www]$ cat /etc/opt/remi/php74/php.ini | grep Paris
date.timezone = "Europe/Paris"
```
**3. Install NextCloud :**
```bash 
[arthur@web www]$ cd
[arthur@web ~]$ curl -SLO https://download.nextcloud.com/server/releases/nextcloud-21.0.1.zip
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  148M  100  148M    0     0  3408k      0  0:00:44  0:00:44 --:--:-- 3823k
[arthur@web ~]$ ls
nextcloud-21.0.1.zip
```
Ranger la chambre : 
```bash 
[arthur@web ~]$ unzip nextcloud-21.0.1.zip
[arthur@web ~]$ ls
nextcloud  nextcloud-21.0.1.zip
[arthur@web ~]$ sudo mv nextcloud/* /var/www/nextcloud/html/
[sudo] password for arthur: 
[arthur@web html]$ sudo chown -R apache:apache var/www/nextcloud
[arthur@web nextcloud]$ sudo systemctl restart httpd
```
**4. Test :**
```bash 
❯ cat /etc/hosts | grep web
10.5.1.11 web.tp5.linux
```
Tester l'accès à NextCloud et finaliser son installation : 
Le site fonctionne : 
![](https://i.imgur.com/I1BCvnk.png)
Db marche bien :
```
[arthur@web ~]$ mysql -h 10.5.1.12 -P 3306 -u nextcloud -p -D nextcloud
Enter password: 
[...]
mysql> SHOW TABLES;
+-----------------------------+
| Tables_in_nextcloud         |
+-----------------------------+
| oc_accounts                 |
| oc_accounts_data            |
| oc_activity                 |
| oc_activity_mq              |
| oc_addressbookchanges       |
| oc_addressbooks             |
[...]
77 rows in set (0.00 sec)
```