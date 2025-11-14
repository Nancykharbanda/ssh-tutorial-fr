# 💼 Exemples pratiques SSH

Ce document présente des cas d'usage concrets et des scénarios réels d'utilisation de SSH.

## 📑 Table des matières

- [Développement Web](#-d%C3%A9veloppement-web)
- [Administration système](#%EF%B8%8F-administration-syst%C3%A8me)
- [DevOps et automatisation](#-devops-et-automatisation)
- [Sécurité et tunneling](#-s%C3%A9curit%C3%A9-et-tunneling)
- [Gestion de bases de données](#%EF%B8%8F-gestion-de-bases-de-donn%C3%A9es)
- [Développement avec Git](#-d%C3%A9veloppement-avec-git)

---

## 🌐 Développement Web

### Déploiement d'une application web

```bash
#!/bin/bash
# Script de déploiement automatique

SERVER="prod"
APP_DIR="/var/www/monapp"
BACKUP_DIR="/var/backups/monapp"

# Créer une sauvegarde
ssh $SERVER "sudo tar -czf $BACKUP_DIR/backup_$(date +%Y%m%d_%H%M%S).tar.gz $APP_DIR"

# Mettre à jour le code
ssh $SERVER "cd $APP_DIR && git pull origin main"

# Installer les dépendances
ssh $SERVER "cd $APP_DIR && npm install --production"

# Redémarrer le service
ssh $SERVER "sudo systemctl restart monapp"

# Vérifier le statut
ssh $SERVER "sudo systemctl status monapp"

echo "Déploiement terminé !"
```

### Accéder à un site en développement local depuis l'extérieur

```bash
# Sur votre machine locale (port 3000 = votre app)
ssh -R 8080:localhost:3000 utilisateur@serveur-public.com

# Votre application est maintenant accessible via:
# http://serveur-public.com:8080
```

### Synchroniser des fichiers en développement

```bash
# Observer et synchroniser automatiquement
# Installation de fswatch (macOS/Linux)
brew install fswatch  # macOS
sudo apt install inotify-tools  # Linux

# Script de synchronisation automatique
fswatch -o ./src | while read change; do
    rsync -avz -e "ssh -p 2222" ./src/ user@dev:/var/www/app/src/
    echo "Fichiers synchronisés: $(date)"
done
```

---

## ⚙️ Administration système

### Surveillance de serveurs multiples

```bash
#!/bin/bash
# Vérifier l'espace disque sur plusieurs serveurs

SERVERS=("prod" "dev" "staging")

for server in "${SERVERS[@]}"; do
    echo "=== $server ==="
    ssh $server "df -h | grep -E '^/dev/'" || echo "Erreur de connexion"
    echo ""
done
```

### Mise à jour de plusieurs serveurs

```bash
#!/bin/bash
# Script de mise à jour multi-serveurs

SERVERS=("server1" "server2" "server3")

for server in "${SERVERS[@]}"; do
    echo "Mise à jour de $server..."
    ssh $server "sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y"
    echo "$server mis à jour avec succès"
done
```

### Collecter des informations système

```bash
#!/bin/bash
# Rapport système automatique

SERVER="prod"
REPORT_DIR="./reports"
REPORT_FILE="$REPORT_DIR/report_$(date +%Y%m%d).txt"

mkdir -p $REPORT_DIR

{
    echo "=== RAPPORT SYSTÈME - $(date) ==="
    echo ""
    echo "=== Informations OS ==="
    ssh $SERVER "cat /etc/os-release"
    echo ""
    echo "=== Uptime ==="
    ssh $SERVER "uptime"
    echo ""
    echo "=== Utilisation disque ==="
    ssh $SERVER "df -h"
    echo ""
    echo "=== Utilisation mémoire ==="
    ssh $SERVER "free -h"
    echo ""
    echo "=== Charge système ==="
    ssh $SERVER "top -bn1 | head -20"
    echo ""
    echo "=== Services actifs ==="
    ssh $SERVER "systemctl list-units --type=service --state=running"
} > $REPORT_FILE

echo "Rapport généré: $REPORT_FILE"
```

### Backup automatique

```bash
#!/bin/bash
# Script de backup via SSH

SOURCE_SERVER="prod"
SOURCE_DIR="/var/www"
BACKUP_DIR="./backups"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/backup_$DATE.tar.gz"

mkdir -p $BACKUP_DIR

# Créer l'archive sur le serveur et la télécharger
ssh $SOURCE_SERVER "tar -czf - $SOURCE_DIR" > $BACKUP_FILE

# Vérifier la réussite
if [ $? -eq 0 ]; then
    echo "Backup réussi: $BACKUP_FILE"
    # Supprimer les backups de plus de 7 jours
    find $BACKUP_DIR -name "backup_*.tar.gz" -mtime +7 -delete
else
    echo "Erreur lors du backup"
    exit 1
fi
```

---

## 🚀 DevOps et automatisation

### Déploiement avec Git hooks

```bash
# Sur le serveur de production
# Fichier: /var/repo/monapp.git/hooks/post-receive

#!/bin/bash
TARGET="/var/www/monapp"
GIT_DIR="/var/repo/monapp.git"
BRANCH="main"

while read oldrev newrev ref
do
    # Vérifier la branche
    if [[ $ref =~ .*/main$ ]]; then
        echo "Déploiement de la branche main..."
        git --work-tree=$TARGET --git-dir=$GIT_DIR checkout -f $BRANCH
        cd $TARGET
        npm install --production
        sudo systemctl restart monapp
        echo "Déploiement terminé!"
    fi
done
```

### Exécution de commandes sur plusieurs serveurs

```bash
#!/bin/bash
# Exécuter une commande sur plusieurs serveurs

SERVERS=("web1" "web2" "web3")
COMMAND="sudo systemctl restart nginx"

for server in "${SERVERS[@]}"; do
    echo "Exécution sur $server..."
    ssh -o ConnectTimeout=5 $server "$COMMAND" &
done

wait
echo "Commande exécutée sur tous les serveurs"
```

### Monitoring avec notification

```bash
#!/bin/bash
# Vérifier la disponibilité d'un service

SERVER="prod"
SERVICE="nginx"
EMAIL="admin@example.com"

# Vérifier le service
if ! ssh $SERVER "sudo systemctl is-active $SERVICE" &>/dev/null; then
    # Envoyer une alerte
    echo "Le service $SERVICE est DOWN sur $SERVER" | mail -s "ALERTE: Service DOWN" $EMAIL

    # Tenter un redémarrage
    ssh $SERVER "sudo systemctl restart $SERVICE"

    echo "Alerte envoyée et redémarrage tenté"
fi
```

---

## 🔒 Sécurité et tunneling

### Naviguer en sécurité via un proxy SOCKS

```bash
# Créer un tunnel SOCKS
ssh -D 1080 -C -N utilisateur@serveur-securise.com

# Configuration Firefox:
# Préférences → Paramètres réseau → Configuration manuelle du proxy
# SOCKS v5: localhost, Port: 1080
# ✓ Proxy DNS lors de l'utilisation de SOCKS v5
```

### Accéder à un réseau d'entreprise

```bash
# Configuration dans ~/.ssh/config

Host bastion
    HostName bastion.entreprise.com
    User moi
    Port 22
    IdentityFile ~/.ssh/entreprise_key

Host serveur-interne
    HostName 10.0.0.50
    User admin
    ProxyJump bastion
    IdentityFile ~/.ssh/interne_key

# Connexion simplifiée
ssh serveur-interne
```

### Port forwarding pour accéder à des services internes

```bash
# Accéder à plusieurs services internes
ssh -L 8080:serveur-web-interne:80 \
    -L 3306:serveur-db-interne:3306 \
    -L 6379:serveur-redis-interne:6379 \
    utilisateur@bastion.com

# Maintenant accessible localement:
# http://localhost:8080 → Serveur web
# localhost:3306 → MySQL
# localhost:6379 → Redis
```

---

## 🗄️ Gestion de bases de données

### Backup de base de données MySQL/MariaDB

```bash
#!/bin/bash
# Backup MySQL via SSH

DB_SERVER="prod"
DB_NAME="ma_base"
DB_USER="backup_user"
BACKUP_DIR="./db_backups"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p $BACKUP_DIR

# Dump la base de données et la télécharger
ssh $DB_SERVER "mysqldump -u $DB_USER -p $DB_NAME | gzip" > "$BACKUP_DIR/${DB_NAME}_$DATE.sql.gz"

echo "Backup terminé: $BACKUP_DIR/${DB_NAME}_$DATE.sql.gz"
```

### Tunnel SSH vers MySQL

```bash
# Créer le tunnel (dans un terminal)
ssh -L 3306:localhost:3306 -N utilisateur@serveur-db.com

# Se connecter à la base (dans un autre terminal)
mysql -h 127.0.0.1 -P 3306 -u utilisateur -p ma_base

# Ou avec une GUI (MySQL Workbench, DBeaver, etc.)
# Host: 127.0.0.1
# Port: 3306
```

### Restauration de base de données

```bash
#!/bin/bash
# Restaurer une base de données MySQL

DB_SERVER="dev"
DB_NAME="ma_base"
DB_USER="root"
BACKUP_FILE="./db_backups/ma_base_20250114.sql.gz"

# Envoyer et restaurer
zcat $BACKUP_FILE | ssh $DB_SERVER "mysql -u $DB_USER -p $DB_NAME"

echo "Restauration terminée"
```

### Tunnel pour PostgreSQL

```bash
# Tunnel SSH vers PostgreSQL
ssh -L 5432:localhost:5432 -N utilisateur@serveur-pg.com

# Connexion avec psql
psql -h localhost -p 5432 -U postgres -d ma_base

# Connexion avec GUI (pgAdmin, DBeaver, etc.)
# Host: localhost
# Port: 5432
```

---

## 🐙 Développement avec Git

### Configuration multi-comptes GitHub

```bash
# Dans ~/.ssh/config

Host github-perso
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_perso_ed25519

Host github-travail
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_travail_ed25519

# Cloner avec le compte perso
git clone git@github-perso:utilisateur/repo.git

# Cloner avec le compte travail
git clone git@github-travail:entreprise/repo.git

# Dans un dépôt existant, modifier le remote
git remote set-url origin git@github-travail:entreprise/repo.git
```

### Déploiement automatique via Git

```bash
# Sur votre machine locale
# Ajouter le serveur comme remote

git remote add production ssh://utilisateur@serveur.com/var/repo/monapp.git

# Déployer
git push production main

# Le hook post-receive sur le serveur s'occupe du déploiement
```

### Synchroniser un fork avec l'original

```bash
# Ajouter le dépôt original
git remote add upstream git@github.com:original/repo.git

# Récupérer les modifications
git fetch upstream

# Fusionner
git checkout main
git merge upstream/main

# Pousser vers votre fork
git push origin main
```

---

## 🎯 Cas d'usage avancés

### Jump Host avec double saut

```bash
# Configuration ~/.ssh/config

Host bastion1
    HostName bastion1.com
    User jumpuser1
    IdentityFile ~/.ssh/bastion1_key

Host bastion2
    HostName 10.0.0.10
    User jumpuser2
    ProxyJump bastion1
    IdentityFile ~/.ssh/bastion2_key

Host serveur-final
    HostName 10.10.0.50
    User finaluser
    ProxyJump bastion1,bastion2
    IdentityFile ~/.ssh/final_key

# Connexion en une commande
ssh serveur-final
```

### Copie de fichiers entre deux serveurs distants

```bash
# Via votre machine (avec transfert de clé d'agent)
ssh -A -t serveur1 "scp fichier.txt serveur2:/destination/"

# Direct entre deux serveurs
scp utilisateur1@serveur1:/chemin/fichier.txt utilisateur2@serveur2:/destination/
```

### Exécution de scripts complexes

```bash
# Script local exécuté sur le serveur avec variables
#!/bin/bash

APP_NAME="monapp"
SERVER="prod"

ssh $SERVER /bin/bash << EOF
    export APP_NAME=$APP_NAME
    cd /var/www/\$APP_NAME
    git pull
    npm install
    npm run build
    sudo systemctl restart \$APP_NAME
    echo "Déploiement de \$APP_NAME terminé"
EOF
```

---

- **📧 Contact :** NDXDev@gmail.com
- **📚 Retour au tutoriel :** [README.md](README.md)


