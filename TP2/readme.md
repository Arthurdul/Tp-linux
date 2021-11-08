# TP 2 : Manipuler des services.

---

**Intro :**

**➜ Nommer la machine :**

    arthur@arthur:~$ sudo hostname node1.tp1.linux
    arthur@node1:~$ hostname
    node1.tp1.linux

**➜Configuration réseaux :**

    arthur@node1:~$ ping 1.1.1.1
    PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
    64 bytes from 1.1.1.1: icmp_seq=1 ttl=57 time=49.7 ms
    64 bytes from 1.1.1.1: icmp_seq=2 ttl=57 time=48.4 ms
    64 bytes from 1.1.1.1: icmp_seq=3 ttl=57 time=42.3 ms
    --- 1.1.1.1 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2004ms
    rtt min/avg/max/mdev = 42.275/46.816/49.737/3.254 ms

On peut ping 1.1.1.1

    arthur@node1:~$ ping ynov.com
    PING ynov.com (92.243.16.143) 56(84) bytes of data.
    64 bytes from xvm-16-143.dc0.ghst.net (92.243.16.143): icmp_seq=1 ttl=55 time=40.7 ms
    64 bytes from xvm-16-143.dc0.ghst.net (92.243.16.143): icmp_seq=2 ttl=55 time=39.9 ms
    64 bytes from xvm-16-143.dc0.ghst.net (92.243.16.143): icmp_seq=3 ttl=55 time=40.3 ms
    --- ynov.com ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2003ms
    rtt min/avg/max/mdev = 39.850/40.284/40.714/0.352 ms

On peut ping ynov.com

    arthur@node1:~$ ping 192.168.56.116
    PING 192.168.56.116 (192.168.56.116) 56(84) bytes of data.
    64 bytes from 192.168.56.116: icmp_seq=1 ttl=64 time=0.130 ms
    64 bytes from 192.168.56.116: icmp_seq=2 ttl=64 time=0.079 ms
    64 bytes from 192.168.56.116: icmp_seq=3 ttl=64 time=0.073 ms
    --- 192.168.56.116 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2031ms
    rtt min/avg/max/mdev = 0.073/0.094/0.130/0.025 ms

On peut ping la machine 


**SSH :**

**➜ Installation :**

    arthur@node1:~$ sudo apt install openssh-server

**➜ Lancement du service ssh :**

    arthur@node1:~$ systemctl start sshd
    arthur@node1:~$ systemctl status sshd
    ● ssh.service - OpenBSD Secure Shell server
        Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
        Active: active (running) since Sun 2021-11-07 13:51:04 CET; 23min ago
        Docs: man:sshd(8)
             man:sshd_config(5)
    Main PID: 578 (sshd)
      Tasks: 1 (limit: 1105)
    Memory: 4.6M
    CGroup: /system.slice/ssh.service
            └─578 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups

    nov. 07 13:51:03 node1.tp2.linux systemd[1]: Starting OpenBSD Secure Shell server...
    nov. 07 13:51:04 node1.tp2.linux sshd[578]: Server listening on 0.0.0.0 port 22.
    nov. 07 13:51:04 node1.tp2.linux sshd[578]: Server listening on :: port 22.
    nov. 07 13:51:04 node1.tp2.linux systemd[1]: Started OpenBSD Secure Shell server.
    nov. 07 13:52:54 node1.tp2.linux sshd[1830]: Accepted password for arthur from 192.168.56.1 port 61759 ssh2
    nov. 07 13:52:54 node1.tp2.linux sshd[1830]: pam_unix(sshd:session): session opened for user arthur by (uid=0)

**➜ Etude du service SSH:**

    arthur@node1:~$ ps -aux | grep sshd
    root         578  0.0  0.5  12184  5104 ?        Ss   13:51   0:00 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
    arthur@node1:~$ sudo ss -l | grep ssh
    u_str   LISTEN   0        4096             /run/user/1000/gnupg/S.gpg-agent.ssh 27446                                           * 0
    u_str   LISTEN   0        128                  /tmp/ssh-IiSzbznYQY5t/agent.1367 27732                                           * 0
    u_str   LISTEN   0        10                         /run/user/1000/keyring/ssh 28008                                           * 0
    tcp     LISTEN   0        128                                           0.0.0.0:ssh                                       0.0.0.0:*
    tcp     LISTEN   0        128                                              [::]:ssh                                          [::]:*
    arthur@node1:~$ journalctl | grep sshd
    oct. 23 14:43:49 arthur-VM useradd[1516]: new user: name=sshd, UID=123, GID=65534, home=/run/sshd, shell=/usr/sbin/nologin, from=none
    oct. 23 14:43:49 arthur-VM usermod[1524]: change user 'sshd' password
    oct. 23 14:43:49 arthur-VM chage[1531]: changed password expiry for sshd

On se connecte au serveur en SSH

    ssh arthur@192.168.56.
    arthur@192.168.56.116's password:
    arthur@node1:~$

**➜ Modification de la configuration du serveur :**

    arthur@node1:~$ cd /etc/ssh
    arthur@node1:/etc/ssh$ cat sshd_config
    #       $OpenBSD: sshd_config,v 1.103 2018/04/09 20:41:22 tj Exp $

    # This is the sshd server system-wide configuration file.  See
    # sshd_config(5) for more information.

    # This sshd was compiled with PATH=/usr/bin:/bin:/usr/sbin:/sbin

    # The strategy used for options in the default sshd_config shipped with
    # OpenSSH is to specify options with their default value where
    # possible, but leave them commented.  Uncommented options override the
    # default value.
    Include /etc/ssh/sshd_config.d/*.conf

    #Port 22

Le nouveau port est le port 22

    arthur@node1:~$ sudo ss -l | grep 22
    tcp   LISTEN 0      128                                       0.0.0.0:22                      0.0.0.0:*           
    tcp   LISTEN 0      128                                          [::]:22                         [::]:*

Pour se connecter au serveur SSH :

    ssh arthur@192.168.56.116 -p 22
    arthur@192.168.56.116's password: 
    arthur@node1:~$

**FTP :**

**➜ Installation du serveur :**

    arthur@node1:~$ sudo apt install vsftpd

**➜ Lancement du service FTP :**

    arthur@node1:~$ systemctl start vsftpd

**➜ Etude du service FTP :**

    arthur@node1:~$ systemctl status vsftpd
    ● vsftpd.service - vsftpd FTP server
        Loaded: loaded (/lib/systemd/system/vsftpd.service; enabled; vendor preset: e>
        Active: active (running) since Mon 2021-11-08 14:39:05 CET; 6min ago
        Process: 516 ExecStartPre=/bin/mkdir -p /var/run/vsftpd/empty (code=exited, st>
    Main PID: 519 (vsftpd)
      Tasks: 1 (limit: 1105)
     Memory: 716.0K
     CGroup: /system.slice/vsftpd.service
             └─519 /usr/sbin/vsftpd /etc/vsftpd.conf

    nov. 08 14:39:04 node1.tp2.linux systemd[1]: Starting vsftpd FTP server...
    nov. 08 14:39:05 node1.tp2.linux systemd[1]: Started vsftpd FTP server.

    arthur@node1:~$ ps -aux | grep vsftpd
    root         519  0.0  0.3   6816  3016 ?        Ss   14:39   0:00 /usr/sbin/vsftp
    /etc/vsftpd.conf
    sudo ss -l | grep ftp
    tcp     LISTEN   0        32                                                  *:ft
    journalctl | grep vsftpd
    nov. 08 14:55:56 node1.tp1.linux sudo[2567]:   arthur : TTY=pts/0 ; PWD=/home/arthur ; USER=root ; COMMAND=/usr/bin/apt install vsftpd
    arthur@node1:~$ sudo cat /var/log/vsftpd.log

**➜ Upload et Download :** 

    arthur@node1:~$ sudo nano /etc/vsftpd.conf
    write_enable=YES

On a les permitions d'écrire via filezilla.

    arthur@node1:~$ sudo cat /var/log/vsftpd.log
    Mon Nov  8 16:05:37 2021 [pid 2231] [arthur] OK UPLOAD: Client "::ffff:192.168.56.1", "/home/arthur/test tp2.txt", 0.00Kbyte/sec

Le fichier text à donc bien été upload et download.

**➜ Modification de la configuration serveur :**

    arthur@node1:~$ sudo nano /etc/vsftpd.conf
    listen_port=33450
    arthur@node1:~$ sudo systemctl restart vsftpd

J'essaye de me connecter au FTP avec le nouveau port.

    arthur@node1:~$ ftp 192.168.56.116 33450
    Connected to 192.168.56.116.
    220 (vsFTPd 3.0.3)
    Name (192.168.56.116:arthur): arthur
    331 Please specify the password.
    Password:
    230 Login successful.

**Création de votre propre service :**

**➜ Jouer avec NetCat**

Sur mon pc :

    arthur@node1:~$ sudo apt install netcat
    arthur@node1:~$ nc -l 7654

Sur la VM :

    arthur@arthur:~$ sudo apt install netcat
    arthur@arthur:~$ nc 192.168.56.116 7654

Les deux peuvent maintenant discuter ensemble :

    arthur@node1:~$ nc -l 7564
    Hello

**➜ Un service basé sur NetCat :**

    arthur@node1:~$ sudo nano /etc/systemd/system/chat_tp2.service
        [Unit]
        Description=Little chat service (TP2)

        [Service]
        ExecStart=nc -l 7564

        [Install]
        WantedBy=multi-user.target
    chmod 777 /etc/systemd/system/chat_tp2.service

**Test test et retest :**

    arthur@node1:~$ systemctl start chat_tp2.service
    arthur@node1:~$ systemctl status chat_tp2
    ● chat_tp2.service - Little chat service (TP2)
        Loaded: loaded (/etc/systemd/system/chat_tp2.service; disabled; vendor preset: enabled)
        Active: active (running) since Mon 2021-11-08 17:09:06 CET; 6s ago
   Main PID: 2721 (nc)
      Tasks: 1 (limit: 1105)
     Memory: 188.0K
     CGroup: /system.slice/chat_tp2.service
             └─2721 /usr/bin/nc -l 7564

    arthur@node1:~$ journalctl -xe -u chat_tp2
    -- Logs begin at Sat 2021-10-23 14:39:58 CEST, end at Mon 2021-11-08 17:09:26 CET. --
    nov. 08 17:09:06 node1.tp2.linux systemd[1]: Started Little chat service (TP2).
    -- Subject: A start job for unit chat_tp2.service has finished successfully

    arthur@node1:~$ journalctl -xe -u chat_tp2 -f
    -- Logs begin at Sat 2021-10-23 14:39:58 CEST. --
    nov. 08 17:09:06 node1.tp2.linux systemd[1]: Started Little chat service (TP2).
    -- Subject: A start job for unit chat_tp2.service has finished successfully
    -- Defined-By: systemd
    -- Support: http://www.ubuntu.com/support
    --
    -- A start job for unit chat_tp2.service has finished successfully.
    --
    -- The job identifier is 3955.
    nov. 08 17:15:25 node1.tp2.linux systemd[1]: chat_tp2.service: Succeeded.
    -- Subject: Unit succeeded
    -- Defined-By: systemd
    -- Support: http://www.ubuntu.com/support
    --
    -- The unit chat_tp2.service has successfully entered the 'dead' sta
