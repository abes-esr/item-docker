# Watchtower: scope uniquement

Cette note explique la configuration actuelle utilisée dans ce dépôt.

## Ce que fait `WATCHTOWER_SCOPE`

- Le scope (`item-watchtower-scope`) définit le périmètre de conteneurs que cette instance watchtower peut considérer.
- Un conteneur hors scope est ignoré.

## Résumé

- `scope` = quels conteneurs sont visibles par watchtower.
- Le dépôt n'utilise pas `WATCHTOWER_LABEL_ENABLE` ni `com.centurylinklabs.watchtower.enable`.
