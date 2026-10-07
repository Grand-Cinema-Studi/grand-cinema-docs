# Journal de bord — Titre CDA « Le Grand Cinéma »

Suivi résumé de tout ce qui a été produit, session par session.
Une ligne par livrable : quoi, où, état. Les détails restent dans les fichiers cités.

**Projet imposé :** Le Grand Cinéma — 10 salles, 1 700 places, billetterie dématérialisée.
**Organisation GitHub :** `Grand-Cinema-Studi` (type : Organization)
**Project :** `Grand Cinema Backlog` — n° 3 — https://github.com/orgs/Grand-Cinema-Studi/projects/3
**Dépôts :** `grand-cinema-docs` · `grand-cinema-backend` · `grand-cinema-frontend`

---

## Jalons

| Échéance | Attendu | État |
| :--- | :--- | :--- |
| Mardi 20 octobre 2026 | 1er dépôt Blocs 1, 2 et 3 (retours jurys) | En cours |
| Vendredi 20 novembre 2026 | Dépôt final obligatoire + vidéos | À venir |
| Mardi 24 novembre 2026 | Vidéos de réponse aux formateurs | À venir |

---

## État d'avancement global

| Bloc | Dossier | Question | État |
| :--- | :--- | :--- | :--- |
| **1** | 1 — La demande du client | Q1 — Demande, objectifs, utilisateurs, besoins | ✅ Rédigée et intégrée au .docx |
| **1** | 1 — La demande du client | Q2 — Périmètre MVP et justifications | ✅ Rédigée et intégrée au .docx |
| **1** | 2 — Les solutions | Q1 — Lister les solutions | ⬜ À faire |
| **1** | 2 — Les solutions | Q2 — Utilisables ou non, justifier | ⬜ À faire |
| **1** | 2 — Les solutions | Q3 — Coût de mise en place | ⬜ À faire |
| **1** | 3 — L'architecture | Q1 — Justifier le choix d'hébergement | ⬜ À faire |
| **1** | 3 — L'architecture | Q2 — Schéma d'architecture logicielle | ⬜ À faire |
| **1** | 3 — L'architecture | Q3 — Justifier le choix d'architecture | ⬜ À faire |
| **2** | — | Backlog, plan de recette, tests | ⬜ À faire |
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
décomposition en tâches techniques, un script PowerShell d'automatisation, les vues du Project et la
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

1. **Retirer le champ d'issue `Priority`** du Project 3, puis le passer en **public**.
2. **Vérifier en navigation privée** : Project 3 et les 3 dépôts accessibles sans compte.
3. **Renseigner les champs** sur les 12 US : Story Points (97 au total), Priorite MoSCoW, Version.
4. **Nettoyer le project personnel n° 4 (GLOOMQUEST)** : retirer les 16 issues et les 4 champs ajoutés par erreur.
5. **Traiter le Dossier 2** : solutions techniques, faisabilité, coûts.
6. **Traiter le Dossier 3** : hébergement, schéma d'architecture, justification.

---

## Fichiers du projet

| Fichier | Rôle |
| :--- | :--- |
| `app_creation_from_scratch.md` | Ce journal de bord |
| `tuto_github_epics.md` | Tutoriel backlog agile sur GitHub (multi-repo) |
| `deadlines.md` · `planning_global.md` · `calendrier_blocs.md` | Rétroplanning |
| `Enoncé_BDA_Bloc 1_…pdf` | Énoncé officiel du sujet imposé |
| `Bloc 1/Bloc 1_…digitale.docx` | **Livrable officiel à déposer** |
| `Bloc 1/exam_bloc_1_answer.md` | Brouillon de travail des réponses du Bloc 1 |
| `Bloc 1/planning_bloc1_fasttrack.md` | Planning resserré du Bloc 1 |

---

<!--
MODÈLE POUR LA PROCHAINE ENTRÉE — copier-coller sous la dernière date

## JJ mois 2026

### <Bloc / Dossier / Sujet>
<Ce qui a été produit, en 3 à 5 lignes. Où ça vit. État : terminé / en cours / à reprendre.>

-->
