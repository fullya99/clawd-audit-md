---
description: Raccourcit le CLAUDE.md du projet en sortant le détail vers des fiches, des règles ou des skills, avec une preuve à côté de chaque coupe.
argument-hint: [fichier ou dossier à auditer, vide = le dépôt courant]
---

Cible de l'audit : $ARGUMENTS

Lance le skill **clawd-audit-md**.

Lis d'abord son `SKILL.md`, puis applique les six phases dans l'ordre : inventaire, vérification
contre le code, classement, contradictions, restructuration, rapport. **Le but est de raccourcir**,
la vérification sert à rendre les coupes défendables. Charge le skill
`style-redaction` avant d'écrire la moindre ligne, et les références du skill seulement quand une
phase les demande.

Quatre choses coûtent cher si tu les oublies.

- **Tu vérifies que l'arbre git est propre avant de commencer.** S'il ne l'est pas, tu t'arrêtes et
  tu demandes de commiter. C'est ce qui rend l'audit annulable d'un `git checkout`.
- **Rien ne s'écrit avant le oui de l'opérateur**, à la fin de la phase 6. Le rapport se présente
  en entier, il approuve, corrige ou refuse bloc par bloc.
- **Aucun `FAUX` sans preuve relançable.** Le chemin qui n'existe pas, la ligne du manifeste, le
  comptage. Sans preuve, c'est `NON VÉRIFIABLE`, et la ligne ne bouge pas.
- **Aucune invention.** Tu déplaces, tu supprimes, tu corriges une valeur fausse par la valeur
  vraie. Tu ne reformules pas pour faire joli et tu ne complètes pas une règle incomplète.

Si le projet n'a aucun fichier d'instructions, dis-le et arrête-toi. En écrire un de zéro est un
autre travail.
