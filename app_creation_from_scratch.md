# Journal de bord — Titre CDA « Le Grand Cinéma »

Suivi résumé de tout ce qui a été produit, session par session.
Une ligne par livrable : quoi, où, état. Les détails restent dans les fichiers cités.

**Projet imposé :** Le Grand Cinéma — 10 salles, 1 700 places, billetterie dématérialisée.
**Organisation GitHub :** `Grand-Cinema-Studi` (type : Organization)
**Project :** `Grand Cinema Backlog` — n° 3 — https://github.com/orgs/Grand-Cinema-Studi/projects/3
**Dépôts :** `grand-cinema-docs` · `grand-cinema-backend` · `grand-cinema-frontend`
**Dossier local :** `STUDI/Grand Cinema App/` (un sous-dossier par dépôt ; `grand-cinema-docs` est cloné et à jour)

---

## Jalons

| Échéance | Attendu | État |
| :--- | :--- | :--- |
| Mardi 20 octobre 2026 | 1er dépôt Blocs 1, 2 et 3 (retours jurys) | En cours — Bloc 1 rédigé, Blocs 2 et 3 à produire |
| Vendredi 20 novembre 2026 | Dépôt final obligatoire + vidéos | À venir |
| Mardi 24 novembre 2026 | Vidéos de réponse aux formateurs | À venir |

---

## État d'avancement global

| Bloc | Dossier | Question | État |
| :--- | :--- | :--- | :--- |
| **1** | 1 — La demande du client | Q1 — Demande, objectifs, utilisateurs, besoins | ✅ Rédigée et intégrée au .docx |
| **1** | 1 — La demande du client | Q2 — Périmètre MVP et justifications | ✅ Rédigée et intégrée au .docx |
| **1** | 2 — Les solutions | Q1 — Lister les solutions | ✅ Rédigée (3 familles : SaaS, CMS, sur-mesure) |
| **1** | 2 — Les solutions | Q2 — Utilisables ou non, justifier | ✅ Rédigée |
| **1** | 2 — Les solutions | Q3 — Coût de mise en place | 🟨 Rédigée, à équilibrer : chiffrage détaillé seulement pour la Famille C (sur-mesure) ; A et B uniquement dans le tableau comparatif |
| **1** | 3 — L'architecture | Q1 — Justifier le choix d'hébergement | ✅ Rédigée (AWS, région Paris, 4 critères) |
| **1** | 3 — L'architecture | Q2 — Schéma d'architecture logicielle | 🟨 Schéma inséré, à reprendre (thème clair, flèche Stripe depuis la couche applicative, verrou anti-surbooking visible) |
| **1** | 3 — L'architecture | Q3 — Justifier le choix d'architecture | ✅ Rédigée (modularité, TMA, conduite du changement) |
| **2** | — | Backlog, plan de recette, tests | 🟨 Backlog GitHub fait (4 EPICs, 12 US, 97 pts) ; plan de recette, tests et anomalies à faire (gabarit `Bloc 2/` encore vide) |
| **3** | — | Développement, API, BDD, sécurité | ⬜ À faire |

---

## 18 septembre 2026

### Bloc 1 — Dossier 1, Questions 1 et 2 → intégrées au livrable Word

Les réponses rédigées dans `Bloc 1/exam_bloc_1_answer.md` ont été insérées dans le document officiel
`Bloc 1/Bloc 1_Concevoir et spécifier une solution digitale.docx`, sous les intitulés de questions existants.

Contenu de la **Question 1** : diagnostic de l'existant (parc de 10 salles, 1 700 places, files de 30 min),
démarcation objectifs économiques / enjeux métiers (compétence C1.1), 4 personas d'ingénierie
(Marc Dupont, Elena Rostova, Thomas Martin, Sophie Laurent), backlog en 4 EPICs et 12 User Stories.

Contenu de la **Question 2** : définition académique du MVP (Robinson / Ries, Lean Startup), périmètre
fonctionnel V1, matrice d'arbitrage V1/V2/V3 avec justification de chaque report, anticipation
architecturale non figée (modèle de données extensible, API-First, isolation du verrou de concurrence).

Traitement appliqué au passage : suppression des marqueurs `[cite: …]` et conversion des 7 schémas ASCII
en **11 vrais tableaux Word**. Rendu vérifié page par page, document validé sur 18 pages.

### Outillage — Tutoriel GitHub pour le backlog agile

Décision : **abandon de Trello au profit de GitHub Issues + Projects**, pour la traçabilité
besoin → code que Trello ne permet pas.

Livrable : `tuto_github_epics.md` — tutoriel complet couvrant la création du Project, la préparation des
3 dépôts, les labels, les 4 EPICs et 12 User Stories entièrement rédigés et prêts à coller, la
décomposition en tâches techniques, un script d'automatisation (converti en Bash le 20 septembre), les vues du Project et la
checklist de conformité jury. Double méthode : interface web et `gh` CLI.

**État : tutoriel rédigé, exécution sur GitHub pas encore faite.**

---

## 19-20 septembre 2026

### Mise en place du backlog sur GitHub

Backlog créé dans `grand-cinema-docs` : **4 EPICs et 12 User Stories**, avec critères d'acceptation
en Given/When/Then. Project **« Grand Cinema Backlog » n° 3**, sous l'organisation.
Champs de pilotage créés : `Story Points`, `Priorite` (MoSCoW), `Version` (MVP/V2/V3).

Trois pièges rencontrés, tous documentés dans le tutoriel :

- **`--owner "@me"` ne cible pas l'organisation.** Les 16 issues sont d'abord parties dans un project
  personnel sans rapport (GLOOMQUEST n° 4). Toutes les commandes `gh project` prennent désormais `--owner "$POWNER"`.
- **Un champ d'issue restreint (`Priority`) bloque la publication** du Project en public.
  À retirer depuis Settings avant toute bascule en visibilité publique.
- **`gh issue create` n'ajoute pas au Project** : `gh project item-add` est une étape distincte.

Le tutoriel `tuto_github_epics.md` a été entièrement converti en **Bash** (Git Bash sous Windows).

### Livrable Word — lien GitHub définitif

Le repère jaune est remplacé par l'URL réelle : `https://github.com/orgs/Grand-Cinema-Studi/projects/3`.
Plus aucun surlignage dans le document. Reste à **vérifier l'accès en navigation privée** — le Project
et les 3 dépôts doivent être publics, sinon le correcteur tombe sur un 404.

---

## Octobre 2026

### Réorganisation du dossier et synchronisation Git (7-8 octobre)

Le dossier local est réorganisé sous `STUDI/Grand Cinema App/grand-cinema-docs/` :
`1 - Conception…`, `2 - Gestion…`, `3 - Développement…` (cours Studi récupérés tels quels, environ 250 PDF),
`Énoncés/` (2 PDF officiels), `Réponses Bloc 1 2 & 3/` (un sous-dossier par bloc) et `TEST_CODES/`.
Les anciens brouillons (`exam_bloc_1_answer.md`, `planning_bloc1_fasttrack.md`, dossier `backlog/`) ont été supprimés volontairement.

Piège rencontré : `git pull` échouait sur Windows (`error: invalid path`) car des dossiers de cours contenaient
un `:` (« Agile : Scrum », « Agile : Kanban »), caractère interdit par NTFS. Dossiers renommés avec un tiret ;
le pull passe. **Règle : pas de `:` ni `? * " < > |` dans les noms de fichiers du dépôt.**

### Relecture du livrable Bloc 1 (7 octobre, lecture seule)

Les 8 questions sont toutes rédigées dans le .docx (~3 800 mots, 11 tableaux, 1 schéma d'architecture).
Points à reprendre avant le 20 octobre :

- **D2 Q3** : chiffrer aussi les Familles A (commissions par billet) et B (licences, plugins, maintenance), sur une base comparable.
- **D3 Q2** : schéma en thème clair, flèche Stripe partant de la couche applicative (pas de la base), verrou anti-surbooking visible, cohérence avec « architecture modulaire » de la Q3.
- **Poids du fichier (10,9 Mo)** : 7 polices incorporées ; décocher « Incorporer les polices » dans Word.
- Paragraphes vides en fin de réponses à nettoyer. La mention « Trello » restante vient de l'énoncé officiel : à laisser.

**Évaluation :** Bloc 1 rédigé à environ 90 %. Le risque est sur les Blocs 2 et 3, non démarrés.

### Plan indicatif jusqu'au 20 octobre (~33 h d'étude)

| Bloc | Charge | Contenu |
| :--- | :--- | :--- |
| 1 | ~3 h | Corrections ci-dessus, puis gel du document |
| 2 | ~12 h | Plan de recette, tests écrits et exécutés, suivi des anomalies ; vérifier cours « Terminés » et QCM validé (sinon énoncé verrouillé) |
| 3 | ~15 h | Squelette qui tourne : API de réservation avec verrou anti-surbooking, base de données, un écran frontend, tests |
| — | ~3 h | Marge |

---

## Décisions structurantes prises

| Décision | Choix retenu | Motif |
| :--- | :--- | :--- |
| Outil de gestion agile | GitHub Issues + Projects | Traçabilité besoin → commit, absente de Trello |
| Découpage des dépôts | 3 dépôts : docs / backend / frontend | Séparer le besoin de son implémentation |
| Emplacement des EPICs et US | `grand-cinema-docs` | Une US est métier, pas technique ; elle mobilise souvent les deux dépôts |
| Emplacement des tâches techniques | `backend` et `frontend`, en sub-issues cross-repo | Le travail vit où le code vit |
| Matérialisation de l'EPIC | Label `epic` + hiérarchie sub-issues | Les types natifs sont disponibles (organisation) mais non notés ; ne pas complexifier avant le dépôt |
| Propriétaire des dépôts et du Project | Organisation `Grand-Cinema-Studi` | `--owner "@me"` cible le compte personnel : source d'erreur silencieuse |
| Nom de l'organisation | `Grand-Cinema-Studi` | Remplace `M-Motors-studi`, sans rapport avec le projet imposé |
| Abandon du champ `Niveau` | Les labels `epic` / `user-story` suffisent | Doublon coûteux en saisie et risque d'incohérence |
| Shell de travail | Git Bash sous Windows | Tutoriel entièrement converti en Bash |
| Périmètre MVP | V1 = socle transactionnel, plan 2D reporté en V2 | Budget limité, salles modulables |
| Chiffrage du backlog | 97 story points, 4 sprints de 2 semaines | Vélocité 25 pts, 2 développeurs |

---

## Prochaines actions

À vérifier / finir côté GitHub (état exact non confirmé dans ce journal) :

1. **Retirer le champ d'issue `Priority`** du Project 3, puis le passer en **public**.
2. **Vérifier en navigation privée** : Project 3 et les 3 dépôts accessibles sans compte.
3. **Renseigner les champs** sur les 12 US : Story Points (97 au total), Priorite MoSCoW, Version.
4. **Nettoyer le project personnel n° 4 (GLOOMQUEST)** : retirer les 16 issues et les 4 champs ajoutés par erreur.

Livrables, par priorité :

5. **Bloc 2** : plan de recette, tests, anomalies (gabarit dans `Réponses Bloc 1 2 & 3/Bloc 2/`).
6. **Bloc 3** : squelette backend/frontend fonctionnel (dépôts encore vides).
7. **Bloc 1** : corriger D2 Q3 et le schéma D3 Q2, retirer les polices incorporées, puis geler.
8. **Vidéos** (20 et 24 novembre) : trames de script, à planifier après le premier dépôt.

---

## Fichiers du projet

Racine : `STUDI/Grand Cinema App/grand-cinema-docs/`

| Fichier / dossier | Rôle |
| :--- | :--- |
| `app_creation_from_scratch.md` | Ce journal de bord |
| `tuto_github_epics.md` | Tutoriel backlog agile sur GitHub (multi-repo, Bash) |
| `deadlines.md` · `blocs.md` | Rétroplanning et cadrage des blocs |
| `README.md` · `.gitignore` | Présentation du dépôt ; `node_modules` ignoré |
| `Énoncés/` | Énoncés officiels Bloc 1 et Bloc 2 (PDF) |
| `1 - Conception…/` · `2 - Gestion…/` · `3 - Développement…/` | Cours Studi (PDF), un sous-dossier par module |
| `Réponses Bloc 1 2 & 3/Bloc 1/` | **Livrable Bloc 1 à déposer** (.docx) |
| `Réponses Bloc 1 2 & 3/Bloc 2/` | Gabarit Bloc 2 à remplir (.docx ; .odt de 35 Mo : ne pas déposer) |
| `TEST_CODES/typescript/` | Essais TypeScript |

---

<!--
MODÈLE POUR LA PROCHAINE ENTRÉE — copier-coller sous la dernière date

## JJ mois 2026

### <Bloc / Dossier / Sujet>
<Ce qui a été produit, en 3 à 5 lignes. Où ça vit. État : terminé / en cours / à reprendre.>

-->
