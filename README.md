# item-docker

[![Docker Pulls](https://img.shields.io/docker/pulls/abesesr/item.svg)](https://hub.docker.com/r/abesesr/item/)

Ce dépôt contient la configuration docker 🐳 pour déployer l'application item (cf sources de l'[api](https://github.com/abes-esr/item-api) et du [front](https://github.com/abes-esr/item-client)) en local sur le poste d'un développeur, ou bien sur les serveurs de dev, test et prod. 

## URLs de item

Les URLs correspondantes aux déploiements en local, dev, test et prod de item sont les suivantes :

- local :
  - http://127.0.0.1:11080/ : URL interne de item-client
  - http://127.0.0.1:11081/api/v1/143519379 : URL interne de item-api
  - http://127.0.0.1:11082/ : URL interne de l'adminer
- dev :
  - http://diplotaxis1-dev.v212.abes.fr:11080/ : URL interne de item-client
  - http://diplotaxis1-dev.v212.abes.fr:11081/ : URL interne de item-api
  - http://diplotaxis1-dev.v212.abes.fr:11082/ : URL interne de l'adminer
- test :
  - http://diplotaxis1-test.v202.abes.fr:11080/ : URL interne de item-client
  - http://diplotaxis1-test.v202.abes.fr:11081/api/v1/143519379 : URL interne de item-api
  - http://diplotaxis1-test.v202.abes.fr:11082/ : URL interne de l'adminer
- prod
  - http://diplotaxis1-prod.v102.abes.fr:11080/ : URL interne de item-client
  - http://diplotaxis1-prod.v102.abes.fr:11081/api/v1/143519379 : URL interne de item-api
  - http://diplotaxis1-prod.v102.abes.fr:11082/ : URL interne de l'adminer

## Prérequis
Disposer de :
- ``docker``
- ``docker-compose``

## Installation

Déployer la configuration docker dans un répertoire :

```bash

# adaptez /opt/pod/ avec l'emplacement où vous souhaitez déployer l'application et cloner le projet

git clone https://github.com/abes-esr/item-docker.git
```
Configurer l'application depuis l'exemple du [fichier ``.env-dist``](./.env-dist) (ce fichier contient la liste des variables avec des explications et des exemples de valeurs) :

```bash
cp .env-dist .env
# personnaliser alors le contenu du .env
```
Démarrer l'application :

```bash
docker-compose up -d
```

Remarque : retirer le ``-d`` pour voir passer les logs dans le terminal et utiliser alors CTRL+C pour stopper l'application

Pour information, une base de données postgresql vide sera alors automatiquement initialisée. Ses données binaires seront placées dans le répertoire persistant suivante (attention le user unix de ce répertoire est celui du conteneur postgresql qui n'est pas le même que celui que vous utilisez pour installer l'application) : ``volumes/item-db/pgdata/``
```bash
# pour stopper l'application
docker-compose stop

# pour redémarrer l'application
cd /opt/pod/qualimarc-docker/
docker-compose restart
```
## Supervision

```bash
# pour visualiser les logs de l'appli
docker-compose logs -f --tail=100

# pour visualiser les logs d'un containeur
docker-compose logs -f --tail=100 nom_du_containeur
```

Cela va afficher les 100 dernière lignes de logs générées par l'application et toutes les suivantes jusqu'au CTRL+C qui stoppera l'affichage temps réel des logs.

## Déploiement continu

Les objectifs des déploiements continus de item sont les suivants (cf [poldev](https://github.com/abes-esr/abes-politique-developpement/blob/main/01-Gestion%20du%20code%20source.md#utilisation-des-branches)) :
- git push sur la branche ``develop`` provoque un déploiement automatique sur le serveur ``diplotaxis4-dev``
- git push (le plus couramment merge) sur la branche ``main`` provoque un déploiement automatique sur le serveur ``diplotaxis4-test``
- git tag X.X.X (associé à une release) sur la branche ``main`` permet un déploiement (non automatique) sur le serveur ``diplotaxis4-prod``

Item est déployé automatiquement en utilisant l'outil watchtower. Pour permettre ce déploiement automatique avec watchtower, il suffit de positionner à ``false`` la variable suivante dans le .env:
```env
ITEM_WATCHTOWER_RUN_ONCE=false
```

Le fonctionnement de watchtower est de surveiller régulièrement l'éventuelle présence d'une nouvelle image docker de ``item-api``, ``item-client`` et ``item-batch``, si oui, de récupérer l'image en question, de stopper le ou les les vieux conteneurs et de créer le ou les conteneurs correspondants en réutilisant les mêmes paramètres ceux des vieux conteneurs. Pour le développeur, il lui suffit de faire un git commit+push par exemple sur la branche ``develop`` d'attendre que la github action build et publie l'image, puis que watchtower prenne la main pour que la modification soit disponible sur l'environnement cible, par exemple la machine ``diplotaxis4-dev``.

Le fait de passer ``ITEM_WATCHTOWER_RUN_ONCE`` à false va faire en sorte d'exécuter périodiquement watchtower. Par défaut cette variable est à ``true`` car ce n'est pas utile voir cela peut générer du bruit dans le cas d'un déploiement sur un PC en local.


## Sauvegardes

Les éléments suivants sont à sauvegarder:
- ``.env`` : contient la configuration spécifique de notre déploiement
- ``volumes/item-db/dump/`` : contient les dumps quotidiens de la base de données postgresql de item

Le répertoire suivant est à exclure des sauvegardes :
- ``volumes/item-db/pgdata/`` : contient les données binaires de la base de données postgresql item

### Restauration depuis une sauvegarde

Réinstallez l'application item depuis la [procédure d'installation ci-dessus](#installation)

Restaurez ensuite le dernier dump de la base de données postgresql de item :
- récupérer le dernier dump généré par ``item-db-dumper`` depuis le système de sauvegarde (le fichier dump ressemble à ceci ``pgsql_item_item-db_20220801-143201.sql.gz``) et placez le fichier dump récupéré (sans le décompresser) dans ``=volumes/item-db/dump/`` sur la machine qui doit faire repartir item
- ensuite lancez uniquement les conteneurs ``item-db`` et ``item-db-dumper`` :
   ```bash
   docker-compose up -d item-db item-db-dumper
   ```
- lancez le script de restauration ``restore`` comme ceci et suivez les instructions :
   ```bash
   docker exec -it item-db-dumper restore
   ```
- C'est bon, la base de données item est alors restaurée

Lancez alors toute l'application item et vérifiez qu'elle fonctionne bien :
```bash
docker-compose up -d
```

### Mise à jour de la dernière version

Pour récupérer et démarrer la dernière version de l'application vous pouvez le faire manuellement comme ceci :
```bash
docker-compose pull
docker-compose up -d
```
Le ``pull`` aura pour effet de télécharger l'éventuelle dernière images docker disponible pour la version glissante en cours (ex: ``develop-api`` ou ``main-api``). Sans le pull c'est la dernière image téléchargée qui sera utilisée.

Ou bien [lancer le conteneur ``item-watchtower``](https://github.com/abes-esr/item-docker/blob/develop/README.md#d%C3%A9ploiement-continu) qui le fera automatiquement toutes les quelques secondes pour vous.


