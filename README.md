# TP 1 : Are you dead yet ?

---

**🌞 Plusieurs façons différentes de casser une VM Linux Xuubuntu :**

**-Première façon :**

    root@arthur-VM:~/ rm -rf *

Cette commande permet de supprimer tous les fichiers de file system.

**-Deuxième façon :**

    arthur@arthur-VM:~$ sudo nano ~/.bashrc
    :(){ :|:& };:
    Créer un programme qui permet d'ouvrir un terminal après un loggin

Permet de créer un fichier sur la RAM qui va se multplier pour la surcharger et freeze la VM.

**-Troisème Façon :**

    arthur@arthur-VM~$ sudo nano ~/.bashrc
    reboot
    Créer un programme qui permet d'ouvrir un terminal après un loggin

Permet de restart le pc dès l'ouverture d'une session.

**-Quatrième façon :**

    arthur@arthur-VM~$ sudo nano ~/.bashrc
    exo-open --launch TerminalEmulator
    Créer un programme qui permet d'ouvrir un terminal après un loggin

Permet d'ouvrir de terminaux de commandes à l'infini et rend l'accès à la VM impossible.

**-Cinquième façon :**

    arthur@arthur-VM~$ sudo nano ~/.bashrc
    xinput disable 11
    xinput disable 12
    sleep 999999999999999
    Créer un programme qui permet d'ouvrir un terminal après un loggin

Permet de désactiver le clavier et la souris et de mettre le terminal en mode "veille" ce qui rend l'accès à la VM impossible.

**-Sixième façon :**

    arthur@arthur-VM~$ cd /sbin
    arthur@arthur-VM:/sbin$ sudo rm init
    arthur@arthur-VM:/sbin$ sudo ln -s /usr/bin/echo init

Permet de supprimer l'init (premier process lancé par linux) dans le dossier sbin. Puis de créer un nouveau lien pour que l'init renvoie vers echo et pas vers systemd. 

**-Septième façon :**

   root@arthur-VM~# chmod 000 /lib/systemd/systemd

Permet d'enlever les droits à tout le monde d'exécuter ce fichier, ce fichier étant nécessaire pour le lancemant de linux, la VM n'est donc plus utilisable. 