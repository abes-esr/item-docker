# item-docker

[![Docker Pulls](https://img.shields.io/docker/pulls/abesesr/item.svg)](https://hub.docker.com/r/abesesr/item/)

Ce dépôt contient la configuration docker 🐳 pour déployer l'application Item (cf sources de l'[api](https://github.com/abes-esr/item-api) et du [front](https://github.com/abes-esr/item-client)) en local sur le poste d'un développeur ou sur les serveurs de dev, test et prod. 

# URLs de item

Les URLs correspondantes aux déploiements en local, dev, test et prod de item sont :

- local :
  - http://127.0.0.1:18080/ : URL interne de item-client
  - http://127.0.0.1:18081/ : URL interne de item-api
  - http://127.0.0.1:18082/ : URL interne de l'adminer
  - http://127.0.0.1:18083/ : Port ouvert pour la gestion de la BDD avec un client SQL
- dev :
  - http://diplotaxis5-dev.v212.abes.fr:18080/ : URL interne de item-client
  - http://diplotaxis5-dev.v212.abes.fr:18081/ : URL interne de item-api
  - http://diplotaxis5-dev.v212.abes.fr:18082/ : URL interne de l'adminer
  - http://diplotaxis5-dev.v212.abes.fr:18083/ : Port ouvert pour gérer la BDD du conteneur
- test :
  - http://diplotaxis5-test.v202.abes.fr:18080/ : URL interne de item-client
  - http://diplotaxis5-test.v202.abes.fr:18081/ : URL interne de item-api
  - http://diplotaxis5-test.v202.abes.fr:18082/ : URL interne de l'adminer
  - http://diplotaxis5-test.v202.abes.fr:18083/ : Port ouvert pour gérer la BDD du conteneur
- prod
  - http://diplotaxis5-prod.v102.abes.fr:18080/ : URL interne de item-client
  - http://diplotaxis5-prod.v102.abes.fr:18081/ : URL interne de item-api
  - http://diplotaxis5-prod.v102.abes.fr:18082/ : URL interne de l'adminer
  - http://diplotaxis5-prod.v102.abes.fr:18083/ : Port ouvert pour gérer la BDD du conteneur
 
# Utilisation de DBeaver (client SQL) pour gérer les bases de données dans Item sur le port 18083

Le port 18083 est ouvert sur les environnements de développement, de test et de production afin de permettre la gestion des bases de données d'Item via un client SQL externe.
Par exemple, avec DBeaver, il est possible de créer une nouvelle connexion : 

- Host: diplotaxis5-test.v202.abes.fr
- Port: 18083
- Database: item
- Nom d'utilisateur: variable ITEM_DB_POSTGRES_USER du fichier .env
- Mot de passe: variable ITEM_DB_POSTGRES_PASSWORD du fichier .env

# URL de consultation des logs avec dozzle

http://diplotaxis5-dev.v212.abes.fr:29999

# Installation

***Docker doit être installé sur la machine de déploiement***

- Déployer la configuration docker dans un répertoire :
*adapter /opt/pod/ avec l'emplacement souhaité où déployer l'application*

```bash
git clone https://github.com/abes-esr/item-docker.git
```

- Configurer l'application depuis l'exemple du [fichier ``.env-dist``](./.env-dist) (Ce fichier contient la liste des variables, accompagnées d'explications et d'exemples de valeurs) :

```bash
cp .env-dist .env
```
- Ajouter les valeurs des variables dans le fichier .env

- Démarrer l'application
```bash
sudo docker compose up -d
```

*Retirez l'option -d pour afficher les logs dans le terminal, puis utilisez CTRL+C pour arrêter l'application*

*Une base de données Postgresql vide sera alors automatiquement initialisée. Ses données binaires seront placées dans le répertoire persistant ``volumes/item-db/pgdata/``*

- pour stopper l'application
```bash
sudo docker compose stop
```
- pour redémarrer l'application
```bash
sudo docker compose restart
```

# Supervision

- pour visualiser les logs de l'application
```bash
sudo docker compose logs -f --tail=100
```
- pour visualiser les logs d'un container
```bash
sudo docker compose logs -f --tail=100 nom_du_container
```

Ces commandes afficheront les 100 dernières lignes de logs générées par les conteneurs, ainsi que toutes les nouvelles lignes en temps réel, jusqu'à ce que la commande CTRL+C soit exécutée pour arrêter l'affichage.

# Déploiement continu

Les configurations pour Item sont les suivantes (cf [poldev](https://github.com/abes-esr/abes-politique-developpement/blob/main/01-Gestion%20du%20code%20source.md#utilisation-des-branches)) :
- git push sur la branche ``develop`` provoque un déploiement automatique sur le serveur ``diplotaxis5-dev``
- git push (le plus couramment merge) sur la branche ``main`` provoque un déploiement automatique sur le serveur ``diplotaxis5-test``
- git tag X.X.X (associé à une release) sur la branche ``main`` permet un déploiement (non automatique) sur le serveur ``diplotaxis5-prod``

Item est déployé automatiquement en utilisant l'outil WUD (What's Up Docker). WUD surveille la présence éventuelle de nouvelles images Docker pour item-front, item-api et item-batch grâce aux labels ``wud.watch=true`` et ``wud.watch.digest=true`` déclarés dans le ``docker-compose.yml``.

Si une nouvelle image est disponible, WUD déclenche la mise à jour du conteneur concerné. Pour le développeur, il suffit de faire un git commit + push sur la branche develop, d'attendre que l'action GitHub construise et publie l'image, puis de laisser WUD intervenir pour que la modification soit déployée sur l'environnement cible, comme par exemple la machine diplotaxis5-dev.

# Sauvegardes

Les éléments suivants sont sauvegardés par le service Infrastructure et Architecture Technique :
- ``.env`` : il contient la configuration spécifique de notre déploiement. Il doit être restauré et non crée depuis le .env-dist qui est généré à partir de la commande git clone du depot ou git pull.
- ``volumes/item-db/dump/`` : il contient les dumps quotidiens de la base de données postgresql de item : pour pourvoir afficher les sauvegardes sur le serveur : ``sudo ls -ll dump``

Le répertoire suivant est à exclure des sauvegardes :
- ``/opt/pod/item-docker/volumes/item-db/pgdata/`` : il contient les données binaires de la base de données Postgresql item

Les dumps sont effectués à l'aide de https://github.com/tiredofit/docker-db-backup

La configuration des dumps est définie dans le fichier docker-compose, au niveau de l'image item-db-dumper.
Ainsi, l'heure de sauvegarde quotidienne est configurée via : 
```bash
DEFAULT_BACKUP_BEGIN: "0130"
```
Cette valeur "0130" indique que la sauvegarde des dumps commence chaque jour à 1h30 du matin, heure GMT.
Le format utilisé est "HHmm", où :
- "HH" représente les heures sur 2 chiffres (de 00 à 23)
- "mm" représente les minutes sur 2 chiffres (de 00 à 59)


# Restauration depuis une sauvegarde

## Restauration de l'application



- Se Connecter avec son compte développeur sur la machine de déploiement diplotaxis5-prod (via Putty etc.)

- Se positionner dans le répertoire des applications :
```bash
cd /opt/pod
```
- Récupérer le projet item-docker :
```bash
git clone https://github.com/abes-esr/item-docker.git
```
- Récupérer le .env depuis sotora (authentification nécessaire) :
```bash
rsync -av devel@sotora.v104.abes.fr:/backup_pool/diplotaxis5-prod/daily.0/racine/opt/pod/item-docker/.env /opt/pod/item-docker/.env
```
*Pour sélectionner une sauvegarde autre que la plus récente, il suffit de remplacer daily.0 dans la commande par le jour souhaité (daily.1 pour la veille, daily.2 pour l'avant-veille, etc.)*

## Restauration des données de l'application

- Se Connecter avec son compte développeur sur la machine de déploiement diplotaxis5-prod (via Putty etc.)

- Se positionner dans le répertoire de l'application :
```bash
cd /opt/pod/item-docker
```

- Vérifier que les conteneurs sont arrêtés :
```bash
sudo docker compose down --remove-orphans	
```
- Redémarrer uniquement item-db et item-db-dumper : 
```bash
sudo docker compose up -d item-db item-db-dumper
```
*Ne pas redémarrer les containers item-batch ou item-api dont la couche JPA recrée la base de données automatiquement.*

Attention: recréer à la main /opt/pod/item-docker/volumes/item-db/dump via winiscp ou la ligne de commande pour bénéficier des droits qui permetteront de glisser déposer les sauvegardes

**Les sept dernières sauvegardes sont conservées et accessibles sur la machine diplotaxis5-prod, qui est également sauvegardée sur la machine sotora. Ainsi, la restauration de la base peut se faire soit directement à partir des sauvegardes de diplotaxis5-prod, soit, en cas d'indisponibilité ou pour des sauvegardes plus anciennes que 7 jours, depuis sotora.**

- Choisir l'une des deux options suivantes : 
  - [Restauration depuis diplotaxis5-prod](#restauration-depuis-diplotaxis5-prod)
  - [Restauration depuis sotora](#restauration-depuis-sotora)

### Restauration depuis diplotaxis5-prod

- Supprimer le schéma, la base de données existante et recréer la base vide :
```bash
sudo docker exec -it item-db bash -c 'psql -U $POSTGRES_USER -d $POSTGRES_DB -c "DROP SCHEMA public CASCADE;"'
sudo docker exec -it item-db bash -c 'dropdb -f -U $POSTGRES_USER $POSTGRES_DB'
sudo docker exec -it item-db bash -c 'createdb -U $POSTGRES_USER $POSTGRES_DB'
# 'bash -c' est utilisé pour permettre l'interprétation des variables d'environnement 
# du conteneur (POSTGRES_USER, POSTGRES_DB) par les commandes psql, dropdb et createdb.

```
- Choisir l'une des deux options suivantes : 
  - [Restauration du schéma et des données avec la sauvegarde la plus récente](#restauration-du-schéma-et-des-données-avec-la-sauvegarde-la-plus-récente)
  - [Restauration du schéma et des données avec une sauvegarde choisie](#restauration-du-schéma-et-des-données-avec-une-sauvegarde-choisie)

#### Restauration du schéma et des données avec la sauvegarde la plus récente
```bash
sudo docker exec -it item-db-dumper bash -c 'restore $(readlink -f /backup/latest-pgsql_item_item-db) $DB_TYPE $DB_HOST $DB_NAME $DB_USER $DB_PASS 5432'	
# 'bash -c' est utilisé pour permettre l'interprétation des variables d'environnement 
# du conteneur (DB_TYPE, DB_HOST, DB_NAME, DB_USER, DB_PASS) par la commande restore.
# Pour utiliser le fichier de sauvegarde correct, 'readlink -f' permet de remplacer l'alias 'latest-pgsql_item_item-db'"
# par son chemin absolu, nécessaire à la commande de restauration.
```
#### Restauration du schéma et des données avec une sauvegarde choisie
- Lister les sauvegardes disponibles : 
```bash
ll volumes/item-db/dump/
```
- Compléter la commande avec le nom de la sauvegarde à restaurer, par exemple : 
```bash
sudo docker exec -it item-db-dumper bash -c 'restore /backup/pgsql_item_item-db_20250221-144114.sql.gz $DB_TYPE $DB_HOST $DB_NAME $DB_USER $DB_PASS 5432'
# 'bash -c' est utilisé pour permettre l'interprétation des variables d'environnement 
# du conteneur (DB_TYPE, DB_HOST, DB_NAME, DB_USER, DB_PASS) par la commande restore.
```

### Restauration depuis sotora

- Récupérer la sauvegarde depuis sotora : 
```bash
rsync -avL devel@sotora.v104.abes.fr:/backup_pool/diplotaxis5-prod/daily.0/racine/opt/pod/item-docker/volumes/item-db/dump/latest-pgsql_item_item-db /opt/pod/item-docker/volumes/item-db/dump/pgsql_item_item-db_sotora.sql.gz
```
*Pour sélectionner une sauvegarde autre que la plus récente, il suffit de remplacer daily.0 dans la commande par le jour souhaité (daily.1 pour la veille, daily.2 pour l'avant-veille, etc.)*

- Supprimer le schéma, la base de données existante et recréer la base vide : 
```bash
sudo docker exec -it item-db bash -c 'psql -U $POSTGRES_USER -d $POSTGRES_DB -c "DROP SCHEMA public CASCADE;"'
sudo docker exec -it item-db bash -c 'dropdb -f -U $POSTGRES_USER $POSTGRES_DB'
sudo docker exec -it item-db bash -c 'createdb -U $POSTGRES_USER $POSTGRES_DB'
# 'bash -c' est utilisé pour permettre l'interprétation des variables d'environnement 
# du conteneur (POSTGRES_USER, POSTGRES_DB) par les commandes psql, dropdb et createdb.
```
- Restaurer le schéma et les données : 
```bash
sudo docker exec -it item-db-dumper bash -c 'restore /backup/pgsql_item_item-db_sotora.sql.gz $DB_TYPE $DB_HOST $DB_NAME $DB_USER $DB_PASS 5432' 
# 'bash -c' est utilisé pour permettre l'interprétation des variables d'environnement 
# du conteneur (DB_TYPE, DB_HOST, DB_NAME, DB_USER, DB_PASS) par la commande restore.
```
La restauration est terminée.

on peut maintenant lancer la commande suivante pour redémarrer l'application
```bash
sudo docker compose up -d
```
# Mise à jour manuelle de la dernière version

La récupération et le démarrage de la dernière version de l'application peuvent être réalisés ainsi : 
```bash
sudo docker compose pull
sudo docker compose up -d
```
Le pull permet de télécharger la dernière image Docker disponible pour la version en cours (par exemple, develop-api ou main-api). Sans effectuer de pull, c'est la dernière image téléchargée qui sera utilisée.
