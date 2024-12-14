# Déploiement d'une Application Next.js sur AWS EC2

Ce guide décrit les étapes nécessaires pour déployer une application Next.js sur une instance AWS EC2 en utilisant `pm2` pour gérer le processus et `nginx` comme proxy inverse. Il inclut également la configuration d'un nom de domaine personnalisé et l'utilisation de Let's Encrypt pour HTTPS.

## Étape 1 : Configurer votre Instance EC2

1. **Lancer une Instance EC2 :**
    - Connectez-vous à AWS Management Console.
    - Lancez une nouvelle instance EC2 (Amazon Linux 2 ou Ubuntu).
    - Sélectionnez le type d'instance (t2.micro pour les tests).
    - Configurez les paramètres de votre instance et ajoutez une paire de clés pour SSH.
    - Configurez le groupe de sécurité pour autoriser le trafic HTTP (port 80) et SSH (port 22).

2. **Connectez-vous à votre Instance EC2 :**
   ```bash
   votre instance
## Étape 2 : Installer les Dépendances Nécessaires

1. **Mettre à jour les packages et installer Node.js, npm et pm2 :** 
```bash
  sudo apt update && sudo apt upgrade -y
  curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
  sudo apt-get install -y nodejs
  sudo npm install -g pm2 
```
## Étape 3 : Cloner votre Dépôt et Installer les Dépendances

```bash
  git clone git@github.com:username/repo.git
  cd repo
  npm install
  npm run build
```

## Étape 4 : Démarrer l'Application avec pm2

```bash
  pm2 start npm --name "next-app" -- start
  pm2 startup
  pm2 save
```

## Étape 5 : Configurer Nginx comme Proxy Inverse

1. **Installer Nginx :**
```bash
  sudo apt-get install -y nginx
  sudo nano /etc/nginx/sites-available/default
```
2. **Configurer Nginx :**
```nginx
  server {
    listen 80;
    server_name example.com www.example.com;

    location / {
      proxy_pass http://localhost:3000;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection 'upgrade';
      proxy_set_header Host $host;
      proxy_cache_bypass $http_upgrade;
    }
  }
```
3. **Redémarrez Nginx :**
```bash
  sudo systemctl restart nginx
```

## Étape 6 : Configurer un Nom de Domaine Personnalisé

1. **Acheter un Nom de Domaine :**
    - Achetez un nom de domaine sur un registraire de domaine.
    - Configurez les enregistrements DNS pour pointer vers l'adresse IP de votre instance EC2.
    - Modifiez la configuration Nginx pour utiliser votre nom de domaine.
```nginx
  server {
    listen 80;

    server_name yourdomain.com www.yourdomain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

## Étape 7 : Configurer Let's Encrypt pour HTTPS (Optionnel)

1. **Installer Certbot :**
```bash
  sudo apt-get install -y certbot python3-certbot-nginx
```
2. **Obtenir un Certificat SSL :**
```bash
  sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```
3. **Configurer le Renouvellement Automatique :**
```bash
  sudo certbot renew --dry-run
```
