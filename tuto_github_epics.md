# Tutoriel — Gérer le backlog agile (EPICs / User Stories) sur GitHub en multi-repo

**Contexte :** Titre professionnel CDA (niveau 6) — Bloc 1 « Concevoir et spécifier une solution digitale », projet imposé **« Le Grand Cinéma »**.
**Objectif :** remplacer Trello par GitHub Issues + GitHub Projects pour produire le backlog attendu par le jury, avec un lien public consultable sans authentification.
**Organisation GitHub :** `Grand-Cinema-Studi` — c'est elle qui possède les dépôts **et** le Project.
**Compte authentifié dans `gh` :** ton compte personnel, distinct de l'organisation. Voir section 5.5, c'est le piège n° 1.
**Dépôts :** `grand-cinema-frontend`, `grand-cinema-backend`, `grand-cinema-docs`.
**Shell :** **Git Bash sous Windows** — toutes les commandes de ce tutoriel sont en Bash.
**Rédigé le :** 19 septembre 2026 — commandes vérifiées sur GitHub CLI ≥ 2.94.
**Mis à jour le :** 8 octobre 2026 — backlog v2 aligné sur les copies des Blocs 1 et 2 : **5 EPICs, 14 User Stories (11 MVP, 3 V2), 89 points**. L'ancien backlog (4 EPICs, 12 US) est mis en conformité en modifiant, fermant ou créant les issues : voir l'étape 4 bis.

---

## Sommaire

1. Ce que le jury attend réellement
2. Comment GitHub remplace Trello (modèle de données)
3. Architecture du backlog sur 3 dépôts — **la décision structurante**
4. Décision secondaire : label `epic` ou type d'issue natif
5. Prérequis, installation et spécificités Git Bash
6. Étape 1 — Préparer les 3 dépôts
7. Étape 2 — Créer le Project transverse et le rendre public
8. Étape 3 — Configurer les champs de pilotage
9. Étape 4 — Créer les labels dans les 3 dépôts (EPIC, version, profil)
9 bis. Étape 4 bis — Mettre le backlog existant en conformité avec les copies
10. Étape 5 — Mettre à jour les EPICs (contenu prêt à coller)
11. Étape 6 — Mettre à jour les 14 User Stories
12. Étape 7 — Décomposer en tâches techniques cross-repo
13. Étape 8 — Script Bash de mise en conformité
14. Étape 9 — Configurer les vues du Project
15. Étape 10 — Publier et récupérer le lien pour les dossiers Word
16. Checklist de conformité jury
17. Erreurs fréquentes et dépannage
18. Annexe A — Tableau de chiffrage consolidé
19. Annexe B — Matrice de décomposition US → tâches techniques

---

## 1. Ce que le jury attend réellement

Le référentiel CDA n'impose aucun outil. Il impose des **preuves**. Trello ou GitHub, peu importe : ce sont les artefacts suivants qui sont notés.

| Attendu du référentiel | Preuve à produire sur GitHub | Où ça se voit |
| :--- | :--- | :--- |
| Backlog hiérarchisé | Issues EPIC avec sub-issues | Onglet Issues + vue Table |
| User Stories formalisées | Corps d'issue « En tant que… je veux… afin de… » | Chaque issue US |
| Critères d'acceptation | Section Gherkin dans chaque US | Chaque issue US |
| Chiffrage / estimation | Champ `Story Points` du Project | Vue Table, colonne dédiée |
| Priorisation justifiée | Champ `Priorité (MoSCoW)` + champ `Version` | Vue Table groupée |
| Suivi d'avancement agile | Vue Board (kanban) + itérations | Vue Board |
| Architecture technique | Répartition des tâches sur les 3 dépôts | Colonne `Repository` du Project |
| Accessibilité du jury | Project en visibilité **Public** | Lien dans le dossier Word |

> **Point critique :** un backlog magnifique mais privé vaut zéro. Le correcteur ouvre le lien sans être connecté à ton compte. Si la page affiche un 404, la compétence n'est pas démontrée. L'étape 10 traite ce point en détail — ne la saute pas.

---

## 2. Comment GitHub remplace Trello (modèle de données)

| Notion agile | Trello | GitHub |
| :--- | :--- | :--- |
| EPIC | Carte dans une liste « EPICs » | **Issue** labellisée `epic` dans le dépôt pilote |
| User Story | Carte | **Issue** labellisée `user-story`, rattachée à l'EPIC en **sub-issue** |
| Tâche technique | Checklist dans la carte | **Issue** dans le dépôt concerné, sub-issue de l'US |
| Critères d'acceptation | Checklist | Section markdown avec cases à cocher |
| Estimation | Plugin payant | **Champ numérique** natif du Project |
| Colonnes du tableau | Listes | Champ `Status` affiché en vue **Board** |
| Sprint | Plugin | Champ **Iteration** natif |
| Traçabilité code ↔ besoin | Inexistante | Commit `Closes #12` qui ferme automatiquement la tâche |

**L'argument décisif à l'oral :** sur GitHub, un commit contenant `Closes #12` ferme l'issue n° 12 et la déplace en « Done ». Tu démontres la traçabilité **besoin → spécification → code → livraison** à travers trois dépôts distincts. Trello ne permet rien de tel.

**Les trois briques à ne pas confondre :**

- **Repository** — contient le code *et* les Issues. Une issue appartient toujours à un dépôt précis.
- **Issue** — une unité de travail (EPIC, US ou tâche). C'est là que vit le contenu rédigé.
- **Project** — une vue de pilotage **transverse aux 3 dépôts** qui agrège des issues et leur ajoute des champs. Le Project ne contient pas les issues, il les référence. C'est cette propriété qui rend le multi-repo indolore.

---

## 3. Architecture du backlog sur 3 dépôts — la décision structurante

Tu disposes de trois dépôts. La question n'est donc plus « où créer le repo » mais **« dans quel dépôt vit chaque type d'issue »**. Répondre au hasard produit un backlog éclaté et illisible ; répondre correctement transforme la contrainte multi-repo en argument d'architecture devant le jury.

### 3.1 Le principe : séparer le besoin de son implémentation

```
grand-cinema-docs        →  LE BESOIN         →  5 EPICs + 14 User Stories
       │                                          (métier, indépendant de la technique)
       ├── sub-issues ──> grand-cinema-backend →  Tâches techniques API, BDD, sécurité
       └── sub-issues ──> grand-cinema-frontend → Tâches techniques UI, PWA, intégration
```

| Dépôt | Rôle dans le backlog | Contenu |
| :--- | :--- | :--- |
| **`grand-cinema-docs`** | **Dépôt pilote du backlog** | Les 5 EPICs, les 14 User Stories, les livrables du dossier (UML, MCD/MLD, maquettes) |
| **`grand-cinema-backend`** | Dépôt d'implémentation serveur | Tâches techniques : endpoints API, schéma BDD, verrous de concurrence, passerelle de paiement |
| **`grand-cinema-frontend`** | Dépôt d'implémentation client | Tâches techniques : écrans, composants, PWA de scan, responsive |

**Pourquoi les US vivent dans `docs` et non dans le dépôt technique :** une User Story exprime un besoin utilisateur, pas une technologie. « Réserver un nombre de places et passer au paiement » mobilise simultanément le frontend (choix du nombre de places, compte à rebours) et le backend (blocage en base, transaction). La placer dans l'un des deux dépôts serait un choix arbitraire qui casserait la lecture du backlog. Elle reste donc dans le dépôt pilote, et ses **tâches techniques** descendent dans les dépôts concernés.

Cette séparation est exactement ce que le jury veut entendre quand il demande de justifier une architecture en trois dépôts. Prépare cette phrase :

> « Le dépôt `docs` porte la spécification et le backlog fonctionnel, indépendants de toute technologie. Les dépôts `frontend` et `backend` portent l'implémentation. La hiérarchie de sub-issues traverse les trois dépôts, ce qui garantit qu'aucune tâche technique n'existe sans être rattachée à un besoin utilisateur formalisé. »

### 3.2 La hiérarchie sur trois niveaux

```
[EPIC 2] Réservation et choix des places en temps réel        (docs)
   └── [US 2.3] Réserver un nombre de places et passer au paiement    (docs)
          ├── [BACK] Bloquer les places en base pendant 10 minutes   (backend)
          └── [FRONT] Compte à rebours du blocage                    (frontend)
```

GitHub autorise **8 niveaux d'imbrication et 100 sub-issues par parent** : trois niveaux passent largement. Et les sub-issues **franchissent les frontières de dépôt** — c'est la fonctionnalité qui rend toute cette architecture possible.

### 3.3 Ce qui change concrètement par rapport à un dépôt unique

Trois conséquences pratiques, chacune source d'erreur si elle est ignorée :

1. **Les labels sont propres à chaque dépôt.** Créer le label `epic` dans `docs` ne le crée pas dans `backend`. L'étape 4 boucle donc sur les trois dépôts.
2. **Le Project doit être lié aux trois dépôts** pour que les issues des trois remontent dans les mêmes vues.
3. **Le rattachement parent-enfant cross-repo exige une URL complète**, pas un numéro d'issue. Le numéro `#12` est ambigu entre trois dépôts ; l'URL ne l'est jamais.

### 3.4 Variante si tu manques de temps

Créer 5 EPICs + 14 US + une vingtaine de tâches représente une quarantaine d'issues. Si le 20 octobre approche, applique cette priorité :

- **Obligatoire pour les Blocs 1 et 2 :** les 5 EPICs et les 14 US dans `docs`, identiques aux copies et chiffrés. C'est ce qui est noté.
- **Fortement recommandé :** les tâches techniques d'**une seule US** (l'US 2.3, la plus riche techniquement), pour démontrer la méthode de décomposition et l'usage des trois dépôts.
- **À faire pendant le Bloc 3 :** les tâches des autres US, créées au fil du développement.

Un backlog où une US est décomposée jusqu'à la tâche et les autres restent au niveau US se défend parfaitement : c'est le fonctionnement normal d'une équipe agile, qui n'affine que le haut du backlog.

---

## 4. Décision secondaire : label `epic` ou type d'issue natif

`Grand-Cinema-Studi` étant une **organisation**, les **types d'issues natifs** (`Epic`, `Feature`, `Task`) te sont accessibles — ils se configurent dans **Settings → Planning → Issue types** de l'organisation, où l'on peut créer jusqu'à 25 types personnalisés. Le choix est donc ouvert.

**Option A — le label `epic` (retenue ici).** L'EPIC est matérialisé par un label et par la hiérarchie de sub-issues. Le jury voit la hiérarchie, la barre de progression automatique de chaque EPIC et tous les champs de chiffrage. **Rien de ce qui est noté ne manque.**

**Option B — le type d'issue natif `Epic`.** Plus élégant : une pastille typée plutôt qu'un label, et un filtrage par type. Mise en œuvre : créer le type dans les réglages de l'organisation, puis l'appliquer aux issues existantes.

```bash
gh issue edit <numero> --repo "$REPO_DOCS" --type "Epic"
```

> **Recommandation :** reste sur l'option A jusqu'au dépôt du 20 octobre. Deux raisons concrètes. D'abord le gain est esthétique, pas académique : aucun critère du référentiel ne distingue un label d'un type. Ensuite, les types d'issues relèvent de la même famille que les **champs d'issue**, qui peuvent bloquer la publication du Project en visibilité publique (voir section 15.1) — or c'est cette publication qui conditionne l'accès du jury. Ne complexifie pas ce qui marche avant d'avoir sécurisé le dépôt.

---

## 5. Prérequis, installation et spécificités Git Bash

### 5.1 Installer GitHub CLI

`winget` est un exécutable Windows, accessible depuis Git Bash :

```bash
winget install --id GitHub.cli -e
```

Ferme puis rouvre le terminal. Il faut la version **2.94 minimum** pour les flags `--parent` et `--type` :

```bash
gh --version
winget upgrade --id GitHub.cli -e   # si version inférieure
```

### 5.2 S'authentifier avec le bon scope

Les commandes `gh project` exigent le scope `project`, absent par défaut. C'est la cause n° 1 des échecs de script.

```bash
gh auth login
gh auth refresh -s project,read:project
gh auth status
```

La sortie doit mentionner `project` et `read:project`.

> **Spécificité Git Bash :** si `gh auth login` reste figé sans afficher le menu interactif, c'est le problème classique des TTY sous MinTTY. Préfixe la commande par `winpty` :
> ```bash
> winpty gh auth login
> ```

### 5.3 Les trois pièges Git Bash à connaître

**Piège 1 — l'affectation de variable.** C'est l'erreur la plus fréquente quand on vient de PowerShell :

```bash
PROJECT=2          # CORRECT : pas de $, pas d'espace autour du =
$PROJECT = 2       # FAUX : "bash: =: command not found"
PROJECT = 2        # FAUX aussi : bash cherche une commande nommée PROJECT
```

Le `$` ne sert **qu'à lire** une variable, jamais à l'affecter.

**Piège 2 — la conversion automatique des chemins.** Git Bash (MSYS2) convertit les arguments qui commencent par `/` en chemins Windows. Aucune commande de ce tutoriel n'est concernée, mais si tu utilises un jour `gh api` avec un chemin de ce type, neutralise la conversion :

```bash
MSYS_NO_PATHCONV=1 gh api /repos/Grand-Cinema-Studi/grand-cinema-docs
```

**Piège 3 — la continuation de ligne.** En Bash c'est l'antislash `\` en fin de ligne, pas l'accent grave de PowerShell. Les blocs de ce tutoriel sont écrits sans continuation pour éviter toute ambiguïté.

### 5.4 Les variables de travail

À coller au début de chaque session Git Bash — **elles disparaissent quand tu fermes la fenêtre** :

```bash
OWNER="Grand-Cinema-Studi"          # proprietaire des depots ET du Project
POWNER="$OWNER"                     # proprietaire du Project : JAMAIS "@me"
REPO_DOCS="$OWNER/grand-cinema-docs"
REPO_BACK="$OWNER/grand-cinema-backend"
REPO_FRONT="$OWNER/grand-cinema-frontend"
PROJECT=___                         # numero reel, lu a l'etape 2 — ne pas inventer

echo "Project $PROJECT chez $POWNER | $REPO_DOCS"   # verification
```

> **Astuce :** mets ce bloc dans un fichier `~/gc-env.sh`, puis charge-le d'un seul coup à chaque ouverture de terminal avec `source ~/gc-env.sh`.

### 5.5 Le piège n° 1 : `@me` n'est pas ton organisation

**C'est l'erreur qui coûte le plus de temps, et elle est silencieuse.**

`--owner "@me"` désigne le **compte authentifié dans `gh`** — ton compte personnel. Or les dépôts et le Project du projet appartiennent à l'**organisation** `Grand-Cinema-Studi`. Ce sont deux propriétaires différents.

Conséquence : une commande `gh project ... --owner "@me"` s'exécute sans erreur, mais **sur tes projects personnels**. Tu peux ainsi verser seize issues dans un projet sans rapport, pendant que le Project de l'organisation reste vide — et chercher longtemps un filtre fautif qui n'existe pas.

Vérifie qui est qui, une fois pour toutes :

```bash
gh api user --jq .login                       # ton compte personnel
gh api users/Grand-Cinema-Studi --jq .type    # doit repondre : Organization

gh project list --owner "@me"                 # TES projects personnels
gh project list --owner "Grand-Cinema-Studi"  # ceux de l'ORGANISATION  <- la bonne liste
```

**La règle à retenir :** dans toutes les commandes `gh project` de ce tutoriel, `--owner` prend `"$POWNER"`, c'est-à-dire l'organisation. `"@me"` ne sert qu'à inspecter tes propres projects personnels.

Le symptôme d'un mauvais ciblage : `GraphQL: Could not resolve to a ProjectV2 with the number N`. Le numéro existe peut-être — mais pas chez le propriétaire que tu as indiqué.

> **Ne devine jamais le numéro du Project.** Les numéros sont propres à chaque propriétaire : le n° 4 de ton compte et le n° 4 de l'organisation sont deux projects différents. Lis-le toujours dans `gh project list --owner "$POWNER"`.

---

## 6. Étape 1 — Préparer les 3 dépôts

### 6.1 Vérifier l'existence et la visibilité

```bash
gh repo list "$OWNER" --json name,visibility,description
```

Les trois dépôts `grand-cinema-*` doivent être en **PUBLIC**. S'ils n'existent pas encore :

```bash
gh repo create "$REPO_DOCS"  --public --add-readme --description "Titre CDA Bloc 1 - Le Grand Cinema - Specifications, backlog agile et livrables de conception"
gh repo create "$REPO_BACK"  --public --add-readme --description "Titre CDA - Le Grand Cinema - API REST, base de donnees et logique metier"
gh repo create "$REPO_FRONT" --public --add-readme --description "Titre CDA - Le Grand Cinema - Application web responsive et PWA de controle d'acces"
```

Si l'un existe mais reste privé :

```bash
for r in "$REPO_DOCS" "$REPO_BACK" "$REPO_FRONT"; do
  gh repo edit "$r" --visibility public --accept-visibility-change-consequences
done
```

### 6.2 Renseigner les descriptions

Le correcteur arrive sur ton profil et voit trois dépôts. Leur description doit lui indiquer immédiatement le rôle de chacun.

```bash
gh repo edit "$REPO_DOCS"  --description "Titre CDA Bloc 1 - Le Grand Cinema - Specifications, backlog agile (EPICs et User Stories) et livrables de conception"
gh repo edit "$REPO_BACK"  --description "Titre CDA - Le Grand Cinema - API REST, base de donnees et logique metier"
gh repo edit "$REPO_FRONT" --description "Titre CDA - Le Grand Cinema - Application web responsive et PWA de controle d'acces"
```

### 6.3 Le README d'orientation dans `grand-cinema-docs`

C'est la porte d'entrée du jury. Depuis un clone local de `grand-cinema-docs` :

```bash
cat > README.md << 'EOF'
# Le Grand Cinéma — Titre professionnel CDA

Projet d'examen — Bloc 1 « Concevoir et spécifier une solution digitale ».

## Organisation du projet

| Dépôt | Rôle |
| :--- | :--- |
| **grand-cinema-docs** (ce dépôt) | Spécifications, backlog agile, livrables de conception |
| [grand-cinema-backend](https://github.com/Grand-Cinema-Studi/grand-cinema-backend) | API REST, base de données, logique métier |
| [grand-cinema-frontend](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend) | Application web responsive et PWA de contrôle d'accès |

## Backlog agile

Le backlog complet — 5 EPICs, 14 User Stories, 89 story points (63 MVP, 26 V2) — est consultable
publiquement ici : **[Grand Cinema Backlog](https://github.com/orgs/Grand-Cinema-Studi/projects/3)**

Les EPICs et User Stories sont créés dans ce dépôt. Les tâches techniques sont
rattachées en sub-issues dans les dépôts backend et frontend.
EOF

git add README.md
git commit -m "docs: README d'orientation du projet"
git push
```

> Le `<< 'EOF'` avec les quotes simples est important : il empêche Bash d'interpréter le contenu du heredoc. Sans les quotes, tout `$` du texte serait remplacé par une variable.

---

## 7. Étape 2 — Créer le Project transverse et le rendre public

Un seul Project pour les trois dépôts. C'est lui qui recompose l'unité du projet.

### 7.1 Quel modèle choisir

**Prends « Iterative development ».**

Le modèle n'est qu'une amorce de champs et de vues, tout reste modifiable ensuite — ne passe pas plus de trente secondes sur ce choix. Mais un critère objectif le départage : **le champ Iteration est le seul que `gh project field-create` ne sait pas créer** (le CLI n'accepte que `TEXT`, `SINGLE_SELECT`, `DATE` et `NUMBER`). Un modèle qui le fournit déjà t'évite la seule manipulation obligatoirement manuelle, et ce champ matérialise directement tes sprints de 2 semaines.

| Modèle | Apport | Verdict |
| :--- | :--- | :--- |
| **Iterative development** | Sprints (Iteration) + kanban | **À retenir** |
| Kanban / Board | Le champ Status seul | Tout à recréer |
| Roadmap | Dates Start/Target | Pas de calendrier de dev réel à y mettre |
| Table | Grille nue | Aucun apport |

GitHub affiche un aperçu des champs et vues avant validation : vérifie qu'un champ d'itération y figure. Sinon, prends n'importe quel modèle et crée-le à la main.

> **⚠️ Le piège du modèle « Iterative development » :** il attache au Project un **champ d'issue** (*issue field*) nommé `Priority`. Les champs d'issue ont leur propre visibilité, et ceux en « Organization only » **interdisent de rendre le Project public** — tu obtiendras `Visibility cannot be changed while the project has restricted issue fields attached: Priority`. La section 15.1 explique comment le retirer en trente secondes. Ce n'est pas une raison d'éviter ce modèle, mais il faut le savoir avant de publier.

> **À dissiper :** aucun modèle ne crée de structure d'EPICs. La hiérarchie EPIC → US → tâche vient des **issues** (label + sub-issues), pas du Project. Tu peux changer de modèle ou recréer le Project sans affecter ton backlog.

### 7.2 Création

Interface web : sur `github.com/Grand-Cinema-Studi`, onglet **Projects** → **New project** → modèle **Iterative development** → titre `Grand Cinema Backlog`.

En CLI (crée un project vide, sans modèle) :

```bash
gh project create --owner "$POWNER" --title "Grand Cinema Backlog"
gh project list --owner "$POWNER"    # pour relire le numéro
```

Note le numéro affiché dans l'URL `github.com/orgs/Grand-Cinema-Studi/projects/<numéro>` et reporte-le dans ta variable `PROJECT`.

**Renommer un Project existant**, si tu as déjà créé le tien sous un autre nom :

```bash
gh project edit "$PROJECT" --owner "$POWNER" --title "Grand Cinema Backlog"
```

Ou dans l'interface : Project → **⋯** → **Settings** → champ du nom en haut de la page.

> Le renommage ne casse rien : l'URL d'un Project repose sur son **numéro**, pas sur son titre. Tes issues, champs et vues sont conservés, et le lien que tu mettras dans le dossier Word reste valable.

### 7.3 Le dépôt par défaut et le rattachement des dépôts

Deux réglages distincts, souvent confondus. Sur un Project de **compte personnel**, les réglages du Project ne proposent **pas** de liste « Repositories » : uniquement un **« Default repository »**, qui n'accepte qu'un seul dépôt. C'est normal.

**Le dépôt par défaut.** Project → **⋯** → **Settings** → **Default repository** → choisis **`grand-cinema-docs`**. Ce réglage ne limite rien : il désigne simplement le dépôt utilisé quand tu convertis une note du Project en vraie issue, et celui proposé à l'autocomplétion. Comme tes EPICs et US vivent dans `docs`, c'est le bon choix.

**Le rattachement des dépôts se fait depuis les dépôts, pas depuis le Project.** Pour chacun des trois :

1. Ouvre le dépôt sur GitHub → onglet **Projects**.
2. Clique **Link a project**.
3. Recherche et sélectionne `Grand Cinema Backlog`.

À répéter pour `grand-cinema-docs`, `grand-cinema-backend` et `grand-cinema-frontend`.

> **Ce n'est pas un prérequis pour alimenter le Project.** `gh project item-add --url <url-de-l-issue>` fonctionne avec n'importe quelle issue à laquelle tu as accès, liée ou non. Le rattachement sert au confort : le Project apparaît dans l'onglet Projects de chaque dépôt, et se propose directement dans la barre latérale des issues. Si tu es pressé, tu peux le faire après avoir créé tout le backlog.

---

## 8. Étape 3 — Configurer les champs de pilotage

Ce sont ces champs qui matérialisent le **chiffrage** et la **priorisation** notés par le jury.

| Nom du champ | Type | Options |
| :--- | :--- | :--- |
| `Story Points` | Number | — |
| `Priorite` | Single select | `Must have`, `Should have`, `Could have`, `Won't have` |
| `Version` | Single select | `MVP (V1)`, `V2`, `V3` |
| `Sprint` | Iteration | Fourni par le modèle, sinon à créer dans l'interface web |

```bash
gh project field-create "$PROJECT" --owner "$POWNER" --name "Story Points" --data-type NUMBER

gh project field-create "$PROJECT" --owner "$POWNER" --name "Priorite" --data-type SINGLE_SELECT --single-select-options "Must have,Should have,Could have,Won't have"

gh project field-create "$PROJECT" --owner "$POWNER" --name "Version" --data-type SINGLE_SELECT --single-select-options "MVP (V1),V2,V3"
```

> **Pas de champ `Niveau`.** Le niveau hiérarchique est déjà porté par les labels (`epic`, `user-story`, `tache`) et par la hiérarchie de sub-issues. Un champ supplémentaire ferait doublon, coûterait de la saisie et risquerait de diverger de son label.

> **Le champ `Composant` est inutile.** Le Project expose automatiquement une colonne **`Repository`** : c'est elle qui indique si une tâche est frontend, backend ou docs. Un champ redondant introduirait un risque d'incohérence.

Vérification :

```bash
gh project field-list "$PROJECT" --owner "$POWNER"
```

---

## 9. Étape 4 — Créer les labels dans les 3 dépôts

**Les labels sont propres à chaque dépôt.** La copie du Bloc 2 annonce trois familles d'étiquettes : **l'EPIC** (1 à 5), **la version** (MVP ou V2) et **le profil concerné**. Le jury comparera : les trois familles doivent exister.

**Labels de structure et de version — dans les 3 dépôts :**

```bash
for r in "$REPO_DOCS" "$REPO_BACK" "$REPO_FRONT"; do
  gh label create "epic"       --repo "$r" --color "6E40C9" --description "Conteneur de User Stories" --force
  gh label create "user-story" --repo "$r" --color "0E8A16" --description "Besoin utilisateur unitaire" --force
  gh label create "tache"      --repo "$r" --color "1D76DB" --description "Tache technique d'implementation" --force
  gh label create "mvp"        --repo "$r" --color "D93F0B" --description "Perimetre MVP (V1)" --force
  gh label create "v2"         --repo "$r" --color "FBCA04" --description "Reporte en version 2" --force
done
```

**Labels d'EPIC et de profil — dans `grand-cinema-docs` uniquement** (c'est là que vivent les EPICs et les US) :

```bash
gh label create "EPIC 1" --repo "$REPO_DOCS" --color "BFD4F2" --description "Consultation et programmation" --force
gh label create "EPIC 2" --repo "$REPO_DOCS" --color "BFD4F2" --description "Reservation et choix des places en temps reel" --force
gh label create "EPIC 3" --repo "$REPO_DOCS" --color "BFD4F2" --description "Paiement et billetterie dematerialisee" --force
gh label create "EPIC 4" --repo "$REPO_DOCS" --color "BFD4F2" --description "Controle d'acces en salle" --force
gh label create "EPIC 5" --repo "$REPO_DOCS" --color "BFD4F2" --description "Administration et reporting" --force

gh label create "profil: spectateur"                 --repo "$REPO_DOCS" --color "F9D0C4" --description "Spectateur regulier" --force
gh label create "profil: festivalier"                --repo "$REPO_DOCS" --color "F9D0C4" --description "Festivalier ou touriste" --force
gh label create "profil: caissier"                   --repo "$REPO_DOCS" --color "F9D0C4" --description "Personnel du guichet" --force
gh label create "profil: controleur"                 --repo "$REPO_DOCS" --color "F9D0C4" --description "Personnel a l'entree des salles" --force
gh label create "profil: responsable programmation"  --repo "$REPO_DOCS" --color "F9D0C4" --description "Responsable Programmation / Direction" --force
```

Le flag `--force` écrase un label existant de même nom au lieu d'échouer : les blocs sont rejouables sans erreur. L'ancien label `v3` peut rester : il ne sert plus, mais ne gêne pas.

Vérification :

```bash
gh label list --repo "$REPO_DOCS"
```

---

## 9 bis. Étape 4 bis — Mettre le backlog existant en conformité avec les copies

> **Pourquoi cette étape.** Le premier backlog créé sur GitHub (4 EPICs, 12 US, 97 points) date d'avant la rédaction des copies. Il ne correspond plus à ce que le jury lira : les copies Bloc 1 et Bloc 2 décrivent **5 EPICs et 14 User Stories**, avec un compte obligatoire pour réserver (l'ancienne US 2.3 proposait au contraire un achat sans compte), un tableau de bord reporté en V2 (l'ancienne US 4.3 était en MVP), et sans certaines anciennes stories (jauge modulable, chargement en moins d'une seconde). Le Bloc 2 renvoie explicitement le jury vers le Project : **le Project doit refléter la copie, mot pour mot.**

### 9 bis.1 La stratégie : modifier, fermer, créer

On garde le Project n° 3 (son lien est cité dans les copies) et on travaille sur les issues :

- **Modifier** les issues qui portent déjà le même besoin : leur numéro, leur place dans le Project et leurs tâches techniques sont conservés. L'historique de l'issue montre le changement de titre : c'est la trace d'un affinage de backlog, normal en agilité, et un argument à l'oral.
- **Fermer** (en « Not planned ») celles qui n'ont pas d'équivalent dans les copies. Réutiliser une issue pour un besoin différent serait une mauvaise pratique : un juré qui ouvre son historique verrait une story changer de sens.
- **Créer** celles qui manquent.

| Issue actuelle | Devient | Action |
| :--- | :--- | :--- |
| #1 [EPIC 1] Catalogue | [EPIC 1] Consultation et programmation | Modifier |
| #2 [EPIC 2] Réservation et jauges | [EPIC 2] Réservation et choix des places en temps réel | Modifier |
| #3 [EPIC 3] Billetterie et contrôle d'accès | [EPIC 3] Paiement et billetterie dématérialisée | Modifier |
| #4 [EPIC 4] Administration et pilotage | [EPIC 5] Administration et reporting | Modifier (change de numéro d'EPIC) |
| — | [EPIC 4] Contrôle d'accès en salle | **Créer** |
| #5 [US 1.1] Filtrer la programmation | [US 1.1] Consulter les films et les horaires | Modifier |
| #6 [US 1.2] Fiche détaillée d'un film | [US 1.2] Consulter sans compte | Modifier |
| #7 [US 1.3] Grille en moins d'une seconde | — (absente des copies) | **Fermer** |
| #8 [US 2.1] Places disponibles en temps réel | [US 2.2] Voir les places restantes en temps réel | Modifier |
| #9 [US 2.2] Verrouiller les places pendant le paiement | [US 2.3] Réserver un nombre de places et passer au paiement | Modifier (**garde ses tâches techniques**) |
| #10 [US 2.3] Payer sans création de compte | [US 3.1] Payer ma réservation en ligne | Modifier + **changer de parent** (EPIC 3) |
| #11 [US 3.1] E-billet avec QR code | [US 3.2] Recevoir un billet dématérialisé | Modifier |
| #12 [US 3.2] Scanner en moins de 2 secondes | [US 4.1] Scanner un billet | Modifier + **changer de parent** (nouvel EPIC 4) |
| #13 [US 3.3] Bloquer la réutilisation d'un billet | — (devient un critère de l'US 4.1) | **Fermer** |
| #14 [US 4.1] Créer et planifier les séances | [US 5.2] Gérer la programmation | Modifier |
| #15 [US 4.2] Moduler la jauge des salles | — (absente des copies) | **Fermer** |
| #16 [US 4.3] Tableau de bord analytique | [US 5.4] Analyser les taux de remplissage (V2) | Modifier |
| — | [US 2.1] Créer un compte et me connecter | **Créer** |
| — | [US 2.4] Choisir son siège sur un plan de salle (V2) | **Créer** |
| — | [US 5.1] Consulter les réservations en ligne (caissier) | **Créer** |
| — | [US 5.3] Suivre les réservations | **Créer** |
| — | [US 5.5] Être alerté en cas de forte affluence (V2) | **Créer** |

**Bilan :** 13 issues modifiées (dont 2 changements de parent), 3 fermées, 6 créées. Résultat : 5 EPICs et 14 US ouvertes.

**Ordre des étapes :** labels (étape 4) → colonnes du tableau (9 bis.3) → EPICs (étape 5) → US (étape 6) → champs de pilotage → tâches techniques (étape 7).

### 9 bis.2 Vérifier les numéros avant de toucher à quoi que ce soit

Toutes les commandes des étapes 5 et 6 utilisent les numéros #1 à #16. Vérifie qu'ils correspondent bien :

```bash
gh issue list --repo "$REPO_DOCS" --state open --limit 100
```

Tu dois retrouver `[EPIC 1]` à `[EPIC 4]` sous les n° 1 à 4, et `[US 1.1]` à `[US 4.3]` sous les n° 5 à 16, dans l'ordre du tableau ci-dessus. **Si un numéro diffère, corrige-le dans les commandes avant de les lancer.**

### 9 bis.3 Adapter les colonnes du tableau à la copie

La copie du Bloc 2 annonce ces colonnes : **Backlog → Prêt (DoR respectée) → En cours → En test / revue → Terminé (DoD respectée)**. Le champ `Status` du Project doit porter exactement ces valeurs.

Interface web uniquement (le CLI ne modifie pas les options d'un champ existant) :

1. Project → **⋯** → **Settings** → dans la barre latérale, le champ **Status**.
2. Renomme les options existantes et ajoute les manquantes, dans cet ordre : `Backlog`, `Prêt`, `En cours`, `En test / revue`, `Terminé`.
3. Renommer une option conserve les cartes qui la portaient : rien n'est perdu.
4. Project → **⋯** → **Workflows** : vérifie que « Item closed » envoie bien vers `Terminé` et « Item added to project » vers `Backlog`.

---

## 10. Étape 5 — Mettre à jour les EPICs (contenu prêt à coller)

Les EPICs vivent **dans `grand-cinema-docs`**. Leur contenu reprend la section « Formalisation des besoins en EPICs » du Bloc 1 (Dossier 1, Question 1) : même numérotation, mêmes intitulés, mêmes enjeux. Quatre EPICs existants sont modifiés, un seul est créé.

### 10.1 Structure d'un EPIC de qualité jury

Un EPIC n'est pas un titre. Il porte cinq informations : description, enjeux du Bloc 1 auxquels il répond, profils concernés, stories enfants, critères de fin.

> **Accents.** Les titres et contenus ci-dessous sont accentués pour coller à la copie. Git Bash les transmet correctement dans la grande majorité des cas. Vérifie le premier EPIC créé sur le web : si les accents sont remplacés par des caractères bizarres, lance `export LANG=fr_FR.UTF-8` puis corrige le titre depuis l'interface.

### 10.2 Générer les 5 fichiers de contenu

Un seul bloc à coller dans Git Bash, **depuis le dossier `grand-cinema-docs`** : il crée le dossier `backlog/` et les cinq fichiers.

```bash
mkdir -p backlog

cat > backlog/epic1.md << 'EOF2'
## Description
Permettre à tout visiteur, y compris sans compte, de consulter les films et les séances du Grand Cinéma depuis un ordinateur ou un smartphone, sans se déplacer.

## Enjeux du Bloc 1
Captation de la clientèle touristique, expérience utilisateur.

## Profils concernés
Spectateur régulier, festivalier / touriste.

## User Stories
- US 1.1 Consulter les films et les horaires (MVP)
- US 1.2 Consulter sans compte (MVP)

## Définition de terminé (DoD)
- [ ] Les 2 US enfants sont fermées
- [ ] Consultation testée sans compte, sur mobile et sur ordinateur
EOF2

cat > backlog/epic2.md << 'EOF2'
## Description
Permettre à l'utilisateur de créer un compte, de voir les places restantes en temps réel et de réserver un nombre de places, sans aucun risque de double vente. Le choix d'un siège précis sur un plan de salle est reporté en V2 (US 2.4).

## Règles métier
- La réservation exige un compte utilisateur.
- Il n'existe pas de réservation sans paiement : les places sont bloquées pendant le paiement et libérées si le paiement n'est pas validé au bout de 10 minutes (durée paramétrable).

## Enjeux du Bloc 1
Chiffre d'affaires perdu par abandon d'achat, optimisation du remplissage.

## Profils concernés
Spectateur régulier, festivalier / touriste.

## User Stories
- US 2.1 Créer un compte et me connecter (MVP)
- US 2.2 Voir les places restantes en temps réel (MVP)
- US 2.3 Réserver un nombre de places et passer au paiement (MVP)
- US 2.4 Choisir son siège sur un plan de salle (V2)

## Définition de terminé (DoD)
- [ ] Les 3 US MVP sont fermées
- [ ] Test de concurrence : deux demandes simultanées sur la dernière place, une seule aboutit
EOF2

cat > backlog/epic3.md << 'EOF2'
## Description
Permettre le paiement en ligne sécurisé de la réservation et la remise d'un billet dématérialisé avec QR code unique.

## Enjeux du Bloc 1
Chiffre d'affaires perdu par abandon d'achat, expérience utilisateur.

## Profils concernés
Spectateur régulier, festivalier / touriste.

## User Stories
- US 3.1 Payer ma réservation en ligne (MVP)
- US 3.2 Recevoir un billet dématérialisé (MVP)

## Définition de terminé (DoD)
- [ ] Les 2 US enfants sont fermées
- [ ] Paiement validé en mode test Stripe, aucune donnée de carte stockée par l'application
- [ ] Aucun billet généré sans paiement confirmé
EOF2

cat > backlog/epic4.md << 'EOF2'
## Description
Permettre au contrôleur de valider rapidement l'accès en salle en scannant le billet dématérialisé.

## Enjeux du Bloc 1
Expérience utilisateur : fluidité de l'entrée en salle.

## Profils concernés
Contrôleur.

## User Stories
- US 4.1 Scanner un billet (MVP)

## Définition de terminé (DoD)
- [ ] L'US enfant est fermée
- [ ] Un billet déjà utilisé est refusé au second scan
EOF2

cat > backlog/epic5.md << 'EOF2'
## Description
Donner au caissier une consultation des réservations en ligne, et au Responsable Programmation la gestion des séances et le suivi des réservations.

## Enjeux du Bloc 1
Optimisation du remplissage, anticipation des pics d'affluence (Festival de Jazz).

## Profils concernés
Caissier (lecture seule), Responsable Programmation / Direction.

## User Stories
- US 5.1 Consulter les réservations en ligne (caissier) (MVP)
- US 5.2 Gérer la programmation (MVP)
- US 5.3 Suivre les réservations (MVP)
- US 5.4 Analyser les taux de remplissage (V2)
- US 5.5 Être alerté en cas de forte affluence (V2)

## Définition de terminé (DoD)
- [ ] Les 3 US MVP sont fermées
- [ ] Droits d'accès vérifiés par profil (le caissier n'accède ni à la programmation ni à l'analyse)
EOF2

ls -l backlog/
```

> Le délimiteur interne est `EOF2` (et non `EOF`) pour qu'il ne se confonde pas avec d'éventuels autres heredocs. Les quotes simples autour de `'EOF2'` empêchent Bash d'interpréter le contenu.

### 10.3 Modifier les 4 EPICs existants et créer l'EPIC 4

`gh issue edit` remplace le titre et le contenu, et ajoute le label d'EPIC. Le label `epic` déjà présent est conservé.

```bash
gh issue edit 1 --repo "$REPO_DOCS" --title "[EPIC 1] Consultation et programmation"                --body-file backlog/epic1.md --add-label "EPIC 1"
gh issue edit 2 --repo "$REPO_DOCS" --title "[EPIC 2] Réservation et choix des places en temps réel" --body-file backlog/epic2.md --add-label "EPIC 2"
gh issue edit 3 --repo "$REPO_DOCS" --title "[EPIC 3] Paiement et billetterie dématérialisée"        --body-file backlog/epic3.md --add-label "EPIC 3"
gh issue edit 4 --repo "$REPO_DOCS" --title "[EPIC 5] Administration et reporting"                  --body-file backlog/epic5.md --add-label "EPIC 5"
```

> **Attention à l'issue #4 :** l'ancien EPIC 4 (administration) devient l'**EPIC 5**, pour suivre la numérotation du Bloc 1. Ses enfants (anciennes US 4.1 et 4.3) deviennent les US 5.2 et 5.4 : ils restent donc à leur place, aucun changement de parent n'est nécessaire.

Puis crée le nouvel EPIC 4, directement ajouté au Project grâce à `--project` :

```bash
gh issue create --repo "$REPO_DOCS" --title "[EPIC 4] Contrôle d'accès en salle" --body-file backlog/epic4.md --label "epic,EPIC 4" --project "Grand Cinema Backlog"
```

Vérification — 5 EPICs ouverts, numérotés de 1 à 5 dans leur titre :

```bash
gh issue list --repo "$REPO_DOCS" --label "epic" --state open
```

---

## 11. Étape 6 — Mettre à jour les 14 User Stories

Les US vivent **dans `grand-cinema-docs`**, au même endroit que leurs EPICs parents. Leur contenu reprend **mot pour mot** la Question 2 du Dossier 1 du Bloc 2 : mêmes titres, mêmes descriptions, mêmes critères d'acceptation. Neuf US existantes sont modifiées, trois sont fermées, cinq sont créées.

### 11.1 Structure d'une User Story de qualité jury

Chaque US porte : un titre, une description « En tant que… je veux… afin de… », des critères d'acceptation vérifiables, le profil concerné et la version. Les critères deviennent directement les cas de test du plan de recette (Bloc 2, Dossier 2).

### 11.2 Mémoriser les numéros des 5 EPICs

Les quatre premiers sont connus (n° 1 à 4) ; le nouvel EPIC 4 est relu par son titre.

> **Piège Bash :** n'écris jamais un repère entre chevrons comme `--parent <E1>`. En Bash, `<` est l'opérateur de **redirection d'entrée** : la commande échoue avec `bash: E1: No such file or directory`. Utilise toujours une variable.

```bash
E1=1
E2=2
E3=3
E5=4     # l'issue #4 est devenue l'EPIC 5
E4=$(gh issue list --repo "$REPO_DOCS" --label epic --state open --json number,title --jq '.[] | select(.title | startswith("[EPIC 4]")) | .number')

echo "EPICs : 1=$E1 2=$E2 3=$E3 4=$E4 5=$E5"
```

`E4` doit afficher un numéro **supérieur à 16**. S'il est vide, l'EPIC 4 n'a pas été créé (section 10.3).

### 11.3 Générer les 14 fichiers de contenu

Un seul bloc à coller, toujours depuis le dossier `grand-cinema-docs` :

```bash
cat > backlog/us11.md << 'EOF2'
**En tant que** spectateur ou festivalier, **je veux** consulter les films et leurs séances **afin de** choisir une séance sans me déplacer.

## Critères d'acceptation
- [ ] Étant donné des séances programmées, quand j'ouvre la programmation, alors je vois les films avec leurs séances (jour, heure, salle).
- [ ] Quand je choisis une date, alors seules les séances de ce jour s'affichent.
- [ ] Quand aucune séance n'existe pour cette date, alors un message explicite s'affiche.

Profil : spectateur, festivalier · Version : MVP
EOF2

cat > backlog/us12.md << 'EOF2'
**En tant que** festivalier, **je veux** accéder à la programmation sans compte ni connaissance du lieu **afin de** découvrir l'offre facilement.

## Critères d'acceptation
- [ ] Quand j'arrive sur le site, alors la programmation est accessible sans connexion ni création de compte.
- [ ] Depuis la fiche d'une séance, un accès à la réservation est proposé.
- [ ] Les pages de consultation sont lisibles sur smartphone.

Profil : festivalier · Version : MVP
EOF2

cat > backlog/us21.md << 'EOF2'
**En tant qu'**utilisateur, **je veux** créer un compte et me connecter **afin de** pouvoir réserver et retrouver mes billets.

## Critères d'acceptation
- [ ] Quand je renseigne des informations valides, alors mon compte est créé et je suis connecté.
- [ ] Quand l'adresse e-mail est déjà utilisée, alors la création est refusée avec un message explicite.
- [ ] Quand mes identifiants sont incorrects, alors la connexion est refusée avec un message qui ne précise pas lequel est faux.
- [ ] Mon mot de passe n'est jamais stocké en clair.

Profil : spectateur, festivalier · Version : MVP
EOF2

cat > backlog/us22.md << 'EOF2'
**En tant que** spectateur ou festivalier, **je veux** voir le nombre de places restantes d'une séance **afin de** savoir si je peux réserver.

## Critères d'acceptation
- [ ] Chaque séance affiche son nombre de places restantes.
- [ ] Les places vendues ou en cours de paiement ne sont plus comptées comme disponibles.
- [ ] Quand il ne reste aucune place, alors la séance est affichée « complète » et la réservation est impossible.

Profil : spectateur, festivalier · Version : MVP
EOF2

cat > backlog/us23.md << 'EOF2'
**En tant qu'**utilisateur connecté, **je veux** réserver un nombre de places en ligne **afin d'**éviter la file d'attente au guichet.

## Critères d'acceptation
- [ ] Quand je ne suis pas connecté, alors je suis invité à me connecter ou à créer un compte avant de continuer.
- [ ] Quand je sélectionne un nombre de places disponible et que je passe au paiement, alors ces places sont bloquées pour moi pendant 10 minutes.
- [ ] Aucune réservation n'est enregistrée tant que le paiement n'est pas confirmé.
- [ ] Quand deux utilisateurs visent la dernière place, alors un seul obtient le blocage ; l'autre reçoit un message explicite.
- [ ] Quand 10 minutes s'écoulent sans paiement validé, alors les places sont libérées, le compteur est mis à jour et un message m'en informe.
- [ ] Quand je demande plus de places qu'il n'en reste, alors la demande est refusée avec un message.

Profil : spectateur, festivalier · Version : MVP
EOF2

cat > backlog/us24.md << 'EOF2'
**En tant que** spectateur, **je veux** choisir mon siège sur un plan de la salle **afin de** me placer où je le souhaite.

## Critères d'acceptation
- [ ] Le plan de la salle affiche les sièges libres et occupés.
- [ ] Un siège occupé ou bloqué n'est pas sélectionnable.
- [ ] Les sièges choisis sont bloqués comme à l'US 2.3.

Profil : spectateur · Version : V2
EOF2

cat > backlog/us31.md << 'EOF2'
**En tant qu'**utilisateur, **je veux** payer ma réservation en ligne de façon sécurisée **afin de** finaliser mon achat sans passer au guichet.

## Critères d'acceptation
- [ ] Étant donné des places bloquées, quand je saisis une carte valide, alors le paiement est confirmé, la réservation est enregistrée, mon billet s'affiche et les places passent à « vendues ».
- [ ] Quand la carte est refusée, alors un message d'erreur s'affiche et les places restent bloquées jusqu'à la fin des 10 minutes, ce qui me permet de réessayer, puis sont libérées.
- [ ] Aucune donnée de carte n'est stockée par l'application (le paiement passe par Stripe).

Profil : spectateur, festivalier · Version : MVP
EOF2

cat > backlog/us32.md << 'EOF2'
**En tant qu'**utilisateur, **je veux** recevoir un billet dématérialisé **afin de** me présenter directement en salle.

## Critères d'acceptation
- [ ] Quand le paiement est confirmé, alors un billet avec QR code unique est généré, indiquant film, séance, salle et nombre de places.
- [ ] Le billet reste accessible depuis mon compte.
- [ ] Quand le paiement échoue, alors aucun billet n'est généré.

Profil : spectateur, festivalier · Version : MVP
EOF2

cat > backlog/us41.md << 'EOF2'
**En tant que** contrôleur, **je veux** scanner le billet dématérialisé du spectateur **afin de** valider rapidement son accès à la séance.

## Critères d'acceptation
- [ ] Quand je scanne un billet valide pour la séance en cours, alors l'accès est autorisé et le billet est marqué « utilisé ».
- [ ] Quand je scanne un billet déjà utilisé, alors l'accès est refusé avec un message explicite.
- [ ] Quand je scanne un billet d'une autre séance ou un QR code inconnu ou illisible, alors l'accès est refusé avec un message explicite.
- [ ] Un retour visuel (validé ou refusé) s'affiche immédiatement après le scan.

Profil : contrôleur · Version : MVP
EOF2

cat > backlog/us51.md << 'EOF2'
**En tant que** caissier, **je veux** consulter les réservations en ligne **afin d'**orienter les spectateurs au guichet.

## Critères d'acceptation
- [ ] Je vois, par séance, les places vendues en ligne et les places restantes.
- [ ] Mon accès est en lecture seule.
- [ ] Je n'ai pas accès à la programmation ni à l'analyse.

Profil : caissier · Version : MVP
EOF2

cat > backlog/us52.md << 'EOF2'
**En tant que** Responsable Programmation, **je veux** créer, modifier et supprimer des séances **afin d'**adapter l'offre à la demande.

## Critères d'acceptation
- [ ] Quand je crée une séance (film, salle, date, heure), alors elle apparaît dans la consultation publique.
- [ ] Quand je modifie ou supprime une séance, alors la consultation publique est mise à jour.
- [ ] Quand la séance a déjà des réservations, alors le système m'avertit avant de valider.
- [ ] Seuls les profils autorisés accèdent à cette fonction.

Profil : responsable programmation · Version : MVP
EOF2

cat > backlog/us53.md << 'EOF2'
**En tant que** Responsable Programmation, **je veux** suivre les réservations par séance **afin de** repérer les séances à forte ou faible demande.

## Critères d'acceptation
- [ ] Je vois, par séance, le nombre de places vendues et la capacité de la salle.
- [ ] Les données sont conservées pour l'analyse future (US 5.4).

Profil : responsable programmation · Version : MVP
EOF2

cat > backlog/us54.md << 'EOF2'
**En tant que** Responsable Programmation, **je veux** analyser les taux de remplissage **afin d'**ajuster l'allocation film/salle.

## Critères d'acceptation
- [ ] Je vois le taux de remplissage par séance, salle et période.
- [ ] Je peux comparer avec l'historique déjà collecté.

Profil : responsable programmation · Version : V2
EOF2

cat > backlog/us55.md << 'EOF2'
**En tant que** Responsable Programmation, **je veux** être alerté quand une séance approche de sa capacité **afin d'**anticiper les pics du festival.

## Critères d'acceptation
- [ ] Je peux définir un seuil d'alerte.
- [ ] Quand une séance atteint ce seuil, alors je suis notifié.

Profil : responsable programmation · Version : V2
EOF2

ls backlog/us*.md | wc -l    # doit afficher 14
```

> **US 2.3 et US 2.4 : ne pas confondre.** L'US 2.3 (MVP) réserve une **quantité** de places pour une séance (« 3 places pour la séance de 20 h »), sans choisir les fauteuils. L'US 2.4 (V2) ajoute le **choix du siège** sur un plan de salle. Le compteur de places restantes (US 2.2) et le compte à rebours de 10 minutes pendant le paiement (US 2.3) sont deux choses différentes, toutes deux en MVP.

### 11.4 Modifier les 9 US existantes

Chaque commande remplace le titre et le contenu, et ajoute le label d'EPIC et le ou les profils. Les labels `user-story` et `mvp` déjà présents sont conservés. Pour l'issue #16, qui passe en V2, le label `mvp` est remplacé par `v2`.

```bash
gh issue edit 5  --repo "$REPO_DOCS" --title "[US 1.1] Consulter les films et les horaires"           --body-file backlog/us11.md --add-label "EPIC 1,profil: spectateur,profil: festivalier"
gh issue edit 6  --repo "$REPO_DOCS" --title "[US 1.2] Consulter sans compte"                         --body-file backlog/us12.md --add-label "EPIC 1,profil: festivalier"
gh issue edit 8  --repo "$REPO_DOCS" --title "[US 2.2] Voir les places restantes en temps réel"       --body-file backlog/us22.md --add-label "EPIC 2,profil: spectateur,profil: festivalier"
gh issue edit 9  --repo "$REPO_DOCS" --title "[US 2.3] Réserver un nombre de places et passer au paiement" --body-file backlog/us23.md --add-label "EPIC 2,profil: spectateur,profil: festivalier"
gh issue edit 10 --repo "$REPO_DOCS" --title "[US 3.1] Payer ma réservation en ligne"                 --body-file backlog/us31.md --add-label "EPIC 3,profil: spectateur,profil: festivalier"
gh issue edit 11 --repo "$REPO_DOCS" --title "[US 3.2] Recevoir un billet dématérialisé"              --body-file backlog/us32.md --add-label "EPIC 3,profil: spectateur,profil: festivalier"
gh issue edit 12 --repo "$REPO_DOCS" --title "[US 4.1] Scanner un billet"                             --body-file backlog/us41.md --add-label "EPIC 4,profil: controleur"
gh issue edit 14 --repo "$REPO_DOCS" --title "[US 5.2] Gérer la programmation"                        --body-file backlog/us52.md --add-label "EPIC 5,profil: responsable programmation"
gh issue edit 16 --repo "$REPO_DOCS" --title "[US 5.4] Analyser les taux de remplissage"              --body-file backlog/us54.md --add-label "EPIC 5,v2,profil: responsable programmation" --remove-label "mvp"
```

### 11.5 Changer le parent de deux US

Deux US changent d'EPIC :

| Issue | US | Ancien parent | Nouveau parent |
| :--- | :--- | :--- | :--- |
| #10 | US 3.1 Payer ma réservation en ligne | #2 (EPIC 2) | #3 (EPIC 3) |
| #12 | US 4.1 Scanner un billet | #3 (EPIC 3) | le nouvel EPIC 4 (`$E4`) |

**Interface web (le plus sûr) :** ouvre l'issue → barre latérale **Relationships** → **Parent issue** → modifie le parent et choisis le nouvel EPIC. Une issue ne peut avoir qu'un seul parent : le changement la retire automatiquement de l'ancien.

**En CLI**, si ta version de `gh` accepte `--parent` sur `gh issue edit` :

```bash
gh issue edit 10 --repo "$REPO_DOCS" --parent "$E3"
gh issue edit 12 --repo "$REPO_DOCS" --parent "$E4"
```

Si la commande est refusée, passe par l'interface web : c'est deux clics par issue.

### 11.6 Fermer les 3 US sans équivalent

```bash
gh issue close 7  --repo "$REPO_DOCS" --reason "not planned" --comment "Story retiree lors de l'affinage du backlog : absente des copies des Blocs 1 et 2. La performance mobile releve des criteres de qualite (DoD), pas d'une story."
gh issue close 13 --repo "$REPO_DOCS" --reason "not planned" --comment "Fusionnee dans l'US 4.1 Scanner un billet (critere : un billet deja utilise est refuse)."
gh issue close 15 --repo "$REPO_DOCS" --reason "not planned" --comment "Story retiree lors de l'affinage du backlog : la modulation des jauges ne fait pas partie du perimetre retenu dans les copies."
```

Une issue fermée reste affichée dans le Project. Retire ces trois-là pour que le jury ne voie que le backlog à jour :

```bash
gh project item-list "$PROJECT" --owner "$POWNER" --limit 200 --format json --jq '.items[] | select(.content.url != null) | select(.content.url | test("grand-cinema-docs/issues/(7|13|15)$")) | .id' | while read -r id; do
  gh project item-delete "$PROJECT" --owner "$POWNER" --id "$id"
done
```

**Interface web :** vue Table du Project → sélectionne les trois lignes → clic droit → **Delete from project**. L'issue n'est pas supprimée, seulement retirée du Project.

### 11.7 Créer les 5 US manquantes

Le flag `--project` les ajoute directement au Project.

```bash
gh issue create --repo "$REPO_DOCS" --parent "$E2" --title "[US 2.1] Créer un compte et me connecter"                 --body-file backlog/us21.md --label "user-story,EPIC 2,mvp,profil: spectateur,profil: festivalier" --project "Grand Cinema Backlog"
gh issue create --repo "$REPO_DOCS" --parent "$E2" --title "[US 2.4] Choisir son siège sur un plan de salle"          --body-file backlog/us24.md --label "user-story,EPIC 2,v2,profil: spectateur"                     --project "Grand Cinema Backlog"
gh issue create --repo "$REPO_DOCS" --parent "$E5" --title "[US 5.1] Consulter les réservations en ligne (caissier)"  --body-file backlog/us51.md --label "user-story,EPIC 5,mvp,profil: caissier"                       --project "Grand Cinema Backlog"
gh issue create --repo "$REPO_DOCS" --parent "$E5" --title "[US 5.3] Suivre les réservations"                         --body-file backlog/us53.md --label "user-story,EPIC 5,mvp,profil: responsable programmation"     --project "Grand Cinema Backlog"
gh issue create --repo "$REPO_DOCS" --parent "$E5" --title "[US 5.5] Être alerté en cas de forte affluence"           --body-file backlog/us55.md --label "user-story,EPIC 5,v2,profil: responsable programmation"      --project "Grand Cinema Backlog"
```

### 11.8 Vérifier le résultat

Quatorze US ouvertes, dans l'ordre de la copie :

```bash
gh issue list --repo "$REPO_DOCS" --label user-story --state open --limit 100 --json number,title --jq 'sort_by(.title) | .[] | "#\(.number)  \(.title)"'
```

Sur le web, chaque EPIC doit afficher sa barre de progression avec le bon nombre d'enfants : EPIC 1 `0 / 2`, EPIC 2 `0 / 4`, EPIC 3 `0 / 2`, EPIC 4 `0 / 1`, EPIC 5 `0 / 5`. Si un EPIC affiche un enfant de trop ou de moins, un changement de parent (11.5) n'a pas été fait.

Et dans le Project, 19 issues du backlog (5 EPICs + 14 US) plus les tâches techniques :

```bash
gh project item-list "$PROJECT" --owner "$POWNER" --limit 200 --format json --jq '.items[] | .content.title'
```

### 11.9 Renseigner les champs de pilotage

Les champs vivent dans le Project. Les anciennes valeurs (ancien chiffrage) sont à écraser, et les nouvelles issues naissent avec des champs vides.

**Chiffrage retenu (14 US) :**

```bash
declare -A PTS=( ["US 1.1"]=3 ["US 1.2"]=2 ["US 2.1"]=5 ["US 2.2"]=5 ["US 2.3"]=13 ["US 2.4"]=13 ["US 3.1"]=8 ["US 3.2"]=5 ["US 4.1"]=8 ["US 5.1"]=3 ["US 5.2"]=8 ["US 5.3"]=3 ["US 5.4"]=8 ["US 5.5"]=5 )
declare -A PRIO=( ["US 5.1"]="Should have" ["US 5.3"]="Should have" ["US 2.4"]="Won't have" ["US 5.4"]="Won't have" ["US 5.5"]="Won't have" )   # les autres : Must have
declare -A VERS=( ["US 2.4"]="V2" ["US 5.4"]="V2" ["US 5.5"]="V2" )                                                                                  # les autres : MVP (V1)

gh issue list --repo "$REPO_DOCS" --label user-story --state open --limit 100 --json url,title --jq '.[] | [.url, .title] | @tsv' | while IFS=$'\t' read -r url title; do
  cle=$(echo "$title" | sed -n 's/^\[\(US [0-9]\.[0-9]\)\].*/\1/p')
  [ -z "$cle" ] && { echo "IGNORE (pas une US) : $title"; continue; }
  pts="${PTS[$cle]}"
  prio="${PRIO[$cle]:-Must have}"
  vers="${VERS[$cle]:-MVP (V1)}"
  [ -z "$pts" ] && { echo "IGNORE (non chiffree) : $title"; continue; }
  echo "$cle -> $pts pts, $prio, $vers"
  gh project item-edit "$PROJECT" --owner "$POWNER" --url "$url" --field "Story Points" --number "$pts"
  gh project item-edit "$PROJECT" --owner "$POWNER" --url "$url" --field "Priorite"     --value  "$prio"
  gh project item-edit "$PROJECT" --owner "$POWNER" --url "$url" --field "Version"      --value  "$vers"
done
```

> **Si `gh project item-edit --url` est refusé par ta version de `gh`**, saisis les valeurs directement dans la vue Table du Project : pour quatorze lignes, c'est une grille où l'on tabule d'une cellule à l'autre. Les valeurs à saisir sont dans l'Annexe A.

**Pourquoi « Won't have » pour les US V2 :** dans MoSCoW, *Won't have* signifie « pas dans cette version », pas « jamais ». C'est exactement le statut des trois stories reportées en V2 par l'arbitrage du Bloc 1.

---

## 12. Étape 7 — Décomposer en tâches techniques cross-repo

C'est ici que l'architecture à trois dépôts prend tout son sens.

### 12.1 La règle du rattachement cross-repo

Une tâche créée dans `grand-cinema-backend` doit être rattachée à une US vivant dans `grand-cinema-docs`. Le numéro seul serait ambigu : **il faut l'URL complète du parent**.

```bash
# Rattachement dans le MEME depot : le numero suffit
gh issue create --repo "$REPO_DOCS" --parent 12 --title "..."

# Rattachement CROSS-REPO : URL complete du parent obligatoire
gh issue create --repo "$REPO_BACK" --parent "https://github.com/Grand-Cinema-Studi/grand-cinema-docs/issues/12" --title "..."
```

**Interface web :** ouvre la tâche → **Add existing issue** → le sélecteur permet de **changer de dépôt** pour pointer vers l'US parente dans `docs`.

**Correction après coup**, si une tâche a été créée sans parent :

```bash
gh issue edit 7 --repo "$REPO_BACK" --parent "https://github.com/Grand-Cinema-Studi/grand-cinema-docs/issues/12"
```

### 12.2 Exemple complet — décomposition de l'US 2.3

L'US 2.3 (réservation d'un nombre de places avec blocage de 10 minutes) est la plus riche techniquement : c'est celle à décomposer en priorité, pour démontrer la méthode.

**Tes tâches existantes sont déjà au bon endroit.** Elles étaient rattachées à l'issue #9 (ancienne US 2.2), qui est devenue l'US 2.3 à l'étape 6 : elles la suivent automatiquement, sans rien à refaire. Liste-les pour vérifier :

```bash
gh issue list --repo "$REPO_BACK"  --label tache --state open
gh issue list --repo "$REPO_FRONT" --label tache --state open
```

Il reste à vérifier leur contenu (encadré ci-dessous), puis à créer celles qui manqueraient.

> **Cohérence avec l'architecture du Bloc 1 — à vérifier sur tes tâches existantes.** Le schéma d'architecture (Dossier 3) prévoit une base **RDS PostgreSQL** et ne contient pas de Redis. Le blocage des places se fait donc **en base**, dans une transaction : c'est plus simple, suffisant pour le volume du Grand Cinéma, et cohérent avec le schéma. Si tes tâches existantes parlent d'un verrou Redis, renomme-les et réécris leur contenu avec `gh issue edit <numero> --repo "$REPO_BACK" --title "..." --body "..."` en reprenant les textes ci-dessous. Si tu préfères garder Redis, il faut l'ajouter au schéma et le justifier dans la Question 3 du Dossier 3, sinon le jury relèvera l'écart.

```bash
US23="https://github.com/Grand-Cinema-Studi/grand-cinema-docs/issues/9"    # l'issue #9 est devenue l'US 2.3

gh issue create --repo "$REPO_BACK" --parent "$US23" --label "tache" --title "[BACK] Bloquer les places en base pendant 10 minutes (transaction PostgreSQL)" --body "Tache technique de l'US 2.3 (grand-cinema-docs).

## Perimetre
Enregistrer un blocage temporaire des places choisies, avec une date d'expiration a 10 minutes (parametre), dans une transaction qui verrouille la seance pour eliminer toute double vente.

## Criteres de realisation
- [ ] Blocage enregistre avec date d'expiration parametrable (10 minutes par defaut)
- [ ] Verification et blocage executes dans une seule transaction (verrouillage de la ligne de la seance)
- [ ] Les blocages expires ne sont plus comptes dans les places restantes
- [ ] Test de concurrence : deux demandes simultanees sur la derniere place, une seule aboutit
- [ ] Tests unitaires ecrits et passants"

gh issue create --repo "$REPO_BACK" --parent "$US23" --label "tache" --title "[BACK] Exposer l'endpoint POST /api/reservations/blocage" --body "Tache technique de l'US 2.3 (grand-cinema-docs).

## Perimetre
Exposer l'API qui pose le blocage et retourne l'heure d'expiration au client. Endpoint reserve aux utilisateurs connectes.

## Criteres de realisation
- [ ] Reponse 201 avec identifiant du blocage et heure d'expiration
- [ ] Reponse 401 si l'utilisateur n'est pas connecte
- [ ] Reponse 409 si les places demandees ne sont plus disponibles
- [ ] Contrat d'API documente au format OpenAPI
- [ ] Tests d'integration ecrits et passants"

gh issue create --repo "$REPO_FRONT" --parent "$US23" --label "tache" --title "[FRONT] Afficher le compte a rebours du blocage" --body "Tache technique de l'US 2.3 (grand-cinema-docs).

## Perimetre
Afficher un compte a rebours synchronise sur l'heure d'expiration renvoyee par l'API, et gerer l'expiration cote client.

## Criteres de realisation
- [ ] Compte a rebours visible pendant tout le parcours de paiement
- [ ] Alerte visuelle sous 2 minutes restantes
- [ ] A l'expiration : message explicite et retour a la seance avec le compteur a jour
- [ ] Composant teste unitairement"
```

**Ne crée que les tâches qui manquent** parmi les trois ci-dessus : si une tâche équivalente existe déjà, modifie-la plutôt que d'en créer une seconde.

La matrice complète des tâches pour les 14 US figure en **Annexe B**.

---


## 13. Étape 8 — Script Bash de mise en conformité

Ce script enchaîne les étapes 5 et 6 : modification des 13 issues, fermeture des 3 US sans équivalent, création des 6 issues manquantes. Il ne fait **pas** les deux changements de parent (section 11.5), à faire ensuite sur le web.

Enregistre-le sous `backlog/mettre-a-jour-backlog.sh`, rends-le exécutable avec `chmod +x backlog/mettre-a-jour-backlog.sh`, puis lance-le par `./backlog/mettre-a-jour-backlog.sh` **depuis le dossier `grand-cinema-docs`**.

**Prérequis :** les fichiers `backlog/epic1.md` à `backlog/epic5.md` (section 10.2) et `backlog/us11.md` à `backlog/us55.md` (section 11.3) existent, les labels sont créés (étape 4), et la vérification des numéros (9 bis.2) est faite.

> **À utiliser à la place des commandes des étapes 5 et 6, pas en plus.** Le relancer une seconde fois créerait des doublons pour les 6 issues nouvelles.

```bash
#!/usr/bin/env bash
set -euo pipefail

# --- Parametres -----------------------------------------------------
OWNER="Grand-Cinema-Studi"
POWNER="$OWNER"                     # proprietaire du Project : JAMAIS "@me"
REPO_DOCS="$OWNER/grand-cinema-docs"
PROJECT=3
PNAME="Grand Cinema Backlog"
P_SF="profil: spectateur,profil: festivalier"
P_RP="profil: responsable programmation"

# --- 1. EPICs : 4 modifies, 1 cree ----------------------------------
gh issue edit 1 --repo "$REPO_DOCS" --title "[EPIC 1] Consultation et programmation"                --body-file backlog/epic1.md --add-label "EPIC 1"
gh issue edit 2 --repo "$REPO_DOCS" --title "[EPIC 2] Réservation et choix des places en temps réel" --body-file backlog/epic2.md --add-label "EPIC 2"
gh issue edit 3 --repo "$REPO_DOCS" --title "[EPIC 3] Paiement et billetterie dématérialisée"        --body-file backlog/epic3.md --add-label "EPIC 3"
gh issue edit 4 --repo "$REPO_DOCS" --title "[EPIC 5] Administration et reporting"                  --body-file backlog/epic5.md --add-label "EPIC 5"

url=$(gh issue create --repo "$REPO_DOCS" --title "[EPIC 4] Contrôle d'accès en salle" --body-file backlog/epic4.md --label "epic,EPIC 4" --project "$PNAME")
E4="${url##*/}"
echo "Nouvel EPIC 4 : #$E4"

# --- 2. US : 9 modifiees ---------------------------------------------
gh issue edit 5  --repo "$REPO_DOCS" --title "[US 1.1] Consulter les films et les horaires"           --body-file backlog/us11.md --add-label "EPIC 1,$P_SF"
gh issue edit 6  --repo "$REPO_DOCS" --title "[US 1.2] Consulter sans compte"                         --body-file backlog/us12.md --add-label "EPIC 1,profil: festivalier"
gh issue edit 8  --repo "$REPO_DOCS" --title "[US 2.2] Voir les places restantes en temps réel"       --body-file backlog/us22.md --add-label "EPIC 2,$P_SF"
gh issue edit 9  --repo "$REPO_DOCS" --title "[US 2.3] Réserver un nombre de places et passer au paiement" --body-file backlog/us23.md --add-label "EPIC 2,$P_SF"
gh issue edit 10 --repo "$REPO_DOCS" --title "[US 3.1] Payer ma réservation en ligne"                 --body-file backlog/us31.md --add-label "EPIC 3,$P_SF"
gh issue edit 11 --repo "$REPO_DOCS" --title "[US 3.2] Recevoir un billet dématérialisé"              --body-file backlog/us32.md --add-label "EPIC 3,$P_SF"
gh issue edit 12 --repo "$REPO_DOCS" --title "[US 4.1] Scanner un billet"                             --body-file backlog/us41.md --add-label "EPIC 4,profil: controleur"
gh issue edit 14 --repo "$REPO_DOCS" --title "[US 5.2] Gérer la programmation"                        --body-file backlog/us52.md --add-label "EPIC 5,$P_RP"
gh issue edit 16 --repo "$REPO_DOCS" --title "[US 5.4] Analyser les taux de remplissage"              --body-file backlog/us54.md --add-label "EPIC 5,v2,$P_RP" --remove-label "mvp"

# --- 3. US : 3 fermees ------------------------------------------------
gh issue close 7  --repo "$REPO_DOCS" --reason "not planned" --comment "Story retiree lors de l'affinage du backlog : absente des copies des Blocs 1 et 2."
gh issue close 13 --repo "$REPO_DOCS" --reason "not planned" --comment "Fusionnee dans l'US 4.1 Scanner un billet (critere : un billet deja utilise est refuse)."
gh issue close 15 --repo "$REPO_DOCS" --reason "not planned" --comment "Story retiree lors de l'affinage du backlog : hors du perimetre retenu dans les copies."

# --- 4. US : 5 creees ----------------------------------------------------
gh issue create --repo "$REPO_DOCS" --parent 2 --title "[US 2.1] Créer un compte et me connecter"                --body-file backlog/us21.md --label "user-story,EPIC 2,mvp,$P_SF"                  --project "$PNAME"
gh issue create --repo "$REPO_DOCS" --parent 2 --title "[US 2.4] Choisir son siège sur un plan de salle"         --body-file backlog/us24.md --label "user-story,EPIC 2,v2,profil: spectateur"      --project "$PNAME"
gh issue create --repo "$REPO_DOCS" --parent 4 --title "[US 5.1] Consulter les réservations en ligne (caissier)" --body-file backlog/us51.md --label "user-story,EPIC 5,mvp,profil: caissier"        --project "$PNAME"
gh issue create --repo "$REPO_DOCS" --parent 4 --title "[US 5.3] Suivre les réservations"                        --body-file backlog/us53.md --label "user-story,EPIC 5,mvp,$P_RP"                  --project "$PNAME"
gh issue create --repo "$REPO_DOCS" --parent 4 --title "[US 5.5] Être alerté en cas de forte affluence"          --body-file backlog/us55.md --label "user-story,EPIC 5,v2,$P_RP"                   --project "$PNAME"

echo ""
echo "Termine. Reste a faire sur le web :"
echo "  - changer le parent de #10 vers #3 (EPIC 3)"
echo "  - changer le parent de #12 vers #$E4 (EPIC 4)"
echo "  - retirer #7, #13 et #15 du Project (section 11.6)"
echo "  - renseigner les champs de pilotage (section 11.9)"
```

**Notes de lecture du script :**

- `set -euo pipefail` arrête le script à la première erreur au lieu de continuer sur un backlog à moitié modifié. En cas d'arrêt, lis le message, corrige, puis **ne relance que les lignes qui n'ont pas été exécutées**.
- `${url##*/}` retire tout jusqu'au dernier `/` : c'est ainsi qu'on extrait le numéro d'issue de son URL.
- Les nouvelles US de l'EPIC 5 prennent `--parent 4`, car l'issue #4 est devenue l'EPIC 5.
- `gh issue edit` sans `--body-file` ne toucherait pas au contenu : ici, il est bien remplacé.

---

## 14. Étape 9 — Configurer les vues du Project

Quatre vues à créer (bouton **+** à droite des onglets). Ce sont elles que le correcteur regarde en premier.

### Vue 1 — « Board » (kanban)

Type **Board**, groupé par `Status`, avec les colonnes annoncées dans la copie du Bloc 2 : **Backlog → Prêt → En cours → En test / revue → Terminé** (section 9 bis.3). Place les US MVP en `Backlog` ou `Prêt`, et déplace quelques tâches techniques en `En cours` ou `Terminé` avant le dépôt : un tableau intégralement en `Backlog` suggère un projet qui n'a jamais démarré.

### Vue 2 — « Backlog par EPIC » — la vue de référence du dossier

Type **Table**, filtrée sur `label:user-story is:open`, groupée par **Parent issue**. Colonnes : Title, Labels, Status, `Story Points`, `Priorite`, `Version`. Active la somme des points par groupe : menu de la colonne `Story Points` → **Sum**.

### Vue 3 — « Répartition technique » — spécifique au multi-repo

Type **Table**, groupé par **`Repository`**, filtré sur `label:tache`. Montre la répartition de la charge entre `frontend` et `backend`. C'est ta réponse visuelle si le jury demande comment tu as organisé le travail entre les trois dépôts.

### Vue 4 — « MVP / V2 »

Type **Table**, filtrée sur `label:user-story is:open`, groupée par `Version`. Elle répond à la Question 2 du Dossier 1 du Bloc 1 (périmètre MVP) : 11 stories MVP (63 points) et 3 stories V2 (26 points).

### Vue optionnelle — Sprints

Avec le champ Iteration `Sprint` : vue Table groupée par `Sprint`. Le MVP (63 points) se répartit sur **5 à 6 sprints de 2 semaines**, soit une vélocité d'environ 10 à 13 points par sprint pour deux développeurs. Ce rythme est cohérent avec le chiffrage horaire du Bloc 1 (Dossier 2, Question 3 : environ 685 à 935 heures de développement, soit 10 à 13 semaines à deux).

---

## 15. Étape 10 — Publier et récupérer le lien pour les dossiers Word

### 15.1 Retirer les champs d'issue restreints (à faire en premier)

Si le Project a été créé depuis le modèle « Iterative development », la bascule en public échoue avec :

```
Visibility cannot be changed while the project has restricted issue fields attached: Priority.
Remove them from the project first.
```

**Ce qui se passe.** Les **champs d'issue** (*issue fields*) sont des métadonnées typées, distinctes des champs de Project. Chacun porte sa propre visibilité : « Organization only » ou « Public ». Or seuls les champs d'issue **publics** peuvent figurer dans un Project public. Tant qu'un champ restreint est attaché, GitHub refuse la publication — c'est une protection contre la fuite de métadonnées internes.

**La correction, en trente secondes :**

1. Project → **⋯** (en haut à droite) → **Settings**.
2. Dans la barre latérale gauche, sélectionne le champ d'issue nommé dans le message d'erreur (`Priority`).
3. Clique sur **Remove field**.
4. Recommence tant que la bascule en public est refusée : le message ne cite parfois qu'un champ à la fois.

Retirer un champ d'issue du Project **ne supprime ni le champ ni ses valeurs** ailleurs : tu ne perds rien.

**Puis recrée l'équivalent en champ de Project**, qui lui ne bloque jamais la publication (étape 3) :

```bash
gh project field-create "$PROJECT" --owner "$POWNER" --name "Priorite" --data-type SINGLE_SELECT --single-select-options "Must have,Should have,Could have,Won't have"
```

> Garde le nom `Priorite` sans accent : il se distingue ainsi du champ d'issue `Priority` que tu viens de retirer, et évite toute confusion à la saisie.

### 15.2 Rendre le Project public

Project → **⋯** → **Settings** → **Danger zone** → **Visibility** → **Public** → confirme.

### 15.3 Vérifier comme le fera le correcteur

L'étape que tout le monde oublie et qui coûte des points. En **navigation privée**, sans être connecté :

1. L'URL du Project : `https://github.com/orgs/Grand-Cinema-Studi/projects/<numéro>` → le tableau s'affiche sans invite de connexion.
2. Les **trois** dépôts sont accessibles. Si `grand-cinema-backend` est resté privé, ses tâches apparaîtront comme des lignes vides dans le Project public — effet désastreux.
3. Un EPIC dans `docs` → ses sub-issues et sa barre de progression sont visibles.
4. Une tâche dans `backend` → son lien vers l'US parente dans `docs` fonctionne.

Contrôle rapide en ligne de commande :

```bash
gh repo list "$OWNER" --json name,visibility --jq '.[] | "\(.name) : \(.visibility)"'
```

### 15.4 Mettre à jour les dossiers Word

**Bloc 1** (`Bloc 1_Concevoir et spécifier une solution digitale.docx`) et **Bloc 2** (Dossier 1, Question 2) renvoient tous deux au Project. Vérifie que le texte qui accompagne le lien correspond au backlog réel :

> **Lien vers l'espace de gestion agile (GitHub Projects) :**
> `https://github.com/orgs/Grand-Cinema-Studi/projects/3`
> *(Accès public en lecture, sans authentification — backlog structuré en 5 EPICs et 14 User Stories, dont 11 MVP et 3 V2)*
>
> **Dépôts du projet :**
> `grand-cinema-docs` (spécifications et backlog) · `grand-cinema-backend` (API et base de données) · `grand-cinema-frontend` (application web et contrôle d'accès)

Mets aussi à jour le `README.md` de `grand-cinema-docs` : la phrase « 4 EPICs, 12 User Stories, 97 story points » devient « 5 EPICs, 14 User Stories, 89 story points (63 MVP, 26 V2) ».

---

## 16. Checklist de conformité jury

À passer avant le dépôt du **mardi 20 octobre**.

**Dépôts**

- [ ] Les **trois** dépôts sont en visibilité **publique**
- [ ] Chacun porte une description explicite de son rôle
- [ ] `grand-cinema-docs` contient un README à jour qui oriente vers les deux autres et vers le Project

**Mise en conformité**

- [ ] Les 13 issues conservées portent les titres et contenus des copies
- [ ] Les issues #7, #13 et #15 sont fermées en « Not planned » et retirées du Project
- [ ] Les issues #10 (US 3.1) et #12 (US 4.1) ont changé de parent
- [ ] Les tâches techniques de l'issue #9 (US 2.3) ne mentionnent pas de technologie absente du schéma d'architecture (Redis, par exemple)

**Backlog — conformité avec les copies**

- [ ] Les **5 EPICs** existent dans `grand-cinema-docs`, avec les mêmes intitulés que le Bloc 1
- [ ] Les **14 User Stories** portent les mêmes titres, descriptions et critères que le Bloc 2 (Dossier 1, Question 2)
- [ ] Chaque US est rattachée à son EPIC en sub-issue, et chaque EPIC affiche sa barre de progression
- [ ] Les étiquettes annoncées dans la copie existent : **EPIC 1 à 5**, **mvp / v2**, **profil**
- [ ] Les colonnes du Board sont celles de la copie : Backlog, Prêt, En cours, En test / revue, Terminé
- [ ] Au moins une US (la 2.3) est décomposée en **tâches techniques** réparties entre `frontend` et `backend`

**Pilotage**

- [ ] `Story Points` renseigné sur les 14 US (89 au total, dont 63 MVP)
- [ ] `Priorite` MoSCoW renseignée et justifiable à l'oral
- [ ] `Version` distingue les 11 US MVP des 3 US V2
- [ ] Les 4 vues sont créées et nommées explicitement
- [ ] Quelques issues sont en `En cours` ou `Terminé`

**Publication**

- [ ] Le Project ciblé est bien celui de l'**organisation** (`gh project list --owner "$POWNER"`), pas un project personnel
- [ ] Le Project est **public**, vérifié en navigation privée
- [ ] Le texte accompagnant le lien dans les copies Bloc 1 et Bloc 2 annonce bien 5 EPICs et 14 US

---

## 17. Erreurs fréquentes et dépannage

| Symptôme | Cause | Correction |
| :--- | :--- | :--- |
| `bash: =2: command not found` | Syntaxe PowerShell (`$VAR=valeur`) | `VAR=valeur` — sans `$`, sans espaces |
| `Could not resolve to a ProjectV2 with the number N` | Mauvais propriétaire : `@me` au lieu de l'organisation, ou numéro inventé | `gh project list --owner "$POWNER"` puis corriger `PROJECT` (section 5.5) |
| Les issues partent dans un project sans rapport | `--owner "@me"` cible ton compte, pas l'organisation | Utiliser `--owner "$POWNER"` dans toutes les commandes `gh project` |
| `bash: E1: No such file or directory` | `<E1>` : `<` est une redirection d'entrée | Utiliser une variable : `--parent "$E1"` |
| `gh auth login` reste figé | Problème de TTY sous MinTTY | `winpty gh auth login` |
| Un argument commençant par `/` est transformé | Conversion de chemins MSYS2 | Préfixer par `MSYS_NO_PATHCONV=1` |
| `gh project ...` erreur de scope | Token sans le scope `project` | `gh auth refresh -s project,read:project` |
| Le flag `--parent` est refusé | GitHub CLI antérieur à 2.94 | `winget upgrade --id GitHub.cli -e` |
| `--parent 12` échoue depuis `backend` | Numéro ambigu en cross-repo | Utiliser l'**URL complète** du parent |
| `declare -A` : erreur de syntaxe | Bash 3 (rare sous Git Bash) | `bash --version` ; sinon remplacer par des variables simples |
| Le label `epic` n'existe pas dans `backend` | Les labels sont propres à chaque dépôt | Rejouer la boucle de l'étape 4 |
| Les issues existent dans le dépôt mais le Project est vide | `gh issue create` n'ajoute pas au Project | Boucle `gh project item-add` de la section 11.4, ou flag `--project` à la création |
| Une issue de `frontend` n'apparaît pas dans le Project | Elle n'a jamais été ajoutée au Project | `gh project item-add "$PROJECT" --owner "$POWNER" --url <url>` |
| Pas de section « Repositories » dans les réglages du Project | Normal sur un compte personnel : seul « Default repository » existe | Le rattachement se fait depuis chaque dépôt → onglet Projects → **Link a project** (section 7.3) |
| Des lignes vides dans le Project public | Un dépôt est resté privé | `gh repo edit <repo> --visibility public --accept-visibility-change-consequences` |
| Impossible de créer un champ `Iteration` en CLI | Type non supporté par `field-create` | Modèle « Iterative development », ou création dans l'interface web |
| `--type "Epic"` échoue | Types d'issues réservés aux organisations | Rester sur le label `epic` |
| `Visibility cannot be changed while the project has restricted issue fields attached` | Un champ d'issue « Organization only » est attaché au Project | Settings → sélectionner le champ → **Remove field**, puis republier (section 15.1) |
| Le correcteur voit un 404 | Project resté privé | Settings → Visibility → Public, puis test en navigation privée |
| Le total des points ne s'affiche pas | Somme non activée | Menu de la colonne `Story Points` → **Sum** |
| Un EPIC affiche un enfant de trop ou de moins | Un changement de parent n'a pas été fait | Section 11.5 (issues #10 et #12) |
| Les issues #7, #13 ou #15 apparaissent encore dans le Project | Une issue fermée reste dans le Project | Les retirer du Project (section 11.6) |
| Les accents des titres sont remplacés par des caractères bizarres | Encodage du terminal | `export LANG=fr_FR.UTF-8`, puis corriger le titre sur le web |
| `--label` ou `--add-label "profil: spectateur"` échoue | Le label n'existe pas encore dans `docs` | Rejouer le second bloc de l'étape 4 |
| `--project "Grand Cinema Backlog"` échoue | Nom du Project différent, ou scope `project` manquant | Vérifier le nom avec `gh project list --owner "$POWNER"`, sinon `gh auth refresh -s project` ; à défaut, ajouter l'issue au Project avec `gh project item-add` |

---

## 18. Annexe A — Tableau de chiffrage consolidé

| EPIC | US | Points | Priorité | Version |
| :--- | :--- | :---: | :--- | :--- |
| 1 — Consultation et programmation | US 1.1 Consulter les films et les horaires | 3 | Must have | MVP (V1) |
| 1 — Consultation et programmation | US 1.2 Consulter sans compte | 2 | Must have | MVP (V1) |
| 2 — Réservation et places en temps réel | US 2.1 Créer un compte et me connecter | 5 | Must have | MVP (V1) |
| 2 — Réservation et places en temps réel | US 2.2 Voir les places restantes en temps réel | 5 | Must have | MVP (V1) |
| 2 — Réservation et places en temps réel | US 2.3 Réserver un nombre de places et passer au paiement | 13 | Must have | MVP (V1) |
| 2 — Réservation et places en temps réel | US 2.4 Choisir son siège sur un plan de salle | 13 | Won't have | V2 |
| 3 — Paiement et billetterie | US 3.1 Payer ma réservation en ligne | 8 | Must have | MVP (V1) |
| 3 — Paiement et billetterie | US 3.2 Recevoir un billet dématérialisé | 5 | Must have | MVP (V1) |
| 4 — Contrôle d'accès | US 4.1 Scanner un billet | 8 | Must have | MVP (V1) |
| 5 — Administration et reporting | US 5.1 Consulter les réservations en ligne (caissier) | 3 | Should have | MVP (V1) |
| 5 — Administration et reporting | US 5.2 Gérer la programmation | 8 | Must have | MVP (V1) |
| 5 — Administration et reporting | US 5.3 Suivre les réservations | 3 | Should have | MVP (V1) |
| 5 — Administration et reporting | US 5.4 Analyser les taux de remplissage | 8 | Won't have | V2 |
| 5 — Administration et reporting | US 5.5 Être alerté en cas de forte affluence | 5 | Won't have | V2 |
| **TOTAL** | **14 User Stories** | **89** | 9 Must / 2 Should / 3 Won't | **63 MVP + 26 V2** |

> **Les points se portent sur les User Stories uniquement.** Ne chiffre pas les tâches techniques : tu compterais deux fois la même charge et fausserais le total présenté au jury.

**Lecture du chiffrage à l'oral :**

- **Le MVP représente 63 points**, soit 5 à 6 sprints de 2 semaines pour deux développeurs (vélocité d'environ 10 à 13 points par sprint). Ce rythme recoupe le chiffrage horaire du Bloc 1 (environ 685 à 935 heures de développement). Le raisonnement points → vélocité → durée → coût est exactement ce que le jury attend.
- **La part de Must have** atteint 57 points sur 89, soit 64 % du backlog complet. Sur le seul périmètre MVP, elle est plus élevée (57 sur 63) : c'est attendu, puisque le MVP a été défini au Bloc 1 comme le parcours minimal indispensable.
- **Pourquoi l'US 2.3 vaut 13 points :** elle porte le verrou anti-surbooking, la gestion de l'expiration et la concurrence sur la dernière place. C'est la story la plus risquée techniquement.
- **Ce chiffrage remplace celui de l'ancien backlog** (12 US, 97 points, vélocité de 25) : si une ancienne valeur apparaît encore dans un document, aligne-la.

---

## 19. Annexe B — Matrice de décomposition US → tâches techniques

Répartition indicative des tâches entre les deux dépôts d'implémentation. Crée en priorité celles de l'US 2.3 (en gras) ; les autres peuvent être créées au fil du Bloc 3. Les technologies citées sont celles du schéma d'architecture du Bloc 1 (PostgreSQL sur RDS, stockage S3, Stripe).

| US | Tâches `grand-cinema-backend` | Tâches `grand-cinema-frontend` |
| :--- | :--- | :--- |
| US 1.1 Films et horaires | Endpoint `GET /api/seances?date=` | Page programmation avec choix de la date, message si aucune séance |
| US 1.2 Sans compte | Routes de consultation publiques (sans authentification) | Pages de consultation responsive, lien vers la réservation |
| US 2.1 Compte | Inscription, connexion, mot de passe haché, unicité de l'e-mail | Formulaires d'inscription et de connexion, messages d'erreur |
| US 2.2 Places restantes | Calcul des places restantes (vendues + bloquées non expirées) | Affichage du compteur et de l'état « complète » |
| **US 2.3 Réservation et blocage** | **Blocage en base avec expiration à 10 minutes, en transaction** · **Endpoint `POST /api/reservations/blocage`** | **Compte à rebours du blocage** |
| US 2.4 Plan de salle (V2) | Modèle des sièges par salle, blocage par siège | Plan de salle interactif |
| US 3.1 Paiement | Intégration Stripe, confirmation du paiement, passage des places à « vendues » | Parcours de paiement, pages succès et échec |
| US 3.2 Billet | Génération du QR code unique, stockage S3, accès depuis le compte | Écran « mes billets » avec QR affichable |
| US 4.1 Scan | Endpoint de validation, statut « utilisé », contrôle de la séance | Écran de scan caméra, retour visuel validé / refusé |
| US 5.1 Caissier | Endpoint de consultation en lecture seule, droits par profil | Écran des réservations par séance |
| US 5.2 Programmation | CRUD des séances, avertissement si réservations existantes | Formulaire de gestion des séances |
| US 5.3 Suivi | Agrégation places vendues / capacité par séance | Tableau de suivi par séance |
| US 5.4 Analyse (V2) | Calcul des taux de remplissage et historique | Graphiques par séance, salle et période |
| US 5.5 Alertes (V2) | Seuils paramétrables, déclenchement des notifications | Paramétrage du seuil, affichage des alertes |

---

## Pour aller plus loin — lier le code au backlog (Bloc 3)

En multi-repo, la convention de commit diffère selon la position de l'issue.

**Tâche dans le même dépôt que le commit** — fermeture automatique par numéro :

```bash
git commit -m "feat(reservation): blocage des places en base pendant 10 minutes

Closes #7"
```

**US dans `docs`, commit dans `backend`** — référence cross-repo par nom complet :

```bash
git commit -m "feat(reservation): blocage des places en base pendant 10 minutes

Implemente la tache #7.
Contribue a Grand-Cinema-Studi/grand-cinema-docs#<numero de l'US 2.3>"
```

> **Attention :** GitHub ne ferme automatiquement une issue d'un **autre** dépôt que si celui-ci l'autorise. Ferme donc tes US manuellement, une fois toutes leurs tâches terminées — c'est de toute façon la bonne pratique : une US se ferme quand ses critères d'acceptation sont validés, pas quand le dernier commit est poussé.

À la fusion d'une branche, la tâche se ferme, passe en `Done` dans le Project, et la barre de progression de l'US puis de l'EPIC avance. La chaîne **besoin → US → tâche → commit → livraison**, traversant trois dépôts, devient démontrable en trois clics devant le jury.

---

*Sources techniques : documentation GitHub (sub-issues, issue types), GitHub Changelog du 10 juin 2026 (GitHub CLI 2.94 — sub-issues, types et dépendances), manuel GitHub CLI (`gh issue create`, `gh issue edit`, `gh project field-create`, `gh project item-edit`).*
