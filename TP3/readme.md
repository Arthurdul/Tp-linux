# TP 3 : A little script

# I. Script carte d'identité

Script idcard -> https://github.com/Arthurdul/Tp-linux/blob/main/TP3/idcard.sh.txt
```
Machine name : arthur-VM
OS : NAME="Ubuntu"
VERSION="20.04.3 LTS (Focal Fossa)" and kernel version is Linux version 5.11.0-38-generic
IP : 192.168.56.119/24
RAM : 449Mi RAM remaining of 978Mi
Disk : 2,6G remaining
Top 5 Process by RAM usage
   1390 arthur    20   0   22212   3812   3296 R   5,9   0,4   0:00.03 top
      1 root      20   0  167344  11168   8312 S   0,0   1,1   0:03.95 systemd
      2 root      20   0       0      0      0 S   0,0   0,0   0:00.00 kthreadd
      3 root       0 -20       0      0      0 I   0,0   0,0   0:00.00 rcu_gp
      4 root       0 -20       0      0      0 I   0,0   0,0   0:00.00 rcu_par_gp
Listening ports :
tcp   LISTEN 0      4096    127.0.0.53%lo:domain           0.0.0.0:*
tcp   LISTEN 0      128           0.0.0.0:ssh              0.0.0.0:*
tcp   LISTEN 0      5           127.0.0.1:ipp              0.0.0.0:*
tcp   LISTEN 0      128              [::]:ssh                 [::]:*
tcp   LISTEN 0      5               [::1]:ipp                 [::]:*
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   115  100   115    0     0    195      0 --:--:-- --:--:-- --:--:--   195
Here's your random cat https://cdn2.thecatapi.com/images/MTgxNTQwNQ.jpg
  ```
  
# II. Script youtube-dl

Script yt.sh -> https://github.com/Arthurdul/Tp-linux/blob/main/TP3/yt.sh.txt

Si tous les dossiers sont présents :

```
arthur@arthur-VM:/srv/yt$ sudo bash yt.sh https://www.youtube.com/watch?v=CxcNh6xo32w
Video https://www.youtube.com/watch?v=CxcNh6xo32w was downloaded in mp4 format.
File path : /srv/yt/downloads/Jojo earthquake meme/Jojo earthquake meme.mp4
```

Si il manque un dossier :

```
arthuri@arthur-VM:~/Tp-Linux/TP3/srv/yt$ ls
yt.sh
arthur@arthur-VM:~/srv/yt$ sudo bash yt.sh https://www.youtube.com/watch?v=CxcNh6xo32w
Dossier downloads manquant, error..
```

Si le lien ne marche pas :

```
arthur@arthur-VM:/srv/yt$ sudo bash yt.sh https://www.youtube.com/watch?v=syuzgefizuefz
Link not working
```

le fichier log après plusieurs dl :

```
arthur@arthur-VM:~$ cat /var/log/yt/download.log
[11/22/21 18:55:16] Video https://www.youtube.com/watch?v=lg5WKsVnEA4 was downloaded. File path : /srv/yt/downloads/Michael Rosen - Nice/Michael Rosen - Nice.mp4'
[11/22/21 18:59:37] Video https://www.youtube.com/watch?v=CxcNh6xo32w was downloaded. File path : /srv/yt/downloads/Jojo earthquake meme/Jojo earthquake meme.mp4'
[11/22/21 18:59:41]https://www.youtube.com/watch?v=Hazzioajazeazodh is not a working url'
[11/27/21 13:44:11] Video https://www.youtube.com/watch?v=CxcNh6xo32w was downloaded. File path : /srv/yt/downloads/Jojo earthquake meme/Jojo earthquake meme.mp4'
[11/27/21 13:58:40] Link https://www.youtube.com/watch?v=syuzgefizuefz is not working.'
[11/27/21 14:01:32] Video https://www.youtube.com/watch?v=BW1aX0IbZOE was downloaded. File path : /srv/yt/downloads/Meme 'Okay"/Meme 'Okay".mp4'
[11/27/21 14:04:07] Video https://www.youtube.com/watch?v=05yQW0Sbrv8 was downloaded. File path : /srv/yt/downloads/Mon sac est fait!!!!/Mon sac est fait!!!!.mp4'
```

# III. MAKE IT A SERVICE

Script yt-v2.sh -> https://github.com/Arthurdul/Tp-linux/blob/main/TP3/yt-v2.sh.txt

Le service :

```
arthur@arthur-VM:~$ cat /etc/systemd/system/yt.service
[Unit]
Description="Processus pour download des vidéos"

[Service]
ExecStart=/usr/bin/bash /srv/yt/yt-v2.sh

[Install]
WantedBy=multi-user.target
```

```
arthur@arthur-VM:/srv/yt$ sudo systemctl start yt.service
arthur@arthur-VM:/srv/yt$ sudo systemctl status yt.service
● yt.service - "Processus pour download des vidéos"
     Loaded: loaded (/etc/systemd/system/yt.service; disabled; vendor preset: enabled)
     Active: active (running) since Sat 2021-11-27 14:11:56 CET; 9s ago
   Main PID: 1713 (bash)
      Tasks: 2 (limit: 1105)
     Memory: 608.0K
     CGroup: /system.slice/yt.service
             ├─1713 /usr/bin/bash /srv/yt/yt-v2.sh
             └─1715 sleep 5s

nov. 27 14:11:56 arthur-VM systemd[1]: Started "Processus pour download des vidéos".
```

Je met des bon liens et des mauvais lien puis je vais dans mes log : 

```
arthur@arthur-VM:~/srv/yt$ echo "https://www.youtube.com/watch?v=JwFnC1hFwV4" >> url
arthur@arthur-VM:~/srv/yt$ echo "https://www.youtube.com/watch?v=Qu_sVAByvBk" >> url 
arthur@arthur-VM:~/srv/yt$ echo "https://www.youtube.com/watch?v=zozoebfqkzbds" >> url
arthur@arthur-VM:~/srv/yt$ cat url
https://www.youtube.com/watch?v=JwFnC1hFwV4
https://www.youtube.com/watch?v=Qu_sVAByvBk
https://www.youtube.com/watch?v=zozoebfqkzbds
```

Les premiers liens sont téléchargés et donc supprimés des log mais le mauvais lien est lui aussi delete :

```
arthur@arthur-VM:/srv/yt$ tail -f /var/log/yt/download.log
[11/27/21 14:17:12] Video https://www.youtube.com/watch?v=JwFnC1hFwV4 was downloaded. File path : /srv/yt/downloads/Play Giorno's Theme on sax in Protests/Play Giorno's Theme on sax in Protests.mp4'
[11/27/21 14:17:58] Video https://www.youtube.com/watch?v=Qu_sVAByvBk was downloaded. File path : /srv/yt/downloads/James franco "first time" meme (5 seconds)/James franco "first time" meme (5 seconds).mp4'
[11/27/21 14:18:05]https://www.youtube.com/watch?v=zozoebfqkzbds is not a working url'
```

```
arthur@arthur-VM:/srv/yt$ sudo systemctl status yt
● yt.service - "Processus pour download des vidéos"
     Loaded: loaded (/etc/systemd/system/yt.service; disabled; vendor preset: enabled)
     Active: active (running) since Sat 2021-11-27 14:11:56 CET; 15min ago
   Main PID: 1713 (bash)
      Tasks: 2 (limit: 1105)
     Memory: 4.9M
     CGroup: /system.slice/yt.service
             ├─1713 /usr/bin/bash /srv/yt/yt-v2.sh
             └─1949 sleep 5s

nov. 27 14:11:56 arthur-VM systemd[1]: Started "Processus pour download des vidéos".
nov. 27 14:17:12 arthur-VM bash[1713]: Video https://www.youtube.com/watch?v=JwFnC1hFwV4 was downloaded
nov. 27 14:17:12 arthur-VM bash[1713]: File path : /srv/yt/downloads/Play Giorno's Theme on sax in Protests/Play Giorno's Theme on sax in Protests.mp4
nov. 27 14:17:58 arthur-VM bash[1713]: Video https://www.youtube.com/watch?v=Qu_sVAByvBk was downloaded
nov. 27 14:17:58 arthur-VM bash[1713]: File path : /srv/yt/downloads/James franco "first time" meme (5 seconds)/James franco "first time" meme (5 seconds).mp4
```
```
arthur@arthur-VM:/srv/yt$ sudo systemctl enable yt
Created symlink /etc/systemd/system/multi-user.target.wants/yt.service → /etc/systemd/system/yt.service.
```