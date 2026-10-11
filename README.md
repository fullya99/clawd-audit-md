# clawd-audit-md est remplacé par clawd-stack

Ce dépôt n'est plus maintenu. Le plugin `clawd-audit-md` est absorbé par `clawd-stack`, qui reprend ses
commandes et son skill avec le reste de la convention de contexte.

## Passer à clawd-stack

```
/plugin marketplace add fullya99/clawd-stack
/plugin install clawd-stack@clawd-stack
/plugin marketplace remove clawd-audit-md
```

La dernière commande retire aussi l'ancien plugin.

## Si tu as déjà mis à jour ce marketplace

Le champ `renames` de ce dépôt renomme l'ancien plugin en `clawd-stack@clawd-audit-md` dans tes
réglages, mais ne télécharge pas le nouveau. Il ne se charge qu'après un
`/plugin install clawd-stack@clawd-audit-md`. Les trois commandes ci-dessus restent la voie propre.

## Les anciennes versions

Les tags de ce dépôt restent installables. Pour rester sur une version précise, ajoute le
marketplace avec son tag, sous la forme `/plugin marketplace add fullya99/clawd-audit-md#<tag>`.
