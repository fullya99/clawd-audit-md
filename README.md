# clawd-audit-md

Un plugin qui range les fichiers d'instructions d'un projet. Il lit ton `CLAUDE.md` ou ton
`AGENTS.md`, va chercher dans le code ce qui confirme ou contredit chaque ligne, et te rend un
plan de redistribution : ce qui reste à la racine, ce qui part en fiche, ce qui s'archive, ce qui
se supprime. Rien ne s'écrit avant que tu aies dit oui.

Le problème qu'il règle : ces fichiers grossissent tout seuls. Personne ne relit les lignes du
haut, et au bout de six mois le fichier décrit un projet qui n'existe plus. Chaque ligne fausse se
paie deux fois, en tokens à chaque session et en erreurs que l'agent commet parce qu'il y croit.

## Ce qu'il fait, et que les autres ne font pas

Il confronte au code. Un chemin cité doit exister, une commande citée doit être dans le
`package.json` ou le `Makefile`, un glob de règle doit matcher au moins un fichier, une convention
décrite se compte sur un échantillon. Une affirmation contredite est marquée fausse **avec sa
preuve**. Une affirmation qu'il n'arrive pas à trancher est marquée non vérifiable et ne bouge
pas, jamais fausse par défaut.

Le reste suit : classement bloc par bloc, liste des règles qui se contredisent, fichier racine
ramené sous 100 lignes, fiches `docs/` pour le détail, en-tête daté pour ce qui s'archive.

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
