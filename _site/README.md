# Atelier Déploiement Nginx avec Node JS

## 1. Objectifs

Apprendre à déployer un site sous NodeJS avec Nginx

- Nginx et son reverse proxy
- Git
- NodeJS (React et Express)
- PM2
- SSL et let's encrypt

## 2. Pré requis

- Connaissance minimal en terminal Bash (Se déplacer dans les dossier, afficher son arborescence, créer un fichier, éditer un fichier, ...)
- Connexion en ssh au serveur
- Connaitre git, React et express

## 3. Explication et context

### 3.1 Le serveur (machine physique) est souvent dédié à un service :

- Il peut recevoir et traité plusieurs requêtes en même temps.
- Il est spécialisé dans certaines tâches précises (stockage, calcul, hébergement).
- Il fonctionne en continu, 24h/24, 7j/7.
- Il est rarement manipulé directement et fonctionne souvent sans interface graphique (Connexion SSH).
- Sa puissance est optimisée pour gérer de multiples requêtes en même temps.

### 3.2 Le serveur web (Logiciel)

Pour fonctionner sur le web, un serveur a besoin d'un Serveur Web (Logiciel). Ce logiciel permet d'écouter les ports, par défaut pour le web :

- Port 80 pour HTTP (Non sécurisé)
- Port 443 pour HTTPS (Sécurisé).
  Si une demande (requête) arrive sur un port, le serveur web va prendre en charge la demande et y répondre en fonction des instructions que nous développeurs lui auront donnés

Les 2 serveurs web les plus courants sont Nginx et Apache. Leurs grands missions sont donc :

- Distribuer des pages web : Il reçoit les requêtes HTTP/HTTPS des navigateurs et renvoie les fichiers HTML, CSS, JavaScript et médias demandés.
- Équilibrage de charge (Load Balancing) : Il peut répartir le trafic entre plusieurs serveurs d'application pour optimiser les performances et assurer une haute disponibilité.
- Reverse Proxy : Il agit comme intermédiaire entre les clients et vos serveurs d'applications (comme Node.js, PHP, Ruby), redirigeant les requêtes vers le bon service en arrière-plan.
- Mise en cache : Il stocke temporairement des copies des ressources fréquemment demandées pour accélérer les temps de réponse.
- Sécurité : Il filtre les requêtes malveillantes, limite les taux de connexion (rate limiting), et gère les certificats SSL/TLS pour les connexions sécurisées.
- Compression : Il peut compresser les données avant de les envoyer pour réduire la bande passante utilisée.
- Servir du contenu statique : Il est particulièrement efficace pour délivrer rapidement des fichiers qui ne changent pas (images, CSS, etc.).

Nous allons donc ajouter Nginx à notre serveur. Nginx est apprécié pour sa légèreté, sa rapidité et sa capacité à gérer un grand nombre de connexions simultanées, ce qui en fait un choix populaire pour les sites à fort trafic.

## 4. Installation des outils

Sur linux, avant toute installation, nous allons prendre le reflexe de mettre à jour notre dictionnaire `apt`. C'est un peu la liste des packages possibles pour Linux/Ubuntu avec leur version.

```bash
sudo apt update
```

Puis nous allons, effectivement mettre à jour les paquets à partir de ce nouveau dictionnaire.

```bash
sudo apt upgrade
```

### 4.1 Git

Le premier utilitaire que nous allons avoir besoin vous est bien connu : **GIT**. IL va nous permettre, un peu plus tard dans l'atelier de récupérer notre code sur notre repo GitHub

```bash
sudo apt install git
```

Puis vérifie que la CLI est correctement installée.

```bash
git --version
```

### 4.2 Node

Le deuxième utilitaire à installer est notre runtime: **NodeJS** et son meilleur ami, notre gestionnaire de package.
:warning: pour optimiser les ressources serveurs et éviter les doublons de packages npm, nous pourrions également installer `pnpm`.

```bash
sudo apt install nodejs npm
```

Puis vérifions que l'installation s'est bien passée

```bash
node --version
npm --version
```

### 4.3 PM2

PM2 est un gestionnaire de processus qui va nous aider à gérer et à garder notre application en ligne. Si nous lançons un processus node (npm run dev) dans notre terminal, celui ci sera coupé, lorsque nous nous déconnecterons. PM2 va nous permettre de lancer ces processus en mode détaché (en arrière plan).

PM2 offre beaucoup d'autres possibilités également dans la gestion de nos logs, nos ressources [plus d'infos](https://pm2.keymetrics.io/docs/usage/quick-start/)

```bash
npm install pm2@latest -g
```

Ok, je crois que nous sommes bon pour les installation de base...
Il nous restera **Let's encrypt**, mais plus tard...

## 5 Configuration du serveur web

### 5.1 Nginx

Pour commencer, nous allons maintenant installer notre logiciel de serveur Web Nginx.

```
sudo apt install nginx
```

Test avec Bruno... Attention en mode http, bloquer par les navigateurs

Arborescence de fichier : d'où vient cette page...
var/www/html

Y aller et cloner une page web. en https et sudo

- navigation dans l'arborescence via l'URL (Attention: faille de sécurité)

Configuration etc/

/etc/nginx/sites-available/default

    # Empêcher l'accès à la racine www/html sauf pour /memorize
    location / {
        root /var/www/html;
        deny all; # Bloque l'accès à tout sauf aux exceptions ci-dessous
    }

sudo systemctl reload nginx

Test... Plus accès à rien

### 5.2 Créer une route pour accéder à cette ressource particulière

location /memorize {
alias /var/www/html/Memorize;
try_files $uri $uri/ =404;
autoindex on; # active la navigation dans les dossiers enfants
}

Test... Accès à memorize
Créer un dossier test avec un index.html basique à l'intérieur au niveau de var/www/html...
Test

### 5.3 Passons au reverse proxy

- Dans home
  Clone ton projet fullstack et gère à la main les installations, .env, ....

- Configure ton projet
  Run les npm install
  Rempli les variables d'env. (Vide pour l'url du client car on appelle la racine puis /api)

**Pour le server**
cd api
pm2 start "npm run dev" --name server

**Pour le client**
npm run build
pm2 start "npm run preview" --name client (Port 4173)

## 6. Let'encrypt et le certificat SSL

https://help.ovhcloud.com/csm/en-vps-install-ssl-certificate?id=kb_article_view&sysparm_article=KB0066249
jusqu'à la fin de la step 3.
