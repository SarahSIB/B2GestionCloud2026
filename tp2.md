# I. Prérequis

## 1. Starting blocks

## 2. Une paire de clés SSH

### A. Choix de l'algorithme de chiffrement

🌞 **ed25519**
donner une source fiable qui explique pourquoi on évite RSA désormais (pour les connexions SSH notamment) :  source ANSII : https://ninjalinux.com/2025/09/05/durcir-la-configuration-ssh-avec-les-bonnes-pratiques-anssi/#:~:text=Utiliser%20une%20paire%20de%20cl%C3%A9s,%C3%A0%20d%C3%A9faut%2C%20RSA%204096%20bits.&text=Le%20%2Da%20100%20renforce%20la,de%20s%C3%A9curit%C3%A9%20c%C3%B4t%C3%A9%20brute%20force.
ou https://messervices.cyber.gouv.fr/documents-guides/anssi-guide-recommandations_de_securite_relatives_a_tls-v1.2.pdf

### B. Génération de votre paire de clés

🌞 **Générer une paire de clés pour ce TP**
└─$ ssh -keygen -t ed25519 -f ~/.ssh/cloud_tp2
Bad escape character 'ygen'.

┌──(kali㉿kali)-[~]
└─$ ssh-keygen -t ed25519 -f ~/.ssh/cloud_tp2
Generating public/private ed25519 key pair.

┌──(kali㉿kali)-[~]
└─$ ls -l ~/.ssh
total 16
-rw------- 1 kali kali  444 Mar 23 06:23 cloud_tp2
-rw-r--r-- 1 kali kali   91 Mar 23 06:23 cloud_tp2.pub

(Désolée je l'ai appelé tp2)

### C. Agent SSH



🌞 **Configurer un agent SSH sur votre poste**
┌──(kali㉿kali)-[~]
└─$ eval "$(ssh-agent -s)"
Agent pid 42687

┌──(kali㉿kali)-[~]
└─$ ssh-add ~/.ssh/cloud_tp2
Enter passphrase for /home/kali/.ssh/cloud_tp2:
Identity added: /home/kali/.ssh/cloud_tp2 (kali@kali)

![Logo OpenSSH](../../assets/img/logo_openssh.png)

*[Agent SSH]: Un programme qui tourne en fond, auquel on peut ajouter nos clés SSH, qui seront ensuite utilisées automatiquement à chaque connexion SSH (sans préciser leur chemin ou le password pour les déverrouiller).
*[Resource Group] : Prérequis absolu pour créer des trucs dans Azure. Toute ressource sera ensuite attachée à un Resource Group. C'est lui qui détermine dans quelle datacenter sont physiquement placées les ressources. **Je vous recommande `uksouth` cpacher.**
*[RSA]: [RSA](https://en.wikipedia.org/wiki/RSA_cryptosystem) est un algorithme de chiffrement très largement répandu qui tend à être abandonné
*[Subscription ID]: L'identifiant unique de votre abonnement Azure for Students. Vous pouvez le récupérer depuis la WebUI ou avec une commande `az account show --query id`.
*[WebUI Azure]: Aussi appelé le "Portal". Là quoi : https://portal.azure.com

# II. Spawn des VMs


## 1. Depuis la WebUI

➜ **Faites du cliclic partout dans la WebUI Azure pour créer une VM dans Azure.**


🌞 **Connectez-vous en SSH à la VM pour preuve**

┌──(kali㉿kali)-[~]
└─$ ssh sarah@9.205.16.10

The authenticity of host '9.205.16.10 (9.205.16.10)' can't be established.
ED25519 key fingerprint is SHA256:Ch3rNQUmKLNBYDOw7p8Ygw5Cf+5LWxR7KvJvfPBSDVg.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? y
Please type 'yes', 'no' or the fingerprint: yes
Warning: Permanently added '9.205.16.10' (ED25519) to the list of known hosts.


sarah@VM1:~$

## 2. `az` : a programmatic approach

🌞 **Créez une VM depuis le Azure CLI**
vm create --name VM2 --resource-group CLOUD --location denmarkeast --size Standard_B2s_v2 --image almalinux:almalinux-x86_64:10
--admin-username sarah --ssh-key-values /tmp/ssh/cloud_tp2.pub
Consider upgrading security for your workloads using Azure Trusted Launch VMs. To know more about Trusted Launch, please visit https://aka.ms/TrustedLaunch.
{
  "fqdns": "",
  "id": "/subscriptions/44b9b8fe-92cc-4da7-a30a-49be3a5cae04/resourceGroups/CLOUD/providers/Microsoft.Compute/virtualMachines/VM2",
  "location": "denmarkeast",
  "macAddress": "7C-ED-8D-6A-87-98",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.5",
  "publicIpAddress": "9.205.154.129",
  "resourceGroup": "CLOUD"
}
az>>
    - comme ça, dès que la VM pop, on peut se co en SSH !

🌞 **Assurez-vous que vous pouvez vous connecter à la VM en SSH sur son IP publique**
──(kali㉿kali)-[~]
└─$ ssh sarah@9.205.154.129
The authenticity of host '9.205.154.129 (9.205.154.129)' can't be established.
ED25519 key fingerprint is SHA256:40Jh9SbcHLOAfHpglHYTD0A5Kt/bbgOqeK0+ArpxkDg.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
[sarah@VM2 ~]$


🌞 **Une fois connecté, prouvez la présence...**

- **...du service `waagent.service`**
[sarah@VM2 ~]$ systemctl status waagent.service
● waagent.service - Azure Linux Agent
     Loaded: loaded (/usr/lib/systemd/system/waagent.service; enabled; preset: enabled)
     Active: active (running) since Mon 2026-03-23 13:12:37 UTC; 7min ago


- **...du service `cloud-init.service`**
  
  [sarah@VM2 ~]$ systemctl status cloud-init.service
● cloud-init.service - Cloud-init: Network Stage
     Loaded: loaded (/usr/lib/systemd/system/cloud-init.service; enabled; preset: enabled)
     Active: active (exited) since Mon 2026-03-23 13:12:37 UTC; 8min ago

[sarah@VM2 ~]$ cloud-init status
status: done



## 3. Terraforming ~~planets~~ infrastructures

**Une dernière section pour jouer avec Terraform,** on se contente là encore de simplement créer une VM Azure.



🌞 **Utilisez Terraform pour créer une VM dans Azure**

- ┌──(kali㉿kali)-[~]
└─$ mkdir -p ~/terraform-VM3

┌──(kali㉿kali)-[~]
└─$ cd ~/terraform-VM3

┌──(kali㉿kali)-[~/terraform-VM3]
└─$
???+ note

    Vous pouvez couper un peu l'ouput de votre `terraform apply` pour le compte-rendu, il est immense :d

📁 **Fichiers à rendre**

- `main.tf`
- tout autre fichier utilisé par Terraform (je vous propose des fichiers de base plus bas)

┌──(kali㉿kali)-[~/terraform-VM3]
└─$ nano main.tf



┌──(kali㉿kali)-[~/terraform-VM3]
└─$ nano variables.tf


┌──(kali㉿kali)-[~/terraform-VM3]
└─$ nano terraform.tfvars


┌──(kali㉿kali)-[~/terraform-VM3]
└─$ terraform init
Initializing the backend...
Initializing provider plugins...
- Reusing previous version of hashicorp/azurerm from the dependency lock file
- Using previously-installed hashicorp/azurerm v4.65.0

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.

┌──(kali㉿kali)-[~/terraform-VM3]
└─$ terraform apply
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.


🌞 **Prouvez avec une connexion SSH sur l'IP publique que la VM est up**

┌──(kali㉿kali)-[~]
└─$ eval $(ssh-agent)
Agent pid 175167

┌──(kali㉿kali)-[~]
└─$ ssh-add ~/.ssh/cloud_tp2
Enter passphrase for /home/kali/.ssh/cloud_tp2:
Identity added: /home/kali/.ssh/cloud_tp2 (kali@kali)

┌──(kali㉿kali)-[~]
└─$ ssh-add ~/.ssh/cloud_tp2

┌──(kali㉿kali)-[~]
└─$ ssh sarah@9.205.156.118

1 device has a firmware upgrade available.
Run `fwupdmgr get-upgrades` for more information.

Last login: Mon Mar 23 15:14:48 2026 from 159.117.224.16
[sarah@super-vm ~]$

sss