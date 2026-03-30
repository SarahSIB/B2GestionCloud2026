# TP3B Part1 - Create the base VM

## 1. Intro

🌞 **Créez une VM azure** (une commande `az`)
┌──(kali㉿kali)-[~]
└─$ az vm create --name VM5 --resource-group CLOUD --image "almalinux:almalinux-x86_64:9-gen2:latest" --admin-username sarah --ssh-key-values /ssh/cloud_tp2.pub --public-ip-sku Standard --size Standard_B1s --location denmarkeast
The default value of '--size' will be changed to 'Standard_D2s_v5' from 'Standard_DS1_v2' in a future release.
Consider upgrading security for your workloads using Azure Trusted Launch VMs. To know more about Trusted Launch, please visit https://aka.ms/TrustedLaunch.

  "fqdns": "",
  "id": "/subscriptions/44b9b8fe-92cc-4da7-a30a-49be3a5cae04/resourceGroups/CLOUD/providers/Microsoft.Compute/virtualMachines/VM5",
  "location": "denmarkeast",
  "macAddress": "7C-ED-8D-37-9A-E0",
  "powerState": "VM running",
  "privateIpAddress": "",
  "publicIpAddress": "9.205.154.138",
  "resourceGroup": "CLOUD"

🌞 **Connexion SSH**

- connectez-vous en SSH à la VM

┌──(kali㉿kali)-[~]
└─$ ssh -i ~/.ssh/cloud_tp2 sarah@9.205.154.138
Enter passphrase for key '/home/kali/.ssh/cloud_tp2':
[sarah@VM5 ~]$

# TP3B Part2 - Prepare the VM

On continue le process : dans cette partie on va faire vitefé de la conf et préparer la VM à devenir un template clean.

## 1. Poser notre conf custom

Let's go, connectez-vous en SSH à la VM.

🌞 **Effectuez la conf suivante :**

- mettez le système à jour

[sarah@VM5 ~]$ sudo dnf update -y

- les commandes suivantes doivent être dispos : `htop`, `vim`, `dig`, `ping`

[sarah@VM5 ~]$ sudo dnf install -y htop vim bind-utils
Extra Packages for Enterprise Linux 9 openh264 (From Cisco) - x86_64
Complete!

## 2. Clean la VM

Avant d'éteindre la VM pour en faire un template, on va clean un peu l'environnement.

On a 3 trucs à faire :

- **réinitialiser `cloud-init`** : pour qu'il puisse re-run au prochain boot
- **clean le système** : effacer l'historique de commandes, les logs, etc 
- **réinitialiser `waagent`** : l'agent Azure qui gère la VM

**En gros, on veut simuler une VM qui ne s'est jamais allumée !**

### A. Réinitiliser `cloud-init`

🌞 **Go lancer ça :**

[sarah@VM5 ~]$ sudo cloud-init clean --logs
[sarah@VM5 ~]$ sudo rm -rf /var/lib/cloud/*
[sarah@VM5 ~]$ sudo systemctl enable cloud-init


### B. Clean le système

Allez, vous allez bosser un peu. On veut au minimum :

- clean les logs
- clean l'historique de commande (en dernier évidemment)

🌞 **Proposer une suite de commandes**

-[sarah@VM5 ~]$ sudo cloud-init clean --logs
[sarah@VM5 ~]$ sudo rm -rf /var/lib/cloud/*
[sarah@VM5 ~]$ sudo systemctl enable cloud-init
[sarah@VM5 ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        4.0M     0  4.0M   0% /dev
tmpfs           1.8G     0  1.8G   0% /dev/shm
tmpfs           731M   17M  714M   3% /run
efivarfs        128M   32K  128M   1% /sys/firmware/efi/efivars
/dev/sdb4        29G  1.4G   28G   5% /
/dev/sdb3       960M  281M  680M  30% /boot
/dev/sdb2       200M  7.5M  193M   4% /boot/efi
/dev/sda1       7.8G   28K  7.4G   1% /mnt
tmpfs           366M     0  366M   0% /run/user/1000
[sarah@VM5 ~]$ sudo journalctl --vacuum-time=1s
Vacuuming done, freed 0B of archived journals from /run/log/journal/7346247ceaaf495da23a94c8986b7310.
Vacuuming done, freed 0B of archived journals from /run/log/journal.
Vacuuming done, freed 0B of archived journals from /run/log/journal/ab0c600b350d4ab6a3e74d812751e8f0.
[sarah@VM5 ~]$ sudo find /var/log -type f -exec truncate -s 0 {} +
[sarah@VM5 ~]$ sudo dnf clean all
42 files removed
[sarah@VM5 ~]$ sudo rm -rf /tmp/* /var/tmp/*
[sarah@VM5 ~]$ cat /dev/null > ~/.bash_history && history -c

### C. Réinitaliser l'agent Azure
[sarah@VM5 ~]$ sudo rm -rf /var/lib/waagent/*
[sarah@VM5 ~]$ sudo waagent -deprovision+user -force
WARNING! The waagent service will be stopped.
WARNING! All SSH host key pairs will be deleted.
WARNING! Cached DHCP leases will be deleted.
WARNING! root password will be disabled. You will not be able to login as root.
WARNING! /etc/resolv.conf will be deleted.
WARNING! sarah account and entire home directory will be deleted.
2026-03-24T08:48:00.011015Z INFO MainThread Examine /proc/net/route for primary interface
2026-03-24T08:48:00.011911Z INFO MainThread Primary interface is [eth0]

### D. It's ready

C'est tout bon pour la VM, vous pouvez vous déconnecter de la session SSH.

## Go next

# TP3B Part3 - Create a template


## 1. Créer le template

De retour dans votre shell `az`, sur votre PC, on va créer le template.

🌞 **Let's go, balancez :**
┌──(kali㉿kali)-[~]
└─$ az vm deallocate --resource-group CLOUD --name VM5

┌──(kali㉿kali)-[~]
└─$ az vm generalize --resource-group CLOUD --name VM5

┌──(kali㉿kali)-[~]
└─$ az image create --resource-group CLOUD --name alma_chad --source VM5 --hyper-v-generation V2

## 2. Tester le template

🌞 **Lancer une VM à partir de votre template**

- même commande `vm create` que d'habitude, mais choisissez votre image comme base
- avec un `--image alma_chad` donc !

┌──(kali㉿kali)-[~]
└─$az interactive
az>> vm create -g CLOUD --name VM6 --image alma_chad --ssh-key-values /tmp/ssh//cloud_tp2.pub --location denmarkeast --size Standard_B2ats_v2




🌞 **Vérification !**

┌──(kali㉿kali)-[~]
└─$ ssh -i ~/.ssh/cloud_tp2 nonroot@9.205.156.232
Enter passphrase for key '/home/kali/.ssh/cloud_tp2':
[nonroot@VM6 ~]

[nonroot@VM6 ~]$ cloud-init status
status: done
[nonroot@VM6 ~]$ systemctl status waagent
● waagent.service - Azure Linux Agent
     Loaded: loaded (/usr/lib/systemd/system/waagent.service; enabled; preset: disabled)
     Active: active (running)

## 3. La suite

⚠️⚠️⚠️ **On a créé ce template juste pour vous montrer la démarche pour faire ça.**

On ne se reservira pas de cette image `alma-chad` dans la suite, vous repartirez de l'image Alma officielle.

# TP3B Part4 - Hardened template

## Intro

## Setup ur env

🌞 **Créez une VM qui servira à créer le template**
az vm create --resource-group CLOUD II --name VM8 --image4-gen10::latest --size Standard_B2ats_v2 --location denmarkeast --admin-username useradmin --ssh-key-values tmp/ssh/cloud_tp2.pub

┌──(kali㉿kali)-[~]
└─$ ssh -i ~/.ssh/cloud_tp2 useradmin@9.205.154.171
The authenticity of host '9.205.154.171 (9.205.154.171)' can't be established.
ED25519 key fingerprint is SHA256:UPoufuagVbZQi732q/AftK3jxdcGwvrcN4eYCPj5j9w.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '9.205.154.171' (ED25519) to the list of known hosts.
Enter passphrase for key '/home/kali/.ssh/cloud_tp2':
[useradmin@VM7 ~]$

## Go HARDEN

![Harden](assets/meme/metapod.png)

# TP3B Part4 - A. Firewalling baby

![firewall](assets/meme/firewall.png)

🌞 **Firewall conf**

[useradmin@VM7 ~]$ sudo dnf install -y firewalld
AlmaLinux Kitten 10 - AppStream                               3.2 MB/s | 4.4 MB     00:01
AlmaLinux Kitten 10 - BaseOS                                   29 MB/s |  66 MB     00:02
AlmaLinux Kitten 10 - CRB                                     2.9 MB/s | 942 kB     00:00
AlmaLinux Kitten 10 - Extras packages                          22 kB/s | 7.5 kB     00:00
Dependencies resolved.

[useradmin@VM7 ~]$ sudo systemctl enable --now firewalld
[useradmin@VM7 ~]$ systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
     Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; preset: enabled)
     Active: active (running) since Tue 2026-03-24 13:42:48 UTC; 5s ago
     
[useradmin@VM7 ~]$ sudo firewall-cmd --permanent --add-port=22/tcp
success
[useradmin@VM7 ~]$ sudo firewall-cmd --reload
success

[useradmin@VM7 ~]$ sudo firewall-cmd --permanent --remove-service=ssh
success
[useradmin@VM7 ~]$ sudo firewall-cmd --permanent --remove-service=dhcpv6-client
success
[useradmin@VM7 ~]$ sudo firewall-cmd --permanent --remove-service=cockpit
success
[useradmin@VM7 ~]$ sudo firewall-cmd --reload
success
[useradmin@VM7 ~]$ sudo firewall-cmd --list-all
public (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: eth0
  sources:
  services:
  ports: 22/tcp
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:

└─$ ssh -i ~/.ssh/cloud_tp2 useradmin@9.205.154.171
Enter passphrase for key '/home/kali/.ssh/cloud_tp2':
Last login: Tue Mar 24 13:41:49 2026 from 91.164.176.223


# TP3B Part4 - B. Stronk SSH

OpenSSH c'est l'outil de référence pour l'accès distant sous Linux. Pas trop besoin de le présenter non plus si ?

🌞 **Proposez une conf OpenSSH forte**

[useradmin@VM7 ~]$ sudo vi /etc/ssh/sshd_config
[useradmin@VM7 ~]$ sudo sshd -t
[useradmin@VM7 ~]$ sudo systemctl restart sshd

# TP3B Part4 - C. fail2ban

Histoire de subir un peu moins de tentatives d'intrusions des randoms mass-attacks sur internet.

🌞 **Installer et configurer `fail2ban`**

[useradmin@VM7 ~]$ sudo dnf install fail2ban -y
Extra Packages for Enterprise Linux 10 - x86_64    6.2 MB/s | 6.3 MB     00:01
Last metadata expiration check: 0:00:01 ago on Tue 24 Mar 2026 02:17:21 PM UTC.
Dependencies resolved
[useradmin@VM7 ~]$ sudo vi /etc/fail2ban/jail.local
[useradmin@VM7 ~]$ sudo systemctl enable --now fail2ban
Created symlink '/etc/systemd/system/multi-user.target.wants/fail2ban.service' → '/usr/lib/systemd/system/fail2ban.service'.



🌞 **Prouvez que `fail2ban` fonctionne**
[useradmin@VM7 ~]$ sudo fail2ban-client status sshd
Status for the jail: sshd
|- Filter
|  |- Currently failed: 3
|  |- Total failed:     29
|  `- Journal matches:  _SYSTEMD_UNIT=sshd.service + _COMM=sshd + _COMM=sshd-session
`- Actions
   |- Currently banned: 2
   |- Total banned:     2
   `- Banned IP list:   185.91.69.217 91.164.176.223

   Pour me reconnecter je suis allée dans Opérations --> Executer la commande --> RunShellScript : fail2ban-client unban --all

   j'ai pu me reconnecter en ssh 

   ┌──(kali㉿kali)-[~]
└─$ ssh -i ~/.ssh/cloud_tp2 useradmin@9.205.154.171
Enter passphrase for key '/home/kali/.ssh/cloud_tp2':
Last login: Tue Mar 24 14:43:57 2026 from 91.164.176.223


# TP3B Part4 - D. Harden kernel parameters

## 1. Intro

## 2. Setup

🌞 **Proposer une conf `sysctl`**

- doit améliorer le niveau de sécurité de la machine
- proposer au moins 3 paramètres 
- c'est valide que si vous changez le paramètre par rapport à sa valeur par défaut
- vous devez comprendre les implications de la conf que vous ajoutez, faites vos recherches

---

## Go next

➜ [**Next config : Intrusion Detection System**](4e_ids.md)




