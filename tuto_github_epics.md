# Tutoriel — Gérer le backlog agile (EPICs / User Stories) sur GitHub en multi-repo

**Contexte :** Titre professionnel CDA (niveau 6) — Bloc 1 « Concevoir et spécifier une solution digitale », projet imposé **« Le Grand Cinéma »**.
**Objectif :** remplacer Trello par GitHub Issues + GitHub Projects pour produire le backlog attendu par le jury, avec un lien public consultable sans authentification.
**Organisation GitHub :** `Grand-Cinema-Studi` — c'est elle qui possède les dépôts **et** le Project.
**Compte authentifié dans `gh` :** ton compte personnel, distinct de l'organisation. Voir section 5.5, c'est le piège n° 1.
**Dépôts :** `grand-cinema-frontend`, `grand-cinema-backend`, `grand-cinema-docs`.
**Shell :** **Git Bash sous Windows** — toutes les commandes de ce tutoriel sont en Bash.
**Rédigé le :** 19 septembre 2026 — commandes vérifiées sur GitHub CLI ≥ 2.94.

---

## Sommaire

1. Ce que le jury attend réellement
2. Comment GitHub remplace Trello (modèle de données)
3. Architecture du backlog sur 3 dépôts — **la décision structurante**
4. Décision secondaire : compte personnel ou organisation
5. Prérequis, installation et spécificités Git Bash
6. Étape 1 — Préparer les 3 dépôts
7. Étape 2 — Créer le Project transverse et le rendre public
8. Étape 3 — Configurer les champs de pilotage
9. Étape 4 — Créer les labels dans les 3 dépôts
10. Étape 5 — Créer les 4 EPICs (contenu prêt à coller)
11. Étape 6 — Créer les 12 User Stories en sub-issues
12. Étape 7 — Décomposer en tâches techniques cross-repo
13. Étape 8 — Script Bash complet d'automatisation
14. Étape 9 — Configurer les vues du Project
15. Étape 10 — Publier et récupérer le lien pour le dossier Word
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
grand-cinema-docs        →  LE BESOIN         →  4 EPICs + 12 User Stories
       │                                          (métier, indépendant de la technique)
       ├── sub-issues ──> grand-cinema-backend →  Tâches techniques API, BDD, sécurité
       └── sub-issues ──> grand-cinema-frontend → Tâches techniques UI, PWA, intégration
```

| Dépôt | Rôle dans le backlog | Contenu |
| :--- | :--- | :--- |
| **`grand-cinema-docs`** | **Dépôt pilote du backlog** | Les 4 EPICs, les 12 User Stories, les livrables du dossier (UML, MCD/MLD, maquettes) |
| **`grand-cinema-backend`** | Dépôt d'implémentation serveur | Tâches techniques : endpoints API, schéma BDD, verrous de concurrence, passerelle de paiement |
| **`grand-cinema-frontend`** | Dépôt d'implémentation client | Tâches techniques : écrans, composants, PWA de scan, responsive |

**Pourquoi les US vivent dans `docs` et non dans le dépôt technique :** une User Story exprime un besoin utilisateur, pas une technologie. « Payer sans créer de compte » mobilise simultanément le frontend (tunnel d'achat) et le backend (transaction, passerelle). La placer dans l'un des deux dépôts serait un choix arbitraire qui casserait la lecture du backlog. Elle reste donc dans le dépôt pilote, et ses **tâches techniques** descendent dans les dépôts concernés.

Cette séparation est exactement ce que le jury veut entendre quand il demande de justifier une architecture en trois dépôts. Prépare cette phrase :

> « Le dépôt `docs` porte la spécification et le backlog fonctionnel, indépendants de toute technologie. Les dépôts `frontend` et `backend` portent l'implémentation. La hiérarchie de sub-issues traverse les trois dépôts, ce qui garantit qu'aucune tâche technique n'existe sans être rattachée à un besoin utilisateur formalisé. »

### 3.2 La hiérarchie sur trois niveaux

```
[EPIC 2] Réservation et gestion des jauges          (docs)
   └── [US 2.2] Verrouiller les places pendant le paiement   (docs)
          ├── [BACK] Verrou Redis avec TTL de 10 minutes       (backend)
          └── [FRONT] Compte à rebours du panier               (frontend)
```

GitHub autorise **8 niveaux d'imbrication et 100 sub-issues par parent** : trois niveaux passent largement. Et les sub-issues **franchissent les frontières de dépôt** — c'est la fonctionnalité qui rend toute cette architecture possible.

### 3.3 Ce qui change concrètement par rapport à un dépôt unique

Trois conséquences pratiques, chacune source d'erreur si elle est ignorée :

1. **Les labels sont propres à chaque dépôt.** Créer le label `epic` dans `docs` ne le crée pas dans `backend`. L'étape 4 boucle donc sur les trois dépôts.
2. **Le Project doit être lié aux trois dépôts** pour que les issues des trois remontent dans les mêmes vues.
3. **Le rattachement parent-enfant cross-repo exige une URL complète**, pas un numéro d'issue. Le numéro `#12` est ambigu entre trois dépôts ; l'URL ne l'est jamais.

### 3.4 Variante si tu manques de temps

Créer 4 EPICs + 12 US + une vingtaine de tâches représente une quarantaine d'issues. Si le 20 octobre approche, applique cette priorité :

- **Obligatoire pour le Bloc 1 :** les 4 EPICs et les 12 US dans `docs`, entièrement rédigés et chiffrés. C'est ce qui est noté.
- **Fortement recommandé :** les tâches techniques d'**un seul EPIC** (l'EPIC 2, le plus riche techniquement), pour démontrer la méthode de décomposition et l'usage des trois dépôts.
- **À faire pendant le Bloc 3 :** les tâches des trois autres EPICs, créées au fil du développement.

Un backlog où un EPIC est décomposé jusqu'à la tâche et les autres restent au niveau US se défend parfaitement : c'est le fonctionnement normal d'une équipe agile, qui n'affine que le haut du backlog.

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

Le backlog complet — 4 EPICs, 12 User Stories, 97 story points — est consultable
publiquement ici : **[Grand Cinema Backlog](https://github.com/orgs/Grand-Cinema-Studi/projects/NUMERO)**

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

Le modèle n'est qu'une amorce de champs et de vues, tout reste modifiable ensuite — ne passe pas plus de trente secondes sur ce choix. Mais un critère objectif le départage : **le champ Iteration est le seul que `gh project field-create` ne sait pas créer** (le CLI n'accepte que `TEXT`, `SINGLE_SELECT`, `DATE` et `NUMBER`). Un modèle qui le fournit déjà t'évite la seule manipulation obligatoirement manuelle, et ce champ matérialise directement tes 4 sprints de 2 semaines.

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

**Les labels sont propres à chaque dépôt.** Cette boucle les crée partout d'un coup :

```bash
for r in "$REPO_DOCS" "$REPO_BACK" "$REPO_FRONT"; do
  gh label create "epic"       --repo "$r" --color "6E40C9" --description "Conteneur de User Stories" --force
  gh label create "user-story" --repo "$r" --color "0E8A16" --description "Besoin utilisateur unitaire" --force
  gh label create "tache"      --repo "$r" --color "1D76DB" --description "Tache technique d'implementation" --force
  gh label create "mvp"        --repo "$r" --color "D93F0B" --description "Perimetre MVP V1" --force
  gh label create "v2"         --repo "$r" --color "FBCA04" --description "Reporte en version 2" --force
  gh label create "v3"         --repo "$r" --color "C5DEF5" --description "Reporte en version 3" --force
done
```

Le flag `--force` écrase un label existant de même nom au lieu d'échouer : la boucle est rejouable sans erreur.

---

## 10. Étape 5 — Créer les 4 EPICs (contenu prêt à coller)

Les EPICs sont créés **dans `grand-cinema-docs`**.

### 10.1 Structure d'un EPIC de qualité jury

Un EPIC n'est pas un titre. Il porte quatre informations : périmètre, valeur métier, valeur économique, critères de fin.

### 10.2 Générer les 4 fichiers de contenu

Un seul bloc à coller dans Git Bash : il crée le dossier et les quatre fichiers.

```bash
mkdir -p backlog

cat > backlog/epic1.md << 'EOF'
## Description
Fournir une interface web responsive performante permettant a tout visiteur de consulter l'offre cinematographique et evenementielle du Grand Cinema sans friction, 24h/24.

## Valeur metier
Exposer la programmation des 10 salles et l'offre du Festival de Jazz en dehors des horaires d'ouverture du guichet physique.

## Valeur economique
Maximiser la conversion visiteur-acheteur et mettre en avant les seances a forte contribution de marge (Premium/IMAX et VIP).

## Depots concernes
Frontend (affichage, filtres, responsive) et Backend (API catalogue et seances).

## Definition de termine (DoD)
- [ ] Les 3 US enfants sont fermees
- [ ] Parcours de consultation teste sur mobile et desktop
- [ ] Temps de chargement de la grille horaire mesure sous 1 seconde
EOF

cat > backlog/epic2.md << 'EOF'
## Description
Permettre la selection et l'achat securise de places avec synchronisation instantanee des disponibilites, afin d'eliminer tout risque de surreservation lors des pics d'affluence du Festival de Jazz.

## Valeur metier
Supprimer la file d'attente physique de plus de 30 minutes et fiabiliser la concurrence d'acces sur les 1 700 places du parc.

## Valeur economique
Securiser l'encaissement 24h/24 et supprimer le cout d'attrition lie aux spectateurs qui renoncent a l'achat devant la file.

## Depots concernes
Backend (verrous de concurrence, transactions, passerelle de paiement) et Frontend (tunnel d'achat, compte a rebours).

## Definition de termine (DoD)
- [ ] Les 3 US enfants sont fermees
- [ ] Test de charge simulant 200 reservations simultanees sans surbooking
- [ ] Passerelle de paiement validee en environnement de test
EOF

cat > backlog/epic3.md << 'EOF'
## Description
Emettre automatiquement des titres d'acces dematerialises infalsifiables et fournir aux agents d'accueil un dispositif rapide de validation aux portes des 10 salles.

## Valeur metier
Permettre a un agent de faire entrer jusqu'a 600 personnes en moins de 15 minutes sur les salles Premium/IMAX.

## Valeur economique
Reduire le temps d'acces unitaire a moins de 2 secondes, eliminer les couts de consommables papier et eradiquer la fraude au billet.

## Depots concernes
Backend (generation et validation cryptographique des QR codes) et Frontend (PWA de scan, mode hors ligne).

## Definition de termine (DoD)
- [ ] Les 3 US enfants sont fermees
- [ ] Scan valide en moins de 2 secondes sur terminal mobile
- [ ] Rejeu d'un billet deja scanne correctement bloque
EOF

cat > backlog/epic4.md << 'EOF'
## Description
Offrir a l'equipe de direction une interface centralisee pour planifier la programmation, configurer la modularite des salles et monitorer les indicateurs de performance commerciale.

## Valeur metier
Donner a la direction d'exploitation la visibilite sur les preventes qui lui manque totalement aujourd'hui.

## Valeur economique
Piloter l'objectif de +30 % de croissance annuelle et optimiser le taux d'occupation des 1 700 places.

## Depots concernes
Backend (CRUD seances, agregation des KPI) et Frontend (back-office, tableau de bord).

## Definition de termine (DoD)
- [ ] Les 3 US enfants sont fermees
- [ ] CRUD des seances teste de bout en bout
- [ ] Tableau de bord alimente par des donnees reelles de test
EOF

ls -l backlog/
```

### 10.3 Créer les 4 EPICs

```bash
gh issue create --repo "$REPO_DOCS" --title "[EPIC 1] Consultation et decouverte du catalogue" --body-file backlog/epic1.md --label "epic"
gh issue create --repo "$REPO_DOCS" --title "[EPIC 2] Reservation et gestion des jauges en temps reel" --body-file backlog/epic2.md --label "epic"
gh issue create --repo "$REPO_DOCS" --title "[EPIC 3] Billetterie dematerialisee et controle d'acces" --body-file backlog/epic3.md --label "epic"
gh issue create --repo "$REPO_DOCS" --title "[EPIC 4] Administration, programmation et pilotage metier" --body-file backlog/epic4.md --label "epic"
```

Chaque commande affiche l'URL de l'issue créée. **Conserve ces 4 numéros.** Pour les relire :

```bash
gh issue list --repo "$REPO_DOCS" --label "epic"
```

---

## 11. Étape 6 — Créer les 12 User Stories en sub-issues

Les US sont créées **dans `grand-cinema-docs`**, au même endroit que leurs EPICs parents : le rattachement se fait donc par **numéro**.

### 11.1 Structure d'une User Story de qualité jury

Les critères en **Given / When / Then** ne sont pas décoratifs : ils deviennent directement les cas de test de ton plan de recette du Bloc 2. Bien les rédiger maintenant t'économise une journée en novembre.

### 11.2 Mémoriser les numéros des 4 EPICs

Les commandes ci-dessous utilisent les variables `$E1` à `$E4`, qui doivent contenir les **numéros d'issue** de tes EPICs.

> **Piège Bash :** n'écris jamais un repère entre chevrons comme `--parent <E1>`. En Bash, `<` est l'opérateur de **redirection d'entrée** : la commande échoue avec `bash: E1: No such file or directory`. Utilise toujours une variable.

Relève d'abord les numéros :

```bash
gh issue list --repo "$REPO_DOCS" --label epic
```

Puis affecte-les à la main :

```bash
E1=1    # numero de l'issue [EPIC 1]
E2=2
E3=3
E4=4

echo "EPICs : $E1 $E2 $E3 $E4"   # verification
```

Ou laisse Bash les récupérer automatiquement par leur titre :

```bash
num_epic() { gh issue list --repo "$REPO_DOCS" --label epic --state all --json number,title --jq ".[] | select(.title | startswith(\"[EPIC $1]\")) | .number"; }

E1=$(num_epic 1)
E2=$(num_epic 2)
E3=$(num_epic 3)
E4=$(num_epic 4)

echo "EPICs : $E1 $E2 $E3 $E4"
```

Si l'une des variables ressort vide, l'EPIC correspondant n'a pas été créé ou son titre diffère : reprends l'étape 5 avant d'aller plus loin.

### 11.3 Les 12 User Stories

Les chaînes multilignes entre guillemets doubles fonctionnent nativement en Bash : les blocs ci-dessous se collent tels quels.

#### EPIC 1 — Catalogue

```bash
gh issue create --repo "$REPO_DOCS" --parent "$E1" --label "user-story,mvp" --title "[US 1.1] Filtrer la programmation par date, genre et type de salle" --body "**En tant que** spectateur resident
**Je souhaite** filtrer la programmation par date, par genre et par type de salle (IMAX, Standard, VIP, Evenementielle)
**Afin de** trouver rapidement la seance adaptee a mes envies

## Criteres d'acceptation
- [ ] Etant donne la page programmation, quand je selectionne une date, alors seules les seances de cette date s'affichent
- [ ] Etant donne un filtre type de salle actif, quand je le combine avec un filtre genre, alors les deux criteres s'appliquent simultanement
- [ ] Etant donne un filtre sans resultat, quand la liste est vide, alors un message explicite invite a elargir la recherche

## Estimation
Story points : 5 | Version : MVP (V1) | Priorite : Must have"

gh issue create --repo "$REPO_DOCS" --parent "$E1" --label "user-story,mvp" --title "[US 1.2] Consulter la fiche detaillee d'un film" --body "**En tant que** festivalier
**Je souhaite** consulter la fiche detaillee d'un film (synopsis, duree, bande-annonce, classification)
**Afin de** valider mon choix culturel avant d'acheter

## Criteres d'acceptation
- [ ] Etant donne la liste des seances, quand je clique sur un film, alors sa fiche complete s'affiche
- [ ] Etant donne une fiche film, quand elle contient une classification d'age, alors celle-ci est visible sans scroll
- [ ] Etant donne une fiche film, quand je consulte les seances associees, alors le type de salle est indique pour chacune

## Estimation
Story points : 3 | Version : MVP (V1) | Priorite : Must have"

gh issue create --repo "$REPO_DOCS" --parent "$E1" --label "user-story,mvp" --title "[US 1.3] Charger la grille horaire en moins d'une seconde sur mobile" --body "**En tant qu'** utilisateur mobile en itinerance
**Je souhaite** consulter la grille horaire avec un temps de chargement inferieur a 1 seconde
**Afin de** ne pas abandonner ma recherche sur un reseau cellulaire encombre

## Criteres d'acceptation
- [ ] Etant donne une connexion 4G simulee, quand j'ouvre la grille horaire, alors le premier affichage utile intervient sous 1 seconde
- [ ] Etant donne un affichage mobile, quand la page se charge, alors aucun defilement horizontal n'est necessaire
- [ ] Etant donne un audit Lighthouse mobile, quand il est execute, alors le score Performance est superieur a 90

## Estimation
Story points : 8 | Version : MVP (V1) | Priorite : Should have"
```

#### EPIC 2 — Réservation et jauges

```bash
gh issue create --repo "$REPO_DOCS" --parent "$E2" --label "user-story,mvp" --title "[US 2.1] Visualiser les places disponibles en temps reel" --body "**En tant que** client
**Je souhaite** visualiser en temps reel le nombre exact de places encore disponibles sur la seance choisie
**Afin d'** eviter un deplacement inutile vers une seance complete

## Criteres d'acceptation
- [ ] Etant donne une seance, quand j'affiche sa fiche, alors le solde de places disponibles est affiche (ex. 42 places restantes sur 300)
- [ ] Etant donne une reservation validee par un autre client, quand je rafraichis la page, alors le solde est decremente
- [ ] Etant donne une seance complete, quand je l'affiche, alors le bouton de reservation est desactive et un statut Complet apparait

## Estimation
Story points : 8 | Version : MVP (V1) | Priorite : Must have"

gh issue create --repo "$REPO_DOCS" --parent "$E2" --label "user-story,mvp" --title "[US 2.2] Verrouiller temporairement les places pendant le paiement" --body "**En tant que** client en cours de paiement
**Je souhaite** que mes places soient temporairement reservees pendant 10 minutes
**Afin qu'** aucun autre internaute ne puisse finaliser la commande a ma place

## Criteres d'acceptation
- [ ] Etant donne un panier valide, quand j'accede au paiement, alors un compte a rebours de 10 minutes s'affiche
- [ ] Etant donne des places verrouillees, quand un autre client consulte la seance, alors ces places sont decomptees des disponibilites
- [ ] Etant donne un compte a rebours expire, quand le delai est depasse, alors les places sont automatiquement remises en vente
- [ ] Etant donne 200 reservations simultanees sur la derniere place, quand elles s'executent, alors une seule aboutit et aucun surbooking n'est constate

## Estimation
Story points : 13 | Version : MVP (V1) | Priorite : Must have"

gh issue create --repo "$REPO_DOCS" --parent "$E2" --label "user-story,mvp" --title "[US 2.3] Payer sans creation de compte (Guest Checkout)" --body "**En tant que** festivalier etranger
**Je souhaite** payer via une interface securisee (CB, Apple Pay, Google Pay) sans creer de compte
**Afin de** finaliser mon achat en quelques secondes entre deux concerts

## Criteres d'acceptation
- [ ] Etant donne un panier valide, quand j'accede au paiement, alors seuls un email et un moyen de paiement me sont demandes
- [ ] Etant donne un paiement accepte, quand la transaction aboutit, alors une page de confirmation affiche la reference de commande
- [ ] Etant donne un paiement refuse, quand la transaction echoue, alors les places sont liberees et un message d'erreur explicite s'affiche
- [ ] Etant donne la passerelle de paiement, quand elle traite les donnees bancaires, alors aucune donnee carte ne transite par nos serveurs (conformite PCI-DSS)

## Estimation
Story points : 13 | Version : MVP (V1) | Priorite : Must have"
```

#### EPIC 3 — Billetterie et contrôle d'accès

```bash
gh issue create --repo "$REPO_DOCS" --parent "$E3" --label "user-story,mvp" --title "[US 3.1] Recevoir un e-billet avec QR code apres paiement" --body "**En tant que** client ayant paye
**Je souhaite** recevoir immediatement mon e-billet par email et pouvoir l'afficher en QR code sur mon smartphone
**Afin de** me presenter directement en salle sans passer par le guichet

## Criteres d'acceptation
- [ ] Etant donne un paiement valide, quand la transaction est confirmee, alors l'e-billet est envoye par email en moins de 60 secondes
- [ ] Etant donne un e-billet, quand je l'ouvre, alors il contient un QR code unique, le film, la salle, la date et l'heure
- [ ] Etant donne un e-billet, quand je le telecharge, alors un PDF est disponible et lisible hors ligne
- [ ] Etant donne deux billets distincts, quand on compare leurs QR codes, alors les identifiants cryptographiques sont differents

## Estimation
Story points : 8 | Version : MVP (V1) | Priorite : Must have"

gh issue create --repo "$REPO_DOCS" --parent "$E3" --label "user-story,mvp" --title "[US 3.2] Scanner un billet en moins de 2 secondes" --body "**En tant qu'** agent d'accueil
**Je souhaite** scanner le QR code d'un spectateur depuis mon terminal mobile
**Afin de** valider son acces en moins de 2 secondes et faire entrer 600 personnes en 15 minutes

## Criteres d'acceptation
- [ ] Etant donne l'application de scan, quand je presente un QR code valide, alors la validation s'affiche en moins de 2 secondes
- [ ] Etant donne un billet valide, quand il est scanne, alors son statut passe a Utilise en base
- [ ] Etant donne un billet d'une autre seance, quand il est scanne, alors un statut Seance incorrecte s'affiche
- [ ] Etant donne une micro-coupure reseau, quand elle survient, alors l'application conserve les scans en file et les synchronise au retour du reseau

## Estimation
Story points : 13 | Version : MVP (V1) | Priorite : Must have"

gh issue create --repo "$REPO_DOCS" --parent "$E3" --label "user-story,mvp" --title "[US 3.3] Bloquer la reutilisation d'un billet deja scanne" --body "**En tant qu'** agent d'accueil
**Je souhaite** etre alerte visuellement et par un signal sonore si un billet a deja ete scanne
**Afin d'** interdire toute reutilisation frauduleuse

## Criteres d'acceptation
- [ ] Etant donne un billet deja scanne, quand il est represente, alors un ecran rouge et une alerte sonore se declenchent
- [ ] Etant donne une alerte de doublon, quand elle s'affiche, alors la date et l'heure du premier scan sont indiquees
- [ ] Etant donne un billet refuse, quand l'alerte est fermee, alors l'application revient immediatement en mode scan

## Estimation
Story points : 5 | Version : MVP (V1) | Priorite : Should have"
```

#### EPIC 4 — Administration et pilotage

```bash
gh issue create --repo "$REPO_DOCS" --parent "$E4" --label "user-story,mvp" --title "[US 4.1] Creer et planifier les seances" --body "**En tant que** directrice d'exploitation
**Je souhaite** creer, modifier et planifier des seances en affectant un film, une salle et un tarif
**Afin de** construire le planning hebdomadaire des 10 salles

## Criteres d'acceptation
- [ ] Etant donne le back-office, quand je cree une seance, alors je selectionne un film, une salle, un horaire et un tarif
- [ ] Etant donne deux seances sur la meme salle, quand leurs horaires se chevauchent, alors la creation est refusee avec un message explicite
- [ ] Etant donne une seance ayant deja des reservations, quand je tente de la supprimer, alors une confirmation explicite est exigee

## Estimation
Story points : 8 | Version : MVP (V1) | Priorite : Must have"

gh issue create --repo "$REPO_DOCS" --parent "$E4" --label "user-story,mvp" --title "[US 4.2] Moduler la jauge des salles evenementielles" --body "**En tant que** gestionnaire du parc
**Je souhaite** moduler la capacite des salles evenementielles (70 places ou 140 places fusionnees)
**Afin d'** adapter la configuration au format de l'evenement programme

## Criteres d'acceptation
- [ ] Etant donne une salle modulable, quand je modifie sa jauge, alors la nouvelle capacite s'applique aux seances futures
- [ ] Etant donne une jauge reduite sous le nombre de places deja vendues, quand je valide, alors la modification est refusee
- [ ] Etant donne deux salles fusionnees, quand la fusion est active, alors la capacite cumulee de 140 places est exposee au front-office

## Estimation
Story points : 5 | Version : MVP (V1) | Priorite : Must have"

gh issue create --repo "$REPO_DOCS" --parent "$E4" --label "user-story,mvp" --title "[US 4.3] Visualiser un tableau de bord analytique" --body "**En tant que** responsable financiere
**Je souhaite** visualiser le taux d'occupation par salle, le panier moyen et l'evolution de la frequentation
**Afin de** mesurer l'atteinte de l'objectif de +30 % de croissance

## Criteres d'acceptation
- [ ] Etant donne le tableau de bord, quand je l'ouvre, alors le taux d'occupation des 1 700 places s'affiche en temps reel
- [ ] Etant donne une periode selectionnee, quand je la modifie, alors tous les indicateurs se recalculent
- [ ] Etant donne la periode du Festival de Jazz, quand je la compare a la periode equivalente precedente, alors l'ecart de frequentation est affiche en pourcentage

## Estimation
Story points : 8 | Version : MVP (V1) | Priorite : Must have"
```

### 11.4 Ajouter les 16 issues au Project

> **Point de blocage classique :** `gh issue create` crée l'issue dans le dépôt, **mais ne l'ajoute pas au Project**. Ce sont deux opérations distinctes. Après l'étape 6, tes 16 issues existent dans `grand-cinema-docs` et ton Project est encore vide — c'est normal, il reste à les y verser.

Ajoute-les toutes en une commande :

```bash
gh issue list --repo "$REPO_DOCS" --state all --limit 100 --json url --jq '.[].url' | while read -r url; do
  gh project item-add "$PROJECT" --owner "$POWNER" --url "$url"
done
```

Vérifie le compte — tu dois obtenir 16 :

```bash
gh project item-list "$PROJECT" --owner "$POWNER" --limit 100 --format json --jq '.items | length'
```

> **Pour les issues créées ensuite**, le flag `--project` évite ce rattrapage :
> ```bash
> gh issue create --repo "$REPO_DOCS" --project "Grand Cinema Backlog" --title "..."
> ```

### 11.5 Renseigner les champs de pilotage en masse

La `Version`, sur toutes les User Stories :

```bash
gh issue list --repo "$REPO_DOCS" --label user-story --state all --limit 100 --json url --jq '.[].url' | while read -r url; do
  gh project item-edit "$PROJECT" --owner "$POWNER" --url "$url" --field "Version" --value "MVP (V1)"
done
```

> **Pourquoi c'est nécessaire alors que le label `mvp` est déjà posé :** le label vit sur l'issue, dans le dépôt ; le champ vit dans le Project et naît toujours vide. Rien ne les relie automatiquement. Ces boucles ne font que recopier l'information du label — la source de vérité — vers le champ exploitable par les vues.

Puis les points et priorités, lus dans le titre de chaque US :

```bash
declare -A PTS=( ["US 1.1"]=5 ["US 1.2"]=3 ["US 1.3"]=8 ["US 2.1"]=8 ["US 2.2"]=13 ["US 2.3"]=13 ["US 3.1"]=8 ["US 3.2"]=13 ["US 3.3"]=5 ["US 4.1"]=8 ["US 4.2"]=5 ["US 4.3"]=8 )
declare -A PRIO=( ["US 1.3"]="Should have" ["US 3.3"]="Should have" )   # les autres : Must have

gh issue list --repo "$REPO_DOCS" --label user-story --state all --limit 100 --json url,title --jq '.[] | [.url, .title] | @tsv' | while IFS=$'\t' read -r url title; do
  cle=$(echo "$title" | sed -n 's/^\[\(US [0-9]\.[0-9]\)\].*/\1/p')
  [ -z "$cle" ] && { echo "IGNORE (pas une US) : $title"; continue; }
  pts="${PTS[$cle]}"
  prio="${PRIO[$cle]:-Must have}"
  [ -z "$pts" ] && { echo "IGNORE (non chiffree) : $title"; continue; }
  echo "$cle -> $pts pts, $prio"
  gh project item-edit "$PROJECT" --owner "$POWNER" --url "$url" --field "Story Points" --number "$pts"
  gh project item-edit "$PROJECT" --owner "$POWNER" --url "$url" --field "Priorite"     --value  "$prio"
done
```

En pratique, saisir les points **directement dans la vue Table** du Project reste plus rapide pour douze lignes : c'est une grille, tu tabules d'une cellule à l'autre. Réserve ces boucles au rattrapage ou aux grands volumes.

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

### 12.2 Exemple complet — décomposition de l'US 2.2

L'US 2.2 (verrou TTL) est la plus riche techniquement : c'est celle à décomposer en priorité.

```bash
US22="https://github.com/Grand-Cinema-Studi/grand-cinema-docs/issues/12"   # adapte le numero

gh issue create --repo "$REPO_BACK" --parent "$US22" --label "tache" --title "[BACK] Implementer le verrou de reservation Redis avec TTL de 10 minutes" --body "Tache technique de l'US 2.2 (grand-cinema-docs).

## Perimetre
Mettre en place le verrouillage temporaire des places via un cache cle-valeur Redis, decouple de la base transactionnelle principale, afin de garantir l'integrite de l'inventaire lors des pics de charge.

## Criteres de realisation
- [ ] Cle Redis par couple seance/place avec expiration automatique a 600 secondes
- [ ] Operation de pose de verrou atomique (SET NX EX) pour eliminer toute race condition
- [ ] Liberation automatique des places a l'expiration, sans intervention applicative
- [ ] Test de charge : 200 requetes concurrentes sur la derniere place, une seule aboutit
- [ ] Tests unitaires ecrits et passants"

gh issue create --repo "$REPO_BACK" --parent "$US22" --label "tache" --title "[BACK] Exposer l'endpoint POST /api/panier de pose de verrou" --body "Tache technique de l'US 2.2 (grand-cinema-docs).

## Perimetre
Exposer l'API de creation de panier qui pose les verrous et retourne l'horodatage d'expiration au client.

## Criteres de realisation
- [ ] Reponse 201 avec identifiant de panier et timestamp d'expiration
- [ ] Reponse 409 si les places demandees sont deja verrouillees
- [ ] Contrat d'API documente au format OpenAPI
- [ ] Tests d'integration ecrits et passants"

gh issue create --repo "$REPO_FRONT" --parent "$US22" --label "tache" --title "[FRONT] Afficher le compte a rebours du panier" --body "Tache technique de l'US 2.2 (grand-cinema-docs).

## Perimetre
Afficher un compte a rebours synchronise sur le timestamp d'expiration renvoye par l'API, et gerer l'expiration cote client.

## Criteres de realisation
- [ ] Compte a rebours visible en permanence pendant le tunnel de paiement
- [ ] Alerte visuelle sous 2 minutes restantes
- [ ] Redirection vers le catalogue et message explicite a l'expiration
- [ ] Composant teste unitairement"
```

La matrice complète des tâches pour les 12 US figure en **Annexe B**.

---

## 13. Étape 8 — Script Bash complet d'automatisation

Enregistre sous `backlog/creer-backlog.sh`, rends-le exécutable avec `chmod +x backlog/creer-backlog.sh`, puis lance-le par `./backlog/creer-backlog.sh`.

Le script crée les 4 EPICs, capture leurs numéros, crée les 12 US rattachées, les ajoute au Project et renseigne leurs champs. Les corps des US doivent être placés dans `backlog/us11.md` … `backlog/us43.md` (même méthode heredoc qu'à l'étape 5).

```bash
#!/usr/bin/env bash
set -euo pipefail

# --- Parametres -----------------------------------------------------
OWNER="Grand-Cinema-Studi"
REPO_DOCS="$OWNER/grand-cinema-docs"
REPO_BACK="$OWNER/grand-cinema-backend"
REPO_FRONT="$OWNER/grand-cinema-frontend"
PROJECT=2

declare -A EPIC_NUM     # cle -> numero d'issue
declare -A US_URL       # cle -> URL complete de l'US

# --- Fonctions ------------------------------------------------------
creer_epic() {          # $1=cle  $2=titre  $3=fichier
  local url
  url=$(gh issue create --repo "$REPO_DOCS" --title "$2" --body-file "$3" --label "epic")
  EPIC_NUM[$1]="${url##*/}"          # le numero = ce qui suit le dernier /
  echo "EPIC cree : $2 -> #${EPIC_NUM[$1]}"
  gh project item-add "$PROJECT" --owner "$POWNER" --url "$url"
}

creer_us() {            # $1=cle  $2=cle_epic  $3=titre  $4=fichier  $5=points  $6=priorite
  local url parent
  parent="${EPIC_NUM[$2]}"
  url=$(gh issue create --repo "$REPO_DOCS" --title "$3" --body-file "$4" --label "user-story,mvp" --parent "$parent")
  US_URL[$1]="$url"
  echo "US creee : $3 (parent #$parent)"
  gh project item-add  "$PROJECT" --owner "$POWNER" --url "$url"
  gh project item-edit "$PROJECT" --owner "$POWNER" --url "$url" --field "Story Points" --number "$5"
  gh project item-edit "$PROJECT" --owner "$POWNER" --url "$url" --field "Priorite"     --value  "$6"
  gh project item-edit "$PROJECT" --owner "$POWNER" --url "$url" --field "Version"      --value  "MVP (V1)"
}

creer_tache() {         # $1=repo  $2=cle_us  $3=titre  $4=fichier
  local url parent
  parent="${US_URL[$2]}"             # URL complete : rattachement cross-repo
  url=$(gh issue create --repo "$1" --title "$3" --body-file "$4" --label "tache" --parent "$parent")
  echo "Tache creee dans $1 : $3"
  gh project item-add "$PROJECT" --owner "$POWNER" --url "$url"
}

# --- 1. Les 4 EPICs -------------------------------------------------
creer_epic E1 "[EPIC 1] Consultation et decouverte du catalogue"          backlog/epic1.md
creer_epic E2 "[EPIC 2] Reservation et gestion des jauges en temps reel"  backlog/epic2.md
creer_epic E3 "[EPIC 3] Billetterie dematerialisee et controle d'acces"   backlog/epic3.md
creer_epic E4 "[EPIC 4] Administration, programmation et pilotage metier" backlog/epic4.md

# --- 2. Les 12 User Stories -----------------------------------------
creer_us US11 E1 "[US 1.1] Filtrer la programmation par date, genre et type de salle" backlog/us11.md 5  "Must have"
creer_us US12 E1 "[US 1.2] Consulter la fiche detaillee d'un film"                    backlog/us12.md 3  "Must have"
creer_us US13 E1 "[US 1.3] Charger la grille horaire en moins d'une seconde"          backlog/us13.md 8  "Should have"
creer_us US21 E2 "[US 2.1] Visualiser les places disponibles en temps reel"           backlog/us21.md 8  "Must have"
creer_us US22 E2 "[US 2.2] Verrouiller temporairement les places pendant le paiement" backlog/us22.md 13 "Must have"
creer_us US23 E2 "[US 2.3] Payer sans creation de compte (Guest Checkout)"            backlog/us23.md 13 "Must have"
creer_us US31 E3 "[US 3.1] Recevoir un e-billet avec QR code apres paiement"          backlog/us31.md 8  "Must have"
creer_us US32 E3 "[US 3.2] Scanner un billet en moins de 2 secondes"                  backlog/us32.md 13 "Must have"
creer_us US33 E3 "[US 3.3] Bloquer la reutilisation d'un billet deja scanne"          backlog/us33.md 5  "Should have"
creer_us US41 E4 "[US 4.1] Creer et planifier les seances"                            backlog/us41.md 8  "Must have"
creer_us US42 E4 "[US 4.2] Moduler la jauge des salles evenementielles"               backlog/us42.md 5  "Must have"
creer_us US43 E4 "[US 4.3] Visualiser un tableau de bord analytique"                  backlog/us43.md 8  "Must have"

# --- 3. Taches techniques cross-repo (exemple sur l'US 2.2) ---------
creer_tache "$REPO_BACK"  US22 "[BACK] Implementer le verrou de reservation Redis avec TTL de 10 minutes" backlog/t-us22-back1.md
creer_tache "$REPO_BACK"  US22 "[BACK] Exposer l'endpoint POST /api/panier de pose de verrou"             backlog/t-us22-back2.md
creer_tache "$REPO_FRONT" US22 "[FRONT] Afficher le compte a rebours du panier"                           backlog/t-us22-front1.md

echo ""
echo "Backlog cree : 4 EPICs, 12 User Stories (97 points), 3 taches techniques."
```

**Notes de lecture du script :**

- `set -euo pipefail` arrête le script à la première erreur au lieu de continuer en créant des issues incohérentes.
- `declare -A` crée un tableau associatif — disponible en Bash 4+, ce qui est le cas de Git Bash.
- `${url##*/}` retire tout jusqu'au dernier `/` : c'est ainsi qu'on extrait le numéro d'issue de son URL.
- Les EPICs et US se rattachent par **numéro** (même dépôt), les tâches par **URL** (cross-repo).

> **Avant de lancer :** teste sur un dépôt bac à sable. Un script qui crée quarante issues dans le mauvais dépôt se nettoie à la main, une par une.

---

## 14. Étape 9 — Configurer les vues du Project

Quatre vues à créer (bouton **+** à droite des onglets). Ce sont elles que le correcteur regarde en premier.

### Vue 1 — « Board » (kanban)

Type **Board**, groupé par `Status`. Déplace quelques issues en `In Progress` et `Done` avant le dépôt : un tableau intégralement en `Todo` suggère un projet qui n'a jamais démarré.

### Vue 2 — « Backlog par EPIC » — la vue de référence du dossier

Type **Table**, groupé par **Parent issue** (ou filtré sur `label:user-story`). Colonnes : Title, `Repository`, Status, `Story Points`, `Priorite`, `Version`. Active la somme des points par groupe : menu de la colonne `Story Points` → **Sum**.

### Vue 3 — « Répartition technique » — spécifique au multi-repo

Type **Table**, groupé par **`Repository`**, filtré sur `label:tache`. Montre la répartition de la charge entre `frontend` et `backend`. C'est ta réponse visuelle si le jury demande comment tu as organisé le travail entre les trois dépôts.

### Vue 4 — « Roadmap MVP / V2 / V3 »

Type **Table** ou **Roadmap**, groupé par `Version`. Répond à la Question 2 du Dossier 1 sur le périmètre MVP et la trajectoire d'évolutivité. Ajoutes-y les fonctionnalités V2 et V3 de ta matrice d'arbitrage sous forme d'issues labellisées `v2` et `v3` dans `grand-cinema-docs`.

### Vue optionnelle — Sprints

Avec le champ Iteration `Sprint` fourni par le modèle : vue Table groupée par `Sprint`, 97 points répartis sur 4 sprints de 2 semaines (vélocité ≈ 25 points).

---

## 15. Étape 10 — Publier et récupérer le lien pour le dossier Word

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

### 15.4 Mettre à jour le dossier Word

Dans `Bloc 1_Concevoir et spécifier une solution digitale.docx`, section 4 de la Question 1, le repère surligné en jaune attend :

> **Lien vers l'espace de gestion agile (GitHub Projects) :**
> `https://github.com/orgs/Grand-Cinema-Studi/projects/<numéro>`
> *(Accès public en lecture, sans authentification — backlog structuré en 4 EPICs et 12 User Stories, chiffré à 97 story points)*
>
> **Dépôts du projet :**
> `grand-cinema-docs` (spécifications et backlog) · `grand-cinema-backend` (API et base de données) · `grand-cinema-frontend` (application web et PWA)

---

## 16. Checklist de conformité jury

À passer avant le dépôt du **mardi 20 octobre**.

**Dépôts**

- [ ] Les **trois** dépôts sont en visibilité **publique**
- [ ] Chacun porte une description explicite de son rôle
- [ ] `grand-cinema-docs` contient un README qui oriente vers les deux autres et vers le Project

**Backlog**

- [ ] Les **4 EPICs** existent dans `grand-cinema-docs`, avec description, valeur métier, valeur économique, dépôts concernés et DoD
- [ ] Les **12 User Stories** sont rattachées en sub-issues et la barre de progression apparaît sur chaque EPIC
- [ ] Chaque US respecte le format « En tant que… je souhaite… afin de… »
- [ ] Chaque US porte au moins **3 critères d'acceptation** en Given/When/Then
- [ ] Au moins un EPIC est décomposé en **tâches techniques** réparties entre `frontend` et `backend`
- [ ] Chaque tâche technique est rattachée à son US parente (lien cross-repo fonctionnel)

**Pilotage**

- [ ] `Story Points` renseigné sur les 12 US (total 97)
- [ ] `Priorite` MoSCoW renseignée et justifiable à l'oral
- [ ] `Version` distingue le MVP des reports V2 et V3
- [ ] Les fonctionnalités V2 et V3 de la matrice d'arbitrage figurent dans le Project
- [ ] Les 3 dépôts sont rattachés au Project depuis leur onglet Projects (confort, non bloquant)
- [ ] Les 4 vues sont créées et nommées explicitement
- [ ] Quelques issues sont en `In Progress` ou `Done`

**Publication**

- [ ] Le Project ciblé est bien celui de l'**organisation** (`gh project list --owner "$POWNER"`), pas un project personnel
- [ ] Les champs d'issue restreints ont été retirés du Project (sinon la publication est refusée)
- [ ] Le Project est **public**, vérifié en navigation privée
- [ ] Le lien GitHub remplace le repère jaune dans le document Word
- [ ] Les trois dépôts sont cités dans le document Word

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

---

## 18. Annexe A — Tableau de chiffrage consolidé

| EPIC | US | Points | Priorité | Version |
| :--- | :--- | :---: | :--- | :--- |
| 1 — Catalogue | US 1.1 Filtrage programmation | 5 | Must have | MVP (V1) |
| 1 — Catalogue | US 1.2 Fiche détaillée film | 3 | Must have | MVP (V1) |
| 1 — Catalogue | US 1.3 Performance mobile < 1 s | 8 | Should have | MVP (V1) |
| 2 — Réservation | US 2.1 Jauge temps réel | 8 | Must have | MVP (V1) |
| 2 — Réservation | US 2.2 Verrou TTL 10 minutes | 13 | Must have | MVP (V1) |
| 2 — Réservation | US 2.3 Paiement Guest Checkout | 13 | Must have | MVP (V1) |
| 3 — Billetterie | US 3.1 E-billet QR code | 8 | Must have | MVP (V1) |
| 3 — Billetterie | US 3.2 Scan agent < 2 s | 13 | Must have | MVP (V1) |
| 3 — Billetterie | US 3.3 Anti-doublon | 5 | Should have | MVP (V1) |
| 4 — Administration | US 4.1 CRUD séances | 8 | Must have | MVP (V1) |
| 4 — Administration | US 4.2 Jauges modulables | 5 | Must have | MVP (V1) |
| 4 — Administration | US 4.3 Tableau de bord KPI | 8 | Must have | MVP (V1) |
| **TOTAL** | **12 User Stories** | **97** | 10 Must / 2 Should | MVP complet |

> **Les points se portent sur les User Stories uniquement.** Ne chiffre pas les tâches techniques : tu compterais deux fois la même charge et fausserais le total présenté au jury.

**Lecture du chiffrage à l'oral :** avec deux développeurs, des sprints de deux semaines et une vélocité de 25 points, le MVP représente **4 sprints, soit environ 8 semaines de développement**. Ce raisonnement — points → vélocité → durée → coût — est exactement ce que le jury attend. Prépare-le, la question tombe presque systématiquement.

---

## 19. Annexe B — Matrice de décomposition US → tâches techniques

Répartition indicative des tâches entre les deux dépôts d'implémentation. Crée en priorité celles de l'EPIC 2 (en gras) ; les autres peuvent être créées au fil du Bloc 3.

| US | Tâches `grand-cinema-backend` | Tâches `grand-cinema-frontend` |
| :--- | :--- | :--- |
| US 1.1 Filtrage | Endpoint `GET /api/seances` avec filtres date, genre, type de salle | Composant de filtres + liste de résultats responsive |
| US 1.2 Fiche film | Endpoint `GET /api/films/{id}` | Page fiche film avec lecteur de bande-annonce |
| US 1.3 Performance | Cache des requêtes catalogue, pagination | Optimisation du bundle, lazy loading, audit Lighthouse |
| US 2.1 Jauge | Calcul du solde de places, exposition temps réel | Affichage du solde + état « Complet » |
| **US 2.2 Verrou TTL** | **Verrou Redis SET NX EX 600 s** · **Endpoint `POST /api/panier`** | **Compte à rebours du panier** |
| US 2.3 Paiement | Intégration passerelle PCI-DSS, webhook de confirmation | Tunnel Guest Checkout, pages succès et échec |
| US 3.1 E-billet | Génération QR cryptographique, PDF, envoi email | Écran de confirmation avec QR affichable |
| US 3.2 Scan | Endpoint de validation, statut du billet, file de synchronisation | PWA de scan caméra, mode hors ligne |
| US 3.3 Anti-doublon | Détection du rejeu, horodatage du premier scan | Alerte visuelle et sonore, retour au mode scan |
| US 4.1 CRUD séances | CRUD séances, contrôle de chevauchement | Formulaire de planification back-office |
| US 4.2 Jauges | Modèle de salle modulable, règles de validation | Interface de configuration des jauges |
| US 4.3 KPI | Agrégation des indicateurs, endpoint statistiques | Tableau de bord et graphiques |

---

## Pour aller plus loin — lier le code au backlog (Bloc 3)

En multi-repo, la convention de commit diffère selon la position de l'issue.

**Tâche dans le même dépôt que le commit** — fermeture automatique par numéro :

```bash
git commit -m "feat(panier): verrou Redis avec TTL de 10 minutes

Closes #7"
```

**US dans `docs`, commit dans `backend`** — référence cross-repo par nom complet :

```bash
git commit -m "feat(panier): verrou Redis avec TTL de 10 minutes

Implemente la tache #7.
Contribue a Grand-Cinema-Studi/grand-cinema-docs#12"
```

> **Attention :** GitHub ne ferme automatiquement une issue d'un **autre** dépôt que si celui-ci l'autorise. Ferme donc tes US manuellement, une fois toutes leurs tâches terminées — c'est de toute façon la bonne pratique : une US se ferme quand ses critères d'acceptation sont validés, pas quand le dernier commit est poussé.

À la fusion d'une branche, la tâche se ferme, passe en `Done` dans le Project, et la barre de progression de l'US puis de l'EPIC avance. La chaîne **besoin → US → tâche → commit → livraison**, traversant trois dépôts, devient démontrable en trois clics devant le jury.

---

*Sources techniques : documentation GitHub (sub-issues, issue types), GitHub Changelog du 10 juin 2026 (GitHub CLI 2.94 — sub-issues, types et dépendances), manuel GitHub CLI (`gh issue create`, `gh issue edit`, `gh project field-create`, `gh project item-edit`).*
