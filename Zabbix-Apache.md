# Atelier : Superviser son SI avec Zabbix

## Introduction
Zabbix est une solution open-source de supervision qui permet de surveiller les performances et la disponibilité des éléments de votre SI (serveurs, réseaux, VMs, services, clients).

Dans cet atelier, vous apprendrez à installer, configurer et utiliser Zabbix pour superviser des services critiques.

## 🎯 Objectifs
- **Installer et configurer un serveur Zabbix.**
- **Ajouter et superviser des hôtes.**
- **Créer des alertes personnalisées.**
- **Configurer des notifications.**

## Sommaire
1. 🔧 Prérequis
2. 🔬 Étape 1 : Installation du serveur Zabbix
3. 🔬 Étape 2 : Configuration de Zabbix
4. 🔬 Étape 3 : Configuration de Zabbix depuis la WUI
5. 🔬 Étape 4 : Installation et configuration de l'Agent Zabbix
6. 🔬 Étape 5 : Ajout d'un hôte et création d'un groupe
7. 🔬 Étape 6 : Configuration des alertes et des notifications
8. 🗋 Résumé
9. 🏆 Challenge

---

## 🔧 Prérequis
Pour cet atelier, vous avez besoin :
- D'un serveur avec une distribution Linux (Debian, Ubuntu, CentOS, etc.).
- D'un serveur web (**Apache2**).
- D'un SGBD ou DBMS (MariaDB/MySQL ou PostgreSQL).
- D'une machine Windows 11 pour installer l'agent Zabbix.

> Ce lab a été testé sur Debian 12.8 avec Zabbix 7.2, MariaDB et **Apache2** pour le serveur. Pour la partie cliente, le lab a été testé avec Windows 10 Pro et Windows 11 Pro.

---

## 🔬 Étape 1 : Installation du serveur Zabbix

### 1. Installer le dépôt de Zabbix dans le système :
```bash
wget https://repo.zabbix.com/zabbix/7.2/release/debian/pool/main/z/zabbix-release/zabbix-release_latest_7.2+debian12_all.deb
sudo dpkg -i zabbix-release_latest_7.2+debian12_all.deb
```

### 2. Mettre à jour la liste des paquets et effectuer une mise à niveau :
```bash
sudo apt update && sudo apt upgrade -y
```

### 3. Installer Zabbix server, frontend et agent :
```bash
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent
```

---

## 🔬 Étape 2 : Configuration de Zabbix

### 1. Installer et configurer le SGBD :
#### Installation :
```bash
sudo apt install mariadb-server
```

#### Vérification :
```bash
systemctl status mariadb
```

#### Création et configuration de la base de données :
```bash
sudo mysql -uroot -p
```
Entrez les commandes suivantes dans le terminal MySQL :
```sql
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';
SET GLOBAL log_bin_trust_function_creators = 1;
EXIT;
```

#### Importer le schéma de la base de données :
```bash
zcat /usr/share/zabbix/sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
```

#### Réactiver les restrictions de modification :
```bash
sudo mysql -uroot -p
```
Entrez les commandes suivantes dans le terminal MySQL :
```sql
SET GLOBAL log_bin_trust_function_creators = 0;
EXIT;
```

### 2. Configurer Zabbix pour utiliser la base de données
#### Modifier le fichier de configuration de Zabbix :
```bash
sudo nano /etc/zabbix/zabbix_server.conf
```
Ajoutez ou modifiez la ligne suivante :
```conf
DBPassword=password
```

### 3. Configurer Apache2 pour le frontend de Zabbix
#### Modifier le fichier de configuration Apache :
```bash
sudo nano /etc/zabbix/apache.conf
```
Modifiez ou vérifiez les lignes suivantes :
```conf
<IfModule mod_php7.c>
    php_value date.timezone Europe/Paris
</IfModule>
```
Redémarrez les services :
```bash
sudo systemctl restart zabbix-server zabbix-agent apache2
sudo systemctl enable zabbix-server zabbix-agent apache2
```

---

## 🔬 Étape 3 : Configuration de Zabbix depuis la WUI

Accédez à l'interface web de Zabbix :
1. Ouvrez un navigateur et entrez : `http://<votre-serveur-ip>/zabbix`
2. Configurez Zabbix à l'aide des assistants de configuration.

---

## 🔬 Étape 4 : Installation et configuration de l'Agent Zabbix

1. **Téléchargez l'agent Zabbix** pour Windows depuis : [https://www.zabbix.com/download_agents](https://www.zabbix.com/download_agents)
2. **Installez l'agent** et renseignez l'adresse IP de votre serveur Zabbix pendant l'installation.

---

## 🔬 Étape 5 : Ajout d'un hôte et création d'un groupe

1. Connectez-vous à la WUI avec :
   - **Utilisateur** : `Admin`
   - **Mot de passe** : `zabbix`

2. Créez un groupe d'hôtes :
   - Allez dans **Configuration → Host Groups**.
   - Cliquez sur **Create host group** et nommez-le "Windows hosts".

3. Ajoutez un hôte :
   - Allez dans **Configuration → Hosts**.
   - Cliquez sur **Create host**.
   - Ajoutez l'interface Agent et renseignez l'IP du client.

---

## 🔬 Étape 6 : Configuration des alertes et des notifications

1. **Appliquez un template :**
   - Allez dans **Configuration → Hosts**.
   - Sélectionnez un hôte, cliquez sur **Templates**, puis choisissez "Windows by Zabbix agent".

2. **Créez une alerte RAM :**
   - Ajoutez un item "memory utilization".
   - Configurez un trigger pour générer une alerte si la RAM dépasse un seuil.

3. **Configurez les notifications :**
   - Activez le type de média "Email" dans **Administration → Media Types**.
   - Assurez-vous que votre client email supporte l'authentification à 2 facteurs. (Bien rentrer le mdp délivré lors de l'authentification à 2 facteurs pour la connexion sur ZABBIX)

---

## 🗋 Résumé
Zabbix est un outil puissant et flexible permettant de surveiller en temps réel les performances et la disponibilité de votre infrastructure.

---

## 🏆 Challenge
**Configurez une alerte personnalisée et recevez une notification.**

---

## 🔍 Critères d'acceptation
Si vous recevez un mail indiquant l'alerte, c'est réussi !

