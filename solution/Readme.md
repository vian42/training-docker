# Résolution

## Installation de docker

- Sur debian utiliser le job ansible
- Sur archlinux, installer via pacman

```shell
sudo pacman -Sy docker
```

Démarer le daemon docker

```shell
sudo systemctl start docker
```

> La ligne de commande docker correspond au client docker qui va interragir avec une api que l'on appelle le daemon docker

Lancer la commande `docker run hello-world` 


## Build et run des apis

```shell
# Build quote api
docker run --rm -it -v "$(pwd)":/usr/src/quote-api -w /usr/src/quote-api maven:3.8.4-openjdk-17 mvn clean install

# Run quote api
docker run --rm -v "$(pwd)":/usr/src/quote-api -w /usr/src/quote-api -p 8080:4567  openjdk:17 java -jar target/backend-1.0-SNAPSHOT-jar-with-dependencies.jar
```

```shell
# Build quote front
docker run --rm -v "$(pwd)":/usr/src/quote-front -w /usr/src/quote-front  -it node:16.13.2-alpine npm install
docker run --rm -v "$(pwd)":/usr/src/quote-front -w /usr/src/quote-front  -it node:16.13.2-alpine npm run build
```

## Faire une image quoter-api

Voir le dockerfile dans le dossier resources

```shell
docker build -it quoter-api:1.0 .
docker run --rm -it -p 8080:4567 quoter-api:1.0
```

## Faire communiquer le front avec le back via nginx

- Installer nginx et supprimer la conf par défaut

```shell
# Debian
sudo apt install nginx
sudo rm /etc/nginx/sites-enabled/default
# Arch
sudo pacman -Sy nginx
```

---

### Config nginx ArchLinux

Créer le dossier sites-enabled

```shell
sudo mkdir -p /etc/nginx/sites-enabled
```

Backup de la conf nginx

```shell
sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.backup
``` 

Ouvrez le nginx conf (en sudo) et modifier comme suit

- Décommenter la ligne user et renseigner votre user
- Dans le bloc http rajouter `/etc/nginx/sites-enabled/*.conf` 

```nginx
user votre_user;
worker_processes  1;

....

http {

include       /etc/nginx/sites-enabled/*.conf;
include       mime.types;
default_type  application/octet-stream;

```

Redémarez nginx

```shell
sudo systemctl restart nginx
```

---



- Rajouter le host "quote-picker.local" sur la machine

```shell
sudo echo "127.0.0.1    quote-picker.local" >> /etc/hosts
```

- Copier le fichier *server.local.nginx* sur votre machine et complétez le
- Créez un symlink des sites enabled nginx vers la copie de server.local.nginx
- Redémarrer nginx

```shell
sudo ln -s /path/to/server.local.nginx /etc/nginx/sites-enabled/quote-picker.conf
sudo systemctl restart nginx
```
Dans votre navigateur rendez vous sur http://quote-picker.local

## Faire communiquer le conteneur back avec le conteneur front

- Couper le service nginx

```shell
sudo systemctl stop nginx
```

- Copier le fichier *server.docker.nginx* dans le répertoire du front
- Copier le fichier *dockerfile.quote.front* dans le répertoire du front

```shell
docker build -t quoter-front:1.0 .
```

- Installer docker-compose en suivant la doc officielle https://docs.docker.com/compose/install/

- Dans un répertoire différent copier le fichier docker-compose.yml

```shell
docker-compose up
```

Rendez vous à l'adresse http://quote-picker.local
