# Born2beRoot42

# 42cursus - Born2beroot

Dans ce projet, nous allons créer et configurer une machine virtuelle en suivant des règles strictes. À la fin, on sera en mesure de configurer notre propre système d’exploitation en respectant des contraintes de sécurité.

Qu'est-ce qu'une machine virtuelle ?
Une machine virtuelle (VM) est un logiciel permettant d'installer un système d'exploitation au sein d'un environnement isolé, simulant un ordinateur physique. Ces machines sont hébergées sur un ordinateur hôte et utilisent un hyperviseur pour gérer les ressources matérielles (processeur, mémoire, réseau, stockage). Grâce à cela, il est possible d’exécuter plusieurs systèmes d'exploitation simultanément sur un même matériel.

Comment fonctionnent les machines virtuelles ?
Les machines virtuelles permettent de partitionner les ressources physiques et d'exécuter plusieurs environnements indépendants. Cela présente plusieurs avantages :

Séparation des environnements : différentes machines invitées peuvent fonctionner sur le même hôte avec des OS différents.
Sécurité accrue : possibilité de tester des applications dans un environnement isolé.
Optimisation des ressources : meilleure utilisation des ressources matérielles disponibles.
Flexibilité et portabilité : un système peut être facilement transféré d'un serveur physique à un autre.

Installation
On doit installer et configurer une distribution Linux (Debian ou Rocky Linux) dans une machine virtuelle.

Étapes principales :
Installer VirtualBox (ou UTM sur Mac M1).
Créer une machine virtuelle avec un espace disque dynamique.
Installer Debian/Rocky Linux avec les paramètres requis.
Configurer les partitions en utilisant LVM.
Mettre en place sudo, SSH et une politique de mot de passe stricte.

# sudo

Étape 1 Installation de sudo


```
su - 
apt install sudo
```

Étape 2 : Ajouter un utilisateur au groupe sudo
``` usermod -aG sudo <utilisateur>
```

Étape 3 : Vérifier les permissions
```
sudo -v
```

Étape 4 : Configurer sudo
Modifier les règles avec:
```
sudo visudo
```
Ajouter des règles comme:
Defaults        passwd_tries=3
Defaults        badpass_message="Mot de passe incorrect"
Defaults        logfile="/var/log/sudo/sudo.log"

# SSH

Étape 1 Installation et configuration de SSH:
```
sudo apt install openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh
```

Étape 2 Sécurisation de SSH
Dans /etc/ssh/sshd_config:

-> Port 4242
-> PermitRootLogin no

Étape 3 Redémarrer SSH:
sudo systemctl restart ssh

Étape 4 Connexion distante
ssh <utilisateur>@<ip> -p 4242

# Installer & Configurer UFW

Le pare-feu UFW (Uncomplicated Firewall) permet de sécuriser les connexions entrantes et sortantes.
Étape 1 Installer UFW:
sudo apt install ufw

Étape 2 Autoriser SSH sur un port spécifique (4242)
sudo ufw allow 4242/tcp
Étape 3 Activer UFW:
sudo ufw enable
Étape 4 Verifier status:
sudo ufw status

# Gestion des utilisateurs

Politique de mot de passe
Dans /etc/login.defs:
-> PASS_MAX_DAYS   30
-> PASS_MIN_DAYS   2
-> PASS_WARN_AGE   7

Dans /etc/pam.d/common-password :
password requisite pam_pwquality.so retry=3 minlen=10 ucredit=-1 dcredit=-1 lcredit=-1 maxrepeat=3 reject_username difok=7 enforce_for_root

# Création d’un utilisateur et d’un groupe

sudo adduser <utilisateur>
sudo addgroup user42
sudo usermod -aG user42 <utilisateur>

verification du nouveau user creé
sudo chage -l <unitilisateur>

# cron
Planification d’une tâche automatique
Ajouter une tâche dans crontab:
sudo crontab -u root -e

Exemple : exécuter un script toutes les 10 minutes:
*/10 * * * * /path/to/script.sh



# Surveillance et monitoring
Créer un script monitoring.sh affichant des informations système :

1.Architecture et version du noyau
2.Nombre de CPU physiques et virtuels
3.Utilisation de la mémoire et du disque
4.Charge du processeur
5.Nombre de connexions actives
6.Adresse IP et MAC

#!/bin/bash
1.arc=$(uname -a)
2.pcpu=$(grep "physical id" /proc/cpuinfo | sort | uniq | wc -l)
3.vcpu=$(grep "^processor" /proc/cpuinfo | wc -l)
4.fram=$(free -m | awk '$1 == "Mem:" {print $2}')
5.uram=$(free -m | awk '$1 == "Mem:" {print $3}')
6.pram=$(free | awk '$1 == "Mem:" {printf("%.2f"), $3/$2*100}')
wall "  #Architecture: $arc
        #CPU physique: $pcpu
        #vCPU: $vcpu
        #Mémoire: $uram/${fram}MB ($pram%)"

Donner les permissions d'execution:
chmood +x monitoring.sh

# Bonus - 
## Installation de WordPress avec Lighttpd, MariaDB et PHP

Étape 1 : Installer Lighttpd

Lighttpd est un serveur web léger et performant, idéal pour les environnements à ressources limitées comme un Raspberry Pi.

Installer Lighttpd
sudo apt install lighttpd -y

Activer et démarrer le service :
sudo systemctl enable lighttpd
sudo systemctl start lighttpd

Vérifier que le serveur fonctionne :
Ouvre un navigateur et entre l'IP de ton serveur :
http://192.168.1.100

Étape 2 : Installer MariaDB (MySQL)
Installer MariaDB:
sudo apt install mariadb-server -y

Sécuriser MariaDB:
sudo mysql_secure_installation

Définir un mot de passe root : Oui
Supprimer les utilisateurs anonymes : Oui
Interdire l'accès root à distance : Oui
Supprimer la base de test : Oui
Recharger les privilèges : Oui

Vérifier que MariaDB fonctionne:
sudo systemctl status mariadb

Étape 3 : Créer une base de données pour WordPress
Se connecter à MariaDB:
sudo mysql -u root -p

Créer une base de données pour WordPress:
CREATE DATABASE wordpress;

Créer un utilisateur WordPress avec un mot de passe sécurisé:
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'password';

Donner tous les privilèges à l'utilisateur WordPress:
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
FLUSH PRIVILEGES;

Quitter MariaDB
EXIT;

Installer PHP
Installer PHP et les extensions nécessaires
sudo apt install php-cgi php-mysql -y

Étape 4 Activer le module FastCGI dans Lighttpd:
sudo lighty-enable-mod fastcgi
sudo lighty-enable-mod fastcgi-php
sudo systemctl restart lighttpd

Vérifier que PHP fonctionne :
sudo nano /var/www/html/info.php

Ajoute ce code :
<?php phpinfo(); ?>

Ensuite, va sur http://192.168.1.100/info.php pour voir la page de configuration PHP

Étape 5 : Installer wordPress
Télécharger wordPress:
sudo wget https://wordpress.org/latest.tar.gz -P /var/www/html

Extraire les fichiers:
sudo tar -xzvf /var/www/html/latest.tar.gz -C /var/www/html

Donner les permissions correctes:
sudo chown -R www-data:www-data /var/www/html/wordpress
sudo chmod -R 755 /var/www/html/wordpress

Étape 6 Configurer wordPress
Renommer le fichier de configuration:
sudo cp /var/www/html/wordpress/wp-config-sample.php /var/www/html/wordpress/wp-config.php

Modifier les paramètres de connexion à la base de données:
sudo nano /var/www/html/wordpress/wp-config.php

Changer ces lignes
define('DB_NAME', 'wordpress');
define('DB_USER', 'wpuser');
define('DB_PASSWORD', 'password');
define('DB_HOST', 'localhost');

Redémarrer Lighttpd:
sudo systemctl restart lighttpd

Étape 7 Installer wordPress via le navigateur
Ouvre un navigateur et accède à:
http://192.168.1.100/wordpress

Suis les étapes d’installation :
-> Choisir la langue.
-> Entrer les identifiants de la base de données.
-> Créer un compte administrateur pour WordPress

Étape 8 Sécuriser wordPress
Supprimer le fichier info.php:
sudo rm /var/www/html/info.php

Restreindre l’accès aux fichiers de configuration:
sudo chmod 600 /var/www/html/wordpress/wp-config.php

