# Watchtower: scope vs enable

Cette note explique le comportement quand `WATCHTOWER_SCOPE` et `WATCHTOWER_LABEL_ENABLE=true` sont utilisés ensemble.

## Ce que fait `WATCHTOWER_SCOPE`

- Le scope (`item-watchtower-scope`) définit le périmètre de conteneurs que cette instance watchtower peut considérer.
- Un conteneur hors scope est ignoré.

## Ce que fait `WATCHTOWER_LABEL_ENABLE=true`

- Ce mode active une logique "opt-in".
- Dans le scope, watchtower ne met à jour automatiquement que les conteneurs avec:
  - `com.centurylinklabs.watchtower.enable=true`
- Les conteneurs marqués `enable=false` (ou sans label `enable=true`) ne sont pas auto-mis à jour.

## Implication pour les services exclus

Pour `item-db`, `item-db-adminer`, `item-db-dumper` (et `item-watchtower`), marqués `enable=false`:

- Pas de `pull` automatique.
- Pas de redémarrage automatique lié à watchtower.
- Mise à jour uniquement manuelle (`docker compose pull ...` puis `docker compose up -d ...`).

## Résumé

- `scope` = quels conteneurs sont visibles par watchtower.
- `enable=true/false` = parmi ceux visibles, lesquels sont autorisés à l'auto-update.
