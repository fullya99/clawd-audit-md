# clawd-audit-md

**Il raccourcit ton `CLAUDE.md`.** Il le lit, va chercher dans le code ce qui confirme ou
contredit chaque ligne, et te rend un plan : ce qui reste à la racine, ce qui sort, où ça sort, ce
qui s'archive, ce qui se supprime. Rien ne s'écrit avant que tu aies dit oui.

Le problème qu'il règle : ce fichier grossit tout seul et il est chargé en entier à chaque
session, avant ta première question. Chaque ligne est donc payée pour toujours, qu'elle serve ou
non. Au-delà de 200 lignes la doc officielle annonce une baisse d'obéissance, à 40 000 caractères
l'outil affiche un avertissement. Et personne ne relit les lignes du haut, donc au bout de six mois
le fichier décrit un projet qui n'existe plus.

## Ce qu'il fait, et que les autres ne font pas

**Il sait où sortir un bloc, et les sorties ne se valent pas.** Il y a quatre étages :

| Étage | Quand ça charge |
|---|---|
| `CLAUDE.md` | chaque session, toujours |
| `.claude/rules/<sujet>.md` avec `paths:` | tout seul, quand Claude touche un fichier qui matche |
| un skill | quand Claude reconnaît le cas dans sa description |
| `docs/<sujet>.md`, lien depuis la racine | seulement si Claude décide d'ouvrir le lien |

La règle qui tranche : une contrainte qui doit être **appliquée** ne va jamais dans une fiche liée,
un lien nu ne garantit rien. Elle va en règle liée à des chemins, ou en skill. La fiche porte ce
qu'on consulte. Et si aucune sortie ne la tient, elle reste à la racine, même si ça allonge.

Au passage, il te dira que tes `@import` n'allègent rien. Un import colle le contenu dans le
fichier racine au démarrage, c'est le contresens le plus répandu sur le sujet.

**Et il ne coupe rien sans preuve.** Un chemin cité doit exister, une commande citée doit être dans
le `package.json` ou le `Makefile`, un glob de règle doit matcher au moins un fichier, une
convention décrite se compte sur un échantillon. Une ligne contredite est marquée fausse **avec sa
preuve**. Une ligne qu'il n'arrive pas à trancher est marquée non vérifiable, et elle ne bouge pas.

## Installation

Chez Claude Code, par le marketplace :

```
/plugin marketplace add fullya99/clawd-audit-md
/plugin install clawd-audit-md@clawd-audit-md
```

Ou à la main, en copiant le dossier du skill, pas celui du plugin :

```bash
mkdir -p ~/.claude/skills
cp -r skills/clawd-audit-md ~/.claude/skills/
```

Chez Codex, même dossier, à la portée choisie :

```bash
cp -r skills/clawd-audit-md .agents/skills/      # portée dépôt
cp -r skills/clawd-audit-md ~/.agents/skills/    # portée utilisateur
```

Par une installation à la main, la commande `/audit-md` ne suit pas, elle n'existe qu'en
installation plugin chez Claude Code. Le skill se déclenche quand même sur une demande en
français, par exemple « audite mon CLAUDE.md ».

## Mise à jour

**Rien ne se met à jour tout seul, et deux choses sont figées, pas une.** Le clone du marketplace
reste au commit du jour où tu l'as ajouté, et la version installée reste celle du jour de
l'installation. L'auto-update existe, mais il est désactivé par défaut sur les marketplaces
tierces, donc sur celui-ci. Tu peux croire tourner sur la dernière version pendant des semaines.

```bash
claude plugin marketplace update clawd-audit-md
claude plugin update clawd-audit-md@clawd-audit-md
```

Puis redémarre Claude Code, le CLI le dit lui-même. En session ouverte, `/reload-plugins` suffit
souvent, et s'il avertit qu'il va relire la conversation, relance-le en `/reload-plugins --force`.

L'ordre compte : sans la première commande, la seconde cherche la nouvelle version dans un
catalogue qui ne la connaît pas encore.

Vérifie plutôt que de supposer :

```bash
claude plugin list
```

Une réinstallation nommée avec son marketplace fait le même travail en une commande, parce qu'elle
rafraîchit le catalogue avant de chercher, même auto-update désactivé :

```bash
claude plugin install clawd-audit-md@clawd-audit-md
```

Ça ne vaut que pour la forme complète. `claude plugin install clawd-audit-md` tout court lit le
cache sans rafraîchir.

**Pour ne plus y penser**, active l'auto-update une fois : `/plugin`, onglet **Marketplaces**,
sélectionne `clawd-audit-md`, **Enable auto-update**. Claude Code vérifiera au démarrage et te
proposera `/reload-plugins` si quelque chose a bougé. La session en cours garde ce qu'elle a chargé
au lancement, c'est voulu.

## Usage

```
/audit-md
```

Il vérifie d'abord que ton arbre git est propre, sinon il te demande de commiter. Puis il
inventorie les fichiers d'instructions, lance ses vérifications, te pose les questions des
contradictions qu'il a trouvées, et te présente un rapport en quatre morceaux : le tableau des
preuves, le diff du fichier racine, la liste des fiches à créer avec leur taille, le tableau des
suppressions. Tu approuves, tu corriges, ou tu refuses bloc par bloc. Il écrit après.

Le contrôle mécanique s'utilise aussi tout seul :

```bash
bash skills/clawd-audit-md/scripts/verif-refs.sh CLAUDE.md
bash skills/clawd-audit-md/scripts/verif-refs.sh --format=md CLAUDE.md AGENTS.md
```

Il sort 1 s'il reste une référence fausse, donc il s'accroche à un hook ou à une CI.

## Ce qu'il ne fait pas

Il n'invente rien. Il déplace, il supprime, il corrige une valeur fausse par la valeur vraie. Il
ne reformule pas une règle pour la rendre plus jolie et il ne complète pas une règle incomplète.

Il n'écrit pas un fichier d'instructions de zéro. Un projet qui n'en a pas est hors de son
périmètre, il le dit et s'arrête.

Il ne tranche jamais une contradiction tout seul, même quand une version semble évidente. Celle
qui a l'air périmée est parfois la seule que l'équipe applique.

Il ne touche à rien hors du dépôt. Un `~/.claude/CLAUDE.md` est signalé dans l'inventaire, et c'est
tout.

Il ne travaille pas sur un arbre git sale.
