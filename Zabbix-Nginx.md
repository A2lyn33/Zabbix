# Zabbix
Voici un **récapitulatif** de l’installation, la configuration et l’utilisation de **Zabbix** avec une mise en forme enrichie d’icônes pour faciliter la lecture. Suis pas à pas les étapes pour avoir un serveur de supervision fonctionnel, ajouter un hôte Windows et recevoir des notifications par e-mail lorsqu’un seuil d’alerte est dépassé.

---

## 🏁 1. Installation du serveur Zabbix

1. **📥 Installer le dépôt Zabbix**  
   ```bash
   wget https://repo.zabbix.com/zabbix/7.2/release/debian/pool/main/z/zabbix-release/zabbix-release_latest_7.2+debian12_all.deb
   dpkg -i zabbix-release_latest_7.2+debian12_all.deb
   apt update && apt upgrade -y
   ```

2. **💻 Installer Zabbix (server, frontend, agent)**  
   ```bash
   apt install zabbix-server-mysql zabbix-frontend-php zabbix-nginx-conf zabbix-sql-scripts zabbix-agent
   ```

3. **🛢️ Installer et démarrer MariaDB**  
   ```bash
   apt install mariadb-server
   systemctl status mysql
   ```

---

## 🗃️ 2. Configuration de la base de données

1. **⚙️ Créer la base et l’utilisateur Zabbix**  
   ```sql
   mysql -uroot -p
   # Saisir le mot de passe root
   create database zabbix character set utf8mb4 collate utf8mb4_bin;
   create user zabbix@localhost identified by 'password';
   grant all privileges on zabbix.* to zabbix@localhost;
   set global log_bin_trust_function_creators = 1;
   quit;
   ```

2. **📄 Importer le schéma Zabbix**  
   ```bash
   zcat /usr/share/zabbix/sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
   ```

3. **🔒 Désactiver `log_bin_trust_function_creators`**  
   ```sql
   mysql -uroot -p
   # Saisir le mot de passe root
   set global log_bin_trust_function_creators = 0;
   quit;
   ```

4. **📝 Éditer la conf Zabbix pour la BD**  
   - Dans le fichier `/etc/zabbix/zabbix_server.conf`, ajouter (ou modifier) :  
     ```
     DBPassword=password
     ```

---

## 🌐 3. Configuration de Nginx et démarrage des services

1. **🔧 Configurer l’accès au frontend**  
   - Dans `/etc/zabbix/nginx.conf`, modifier par exemple :
     ```nginx
     listen 8080;
     server_name X.X.X.X;  # Adresse IP du serveur
     ```

2. **▶️ Démarrer et activer Zabbix et Nginx**  
   ```bash
   systemctl restart zabbix-server zabbix-agent nginx php8.2-fpm
   systemctl enable zabbix-server zabbix-agent nginx php8.2-fpm
   ```

3. **🌐 Accéder à l’interface web**  
   - Sur un navigateur (même réseau), aller sur :  
     ```
     http://X.X.X.X:8080
     ```
   - Renseigner les informations demandées (base de données, fuseau horaire, nom du serveur, etc.).

---

## 💽 4. Installation et configuration de l’agent sur Windows

1. **⬇️ Télécharger l’agent**  
   - Depuis la page [Zabbix Download](https://www.zabbix.com/download_agents) et sélectionner l’agent pour Windows.

2. **🚀 Installer l’agent**  
   - Lors de l’installation, préciser l’adresse IP de ton serveur Zabbix dans le champ **Zabbix server IP/DNS**.

---

## 👥 5. Ajout d’un hôte et création d’un groupe dans l’interface Zabbix

1. **👤 Créer un groupe d’hôtes (Host group)**  
   - Menu **Data collection** → **Host groups** → **Create host group**  
   - Nom : `Windows hosts`.

2. **➕ Ajouter l’hôte Windows**  
   - Menu **Data collection** → **Hosts** → **Create host**  
   - Sélectionner le groupe : `Windows hosts`  
   - Dans **Interfaces**, choisir : **Agent**  
   - Renseigner l’IP de la machine Windows.

---

## 🚨 6. Configuration des alertes et notifications

1. **🎛️ Appliquer le template “Windows by Zabbix Agent”**  
   - Menu **Data collection** → **Hosts** → Cliquer sur l’hôte Windows.  
   - Dans **Templates** → **Select**.  
   - Choisir **Template** → cocher **Windows by Zabbix agent** → **Select** → **Update**.  
   - Les premiers indicateurs apparaîtront dans **Monitoring** → **Hosts**.

2. **📧 Configurer l’envoi de mails**  
   - Menu **Administration** → **Media types**.  
   - Activer le média **Email** déjà existant (le configurer avec tes informations SMTP).  
   - Associer ce média à un utilisateur ou groupe d’utilisateurs dans Zabbix pour recevoir les alertes.

3. **🔥 Créer une alerte personnalisée sur la RAM**  
   - Menu **Data collection** → **Hosts** → **Items** (sur la ligne de l’hôte Windows).  
   - Rechercher “memory utilization”.  
   - **Clone** l’item : donner un nom et une clé, par exemple :  
     - **Name** : `Alerte RAM`  
     - **Key** : `AlerteRAM`
   - Menu **Data collection** → **Hosts** → **Triggers** (toujours sur la ligne de l’hôte).  
   - **Create trigger** :  
     - Nom : `WindowsAlerteRam`  
     - Sévérité : *Disaster*, par exemple  
     - **Add** expression : sélectionner l’item `Alerte RAM`, choisir `>=` et indiquer la valeur de déclenchement.  
     - **Insert** → **Add**.

4. **🧪 Tester l’alerte**  
   - Ouvrir plusieurs applications sur la machine Windows pour faire monter l’usage RAM au-dessus du seuil.  
   - Vérifier la réception du mail (alerte) quand le seuil est dépassé, puis un autre mail quand la RAM repasse en dessous du seuil (résolution).

---

## 📝 7. Résumé

- **Zabbix** est un outil de supervision **complet** (serveurs, réseaux, hôtes Windows/Linux, etc.).  
- **Installation** : Installer le serveur Zabbix, configurer la base de données, le frontend web, puis démarrer les services.  
- **Configuration** : Ajouter des hôtes (Windows, Linux, etc.) et appliquer des **templates** prédéfinis.  
- **Alertes et notifications** : Configurer la **messagerie**, créer des **triggers** personnalisés, et recevoir des **emails** d’alerte.

---

## 🏆 8. Challenge

> **Configure une alerte personnalisée et vérifie que tu reçois bien la notification par mail.**

**Critère d’acceptation :**  
- Si tu reçois l’email d’alerte (et celui de retour à la normale), c’est gagné !  

---

### 🤝 En bref
- Tu sais **installer** Zabbix sur Debian (avec Nginx et MariaDB).  
- Tu sais **ajouter un hôte Windows** et appliquer un template de supervision.  
- Tu es **capable de créer un trigger** sur un item personnalisé et de configurer l’envoi de **notifications par mail**.

**Bonne supervision et amuse-toi bien avec Zabbix !**
