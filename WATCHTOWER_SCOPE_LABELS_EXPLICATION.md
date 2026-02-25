# Watchtower: scope et labels enable

Cette note explique la configuration actuelle utilisée dans ce dépôt.

## Ce que fait `WATCHTOWER_SCOPE`

- Le scope (`item-watchtower-scope`) définit le périmètre de conteneurs que cette instance watchtower peut considérer.
- Un conteneur hors scope est ignoré.

## Ce que fait le label `com.centurylinklabs.watchtower.enable`

- Dans la configuration actuelle, `WATCHTOWER_LABEL_ENABLE` n'est pas activé.
- Le label `com.centurylinklabs.watchtower.enable=false` est utilisé pour exclure explicitement certains services.

## Implication pour les services BDD exclus

Pour `item-db`, `item-db-adminer`, `item-db-dumper` (et `item-watchtower`), marqués `enable=false`:

- Pas de `pull` automatique.
- Pas de redémarrage automatique lié à watchtower.
- Mise à jour uniquement manuelle (`docker compose pull ...` puis `docker compose up -d ...`).

## Résumé

- `scope` = quels conteneurs sont visibles par watchtower.
- `enable=false` = conteneurs explicitement exclus de l'auto-update.
