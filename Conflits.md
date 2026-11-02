# Diagnostiquer une erreur APT/dpkg

Le message « Sub-process /usr/bin/dpkg returned an error code (1) » indique un échec ; la cause se trouve généralement dans les lignes précédentes.

## 1. Relever les informations

Conserver la commande lancée, le nom du paquet, la version Debian et les premières erreurs. Consulter :

```bash
cat /etc/os-release
sudo tail -n 60 /var/log/apt/term.log
sudo tail -n 60 /var/log/dpkg.log
sudo dpkg --audit
```

## 2. Si un verrou est signalé

Vérifier si une installation ou une mise à jour est encore en cours :

```bash
ps -ef | grep -E '[a]pt|[d]pkg'
```

Attendre la fin de l'opération active. Ne pas effacer les fichiers de verrouillage : leur suppression ne termine pas le processus qui détient le verrou et peut permettre des accès concurrents à la base des paquets.

## 3. Après une interruption confirmée

Une fois les autres opérations terminées, tenter de reprendre la configuration :

```bash
sudo dpkg --configure -a
```

Si des dépendances restent non satisfaites :

```bash
sudo apt --fix-broken install
```

Lire les changements proposés avant confirmation. En cas de nouvel échec, revenir à son message précis ; éviter une purge générale ou une suppression forcée sans diagnostic.

## 4. Vérifier la résolution

```bash
sudo dpkg --audit
```

Reprendre ensuite l'installation du paquet identifié, puis vérifier séparément le service concerné. Une commande APT réussie ne prouve pas que Zabbix est configuré et opérationnel.

**Statut :** procédure de diagnostic relue en novembre 2026, sans reproduction d'un incident sur une VM Zabbix.

Référence : [FAQ officielle de l'équipe Debian dpkg](https://wiki.debian.org/Teams/Dpkg/FAQ).

[Retour au sommaire](README.md)
