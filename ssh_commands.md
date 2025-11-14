# 🔧 Référence des commandes SSH

## 📑 Table des matières

- [Génération et gestion des clés](#génération-et-gestion-des-clés)
- [Connexions SSH](#connexions-ssh)
- [Agent SSH](#agent-ssh)
- [Transfert de fichiers](#transfert-de-fichiers)
- [Tunneling et Port Forwarding](#tunneling-et-port-forwarding)
- [Gestion des hôtes connus](#gestion-des-hôtes-connus)
- [Sécurité et surveillance](#sécurité-et-surveillance)
- [Gestion du serveur SSH](#gestion-du-serveur-ssh)
- [Diagnostic et dépannage](#diagnostic-et-dépannage)

---

## 🔑 Génération et gestion des clés

### Générer des clés SSH

```bash
# Clé Ed25519 (recommandée - plus rapide et sécurisée)
ssh-keygen -t ed25519 -C "votre@email.com"

# Clé RSA 4096 bits (compatible avec tous les systèmes)
ssh-keygen -t rsa -b 4096 -C "votre@email.com"

# Clé ECDSA
ssh-keygen -t ecdsa -b 521 -C "votre@email.com"

# Générer avec un nom personnalisé
ssh-keygen -t ed25519 -f ~/.ssh/github_key -C "github@email.com"

# Générer sans phrase de passe (attention !)
ssh-keygen -t ed25519 -N "" -f ~/.ssh/deploy_key
```

### Gérer les clés existantes

```bash
# Afficher la clé publique
cat ~/.ssh/id_ed25519.pub

# Copier la clé publique dans le presse-papier
# Linux
xclip -sel clip < ~/.ssh/id_ed25519.pub
# macOS
pbcopy < ~/.ssh/id_ed25519.pub
# Windows (PowerShell)
Get-Content ~/.ssh/id_ed25519.pub | Set-Clipboard

# Afficher l'empreinte d'une clé
ssh-keygen -l -f ~/.ssh/id_ed25519.pub

# Vérifier le type d'une clé
ssh-keygen -l -f ~/.ssh/id_ed25519

# Changer la phrase de passe d'une clé
ssh-keygen -p -f ~/.ssh/id_ed25519

# Convertir une clé au format PEM
ssh-keygen -p -m PEM -f ~/.ssh/id_rsa

# Convertir une clé au format OpenSSH
ssh-keygen -p -m RFC4716 -f ~/.ssh/id_rsa

# Générer une clé publique à partir d'une clé privée
ssh-keygen -y -f ~/.ssh/id_ed25519 > ~/.ssh/id_ed25519.pub
```

### Copier des clés sur un serveur

```bash
# Méthode automatique (recommandée)
ssh-copy-id utilisateur@serveur.com

# Avec port personnalisé
ssh-copy-id -p 2222 utilisateur@serveur.com

# Avec clé spécifique
ssh-copy-id -i ~/.ssh/autre_cle.pub utilisateur@serveur.com

# Méthode manuelle (une ligne)
cat ~/.ssh/id_ed25519.pub | ssh utilisateur@serveur.com "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

---

## 🌐 Connexions SSH

### Connexions de base

```bash
# Connexion standard
ssh utilisateur@serveur.com

# Avec adresse IP
ssh utilisateur@192.168.1.100

# Spécifier le port
ssh -p 2222 utilisateur@serveur.com

# Utiliser une clé spécifique
ssh -i ~/.ssh/custom_key utilisateur@serveur.com
```

### Connexions avancées

```bash
# Mode verbose (débogage)
ssh -v utilisateur@serveur.com
ssh -vv utilisateur@serveur.com   # Plus de détails
ssh -vvv utilisateur@serveur.com  # Verbosité maximale

# Désactiver la vérification de l'hôte (DANGEREUX)
ssh -o StrictHostKeyChecking=no utilisateur@serveur.com

# Désactiver l'ajout à known_hosts
ssh -o UserKnownHostsFile=/dev/null utilisateur@serveur.com

# Avec compression
ssh -C utilisateur@serveur.com

# Combinaison d'options
ssh -p 2222 -i ~/.ssh/key -C -v utilisateur@serveur.com

# Forcer l'utilisation du protocole IPv4
ssh -4 utilisateur@serveur.com

# Forcer l'utilisation du protocole IPv6
ssh -6 utilisateur@serveur.com

# Désactiver l'exécution de commandes locales
ssh -o PermitLocalCommand=no utilisateur@serveur.com
```

### Exécution de commandes distantes

```bash
# Exécuter une commande simple
ssh utilisateur@serveur.com "ls -la /home"

# Exécuter plusieurs commandes
ssh utilisateur@serveur.com "cd /var/www && git pull && npm install"

# Commande avec sudo (nécessite NOPASSWD dans sudoers)
ssh utilisateur@serveur.com "sudo systemctl restart nginx"

# Rediriger la sortie locale
ssh utilisateur@serveur.com "cat /etc/os-release" > os_info.txt

# Utiliser des variables locales
LOCAL_FILE="backup.tar.gz"
ssh user@server "cat > /tmp/$LOCAL_FILE" < $LOCAL_FILE

# Exécuter un script local sur le serveur distant
ssh utilisateur@serveur.com 'bash -s' < script_local.sh

# Pipeline distant
ssh utilisateur@serveur.com "cat /var/log/syslog | grep error" | less

# Commande interactive
ssh -t utilisateur@serveur.com "top"

# Exécuter un éditeur distant
ssh -t utilisateur@serveur.com "nano /etc/config.txt"
```

### Connexions avec configuration

```bash
# Utiliser un fichier de configuration personnalisé
ssh -F ~/.ssh/config_custom utilisateur@serveur.com

# Tester la configuration sans se connecter
ssh -G utilisateur@serveur.com

# Se connecter avec un alias du fichier config
ssh prod  # Au lieu de ssh admin@production.com -p 2222
```

---

## 🔍 Agent SSH

### Gestion de l'agent

```bash
# Démarrer l'agent SSH
eval "$(ssh-agent -s)"

# Alternative pour Fish shell
eval (ssh-agent -c)

# Vérifier si l'agent tourne
ps aux | grep ssh-agent
echo $SSH_AGENT_PID

# Tuer l'agent
ssh-agent -k
# Ou
kill $SSH_AGENT_PID
```

### Gestion des clés dans l'agent

```bash
# Ajouter une clé à l'agent
ssh-add ~/.ssh/id_ed25519

# Ajouter toutes les clés par défaut
ssh-add

# Ajouter avec durée de vie limitée (en secondes)
ssh-add -t 3600 ~/.ssh/id_ed25519  # 1 heure

# Lister les clés dans l'agent
ssh-add -l

# Lister avec empreintes complètes
ssh-add -L

# Supprimer une clé spécifique
ssh-add -d ~/.ssh/id_ed25519

# Supprimer toutes les clés
ssh-add -D

# Verrouiller l'agent (nécessite un mot de passe pour débloquer)
ssh-add -x

# Déverrouiller l'agent
ssh-add -X
```

---

## 📦 Transfert de fichiers

### SCP (Secure Copy)

```bash
# Copier un fichier vers le serveur
scp fichier.txt utilisateur@serveur.com:/chemin/destination/

# Copier vers le répertoire home
scp fichier.txt utilisateur@serveur.com:~

# Copier depuis le serveur
scp utilisateur@serveur.com:/chemin/fichier.txt .

# Copier un dossier récursivement
scp -r mon_dossier/ utilisateur@serveur.com:/destination/

# Avec port personnalisé
scp -P 2222 fichier.txt utilisateur@serveur.com:/chemin/

# Avec clé spécifique
scp -i ~/.ssh/custom_key fichier.txt utilisateur@serveur.com:/chemin/

# Avec compression
scp -C gros_fichier.tar.gz utilisateur@serveur.com:/chemin/

# Limiter la bande passante (en Kbit/s)
scp -l 1000 fichier.txt utilisateur@serveur.com:/chemin/

# Préserver les métadonnées (dates, permissions)
scp -p fichier.txt utilisateur@serveur.com:/chemin/

# Mode verbose
scp -v fichier.txt utilisateur@serveur.com:/chemin/

# Copier entre deux serveurs distants
scp user1@server1:/chemin/fichier.txt user2@server2:/destination/

# Copier plusieurs fichiers
scp fichier1.txt fichier2.txt utilisateur@serveur.com:/chemin/

# Utiliser un wildcard
scp *.txt utilisateur@serveur.com:/chemin/
```

### SFTP (SSH File Transfer Protocol)

```bash
# Se connecter en mode interactif
sftp utilisateur@serveur.com

# Avec port personnalisé
sftp -P 2222 utilisateur@serveur.com

# Mode batch (fichier de commandes)
sftp -b commandes.txt utilisateur@serveur.com

# Commandes SFTP interactives
sftp> help                    # Afficher l'aide
sftp> ls                      # Lister fichiers distants
sftp> lls                     # Lister fichiers locaux
sftp> pwd                     # Répertoire distant actuel
sftp> lpwd                    # Répertoire local actuel
sftp> cd /chemin              # Changer de répertoire distant
sftp> lcd /chemin             # Changer de répertoire local
sftp> get fichier.txt         # Télécharger un fichier
sftp> get -r dossier/         # Télécharger un dossier
sftp> put fichier.txt         # Envoyer un fichier
sftp> put -r dossier/         # Envoyer un dossier
sftp> rm fichier.txt          # Supprimer fichier distant
sftp> rmdir dossier           # Supprimer dossier distant
sftp> mkdir nouveau           # Créer répertoire distant
sftp> lmkdir nouveau          # Créer répertoire local
sftp> chmod 644 fichier.txt   # Changer permissions
sftp> chown uid fichier.txt   # Changer propriétaire
sftp> rename ancien nouveau   # Renommer fichier distant
sftp> symlink cible lien      # Créer lien symbolique
sftp> !ls                     # Exécuter commande locale
sftp> exit                    # Quitter

# Télécharger avec conservation des permissions
sftp> get -p fichier.txt

# Reprendre un téléchargement interrompu
sftp> reget fichier.txt
```

### Rsync sur SSH

```bash
# Synchroniser un dossier
rsync -avz -e ssh dossier/ utilisateur@serveur.com:/destination/

# Avec suppression des fichiers sur la destination
rsync -avz --delete -e ssh dossier/ utilisateur@serveur.com:/destination/

# Avec port personnalisé
rsync -avz -e "ssh -p 2222" dossier/ utilisateur@serveur.com:/destination/

# Mode simulation (dry-run)
rsync -avz --dry-run -e ssh dossier/ utilisateur@serveur.com:/destination/

# Afficher la progression
rsync -avz --progress -e ssh fichier.tar.gz utilisateur@serveur.com:/destination/

# Exclure des fichiers
rsync -avz --exclude='*.log' -e ssh dossier/ utilisateur@serveur.com:/destination/

# Limiter la bande passante (en KB/s)
rsync -avz --bwlimit=1000 -e ssh dossier/ utilisateur@serveur.com:/destination/
```

---

## 🚇 Tunneling et Port Forwarding

### Port Forwarding Local

```bash
# Syntaxe: ssh -L [port_local]:[host_distant]:[port_distant] user@server

# Accéder à une base de données distante
ssh -L 3306:localhost:3306 utilisateur@serveur.com
# Puis: mysql -h 127.0.0.1 -P 3306

# Accéder à un service web distant
ssh -L 8080:localhost:80 utilisateur@serveur.com
# Puis: http://localhost:8080

# Accéder à PostgreSQL
ssh -L 5432:localhost:5432 utilisateur@serveur.com

# Accéder à Redis
ssh -L 6379:localhost:6379 utilisateur@serveur.com

# Tunnel vers un autre serveur via le serveur SSH
ssh -L 8080:autre-serveur.com:80 utilisateur@bastion.com

# Multiple port forwarding
ssh -L 8080:localhost:80 -L 3306:localhost:3306 utilisateur@serveur.com

# En arrière-plan
ssh -f -N -L 8080:localhost:80 utilisateur@serveur.com
```

### Port Forwarding Distant

```bash
# Syntaxe: ssh -R [port_distant]:[host_local]:[port_local] user@server

# Exposer votre serveur web local
ssh -R 9000:localhost:3000 utilisateur@serveur.com
# Le serveur peut accéder via localhost:9000

# Exposer un service local sur Internet
ssh -R 8080:localhost:80 utilisateur@serveur-public.com

# Avec bind sur toutes les interfaces
ssh -R 0.0.0.0:9000:localhost:3000 utilisateur@serveur.com
```

### Dynamic Port Forwarding (Proxy SOCKS)

```bash
# Créer un proxy SOCKS sur le port 1080
ssh -D 1080 utilisateur@serveur.com

# Avec port personnalisé
ssh -D 9050 utilisateur@serveur.com

# En arrière-plan
ssh -f -N -D 1080 utilisateur@serveur.com

# Configurer votre navigateur:
# - Type: SOCKS5
# - Hôte: localhost
# - Port: 1080

# Utiliser avec curl
curl --socks5 localhost:1080 https://example.com

# Utiliser avec Git
git config --global http.proxy 'socks5://127.0.0.1:1080'
```

### Options de tunneling

```bash
# -f : Exécuter en arrière-plan
# -N : Ne pas exécuter de commande distante
# -T : Désactiver l'allocation de pseudo-terminal
# -C : Activer la compression

# Tunnel optimal
ssh -f -N -T -C -L 8080:localhost:80 utilisateur@serveur.com

# Garder le tunnel actif
ssh -f -N -o ServerAliveInterval=30 -L 8080:localhost:80 utilisateur@serveur.com
```

---

## 🔧 Gestion des hôtes connus

### Manipulation de known_hosts

```bash
# Supprimer une entrée pour un hôte
ssh-keygen -R serveur.com

# Supprimer pour un hôte avec port spécifique
ssh-keygen -R [serveur.com]:2222

# Supprimer pour une adresse IP
ssh-keygen -R 192.168.1.100

# Vérifier l'empreinte d'un hôte
ssh-keygen -l -F serveur.com

# Récupérer les clés publiques d'un serveur
ssh-keyscan serveur.com

# Ajouter à known_hosts
ssh-keyscan serveur.com >> ~/.ssh/known_hosts

# Scanner plusieurs serveurs
ssh-keyscan server1.com server2.com >> ~/.ssh/known_hosts

# Scanner avec port personnalisé
ssh-keyscan -p 2222 serveur.com >> ~/.ssh/known_hosts

# Scanner tous les types de clés
ssh-keyscan -t rsa,dsa,ecdsa,ed25519 serveur.com >> ~/.ssh/known_hosts

# Afficher toutes les clés d'un hôte dans known_hosts
ssh-keygen -F serveur.com

# Hasher toutes les entrées du fichier known_hosts
ssh-keygen -H -f ~/.ssh/known_hosts
```

---

## 🛡️ Sécurité et surveillance

### Surveiller les connexions

```bash
# Voir qui est connecté
who
w

# Afficher les connexions SSH actives
who | grep pts

# Historique des connexions
last
last -a

# Connexions actuellement actives
last | grep "still logged in"

# Tentatives de connexion échouées
sudo lastb
sudo lastb | head -20

# Connexions SSH depuis un utilisateur spécifique
last utilisateur
```

### Analyser les logs

```bash
# Logs SSH en temps réel (Debian/Ubuntu)
sudo tail -f /var/log/auth.log

# Logs SSH en temps réel (RHEL/CentOS)
sudo tail -f /var/log/secure

# Avec systemd
sudo journalctl -u sshd -f
sudo journalctl -u ssh -f

# Logs des dernières 24h
sudo journalctl -u sshd --since "24 hours ago"

# Tentatives échouées
sudo grep "Failed password" /var/log/auth.log
sudo grep "Invalid user" /var/log/auth.log

# Connexions réussies
sudo grep "Accepted publickey" /var/log/auth.log
sudo grep "Accepted password" /var/log/auth.log

# Top 10 des IPs qui tentent de se connecter
sudo grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -nr | head -10

# Compter les tentatives par utilisateur
sudo grep "Failed password" /var/log/auth.log | awk '{print $9}' | sort | uniq -c | sort -nr
```

### Permissions des fichiers SSH

```bash
# Vérifier les permissions actuelles
ls -la ~/.ssh/

# Corriger les permissions du répertoire
chmod 700 ~/.ssh

# Corriger les permissions de la clé privée
chmod 600 ~/.ssh/id_*
chmod 600 ~/.ssh/*_rsa
chmod 600 ~/.ssh/*_ed25519

# Corriger les permissions de la clé publique
chmod 644 ~/.ssh/*.pub

# Corriger les permissions de authorized_keys
chmod 600 ~/.ssh/authorized_keys

# Corriger les permissions du fichier config
chmod 600 ~/.ssh/config

# Corriger les permissions de known_hosts
chmod 644 ~/.ssh/known_hosts

# Script de correction complet
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_* 2>/dev/null
chmod 644 ~/.ssh/*.pub 2>/dev/null
chmod 600 ~/.ssh/authorized_keys 2>/dev/null
chmod 600 ~/.ssh/config 2>/dev/null
chmod 644 ~/.ssh/known_hosts 2>/dev/null
```

---

## 🔄 Gestion du serveur SSH

### Contrôle du service

```bash
# Démarrer SSH
sudo systemctl start sshd
sudo service ssh start

# Arrêter SSH
sudo systemctl stop sshd
sudo service ssh stop

# Redémarrer SSH
sudo systemctl restart sshd
sudo service ssh restart

# Recharger la configuration sans interrompre les connexions
sudo systemctl reload sshd
sudo service ssh reload

# Vérifier le statut
sudo systemctl status sshd
sudo service ssh status

# Activer au démarrage
sudo systemctl enable sshd

# Désactiver au démarrage
sudo systemctl disable sshd
```

### Configuration du serveur

```bash
# Tester la configuration avant de redémarrer
sudo sshd -t

# Tester avec verbosité
sudo sshd -t -f /etc/ssh/sshd_config

# Afficher la configuration compilée
sudo sshd -T

# Éditer la configuration
sudo nano /etc/ssh/sshd_config

# Sauvegarder la configuration avant modification
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup.$(date +%Y%m%d)

# Vérifier les ports d'écoute
sudo netstat -tlnp | grep ssh
sudo ss -tlnp | grep ssh
sudo lsof -i :22
```

---

## 🔍 Diagnostic et dépannage

### Tests de connectivité

```bash
# Vérifier si le serveur est accessible
ping serveur.com

# Vérifier si le port SSH est ouvert
telnet serveur.com 22
nc -zv serveur.com 22
nmap -p 22 serveur.com

# Test avec timeout
timeout 5 nc -zv serveur.com 22

# Scanner les ports SSH communs
nmap -p 22,2222,22222 serveur.com
```

### Diagnostic SSH

```bash
# Connexion avec verbosité maximale
ssh -vvv utilisateur@serveur.com

# Tester sans exécuter de commandes
ssh -T utilisateur@serveur.com

# Vérifier quelle configuration est utilisée
ssh -G utilisateur@serveur.com

# Tester avec une clé spécifique
ssh -vvv -i ~/.ssh/test_key utilisateur@serveur.com

# Forcer l'authentification par mot de passe
ssh -o PreferredAuthentications=password utilisateur@serveur.com

# Forcer l'authentification par clé
ssh -o PreferredAuthentications=publickey utilisateur@serveur.com

# Ignorer le fichier config
ssh -F /dev/null utilisateur@serveur.com
```

### Informations système

```bash
# Version SSH client
ssh -V

# Version SSH serveur
ssh -v serveur.com 2>&1 | grep "remote software version"

# Algorithmes supportés par le client
ssh -Q cipher
ssh -Q mac
ssh -Q kex
ssh -Q key

# Processus SSH en cours
ps aux | grep ssh
pgrep -a ssh

# Connexions réseau SSH actives
sudo netstat -tnpa | grep ssh
sudo ss -tnp | grep ssh
```

### Génération de clés de debug

```bash
# Créer une clé de test
ssh-keygen -t ed25519 -f /tmp/test_key -N ""

# Tester avec cette clé
ssh -i /tmp/test_key -v utilisateur@serveur.com

# Comparer les empreintes
ssh-keygen -l -f ~/.ssh/id_ed25519
ssh-keygen -l -f ~/.ssh/id_ed25519.pub
```

### Résolution de problèmes courants

```bash
# Problème: "Permission denied (publickey)"
# 1. Vérifier les permissions
ls -la ~/.ssh/
# 2. Vérifier que la clé est dans l'agent
ssh-add -l
# 3. Ajouter la clé
ssh-add ~/.ssh/id_ed25519
# 4. Mode verbose
ssh -v utilisateur@serveur.com

# Problème: "Connection timed out"
# 1. Tester la connectivité
ping serveur.com
nc -zv serveur.com 22
# 2. Vérifier le pare-feu
sudo ufw status
sudo iptables -L

# Problème: "Host key verification failed"
# Supprimer l'ancienne clé
ssh-keygen -R serveur.com

# Problème: "Too many authentication failures"
# Limiter les clés essayées
ssh -o IdentitiesOnly=yes -i ~/.ssh/cle_correcte utilisateur@serveur.com

# Problème: Agent SSH ne fonctionne pas
# Redémarrer l'agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Problème: Session se déconnecte
# Ajouter dans ~/.ssh/config
Host *
    ServerAliveInterval 30
    ServerAliveCountMax 5
```

---

## 💡 Astuces et raccourcis

```bash
# Échapper une session SSH bloquée
# Taper: ~.  (tilde puis point)

# Suspendre une session SSH
# Taper: ~^Z  (tilde puis Ctrl+Z)

# Afficher les connexions SSH multiplexées
# Taper: ~#

# Relancer un port forwarding
# Taper: ~C puis -L 8080:localhost:80

# Créer un fichier de connexion rapide
alias prod="ssh admin@production.com -p 2222"
alias dev="ssh developer@dev.com"

# SSH avec confirmation avant chaque commande
ssh utilisateur@serveur.com -t "bash --norc"

# Exécuter un script local avec environnement distant
ssh utilisateur@serveur.com 'bash -s' < local_script.sh

# Sauvegarder la configuration SSH
tar -czf ssh_backup_$(date +%Y%m%d).tar.gz ~/.ssh/

# Restaurer la configuration SSH
tar -xzf ssh_backup_20250114.tar.gz -C ~/
```

---

**📧 Contact :** NDXDev@gmail.com
**📚 Retour au tutoriel :** [README.md](README.md)
