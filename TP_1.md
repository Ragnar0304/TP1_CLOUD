# TP 1 — Installation de VirtualBox et création d'une VM Ubuntu

**Cours :** Introduction au Cloud · L3 SI RIST & BDGL
**Enseignant :** Dr BAKAYOKO
**Groupe :** KOUASSI KOUAKOU YANN LOIC · KANGAH KOUAME LUC-MARTIAL · FOFANA CHEICK MANSOUR
**Date :** 14-15 octobre 2026

> Les sorties des commandes sont recopiées en texte dans ce fichier. Les captures d'écran d'origine sont dans le dossier `captures/`, à côté de `TP1.md`.

## Sommaire

1. [Partie 1 — Préparer l'hôte et installer VirtualBox](#partie-1--préparer-lhôte-et-installer-virtualbox)
2. [Partie 2 — Télécharger et vérifier l'image ISO Ubuntu](#partie-2--télécharger-et-vérifier-limage-iso-ubuntu)
3. [Partie 3 — Créer et configurer la machine virtuelle](#partie-3--créer-et-configurer-la-machine-virtuelle)
4. [Partie 4 — Installer Ubuntu Server](#partie-4--installer-ubuntu-server)
5. [Partie 5 — Premiers pas, réseau et accès SSH](#partie-5--premiers-pas-réseau-et-accès-ssh)
6. [Partie 6 — Instantané, clone et arrêt](#partie-6--instantané-clone-et-arrêt)
7. [Difficultés rencontrées](#difficultés-rencontrées)
8. [Conclusion](#conclusion)

---

## Partie 1 — Préparer l'hôte et installer VirtualBox

### Version de VirtualBox et informations sur l'hôte

Commandes (PowerShell) :

```powershell
VBoxManage --version
VBoxManage list hostinfo
```

Sortie :

```text
PS C:\Users\kkoua> VBoxManage --version
7.2.20r175154
PS C:\Users\kkoua> VBoxManage list hostinfo
Host Information:

Host time: 2026-10-14T17:27:08.748000000Z
Processor online count: 12
Processor count: 12
Processor online core count: 10
Processor core count: 10
Processor supports HW virtualization: yes
Processor supports PAE: yes
Processor supports long mode: yes
Processor supports nested paging: yes
Processor supports unrestricted guest: no
Processor supports nested HW virtualization: no
Processor supports virt. vmsave/vmload: no
Processor#0 speed: unknown
Processor#0 description: 12th Gen Intel(R) Core(TM) i5-1235U
[... les processeurs #1 à #11 donnent la même description : 12th Gen Intel(R) Core(TM) i5-1235U ...]
Memory size: 16088 MByte
Memory available: 2480 MByte
Operating system: Windows 11
Operating system version: 10.0.26200.9457
```

![Sortie de VBoxManage --version et list hostinfo](captures/p1_hote.png)

### Hyperviseur de Windows

Commande :

```powershell
bcdedit
```

Sortie (extrait : le « Chargeur de démarrage Windows », qui contient la ligne recherchée) :

```text
Chargeur de démarrage Windows
-------------------------------
identificateur          {current}
device                  partition=C:
path                    \WINDOWS\system32\winload.efi
description             Windows 11
locale                  fr-FR
inherit                 {bootloadersettings}
recoverysequence        {e8fea123-47ed-11f0-941b-ef2e2c88e669}
displaymessageoverride  Recovery
recoveryenabled         Yes
isolatedcontext         Yes
allowedinmemorysettings 0x15000075
osdevice                partition=C:
systemroot              \WINDOWS
resumeobject            {e8fea121-47ed-11f0-941b-ef2e2c88e669}
nx                      OptOut
bootmenupolicy          Standard
hypervisorlaunchtype    Auto
```

![Sortie de bcdedit : hypervisorlaunchtype = Auto](captures/p1_hyperviseur_windows.png)

### Réponses aux questions

**1. Qu'est-ce qu'un hyperviseur de type 1 et de type 2 ? Dans quelle catégorie se range VirtualBox ? Citez un hyperviseur de type 1 utilisé par un fournisseur cloud.**
Un hyperviseur de type 1 s'exécute directement sur le matériel (ESXi, Hyper-V, KVM, Xen). Un hyperviseur de type 2 est une application installée sur un système d'exploitation hôte (VirtualBox, VMware Workstation). **VirtualBox est de type 2.** Chez les fournisseurs cloud, on trouve AWS Nitro (basé sur KVM) et Azure avec Hyper-V.

**2. Pourquoi la virtualisation matérielle (VT-x / AMD-V) est-elle indispensable pour faire tourner une VM 64 bits ?**
Avec VT-x / AMD-V, le processeur exécute directement le code de la VM et l'hyperviseur n'intercepte que les instructions privilégiées. Sans elle, l'émulation est très lente et VirtualBox refuse l'invité 64 bits. Sur notre PC, la sortie ci-dessus indique `Processor supports HW virtualization: yes`.

**3. Quelle version de VirtualBox avez-vous installée ? Que renvoie `VBoxManage list hostinfo` sur le nombre de cœurs et la RAM de l'hôte ?**
VirtualBox **7.2.20** (build `r175154`). `list hostinfo` indique **10 cœurs physiques** (`Processor core count: 10`), **12 processeurs logiques** et **16 088 Mo** de RAM (`Memory size`), dont **2 480 Mo** disponibles (`Memory available`).

---

## Partie 2 — Télécharger et vérifier l'image ISO Ubuntu

### Calcul et comparaison de l'empreinte SHA-256

Commandes :

```powershell
cd $HOME\Downloads
Get-FileHash .\ubuntu-26.04-live-server-amd64.iso -Algorithm SHA256
Select-String -Path .\SHA256SUMS -Pattern "live-server-amd64"
```

Sortie :

```text
PS C:\Users\kkoua\Downloads> Get-FileHash .\ubuntu-26.04-live-server-amd64.iso -Algorithm SHA256

Algorithm       Hash                                                                   Path
---------       ----                                                                   ----
SHA256          DEC49008A71F6098D0BCFC822021F4D042D5F2DB279E4D75BDD981304F1CA5D9       C:\Users\kk...

PS C:\Users\kkoua\Downloads> Select-String -Path .\SHA256SUMS -Pattern "live-server-amd64"

SHA256SUMS:2:dec49008a71f6098d0bcfc822021f4d042d5f2db279e4d75bdd981304f1ca5d9
*ubuntu-26.04-live-server-amd64.iso
SHA256SUMS:5:cc8a95cde20f6ced61a322420de00f10cc3c90ced545daa46cb9c1a117f1d927
*ubuntu-26.04.1-live-server-amd64.iso
```

Le fichier `ubuntu-26.04-live-server-amd64.iso` a une empreinte **identique** à la ligne 2 de `SHA256SUMS` (la casse mise à part). La ligne 5 correspond à une autre version (26.04.1), que nous n'avons pas utilisée. VirtualBox indique une taille de 2,72 Gio pour l'ISO (voir la partie 3).

![Empreinte SHA-256 calculée et ligne de SHA256SUMS](captures/p2_sha256.png)

### Réponses aux questions

**1. Les deux empreintes sont-elles identiques ? Pourquoi faut-il vérifier l'empreinte d'une image avant de l'installer ?**
Oui, elles sont identiques. La vérification garantit l'intégrité de l'ISO (ni corruption pendant le téléchargement, ni modification) avant d'installer un serveur.

**2. Quelle différence entre Ubuntu Desktop et Ubuntu Server ? Pourquoi choisit-on Server pour un serveur cloud ?**
Desktop apporte une interface graphique et des applications. Server fonctionne en ligne de commande seule : il consomme moins de ressources, a une surface d'attaque réduite et s'administre à distance. Il est donc adapté au cloud.

**3. Que signifie LTS ? Combien d'années de mises à jour de sécurité standard une version LTS reçoit-elle ?**
LTS signifie *Long Term Support* : une version tous les deux ans, avec **5 ans** de mises à jour de sécurité (10 ans avec Ubuntu Pro).

---

## Partie 3 — Créer et configurer la machine virtuelle

La VM a été créée avec l'assistant **Nouvelle** de VirtualBox (installation sans surveillance désactivée), puis la redirection de port a été ajoutée en ligne de commande.

### Paramètres de la VM

| Paramètre | Valeur |
| --- | --- |
| Nom | ubuntu-srv |
| Image ISO | `ubuntu-26.04-live-server-amd64.iso` (2,72 Gio) |
| Mémoire vive | 2 048 Mo au départ, puis **1 536 Mo** (voir [Difficultés](#difficultés-rencontrées)) |
| Processeurs | 2 au départ, puis **1** (même raison) |
| Disque dur | `ubuntu-srv.vdi`, 25,00 Gio, alloué dynamiquement |
| Carte réseau 1 | Intel PRO/1000 MT Desktop (NAT) |

La capture ci-dessous montre la VM **telle que créée au départ** (2 048 Mo et 2 processeurs). Nous l'avons réduite ensuite à 1 536 Mo et 1 processeur, valeurs confirmées par la sortie de `free -h` et `nproc` en partie 5.

![La VM ubuntu-srv dans le Gestionnaire de machines, telle que créée au départ](captures/p3_vm_ubuntu-srv.png)

### Redirection de port SSH

Commandes :

```powershell
$env:Path += ";C:\Program Files\Oracle\VirtualBox"
VBoxManage modifyvm ubuntu-srv --natpf1 "ssh,tcp,127.0.0.1,2222,,22"
VBoxManage showvminfo ubuntu-srv | Select-String "NIC 1"
```

Sortie :

```text
NIC 1:                       MAC: 0800276266C9, Attachment: NAT, Cable connected: on, Trace: off
(file: none), Type: 82540EM, Reported speed: 0 Mbps, Boot priority: 0, Promisc Policy: deny,
Bandwidth group: none
NIC 1 Settings:
NIC 1 Rule(0):   name = ssh, protocol = tcp, host ip = 127.0.0.1, host port = 2222, guest ip = ,
guest port = 22
```

![Règle de redirection ssh : 127.0.0.1:2222 vers le port 22 de la VM](captures/p3_redirection_port.png)

### Réponses aux questions

**1. Quelle différence entre un disque alloué dynamiquement et un disque de taille fixe ? Quelle place occupe le fichier `.vdi` juste après sa création ?**
Un disque **dynamique** n'occupe que l'espace réellement écrit et grossit jusqu'à 25 Go. Un disque **fixe** réserve les 25 Go immédiatement. Juste après sa création, le fichier `.vdi` est très petit (la valeur exacte n'a pas été relevée) ; VirtualBox affiche seulement sa taille maximale de 25,00 Gio.

**2. Pourquoi désactive-t-on l'installation sans surveillance pour ce TP ?**
Pour installer à la main et choisir soi-même le réseau, le LVM, l'utilisateur et OpenSSH. L'installation automatique impose un utilisateur et n'installe pas forcément OpenSSH.

**3. En mode NAT, pourquoi l'hôte ne peut-il pas joindre directement la VM ? À quoi sert la redirection 2222 → 22 ?**
La VM est sur un réseau privé (10.0.2.0/24) derrière le NAT : les connexions entrantes ne sont pas routées. La redirection fait écouter l'hôte sur `127.0.0.1:2222` et transmet les connexions au port SSH 22 de la VM (règle `ssh` visible ci-dessus).

---

## Partie 4 — Installer Ubuntu Server

### Choix faits dans l'installateur

| Écran | Choix |
| --- | --- |
| Langue | Français |
| Type d'installation | Ubuntu Server |
| Réseau | enp0s3 en DHCP |
| Stockage | LVM (`/` = 11,5 Go) |
| Profil | serveur `ubuntu-srv`, utilisateur `etudiantufhb` |
| SSH | serveur OpenSSH installé |
| Snaps proposés | aucun |

### Durée de l'installation

Nous n'avons pas chronométré l'installation. Nous l'avons estimée a posteriori à partir de l'horodatage des fichiers de `/var/log/installer/`.

Commandes :

```powershell
VBoxManage startvm ubuntu-srv --type headless
ssh -p 2222 etudiantufhb@127.0.0.1 "ls -l --time-style=full-iso /var/log/installer/"
ssh -p 2222 etudiantufhb@127.0.0.1 "sudo head -n 1 /var/log/installer/subiquity-server-debug.log; sudo tail -n 1 /var/log/installer/subiquity-server-debug.log"
```

Sortie :

```text
PS C:\Users\kkoua\Downloads> VBoxManage startvm ubuntu-srv --type headless
Waiting for VM "ubuntu-srv" to power on...
VM "ubuntu-srv" has been successfully started.
PS C:\Users\kkoua\Downloads> ssh -p 2222 etudiantufhb@127.0.0.1 "ls -l --time-style=full-iso /var/log/installer/"
etudiantufhb@127.0.0.1's password:
total 1372
-r-------- 1 root root   3220 2026-10-14 23:14:27.006344443 +0000 autoinstall-user-data
drwxr-xr-x 2 root root   4096 2026-10-14 23:06:51.066287275 +0000 block
-rw-r--r-- 1 root root     52 2026-10-14 23:10:06.679551621 +0000 casper-md5check.json
-rw------- 1 root root   3286 2026-10-14 23:06:33.429999859 +0000 cloud-init-output.log
-rw-r----- 1 root root  89287 2026-10-14 23:06:33.429999859 +0000 cloud-init.log
drwxr-xr-x 2 root root   4096 2026-10-14 23:10:10.627352948 +0000 curtin-install
-rw-r--r-- 1 root root 118157 2026-10-14 23:14:26.556455341 +0000 curtin-install.log
-rw-r--r-- 1 root root    155 2026-10-14 23:08:46.826897913 +0000 device-map.json
-rw------- 1 root root 746579 2026-10-14 23:20:48.216470061 +0000 installer-journal.txt
-r--r--r-- 1 root root     67 2026-10-14 23:14:25.817467895 +0000 media-info
lrwxrwxrwx 1 root root     31 2026-10-14 23:06:45.210154459 +0000 subiquity-client-debug.log -> subiquity-client-debug.log.1699
-rw------- 1 root root 203380 2026-10-14 23:20:47.941091331 +0000 subiquity-client-debug.log.1699
lrwxrwxrwx 1 root root     30 2026-10-14 23:06:45.210063987 +0000 subiquity-client-info.log -> subiquity-client-info.log.1699
-rw------- 1 root root   3230 2026-10-14 23:12:49.613884504 +0000 subiquity-client-info.log.1699
lrwxrwxrwx 1 root root     31 2026-10-14 23:06:44.981155760 +0000 subiquity-server-debug.log -> subiquity-server-debug.log.1782
-rw------- 1 root root 197658 2026-10-14 23:20:47.960473966 +0000 subiquity-server-debug.log.1782
lrwxrwxrwx 1 root root     30 2026-10-14 23:06:44.981076568 +0000 subiquity-server-info.log -> subiquity-server-info.log.1782
-rw------- 1 root root   4396 2026-10-14 23:20:47.817476148 +0000 subiquity-server-info.log.1782
PS C:\Users\kkoua\Downloads> ssh -p 2222 etudiantufhb@127.0.0.1 "sudo head -n 1 /var/log/installer/subiquity-server-debug.log; sudo tail -n 1 /var/log/installer/subiquity-server-debug.log"
etudiantufhb@127.0.0.1's password:
sudo: A terminal is required to authenticate
sudo: A terminal is required to authenticate
```

La dernière commande a échoué : `sudo` demande un terminal, d'où l'option `-t` utilisée ensuite (voir [Difficultés](#difficultés-rencontrées)).

Lecture des horodatages : le serveur de l'installateur démarre à **23:06:44** (`subiquity-server-debug.log`) et `curtin-install.log` est écrit pour la dernière fois à **23:14:26**. L'installation a donc duré environ **8 minutes**.

![Horodatage des journaux de l'installateur](captures/p4_duree_installation.png)

### Réponses aux questions

**1. Combien de temps a duré l'installation ? Comparez avec la création d'une instance cloud à partir d'une image prête.**
Environ **8 minutes** (de 23:06:44 à 23:14:26, estimées d'après les journaux, faute de chronométrage). Une instance cloud part d'une image prête et démarre en 1 à 2 minutes.

**2. Quelle adresse IP l'installateur a-t-il obtenue sur enp0s3 ? Qui l'a attribuée ?**
**10.0.2.15/24** (voir `ip -4 addr show` en partie 5), attribuée par le serveur DHCP intégré au NAT de VirtualBox.

**3. Pourquoi installe-t-on le serveur OpenSSH dès l'installation ? Que se passerait-il sur un serveur cloud sans SSH ?**
Pour administrer la VM à distance. Sans SSH, un serveur cloud, qui n'a ni écran ni clavier, serait inaccessible.

---

## Partie 5 — Premiers pas, réseau et accès SSH

### Système, ressources et réseau (dans la VM)

Commandes :

```bash
uname -r
hostnamectl
nproc
free -h
df -h /
lsblk
ip -4 addr show
ip route
resolvectl status | grep "DNS Servers"
ping -c 3 ubuntu.com
```

Sortie :

```text
7.0.0-38-generic
 Static hostname: ubuntu-srv
       Icon name: computer-vm
         Chassis: vm
      Machine ID: eb614d7790db48338ca22544b79eed58
         Boot ID: f0aa7cb103334744949e4a69846fda12
  Virtualization: oracle
Operating System: Ubuntu 26.04 LTS
          Kernel: Linux 7.0.0-38-generic
    Architecture: x86-64
 Hardware Vendor: innotek GmbH
  Hardware Model: VirtualBox
Hardware Version: 1.2
Firmware Version: VirtualBox
   Firmware Date: Fri 2006-12-01
    Firmware Age: 19y 10month 1w 6d
1
               total        used        free      shared  buff/cache   available
Mem:           1,4Gi       333Mi       814Mi       1,1Mi       442Mi       1,1Gi
Swap:          2,0Gi          0B       2,0Gi
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv   12G  5,2G  5,5G  49% /
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0   25G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0   23G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0 11,5G  0 lvm  /
sr0                        11:0    1 1024M  1 rom
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    altname enx0800276266c9
    inet 10.0.2.15/24 metric 100 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 85839sec preferred_lft 85839sec
default via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100
10.0.2.2 dev enp0s3 proto dhcp scope link src 10.0.2.15 metric 100
192.168.1.1 via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100
         DNS Servers: 192.168.1.1 fd17:625c:f037:2::3
PING ubuntu.com (91.189.91.44) 56(84) bytes of data.
64 bytes from elvira.canonical.com (91.189.91.44): icmp_seq=1 ttl=64 time=194 ms
64 bytes from elvira.canonical.com (91.189.91.44): icmp_seq=2 ttl=64 time=217 ms
64 bytes from elvira.canonical.com (91.189.91.44): icmp_seq=3 ttl=64 time=240 ms

--- ubuntu.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 193.748/217.091/240.321/19.013 ms
```

![Système, ressources et réseau de la VM](captures/p5_systeme_reseau.png)

### Serveur SSH, connexion depuis l'hôte et copie d'un fichier

Commandes :

```bash
sudo ss -tlnp | grep :22
systemctl status ssh --no-pager
```

Sortie (dans la VM) :

```text
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/usr/lib/systemd/system/ssh.service; disabled; preset: enabled)
     Active: active (running) since Wed 2026-10-14 23:54:32 UTC; 40s ago
 Invocation: c703e95b8ef949daa97f52ff7e76cc53
TriggeredBy: ● ssh.socket
       Docs: man:sshd(8)
             man:sshd_config(5)
    Process: 18376 ExecStartPre=/usr/sbin/sshd -t (code=exited, status=0/SUCCESS)
   Main PID: 18379 (sshd)
      Tasks: 1 (limit: 944)
     Memory: 1.2M (peak: 1.9M)
        CPU: 109ms
     CGroup: /system.slice/ssh.service
             └─18379 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"

oct. 14 23:54:32 ubuntu-srv systemd[1]: Starting ssh.service - OpenBSD Secure Shell server...
oct. 14 23:54:32 ubuntu-srv sshd[18379]: Server listening on 0.0.0.0 port 22.
oct. 14 23:54:32 ubuntu-srv sshd[18379]: Server listening on :: port 22.
oct. 14 23:54:32 ubuntu-srv systemd[1]: Started ssh.service - OpenBSD Secure Shell server.
LISTEN 0      4096         0.0.0.0:22        0.0.0.0:*    users:(("sshd",pid=18379,fd=3),("systemd",pid=1,fd=222))
LISTEN 0      4096            [::]:22           [::]:*    users:(("sshd",pid=18379,fd=4),("systemd",pid=1,fd=224))
```

Depuis PowerShell sur l'hôte :

```powershell
ssh -p 2222 etudiantufhb@127.0.0.1
"Bonjour depuis l'hôte" | Out-File -Encoding utf8 bonjour.txt
scp -P 2222 .\bonjour.txt etudiantufhb@127.0.0.1:~
ssh -p 2222 etudiantufhb@127.0.0.1 "cat ~/bonjour.txt"
```

Sortie :

```text
etudiantufhb@ubuntu-srv:~$ exit
logout
Connection to 127.0.0.1 closed.
PS C:\Users\kkoua\Downloads> "Bonjour depuis l'hôte" | Out-File -Encoding utf8 bonjour.txt
PS C:\Users\kkoua\Downloads> scp -P 2222 .\bonjour.txt etudiantufhb@127.0.0.1:~
etudiantufhb@127.0.0.1's password:
bonjour.txt                                       100%   27     6.6KB/s   00:00
```

Le fichier `bonjour.txt` (27 octets) est bien copié dans la VM ; il apparaît ensuite dans `ls ~` (partie 6).

![Service SSH actif, connexion par 127.0.0.1:2222 et copie avec scp](captures/p5_ssh_scp.png)

### Réponses aux questions

**1. Relevez l'adresse IP, le masque, la passerelle et le serveur DNS de la VM. À quoi correspond la passerelle 10.0.2.2 ?**
Adresse **10.0.2.15**, masque **/24** (255.255.255.0), passerelle **10.0.2.2**, DNS **192.168.1.1** (sortie de `ip -4 addr show`, `ip route` et `resolvectl`). 10.0.2.2 est le **routeur virtuel du NAT** : tout le trafic vers l'extérieur passe par lui. Le ping vers ubuntu.com ne perd aucun paquet (3 envoyés, 3 reçus).

**2. Pourquoi se connecte-t-on à 127.0.0.1:2222 et non directement à 10.0.2.15:22 ?**
10.0.2.15 n'existe que dans le réseau interne de VirtualBox. Seule la redirection sur la boucle locale de l'hôte atteint le port 22 de la VM.

**3. Que contient le message affiché à la première connexion SSH (empreinte) ? Quel risque permet-il d'éviter ?**
Il affiche l'empreinte de la clé du serveur (ED25519). On la confirme, puis elle est mémorisée dans `known_hosts`. Elle protège contre l'usurpation de serveur (attaque de l'homme du milieu).

**4. Quels avantages offre l'authentification par clé par rapport au mot de passe ? Pourquoi les fournisseurs cloud l'imposent-ils par défaut ?**
La clé privée n'est jamais transmise, il n'y a rien à deviner par force brute, et l'automatisation est possible. Les clouds l'imposent parce que leurs instances sont exposées sur Internet. Le bonus (connexion par clé) n'a pas été réalisé.

---

## Partie 6 — Instantané, clone et arrêt

Toutes les commandes sont lancées dans PowerShell sur l'hôte, sauf indication contraire.

### Instantané, modification, restauration

Commandes :

```powershell
VBoxManage snapshot ubuntu-srv take apres-installation --description "Ubuntu installé, OpenSSH actif"
VBoxManage snapshot ubuntu-srv list
ssh -t -p 2222 etudiantufhb@127.0.0.1 "sudo apt install -y htop; touch ~/avant-restauration.txt; sudo poweroff"
VBoxManage snapshot ubuntu-srv restore apres-installation
VBoxManage startvm ubuntu-srv --type headless
ssh -p 2222 etudiantufhb@127.0.0.1 "which htop; ls ~"
```

Sortie :

```text
PS C:\Users\kkoua\Downloads> VBoxManage snapshot ubuntu-srv take apres-installation --description "Ubuntu installé, OpenSSH actif"
0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
Snapshot taken. UUID: a27e224b-e206-4c85-b305-4a5c9853782c
PS C:\Users\kkoua\Downloads> VBoxManage snapshot ubuntu-srv list
   Name: apres-installation (UUID: a27e224b-e206-4c85-b305-4a5c9853782c) *
   Description: Ubuntu installé, OpenSSH actif
PS C:\Users\kkoua\Downloads> ssh -t -p 2222 etudiantufhb@127.0.0.1 "sudo apt install -y htop; touch ~/avant-restauration.txt; sudo poweroff"
etudiantufhb@127.0.0.1's password:
[sudo: authenticate] Password:
htop est déjà la version la plus récente (3.4.1-5build2).
htop passé en « installé manuellement ».
Les paquets suivants ont été installés automatiquement et ne sont plus nécessaires :
  linux-headers-7.0.0-14                 linux-main-modules-zfs-7.0.0-14-generic  linux-tools-7.0.0-14-generic
  linux-headers-7.0.0-14-generic         linux-modules-7.0.0-14-generic           pollinate
  linux-image-unsigned-7.0.0-14-generic  linux-tools-7.0.0-14
Veuillez utiliser « sudo apt autoremove » pour les supprimer.

Sommaire :
  Mise à niveau de : 0. Installation de : 0, Supprimé : 0. Non mis à jour : 2
Connection to 127.0.0.1 closed.
PS C:\Users\kkoua\Downloads> VBoxManage snapshot ubuntu-srv restore apres-installation
Restoring snapshot 'apres-installation' (a27e224b-e206-4c85-b305-4a5c9853782c)
0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
PS C:\Users\kkoua\Downloads> VBoxManage startvm ubuntu-srv --type headless
Waiting for VM "ubuntu-srv" to power on...
VM "ubuntu-srv" has been successfully started.
PS C:\Users\kkoua\Downloads> ssh -p 2222 etudiantufhb@127.0.0.1 "which htop; ls ~"
etudiantufhb@127.0.0.1's password:
/usr/bin/htop
bonjour.txt
```

Après la restauration, `ls ~` ne montre plus que `bonjour.txt` : le fichier `avant-restauration.txt` a disparu, la VM est revenue à l'état de l'instantané. `htop` est toujours là : il était déjà installé dans l'image (message « htop est déjà la version la plus récente »), l'installation n'avait donc rien changé.

![Instantané pris, modification, restauration](captures/p6_instantane.png)

![Restauration vérifiée : avant-restauration.txt a disparu](captures/p6_restauration.png)

### Clone

Commandes :

```powershell
VBoxManage controlvm ubuntu-srv acpipowerbutton
VBoxManage clonevm ubuntu-srv --name ubuntu-clone --register
VBoxManage modifyvm ubuntu-clone --natpf1 delete ssh
VBoxManage modifyvm ubuntu-clone --natpf1 "ssh,tcp,127.0.0.1,2223,,22"
VBoxManage startvm ubuntu-clone --type headless
ssh -t -p 2223 etudiantufhb@127.0.0.1 "sudo hostnamectl set-hostname ubuntu-clone; hostname"
VBoxManage list runningvms
```

Sortie :

```text
PS C:\Users\kkoua\Downloads> VBoxManage controlvm ubuntu-srv acpipowerbutton
PS C:\Users\kkoua\Downloads> VBoxManage list runningvms
PS C:\Users\kkoua\Downloads> VBoxManage clonevm ubuntu-srv --name ubuntu-clone --register
0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
Machine has been successfully cloned as "ubuntu-clone"
PS C:\Users\kkoua\Downloads> VBoxManage modifyvm ubuntu-clone --natpf1 delete ssh
PS C:\Users\kkoua\Downloads> VBoxManage modifyvm ubuntu-clone --natpf1 "ssh,tcp,127.0.0.1,2223,,22"
PS C:\Users\kkoua\Downloads> VBoxManage startvm ubuntu-clone --type headless
Waiting for VM "ubuntu-clone" to power on...
VM "ubuntu-clone" has been successfully started.
PS C:\Users\kkoua\Downloads> ssh -t -p 2223 etudiantufhb@127.0.0.1 "sudo hostnamectl set-hostname ubuntu-clone; hostname"
kex_exchange_identification: read: Connection aborted
banner exchange: Connection to 127.0.0.1 port 2223: Connection aborted
PS C:\Users\kkoua\Downloads> VBoxManage list runningvms
"ubuntu-clone" {ae86e8a5-31fb-471d-8a07-e4405c73d03b}
PS C:\Users\kkoua\Downloads> ssh -t -p 2223 etudiantufhb@127.0.0.1 "sudo hostnamectl set-hostname ubuntu-clone; hostname"
The authenticity of host '[127.0.0.1]:2223 ([127.0.0.1]:2223)' can't be established.
ED25519 key fingerprint is SHA256:OXs9fsAb5BC51Dy5VMVmA+4p6ORXnSAJ6WsZ/1zpgr8.
This host key is known by the following other names/addresses:
    C:\Users\kkoua/.ssh/known_hosts:1: [127.0.0.1]:2222
Are you sure you want to continue connecting (yes/no/[fingerprint])?
Host key verification failed.
```

Le premier essai échoue (`Connection aborted`) parce que le clone n'a pas fini de démarrer. Au deuxième essai, SSH signale que la clé du clone est **déjà connue** sous `[127.0.0.1]:2222` : le clone a hérité des clés SSH de l'original. Nous n'avions pas répondu `yes` en entier, d'où `Host key verification failed`. Après d'autres essais (`Connection reset` pendant le démarrage), la connexion aboutit :

```text
PS C:\Users\kkoua\Downloads> ssh -t -p 2223 etudiantufhb@127.0.0.1 "sudo hostnamectl set-hostname ubuntu-clone; hostname"
The authenticity of host '[127.0.0.1]:2223 ([127.0.0.1]:2223)' can't be established.
ED25519 key fingerprint is SHA256:OXs9fsAb5BC51Dy5VMVmA+4p6ORXnSAJ6WsZ/1zpgr8.
This host key is known by the following other names/addresses:
    C:\Users\kkoua/.ssh/known_hosts:1: [127.0.0.1]:2222
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[127.0.0.1]:2223' (ED25519) to the list of known hosts.
etudiantufhb@127.0.0.1's password:
[sudo: authenticate] Password:
ubuntu-clone
Connection to 127.0.0.1 closed.
```

Le clone répond sur le port 2223 et son nom d'hôte est maintenant `ubuntu-clone`.

![Clone sur le port 2223, renommé ubuntu-clone](captures/p6_clone.png)

### Place occupée sur l'hôte

Commande :

```powershell
Get-ChildItem "$HOME\VirtualBox VMs" -Directory | ForEach-Object { "{0} : {1:N2} Go" -f $_.Name, ((Get-ChildItem $_.FullName -Recurse -File | Measure-Object Length -Sum).Sum / 1GB) }
```

Sortie :

```text
ubuntu-clone : 7.76 Go
ubuntu-srv : 8.63 Go
```

`ubuntu-srv` occupe plus de place que le clone, car son dossier contient aussi l'instantané.

![Place occupée par le clone et par l'original](captures/p6_place_occupee.png)

### Suppression du clone

Commandes :

```powershell
VBoxManage controlvm ubuntu-clone acpipowerbutton
VBoxManage unregistervm ubuntu-clone --delete
VBoxManage list vms
```

Sortie :

```text
PS C:\Users\kkoua\Downloads> VBoxManage controlvm ubuntu-clone acpipowerbutton
PS C:\Users\kkoua\Downloads> VBoxManage list runningvms
PS C:\Users\kkoua\Downloads> VBoxManage unregistervm ubuntu-clone --delete
0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
PS C:\Users\kkoua\Downloads> VBoxManage list vms
"ubuntu-srv" {6e6a719e-8cf9-403e-8ea0-cd15209e9fb5}
```

Seule `ubuntu-srv` reste enregistrée ; nous la gardons pour les TP suivants.

![Suppression du clone : seule ubuntu-srv est conservée](captures/p6_suppression_clone.png)

### Réponses aux questions

**1. Quelle différence entre un instantané et un clone ? Un instantané est-il une sauvegarde ? Justifiez.**
Un instantané est l'état de la VM à un instant donné ; il dépend du disque d'origine. Un clone est une copie indépendante. Un instantané n'est **pas** une sauvegarde : il est stocké sur le même disque et disparaît avec la VM.

**2. Quels sont les équivalents cloud de l'instantané et du clone ?**
Instantané : le snapshot de disque (EBS, disque managé Azure). Clone : la création d'une image (AMI), puis de nouvelles instances à partir d'elle.

**3. Quelle différence entre `acpipowerbutton` et `poweroff` dans `VBoxManage controlvm` ?**
`acpipowerbutton` demande un arrêt propre au système invité, comme un appui sur le bouton d'alimentation. `poweroff` coupe brutalement, comme débrancher la prise, avec un risque de corruption.

**4. Pourquoi démarrer une VM en mode headless ? Quel lien avec une instance cloud ?**
Sans fenêtre graphique, la VM consomme moins de ressources et s'administre par SSH. C'est exactement le fonctionnement d'une instance cloud.

**5. Pourquoi faut-il changer le port hôte et le nom d'hôte du clone ?**
Deux VM ne peuvent pas utiliser ensemble le port hôte 2222. Le clone hérite aussi du nom d'hôte et des clés SSH de l'original (SSH l'a montré : la clé du clone était déjà connue sous `[127.0.0.1]:2222`) : on le renomme pour éviter les confusions.

---

## Difficultés rencontrées

- `VBoxManage` hors du PATH : le dossier VirtualBox a été ajouté à la session PowerShell (`$env:Path += ";C:\Program Files\Oracle\VirtualBox"`).
- Assistant de création : l'installation sans surveillance a dû être désactivée, et le nom et le type de système corrigés.
- Hyperviseur Windows actif et peu de RAM libre : la VM était très lente, nous l'avons réduite à 1 processeur et 1 536 Mo.
- Touches répétées dans l'installateur : l'utilisateur s'appelle `etudiantufhb` au lieu de `etudiant`, et l'installation a été refaite.
- `sudo` via SSH : il faut l'option `-t` (sinon : « sudo: A terminal is required to authenticate »). Clone : attendre son démarrage (sinon `Connection aborted` ou `Connection reset`) et taper `yes` en entier.
- Durée de l'installation non chronométrée : elle a été estimée d'après les journaux de l'installateur.

## Conclusion

Ce TP nous a fait jouer les deux rôles du cloud IaaS : installer l'hyperviseur, puis créer et administrer une machine virtuelle. Le NAT et la redirection de port montrent pourquoi une instance cloud n'est joignable que par les ports ouverts. L'instantané et le clone montrent ce que les snapshots et les images automatisent. Un chronomètre lancé dès le départ nous aurait évité d'estimer la durée de l'installation après coup.
