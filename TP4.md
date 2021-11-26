# TP4 : Une distribution orientée serveur 

## II. Check list :

**➜ Définir une IP à la VM :**

    [arthur@node1 ~]$ cat /etc/sysconfig/network-    scripts/ifcfg-enp0s8
    BOOTPROTO=static
    NAME=enp0s8
    DEVICE=enp0s8
    ONBOOT=yes
    IPADDR=10.200.1.37
    NETMASK=255.255.255.0
    [arthur@node1 ~]$ ip a
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
    2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:58:7e:f1 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
       valid_lft 79580sec preferred_lft 79580sec
    inet6 fe80::a00:27ff:fe58:7ef1/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
    3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:a9:a5:d1 brd ff:ff:ff:ff:ff:ff
    inet 10.200.1.37/24 brd 10.200.1.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fea9:a5d1/64 scope link
       valid_lft forever preferred_lft forever

**➜ Connexion SSH :**

    [arthur@node1 ~]$ sudo systemctl status sshd
    ● sshd.service - OpenSSH server daemon
       Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; vendor preset: enabled)
       Active: active (running) since Tue 2021-11-23 11:36:15 CET; 1h 58min ago
     Docs: man:sshd(8)
           man:sshd_config(5)
     Main PID: 856 (sshd)
    Tasks: 1 (limit: 4943)
       Memory: 5.7M
       CGroup: /system.slice/sshd.service
           └─856 /usr/sbin/sshd -D -oCiphers=aes256-gcm@openssh.com,chacha20-poly1305@openssh.com,aes256-ctr,aes256-cbc,aes128-gcm@openssh.com,aes128-ctr,aes128-cbc -o>

    Nov 23 12:25:37 localhost.localdomain sshd[1696]: pam_unix(sshd:session): session closed for user arthur
    Nov 23 12:26:22 localhost.localdomain sshd[1731]: Accepted password for arthur from 10.200.1.1 port 53043 ssh2
    Nov 23 12:26:22 localhost.localdomain sshd[1731]: pam_unix(sshd:session): session opened for user arthur by (uid=0)
    Nov 23 12:42:43 localhost.localdomain sshd[1796]: Accepted password for arthur from 10.200.1.1 port 54918 ssh2
    Nov 23 12:42:43 localhost.localdomain sshd[1796]: pam_unix(sshd:session): session opened for user arthur by (uid=0)
    Nov 23 12:42:43 localhost.localdomain sshd[1796]: pam_unix(sshd:session): session closed for user arthur
    Nov 23 12:43:09 localhost.localdomain sshd[1839]: Accepted publickey for arthur from 10.200.1.1 port 54947 ssh2: RSA SHA256:vQUXPx3n13x4+6HrpvJ7tmHjhgwhEWws3vK77JNgigc
    Nov 23 12:43:09 localhost.localdomain sshd[1839]: pam_unix(sshd:session): session opened for user arthur by (uid=0)
    Nov 23 13:06:56 node1.tp4.linux sshd[1908]: Accepted publickey for arthur from 10.200.1.1 port 50180 ssh2: RSA SHA256:vQUXPx3n13x4+6HrpvJ7tmHjhgwhEWws3vK77JNgigc
    Nov 23 13:06:56 node1.tp4.linux sshd[1908]: pam_unix(sshd:session): session opened for user arthur by (uid=0)
    
Sur mon PC : 

    $ cat id_rsa.pub
    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC3GN+mp62lXGJJ90DTpgbsDua0kNudlL8EZbSqjfDAVEa9wtWnuCR2JOvCWfqONcBmh+pA2MA9SUHd5dhezGNtijg6WRqV9b+xAgtoVmEgQd6z8MLiw/ZkHENW4pf/JqXRLkigvyjq2YOWAb7MYmfBpuQz/UbdHQaToH66U8C2lZc1Vf8b8ZP0EBYFLwiWt0zPraY/7tFGKbj0/StvoAxN43y0kdSpcgejRNrXdAZh3+nj+d0K5qpB2OW+MetA49+h/qkNyH1lDMKjgNSAgCpVL2Zo2W1YlKVgdcCRPv5K72kDodBhJO27kPjy3pbXVJ22aE6QLUuKXqb2HuiB67WlQ+FQDAsXno7tlDz2Oor4tFftAACVa4mYmOJxWkLbqcrr9Wi2nZqbd+2oKJwgzHyadsaegJbZTjpZwA/0oFD5YRroUZI8QEphNsTO/VH9gjq3HuC7Gu1KOMf3gy5Zt/Ofecu2+3moE0Uq8SKN0nwSpidLKDSHvImhp9ErtdmgSBNaCE5T31U+SPPHufHNowqiBbY+F0qzlz0uQAEK4NQx23yCOP/AbeGuWg9hhWunSpPSbN8TWHUnW6gA4UXJYjBid+HK1hEecSOjaZQewht6u6aGnmUxj4KWAcHuceLmmgIw3RHJDaiY/ExdTLM9aAdrEWJ13u2ZdJY8glX+zOwXtw== arthu@DESKTOP-LBR4JUU
    $ ssh-copy-id -i ~/.ssh/id_rsa.pub arthur@10.200.1.37
    /usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/c/Users/arthu/.ssh/id_rsa.pub"
    /usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed

    /usr/bin/ssh-copy-id: WARNING: All keys were skipped because they already exist on the remote system.
                (if you think this is a mistake, you may want to use -f option)

Etant donné que j'ai déjà copié la clé vers ma VM ça ne marche pas car la clé est déjà affilié. Mais si je me tente de me connecter :
    
    C:\Users\arthu>ssh arthur@10.200.1.37
    Activate the web console with: systemctl enable --now cockpit.socket

    Last login: Tue Nov 23 13:06:56 2021 from 10.200.1.1
    [arthur@node1 ~]$

Sur ma VM : 

    [arthur@node1 .ssh]$ cat authorized_keys

    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC3GN+mp62lXGJJ90DTpgbsDua0kNudlL8EZbSqjfDAVEa9wtWnuCR2JOvCWfqONcBmh+pA2MA9SUHd5dhezGNtijg6WRqV9b+xAgtoVmEgQd6z8MLiw/ZkHENW4pf/JqXRLkigvyjq2YOWAb7MYmfBpuQz/UbdHQaToH66U8C2lZc1Vf8b8ZP0EBYFLwiWt0zPraY/7tFGKbj0/StvoAxN43y0kdSpcgejRNrXdAZh3+nj+d0K5qpB2OW+MetA49+h/qkNyH1lDMKjgNSAgCpVL2Zo2W1YlKVgdcCRPv5K72kDodBhJO27kPjy3pbXVJ22aE6QLUuKXqb2HuiB67WlQ+FQDAsXno7tlDz2Oor4tFftAACVa4mYmOJxWkLbqcrr9Wi2nZqbd+2oKJwgzHyadsaegJbZTjpZwA/0oFD5YRroUZI8QEphNsTO/VH9gjq3HuC7Gu1KOMf3gy5Zt/Ofecu2+3moE0Uq8SKN0nwSpidLKDSHvImhp9ErtdmgSBNaCE5T31U+SPPHufHNowqiBbY+F0qzlz0uQAEK4NQx23yCOP/AbeGuWg9hhWunSpPSbN8TWHUnW6gA4UXJYjBid+HK1hEecSOjaZQewht6u6aGnmUxj4KWAcHuceLmmgIw3RHJDaiY/ExdTLM9aAdrEWJ13u2ZdJY8glX+zOwXtw== arthu@DESKTOP-LBR4JUU
    
**➜ Accès internet :**

    [arthur@node1]$ ping 8.8.8.8
    PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
    64 bytes from 8.8.8.8: icmp_seq=1 ttl=119 time=35.10 ms
    64 bytes from 8.8.8.8: icmp_seq=2 ttl=119 time=39.1 ms
    64 bytes from 8.8.8.8: icmp_seq=3 ttl=119 time=36.10 ms
    ^C
    --- 8.8.8.8 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2003ms
    rtt min/avg/max/mdev = 35.985/37.349/39.084/1.311 ms
    
    
    [arthur@node1]$ ping hentai-paradise.fr
    PING hentai-paradise.fr (104.26.0.84) 56(84) bytes of data.
    64 bytes from 104.26.0.84 (104.26.0.84): icmp_seq=1 ttl=57 time=37.5 ms
    64 bytes from 104.26.0.84 (104.26.0.84): icmp_seq=2     ttl=57 time=38.1 ms
    64 bytes from 104.26.0.84 (104.26.0.84): icmp_seq=3 ttl=57 time=38.3 ms
    ^C
    --- hentai-paradise.fr ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2004ms
    rtt min/avg/max/mdev = 37.538/37.979/38.292/0.320     ms
    
**➜ Nommage de la machine :**

    [arthur@node1]$ cat /etc/hostname
    node1.tp4.linux
    [arthur@node1]$ hostname
    node1.tp4.linux

## III. Mettre en place un service :

    [arthur@node1]$ sudo dnf install nginx
    [...]
    Complete!
    
    [arthur@node1]$ sudo systemctl start nginx
    [arthur@node1 ~]$ sudo systemctl status nginx
    [sudo] password for arthur:
    ● nginx.service - The nginx HTTP and reverse proxy server
       Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
       Active: active (running) since Tue 2021-11-23 13:14:55 CET; 48min ago
     Main PID: 30818 (nginx)
    Tasks: 2 (limit: 4943)
       Memory: 3.7M
       CGroup: /system.slice/nginx.service
           ├─30818 nginx: master process /usr/sbin/nginx
           └─30819 nginx: worker process

    Nov 23 13:14:55 node1.tp4.linux systemd[1]: Starting The nginx HTTP and reverse proxy server...
    Nov 23 13:14:55 node1.tp4.linux nginx[30815]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    Nov 23 13:14:55 node1.tp4.linux nginx[30815]: nginx: configuration file /etc/nginx/nginx.conf test is successful
    Nov 23 13:14:55 node1.tp4.linux systemd[1]: nginx.service: Failed to parse PID from file /run/nginx.pid: Invalid argument
    Nov 23 13:14:55 node1.tp4.linux systemd[1]: Started The nginx HTTP and reverse proxy server.
    
    
    [arthur@node1 ~]$ ps -aux |grep nginx
    root       30818  0.0  0.2 119160  2164 ?        Ss   13:14   0:00 nginx: master process /usr/sbin/nginx
    nginx      30819  0.0  0.9 151820  8072 ?        S    13:14   0:00 nginx: worker process
    arthur     31180  0.0  0.1 221928  1156 pts/1    S+   14:04   0:00 grep --color=auto nginx
    [arthur@node1 ~]$ sudo ss -ltnp
    State        Recv-Q       Send-Q              Local Address:Port               Peer Address:Port       Process
    LISTEN       0            128                       0.0.0.0:80                      0.0.0.0:*           users:(("nginx",pid=30819,fd=8),("nginx",pid=30818,fd=8))
    LISTEN       0            128                       0.0.0.0:22                      0.0.0.0:*           users:(("sshd",pid=856,fd=5))
    LISTEN       0            128                          [::]:80                         [::]:*           users:(("nginx",pid=30819,fd=9),("nginx",pid=30818,fd=9))
    LISTEN       0            128                          [::]:22                         [::]:*           users:(("sshd",pid=856,fd=7))
    
    
    [arthur@node1 ~]$ ls -la /usr/share/nginx/html/
    -rw-r--r--. 1 root root 3332 Jun 10 11:09 404.html
    -rw-r--r--. 1 root root 3404 Jun 10 11:09 50x.html
    -rw-r--r--. 1 root root 3429 Jun 10 11:09 index.html
    -rw-r--r--. 1 root root  368 Jun 10 11:09 nginx-logo.png
    -rw-r--r--. 1 root root 1800 Jun 10 11:09 poweredby.png
    
**➜ Visite du service web :**

    [arthur@node1 ~]$ sudo firewall-cmd --add-port=80/tcp --permanent
    success
    [arthur@node1 ~]$ sudo firewall-cmd --reload
    success
    
    [arthur@node1 ~]$ curl http://10.200.1.37
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN"     "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">

**➜ Modification de la configuration du serveur web :**

Changer le port d'écoute :

    [arthur@node1 ~]$ sudo nano /etc//nginx/nginx.conf
        server {
        listen       8080 default_server;
        listen       [::]:8080 default_server;
        server_name  _;
        root         /usr/share/nginx/html;
    
    
    [arthur@node1 ~]$ systemctl restart nginx
    [arthur@node1 ~]$ systemctl status nginx
    ● nginx.service - The nginx HTTP and reverse proxy server
       Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
       Active: active (running) since Tue 2021-11-23 14:27:16 CET; 1min 21s ago
      Process: 31312 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
      Process: 31309 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
      Process: 31307 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
     Main PID: 31314 (nginx)
    Tasks: 2 (limit: 4943)
       Memory: 3.7M
       CGroup: /system.slice/nginx.service
           ├─31314 nginx: master process /usr/sbin/nginx
           └─31315 nginx: worker process

    Nov 23 14:27:16 node1.tp4.linux systemd[1]: nginx.service: Succeeded.
    Nov 23 14:27:16 node1.tp4.linux systemd[1]: Stopped The nginx HTTP and reverse proxy server.
    Nov 23 14:27:16 node1.tp4.linux systemd[1]: Starting The nginx HTTP and reverse proxy server...
    Nov 23 14:27:16 node1.tp4.linux nginx[31309]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    Nov 23 14:27:16 node1.tp4.linux nginx[31309]: nginx: configuration file /etc/nginx/nginx.conf test is successful
    Nov 23 14:27:16 node1.tp4.linux systemd[1]: nginx.service: Failed to parse PID from file /run/nginx.pid: Invalid argument
    Nov 23 14:27:16 node1.tp4.linux systemd[1]: Started The nginx HTTP and reverse proxy server.
    
    [arthur@node1 ~]$ sudo firewall-cmd --add-port=8080/tcp --permanent
    success
    [arthur@node1 ~]$ sudo firewall-cmd --remove-port=80/tcp --permanent
    success
    [arthur@node1 ~]$ sudo firewall-cmd --reload
    success
    
    [arthur@node1 ~]$ sudo ss -ltpn | grep nginx
    LISTEN 0      128          0.0.0.0:8080       0.0.0.0:*    users:(("nginx",pid=31315,fd=8),("nginx",pid=31314,fd=8))
    LISTEN 0      128             [::]:8080         [::]:*    users:(("nginx",pid=31315,fd=9),("nginx",pid=31314,fd=9))
    
    [arthur@node1 ~]$ curl http://10.200.1.37:8080
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">
    
Changer l'utilisateur qui lance le service : 

    [arthur@node1 ~]$ sudo useradd web -m -s /bin/bash -u 2120
    
    [arthur@node1 ~]$ sudo passwd web
    New password:
    Retype new password:
    passwd: all authentication tokens updated successfully.
    [arthur@node1 ~]$ cat /etc/passwd | grep web
    web:x:2120:2120::/home/web:/bin/bash
    
    [arthur@node1 ~]$ cat /etc/nginx/nginx.conf
    user web;
    web      31417  0.0  0.9 151820  7988 ?        S    14:46   0:00 nginx: worker process
    
Changer l'emplacement de la racine web : 

    [arthur@node1 var]$ sudo mkdir www
    [arthur@node1 var]$ cd www/
    [arthur@node1 www]$ sudo mkdir super_site_web
    [arthur@node1 www]$ cd super_site_web/
    [arthur@node1 super_site_web]$ sudo nano index.html
    [arthur@node1 super_site_web]$ cd ..
    [arthur@node1 www]$ ls -la
    total 4
    drwxr-xr-x.  3 root root   28 Nov 23 14:52 .
    drwxr-xr-x. 22 root root 4096 Nov 23 14:49 ..
    drwxr-xr-x.  2 web  web    29 Nov 23 14:53 super_site_web
    [arthur@node1 super_site_web]$ ls -la
    total 4
    drwxr-xr-x. 2 web  web  24 Nov 23 15:11 .
    drwxr-xr-x. 3 root root 28 Nov 23 14:52 ..
    -rw-r--r--. 1 root root 33 Nov 23 15:11 index.html
    
    [arthur@node1 ~]$ sudo nano /etc/nginx/nginx.conf
    [arthur@node1 ~]$ cat /etc/nginx/nginx.conf | grep root
        root         /var/www/super_site_web;
        
        [arthur@node1 ~]$ curl 10.200.1.37:8080
    <html>
        <h1>Saitame > Goku </h1>
    