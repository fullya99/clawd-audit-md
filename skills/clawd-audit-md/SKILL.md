---
name: clawd-audit-md
description: "Raccourcit le CLAUDE.md d'un projet en sortant le détail vers des fiches liées, des règles liées à des chemins ou des skills, pour que la racine ne porte plus que ce qui doit être vrai à chaque session. Confronte chaque ligne au code réel avant de la déplacer, avec une preuve à côté de chaque coupe, et ne touche pas à ce qu'il ne peut pas vérifier. Traite aussi les AGENTS.md et les fichiers de sous-agents. N'écrit rien tant que le rapport n'a pas été validé. Se déclenche sur : mon CLAUDE.md est trop long, optimise mon CLAUDE.md, allège mes instructions, audit CLAUDE.md, sortir le contenu vers docs, est-ce que les règles sont encore vraies, CLAUDE.md obsolète, nettoyer AGENTS.md, réduire le contexte charge à chaque session, /audit-md."
version: 1.1.1
---

# Clawd audit md

> **Le contrat** : un fichier racine court, où chaque ligne restante est vraie et prouvée, et tout
> le reste rangé là où ça ne coûte rien tant que ça ne sert pas. Rien n'est écrit avant que
> l'opérateur ait lu le rapport et dit oui.

**Le but est de raccourcir.** Un fichier d'instructions grossit tout seul, personne ne relit les
lignes du haut, et il finit par décrire un projet qui n'existe plus. Il est chargé en entier à
chaque session, avant la première question, donc chaque ligne est payée pour toujours. Au-delà de
200 lignes la doc annonce une baisse d'obéissance, à 40 000 caractères l'outil affiche un
avertissement.

La bonne image : ce fichier est de la mémoire vive, pas du disque. Presque tout ce qu'il contient
peut vivre ailleurs et ne charger que quand ça sert, dans une règle liée à des chemins, dans un
skill, ou dans une fiche liée. Il y a quatre étages, ils ne se valent pas, et choisir le bon est
le vrai travail. C'est dans `references/les-quatre-etages.md`.

**La vérification sert à ça, elle n'est pas le but.** On ne déplace pas une ligne qu'on n'a pas
comprise, et on ne supprime pas une ligne fausse sans montrer en quoi elle l'est. La preuve est ce
qui rend la coupe acceptable, pas l'objectif du skill.

**Avant d'écrire quoi que ce soit**, charge le skill **`style-redaction`**. S'il n'est pas
installé, le minimum tient en une ligne : français, ton direct, aucun tiret cadratin, aucun
point-virgule, aucune tournure qui sent le texte généré.

---

## Quand ce skill s'applique

| Situation | Ce que tu fais |
|---|---|
| « il est trop long », « allège-le », « optimise mon CLAUDE.md » | les six phases, en entier. C'est le cas courant |
| « audite mon CLAUDE.md », « il est faux » | pareil, les six phases. Le tri produit le raccourcissement |
| « vérifie juste si c'est encore vrai » | phases 1 et 2 seulement, tu rends le tableau des preuves et tu t'arrêtes |
| « sors ça dans docs/ » sur un bloc précis | phases 3 et 5 sur ce bloc, la preuve reste obligatoire |
| Le projet n'a aucun fichier d'instructions | tu ne fais rien, tu le dis. Écrire un CLAUDE.md de zéro est un autre travail |
| On te demande d'ajouter une règle | hors périmètre. Ce skill déplace et supprime, il n'invente pas |

Le tableau est le routeur. Si la demande n'y entre pas, dis-le plutôt que de forcer.

---

## Sur quel harnais tu tournes

Les fichiers d'instructions ne portent pas le même nom selon le lecteur, et un même dépôt en a
souvent plusieurs. Tu les traites tous de la même façon.

| Lecteur | Dans le dépôt | Hors dépôt, à signaler sans y toucher |
|---|---|---|
| Claude Code | `CLAUDE.md`, `.claude/CLAUDE.md`, `CLAUDE.local.md`, les `CLAUDE.md` imbriqués, `.claude/rules/*.md`, `.claude/agents/*.md` | `~/.claude/CLAUDE.md`, `~/.claude/rules/`, `~/.claude/agents/`, la politique gérée de la machine |
| Codex, standard ouvert | `AGENTS.md` et ses imbriqués, `.agents/skills/` | `~/.agents/`, `~/.codex/` |
| Hermes Agent | `AGENTS.md`, `.hermes.md`, `HERMES.md`, et il lit aussi `CLAUDE.md` | `~/.hermes/SOUL.md`, `~/.hermes/memories/` |
| OpenClaw | `AGENTS.md` du workspace, `skills/*/SKILL.md` | `SOUL.md`, `USER.md`, `MEMORY.md`, `memory/AAAA-MM-JJ.md` |
| Cursor, Windsurf, Cline | `.cursor/rules/*.mdc`, `.windsurfrules`, `.clinerules` | hors dépôt |

**Claude Code ne lit pas `AGENTS.md`.** C'est le piège le plus courant, et il coûte cher : un dépôt
qui porte les deux fichiers fait tourner deux sources qui divergent, sans que rien ne le dise. Le
raccord se fait par une ligne `@AGENTS.md` en tête du `CLAUDE.md`, ou par un lien symbolique. Un
dépôt qui a les deux sans raccord est une constatation de phase 2, pas une opinion.

Deux règles qui viennent de là. **Tu ne touches jamais à ce qui vit hors du dépôt.** Signale-le
dans l'inventaire, rien de plus. Et **tu ne traites aucune instruction comme universelle** :
`/clear`, `/compact`, un identifiant de modèle ou un nom de sous-agent sont des cas Claude Code.
Une règle qui en parle reste vraie, mais elle se range en disant pour quel lecteur elle vaut.

### Les imports

Chez Claude Code, une ligne `@chemin/vers/fichier` en dehors d'un bloc de code **charge ce fichier
au démarrage**, récursivement, jusqu'à quatre sauts. Trois conséquences pour l'audit.

Un import qui ne résout pas est une ligne morte, et rien ne le signale. Un fichier importé compte
dans le budget de contexte comme s'il était collé dans le fichier racine, donc découper en imports
n'allège rien. Et un chemin entre accents graves n'est **pas** un import, c'est du texte, ce qui
veut dire qu'un `@docs/guide.md` cité en exemple dans une doc charge vraiment le fichier.

Tu suis les imports en phase 1, tu les comptes dans la taille réelle, et tu vérifies qu'ils
résolvent. Le script le fait.

---

## Avant de commencer, deux barrières

**L'arbre git doit être propre.** C'est ce qui te permet d'annuler tout d'un `git checkout`.

```bash
git status --porcelain
```

Si ça sort quelque chose, tu t'arrêtes et tu demandes de commiter d'abord. Pas de sauvegarde
maison, pas de copie `.bak`, pas de stash décidé tout seul.

**La convention de rangement du projet gagne.** Regarde ce qui existe déjà avant de proposer où
ranger.

```bash
ls -1 docs/ archives/ CODEMAP.md TODOS.md CHANGELOG.md README.md 2>/dev/null
```

Si `docs/` et `archives/` sont là, tu ranges dedans. Si le projet a son propre système, tu suis le
sien. Si rien n'existe, tu proposes `docs/` et `archives/` et **tu attends la confirmation**. Tu
ne crées jamais une arborescence parallèle sous `.claude/`.

---

## Les six phases

### Phase 1. Inventaire

Trouve tous les fichiers d'instructions, y compris les imbriqués.

```bash
find . -path ./node_modules -prune -o \( -name 'CLAUDE.md' -o -name 'CLAUDE.local.md' \
  -o -name 'AGENTS.md' -o -name '.clinerules' -o -name '.windsurfrules' \
  -o -path '*/.claude/rules/*.md' -o -path '*/.claude/agents/*.md' \
  -o -path '*/.cursor/rules/*' \) -print 2>/dev/null

for f in $(find . -name 'CLAUDE.md' -o -name 'AGENTS.md' 2>/dev/null); do
  printf '%s\t%s lignes\t%s\n' "$f" "$(wc -l < "$f")" "$(git log -1 --format=%ad --date=short -- "$f")"
done

grep -nE '^[[:space:]]*@[^[:space:]`]+' CLAUDE.md 2>/dev/null   # les imports
```

Pour chacun : taille en lignes, date du dernier commit qui l'a touché, portée de chargement.
Un `CLAUDE.md` imbriqué ne se charge que quand on travaille dans son dossier, donc il n'a pas le
même coût qu'un fichier racine. Un fichier importé, lui, coûte comme s'il était collé dans la
racine, donc **sa taille compte dans le total**.

Relève aussi ce qui existe déjà comme documentation dédiée, `docs/`, `CODEMAP.md`, `README.md`,
les ADR. C'est ce qui t'évitera de créer une fiche qui redit ce qui est déjà écrit ailleurs. Une
ligne du CLAUDE.md dont le contenu existe déjà dans `docs/` ne part pas en fiche, elle devient un
lien.

### Phase 2. Vérification contre le code

C'est ce qui rend les coupes défendables, et la partie que les autres outils ne font pas. Lance
d'abord le contrôle mécanique, il traite les chemins, les commandes, les globs et les imports.

```bash
bash "$SKILL/scripts/verif-refs.sh" CLAUDE.md
bash "$SKILL/scripts/verif-refs.sh" --format=md CLAUDE.md   # tableau markdown, pour le rapport
```

`$SKILL` est le répertoire de ce skill. Le skill s'exécute depuis le projet de l'utilisateur, donc
un chemin relatif du genre `scripts/verif-refs.sh` ne résout pas. Résous-le une fois, avant le
premier appel :

```bash
ANCETRES="$(d="$PWD"; while [ "$d" != "/" ]; do echo "$d/.agents/skills/clawd-audit-md"; d="$(dirname "$d")"; done)"

SKILL="$(for d in "$CLAUDE_PLUGIN_ROOT/skills/clawd-audit-md" \
  ".claude/skills/clawd-audit-md" $ANCETRES \
  "$HOME/.claude/skills/clawd-audit-md" \
  "$HOME/.codex/skills/clawd-audit-md" "$HOME/.agents/skills/clawd-audit-md" \
  $(find "$HOME/.claude/plugins/cache" -maxdepth 5 -type d -path '*/skills/clawd-audit-md' 2>/dev/null | sort -r) \
  $(find "$HOME/.claude/plugins/marketplaces" -maxdepth 5 -type d -path '*/skills/clawd-audit-md' 2>/dev/null) \
  $(find "$HOME/.hermes/skills" "$HOME/.openclaw" -maxdepth 4 -type d -name clawd-audit-md 2>/dev/null) \
  $(find skills -maxdepth 2 -type d -name clawd-audit-md 2>/dev/null); do
  [ -f "$d/scripts/verif-refs.sh" ] && echo "$d" && break
done)"
```

L'ordre suit les portées : le plugin d'abord, puis le projet pour Claude Code et pour Codex, puis
l'utilisateur pour les deux, puis les copies propres à Claude Code, `cache/` avant `marketplaces/`
parce que le cache porte la version installée. Codex n'a pas d'équivalent de `CLAUDE_PLUGIN_ROOT`,
cette boucle est son seul mécanisme de résolution. Si elle échoue, tu fais les vérifications à la
main, le script fait gagner du temps mais il n'est pas la règle.

Le script ne couvre pas tout. Ce qui reste à ta charge :

| Affirmation | Comment tu la vérifies |
|---|---|
| Une dépendance, un framework, une version | tu lis le manifeste réel et tu compares la valeur, pas juste le nom |
| Un `paths:` de règle, un glob | il doit matcher au moins un fichier, sinon la règle est morte |
| Une convention de code (« les services exposent `configure()` ») | tu échantillonnes et tu comptes. « 3 fichiers sur 40 » est la preuve |
| Une convention de nommage (« composants en PascalCase ») | pareil, un comptage, jamais une impression |
| Une règle d'organisation (« le métier ne connaît pas la base ») | un grep ciblé qui la contredirait. S'il ne sort rien, c'est confirmé |

Trois verdicts, et un seul par affirmation.

- **`VRAI`** : le dépôt confirme.
- **`FAUX`** : le dépôt contredit, **et tu tiens la preuve**. Le chemin qui n'existe pas, la ligne
  du manifeste, le comptage. Une preuve, c'est quelque chose que l'opérateur peut relancer.
- **`NON VÉRIFIABLE`** : tu n'as pas trouvé de quoi trancher. C'est un verdict honnête, pas un
  échec. Une affirmation invérifiable **n'est jamais marquée faux par défaut** et **reste où elle
  est**.

Le taux de respect compte dans le verdict. Une convention suivie à 90 % est vraie, la règle
tient. À 8 %, elle est fausse, et la preuve est le comptage. Entre les deux, tu poses la question
en phase 4 au lieu de trancher.

### Les fichiers d'agents

Un dépôt qui déclare des sous-agents porte un second jeu de fichiers d'instructions, et celui-là
casse en silence. Un fichier d'agent mal formé n'est pas chargé, et rien ne le dit : pas d'erreur,
pas d'avertissement, l'agent n'existe simplement pas dans la liste.

```bash
bash "$SKILL/scripts/verif-agents.sh" .claude/agents/
```

Le script vérifie les quatre façons de disparaître sans bruit, plus les valeurs d'énumération qui
ne veulent rien dire. Le détail, avec la table complète des clés admises, est dans
`references/fichiers-agents.md`.

Trois vérifications restent à ta charge, elles demandent de lire.

| À vérifier | Comment |
|---|---|
| La `description` déclenche vraiment | c'est elle seule qui décide de la délégation. Une description de trois mots ne se déclenche jamais |
| Deux agents ne se marchent pas dessus | deux descriptions proches, et le mauvais agent gagne une fois sur deux. Tu poses la question en phase 4 |
| Le corps ne redit pas le `CLAUDE.md` | un agent hérite déjà des instructions du projet. Ce qu'il répète coûte deux fois |

**Tu ne réécris jamais le corps d'un fichier d'agent.** Un frontmatter cassé se corrige, parce que
la valeur juste est vérifiable. Le prompt d'un agent est un choix de son auteur, pas une donnée.

### Phase 3. Classement, bloc par bloc

Un bloc, c'est une règle, pas une ligne de texte. Une puce, un paragraphe, une section courte.
Chaque bloc reçoit exactement une étiquette.

| Étiquette | Critère | Destination |
|---|---|---|
| `RACINE` | vrai, et vrai partout dans le dépôt, tout le temps | reste dans le fichier racine |
| `SORTIR` | vrai, mais ne sert que pour une tâche, un dossier ou un module | un des trois étages du bas, voir ci-dessous |
| `ARCHIVE` | était vrai, et garde une valeur de trace, une décision passée | `archives/`, avec sa date et la raison |
| `SUPPRIMER` | faux sans rien à garder, ou vide de sens | supprimé, listé dans le rapport avec la preuve |

**`SORTIR` est l'étiquette qui fait le travail**, et elle demande un second choix. Trois sorties,
elles ne se valent pas.

| Sortie | Où | Quand ça charge |
|---|---|---|
| Règle liée à des chemins | `.claude/rules/<sujet>.md` avec `paths:` | toute seule, quand l'agent touche un fichier qui matche |
| Skill | `.claude/skills/<nom>/SKILL.md` | quand l'agent reconnaît le cas dans sa `description` |
| Fiche | `docs/<sujet>.md`, lien depuis la racine | seulement si l'agent décide d'ouvrir le lien |

La règle qui tranche : **une contrainte qui doit être appliquée ne va jamais dans une fiche.** Un
lien nu ne garantit rien. Elle va en règle liée à des chemins si elle est propre à un dossier, en
skill si c'est une procédure. La fiche porte ce qu'on consulte, du contexte, une décision passée,
un piège documenté.

Et si aucune sortie ne tient la contrainte, elle **reste à la racine**, même si ça allonge le
fichier. Une contrainte perdue coûte plus cher que dix lignes.

Le détail, le classement des sorties, ce que chaque lecteur sait faire, et comment écrire un lien
qui se fait vraiment ouvrir, sont dans `references/les-quatre-etages.md`. Les cas limites du
classement sont dans `references/criteres-classement.md`.

Va dans `SUPPRIMER` ce que l'agent fait déjà par défaut (« écris du code lisible »), ce qu'un
linter ou un formateur applique déjà, ce qui répète le langage du projet (« utilise TypeScript »
dans un projet TypeScript), ce qui est trop vague pour être suivi (« sois rigoureux ») et les
règles formulées seulement en négatif sans dire quoi faire à la place. La liste complète, avec
pour chaque forme la raison, est dans `references/anti-patterns.md`.

Une règle fausse qui redevient vraie en corrigeant une valeur n'est pas à supprimer. Tu corriges
la valeur, tu gardes la règle, et tu montres l'ancienne et la nouvelle dans le rapport.

### Phase 4. Contradictions

Dresse la liste des règles qui s'opposent. Deux blocs qui disent l'inverse, un fichier imbriqué
qui contredit la racine, une règle qui interdit ce qu'une autre impose.

Pour chacune, **tu poses la question à l'opérateur**. Tu montres les deux versions, leur emplacement
et leur date, et tu t'arrêtes là. **Tu ne tranches jamais tout seul**, même quand une version te
semble évidente. Celle qui a l'air périmée est parfois la seule que l'équipe applique.

Une contradiction non tranchée bloque le classement des deux blocs concernés. Elle ne bloque pas
le reste de l'audit.

### Phase 5. Restructuration

Le fichier racine vise **50 lignes**, plafond dur **100**. C'est plus serré que les 200 lignes
conseillées par la doc, et c'est voulu : 200 lignes est le seuil où ça se dégrade, pas une cible.
Il porte quatre choses et rien d'autre : ce qu'est le projet en deux phrases, les commandes
réelles, les contraintes qui n'ont aucune exception, et la table de renvoi.

**La table de renvoi est ce qui fait tenir les 50 lignes.** Chaque ligne dit la situation avant de
donner le chemin, sinon elle ne se fait pas ouvrir.

```markdown
| Quand | Où |
|---|---|
| Tu ajoutes ou modifies une migration | `docs/migrations.md` |
| Un test passe en local et échoue en CI | `docs/ci-flaky.md` |
```

Chaque fiche commence par dire quand elle s'applique, puis donne les règles. Une fiche traite un
sujet, et un sujet c'est ce qu'on a en tête en même temps. Au-delà de 150 lignes, coupe en deux.
En dessous de 15, ne crée pas la fiche, ces lignes rejoignent une fiche voisine ou restent à la
racine.

Une règle qui part en `.claude/rules/` porte son `paths:`, et tu vérifies que le glob matche au
moins un fichier, sinon la règle est morte à la naissance.

**N'utilise jamais un `@import` pour alléger.** Il colle le contenu dans le fichier racine au
démarrage. Si le fichier audité en contient déjà, dis à l'opérateur qu'ils ne l'allègent pas.

Les gabarits sont dans `references/modeles.md`.

**Tu ne casses pas les liens.** Quand un contenu se déplace, tu repasses sur tout le dépôt et tu
corriges les références qui pointaient vers son ancien emplacement.

```bash
grep -rn "CLAUDE.md" --include='*.md' --include='*.yml' --include='*.yaml' . | grep -v node_modules
```

Et tu vérifies que chaque lien du nouveau fichier racine pointe vers un fichier qui existe. Le
script le fait, relance-le sur le résultat :

```bash
bash "$SKILL/scripts/verif-refs.sh" CLAUDE.md
```

### Phase 6. Rapport, puis validation

Tu présentes le plan complet **avant d'écrire quoi que ce soit**. Quatre morceaux, dans cet ordre.

1. Le tableau des vérifications, une ligne par affirmation, avec le verdict et la preuve à côté.
2. Le diff du fichier racine, avec son compte de lignes avant et après, et le gain en pourcentage.
3. La liste de ce qui sort, avec pour chaque morceau son étage de destination et son nombre de
   lignes. C'est la partie que l'opérateur lit en premier.
4. Le tableau des suppressions, une ligne par bloc supprimé, avec la preuve.

Puis tu attends. L'opérateur approuve, corrige, ou refuse bloc par bloc. **Tu n'écris qu'après.**

Une fois l'écriture faite, relance l'audit. Un deuxième passage sur un projet déjà traité ne doit
rien changer. S'il propose encore quelque chose, c'est que la phase 3 a mal classé, pas que le
projet a bougé.

---

## Invariants, jamais négociables

- ❌ **Aucune écriture avant validation du rapport.** Y compris une « petite correction évidente ».
- ❌ **Aucun `FAUX` sans preuve relançable.** Sans preuve, c'est `NON VÉRIFIABLE`, et ça ne bouge pas.
- ❌ **Aucune invention.** Tu ne reformules pas une règle pour la rendre plus jolie, tu ne complètes
  pas une règle incomplète, tu n'ajoutes pas une règle manquante. Tu déplaces, tu supprimes, tu
  corriges une valeur fausse par la valeur vraie. C'est tout.
- ❌ **Aucun travail sur un arbre git sale.** Tu demandes de commiter d'abord.
- ❌ **Aucune contradiction tranchée sans l'opérateur.**
- ❌ **Aucun lien laissé cassé** après un déplacement.
- ❌ **Aucune arborescence parallèle créée** quand le projet a déjà sa convention.
- ❌ **Aucune modification hors du dépôt.** Ce qui vit dans le home se signale, il ne se touche pas.
- ❌ **Aucune réécriture du corps d'un fichier d'agent.** Le frontmatter se corrige, le prompt non.
- ❌ **Aucun texte qui sent le généré.** Voir le skill `style-redaction`.

---

## Références, à charger seulement au besoin

| Fichier | Quand le lire |
|---|---|
| `references/les-quatre-etages.md` | phases 3 et 5, choisir où sort un bloc. La fiche la plus utile du lot |
| `references/criteres-classement.md` | phase 3, pour trancher entre `SORTIR` et `ARCHIVE`, ou sur un cas limite |
| `references/anti-patterns.md` | phase 3, pour justifier un `SUPPRIMER` autrement qu'à l'instinct |
| `references/modeles.md` | phase 5, gabarit du fichier racine et gabarit d'une fiche |
| `references/fichiers-agents.md` | phase 2, quand le dépôt déclare des sous-agents |
| `scripts/verif-refs.sh` | phase 2 et phase 5, chemins, commandes, globs et imports d'un `.md` |
| `scripts/verif-agents.sh` | phase 2, frontmatter des fichiers d'agents |
