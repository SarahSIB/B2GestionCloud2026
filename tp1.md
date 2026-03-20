# Part I : Docker basics

## Sommaire

## 1. Install

🌞 **Installer Docker votre machine Azure**
 sudo apt update
 sudo apt install -y docker.io
 sudo systemctl enable docker --now
 sudo usermod -aG docker $(whoami)
 xfce4-session-logout --logout
 newgrp docker
groups
docker adm dialout cdrom floppy sudo audio dip video plugdev users netdev scanner bluetooth lpadmin wireshark kaboxer vboxsf kali
![Docker](./img/docker_win.png)

## 2. Vérifier l'install

➜ **Vérifiez que Docker est actif est disponible en essayant quelques commandes usuelles :**
docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

┌──(kali㉿kali)-[~]
└─$ docker ps -a
CONTAINER ID   IMAGE         COMMAND    CREATED          STATUS                      PORTS     NAMES
8dbaaea6adc5   hello-world   "/hello"   27 minutes ago   Exited (0) 27 minutes ago             gallant_feistel

┌──(kali㉿kali)-[~]
└─$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    1b44b5a3e06a   7 months ago   10.1kB


## 3. Lancement de conteneurs

La commande pour lancer des conteneurs est `docker run`.

Certaines options sont très souvent utilisées :

```bash
# L'option --name permet de définir un nom pour le conteneur
$ docker run --name web nginx

# L'option -d permet de lancer un conteneur en tâche de fond
$ docker run --name web -d nginx

# L'option -v permet de partager un dossier/un fichier entre l'hôte et le conteneur
$ docker run --name web -d -v /path/to/html:/usr/share/nginx/html nginx

# L'option -p permet de partager un port entre l'hôte et le conteneur
$ docker run --name web -d -v /path/to/html:/usr/share/nginx/html -p 8888:80 nginx
# Dans l'exemple ci-dessus, le port 8888 de l'hôte est partagé vers le port 80 du conteneur
```

🌞 **Utiliser la commande `docker run`** lancer un conteneur `nginx` conf par défaut étou étou, simple pour le moment par défaut il écoute sur le port 80 et propose une page d'accueil le conteneur doit être lancé avec un partage de port le port 9999 de la machine hôte doit rediriger vers le port 80 du conteneur

(kali㉿kali)-[~]
└─$ docker run --name web -d -p 9999:80 nginx

🌞 **Rendre le service dispo sur internet**
┌──(kali㉿kali)-[~]
└─$ sudo ufw allow 9999/tcp
Rules updated
Rules updated (v6)

sudo ufw status
Status: inactive

┌──(kali㉿kali)-[~]
└─$ sudo ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup

┌──(kali㉿kali)-[~]
└─$ sudo ufw status
Status: active

hostname -I
10.100.0.75 192.168.10.108 172.17.0.1


┌──(kali㉿kali)-[~]
└─$ curl -I http://10.100.0.75:9999                                                                  HTTP/1.1 200 OK
Server: nginx/1.29.6
Date: Thu, 19 Mar 2026 10:03:09 GMT
Content-Type: text/html
Content-Length: 896
Last-Modified: Tue, 10 Mar 2026 15:29:07 GMT
Connection: keep-alive
ETag: "69b038c3-380"
Accept-Ranges: bytes

🌞 **Custom un peu le lancement du conteneur**
(kali㉿kali)-[~]
└─$ mkdir ~/tp_docker && cd ~/tp_docker

┌──(kali㉿kali)-[~/tp_docker]
└─$ echo "h<1>Meow</h1>" > index.html

┌──(kali㉿kali)-[~/tp_docker]
└─$ echo "<h1>Meow</h1>" > index.html

┌──(kali㉿kali)-[~/tp_docker]
└─$ cat <<EOF > custom_meow.conf
server {
    listen 7777;
    root /var/www/tp_docker;
    index index.html;
}
EOF

┌──(kali㉿kali)-[~/tp_docker]
└─$ docker run -d \
  --name meow \
  -p 7777:7777 \
  -m 512m \
  -v $(pwd)/index.html:/var/www/tp_docker/index.html:ro \
  -v $(pwd)/custom_meow.conf:/etc/nginx/conf.d/meow.conf:ro \
  nginx
31ec5a4db02e95e300379a0b4c42292dc0cc32f8d45f7f0f7979bc146046a875

🌞 **Call me**

http://10.100.0.75:7777/
![Cloud Linux](./img/cloud_linux.png)



# Part II : Images


## Sommaire


## Exemple de Dockerfile et utilisation



🌞 **Construire votre propre image**
┌──(kali㉿kali)-[~/tp_image]
└─$ ls
index.html

┌──(kali㉿kali)-[~/tp_image]
└─$ nano Dockerfile

┌──(kali㉿kali)-[~/tp_image]
└─$ nano apache2.conf

┌──(kali㉿kali)-[~/tp_image]
└─$ docker build . -t my_apache_nano
[+] Building 1.7s (9/9) FINISHED

┌──(kali㉿kali)-[~/tp_image]
└─$ docker run -d -p 8080:80 --name mon_serveur_apache my_apache_nano
e50a3e109bd15343f582bcebbced9e17ac9ab38ee3a9bf9055b08a49226bf968

└─$ docker ps
CONTAINER ID   IMAGE            COMMAND                  CREATED             STATUS             PORTS                                               NAMES
e50a3e109bd1   my_apache_nano   "/usr/sbin/apache2ct…"   10 seconds ago      Up 9 seconds       0.0.0.0:8080->80/tcp, [::]:8080->80/tcp

┌──(kali㉿kali)-[~/tp_image]
└─$ curl localhost:8080
<h1> Part 2 </h1>


🌞 **Dans les deux cas, j'attends juste votre `Dockerfile` dans le compte-rendu**
GNU nano 8.4                                      Dockerfile
FROM debian:bookworm-slim
RUN apt update -y && apt install -y apache2 && apt clean
COPY apache2.conf /etc/apache2/apache2.conf
COPY index.html /var/www/html/index.html
EXPOSE 80
CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]


# Part III : `docker-compose`

## Sommaire

- [Part III : `docker-compose`]

## 1. Intro


## 2. WikiJS

🌞 **Installez un WikiJS** en utilisant Docker
sudo nmcli con down "Wired connection 1"
sudo nmcli con up "Wired connection 1"
┌──(kali㉿kali)-[~]
└─$ sudo apt install docker-compose -y
┌──(kali㉿kali)-[~]
└─$ docker-compose --version
Docker Compose version 2.32.4-3
┌──(kali㉿kali)-[~]
└─$ mkdir wikijs

┌──(kali㉿kali)-[~]
└─$ cd wikijs

┌──(kali㉿kali)-[~/wikijs]
└─$ nano docker-compose.ym

┌──(kali㉿kali)-[~/wikijs]
└─$ docker compose up -d

┌┌──(kali㉿kali)-[~/wikijs]
└─$ docker ps
CONTAINER ID   IMAGE                     COMMAND                  CREATED        STATUS         PORTS                                               NAMES
1a9d15bf22b8   ghcr.io/requarks/wiki:2   "docker-entrypoint.s…"   19 hours ago   Up 2 seconds   3443/tcp, 0.0.0.0:80->3000/tcp, [::]:80->3000/tcp   wikijs-wikijs-1
7b8d22df3127   postgres:15               "docker-entrypoint.s…"   19 hours ago   Up 3 seconds   5432/tcp                                            wikijs-db-1
e50a3e109bd1   my_apache_nano            "/usr/sbin/apache2ct…"   20 hours ago   Up 20 hours    0.0.0.0:8080->80/tcp, [::]:8080->80/tcp             mon_serveur_apache
31ec5a4db02e   nginx                     "/docker-entrypoint.…"   22 hours ago   Up 22 hours    80/tcp, 0.0.0.0:7777->7777/tcp, :::7777->7777/tcp   meow

curl http://10.100.0.75
<!DOCTYPE html><html lang="en"><head><meta http-equiv="X-UA-Compatible" content="IE=edge"><meta charset="UTF-8"><meta name="viewport" content="user-scalable=yes, width=device-width, initial-scale=1, maximum-scale=5"><meta name="theme-color

🌞 **Call me** when it's done
http://10.100.0.75

## 3. Make your own meow

🌞 **Vous devez :**

- construire une image qui
  - contient `python3`
  - contient l'application et ses dépendances
  - lance l'application au démarrage du conteneur
- écrire un `docker-compose.yml` qui définit le lancement de deux conteneurs :
  - l'app python
  - le Redis dont il a besoin