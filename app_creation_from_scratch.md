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
| **1** | 2 — Les solutions | Q3 — Coût de mise en place | ✅ Rédigée, familles A, B et C chiffrées sur une base commune (coût total sur 3 ans) ; ⚠️ prix de la licence du plugin billetterie (Famille B) non confirmé |
| **1** | 3 — L'architecture | Q1 — Justifier le choix d'hébergement | ✅ Rédigée (AWS, région Paris, 4 critères) |
| **1** | 3 — L'architecture | Q2 — Schéma d'architecture logicielle | 🟨 Nouveau schéma généré (`Bloc 1/schema_architecture_grand_cinema.png` : thème clair, module Authentification, flèche Stripe depuis la couche applicative, verrou anti-surbooking) ; à insérer à la place de l'ancienne image dans le .docx, ou à refaire dans un outil dédié |
| **1** | 3 — L'architecture | Q3 — Justifier le choix d'architecture | ✅ Rédigée (modularité, TMA, conduite du changement) |
| **2** | 1 — Les outils | Q1 — Agile vs approches classiques | ✅ Rédigée dans le .docx Bloc 2 |
| **2** | 1 — Les outils | Q2 — User Stories par EPIC | ✅ Rédigée (5 EPICs, 14 US dont 11 MVP et 3 V2), notes de travail supprimées ; Project GitHub aligné le 9 octobre ; ⚠️ renommer l'US 2.3 dans la copie (« Réserver un nombre de places et passer au paiement ») |
| **2** | 1 — Les outils | Q3 — Tickets DoR et DoD | ⬜ À faire (réponse vide) |
| **2** | 2 — Tests et mise en production | Q1 à Q3 — Stratégie de tests, critères de mise en prod, plan B | ⬜ À faire (réponses vides) |
| **2** | 3 — Gestion des risques | Q1 — Trois approches, choix justifié | ⬜ À faire (réponse vide) |
| **3** | — | Développement, API, BDD, sécurité | ⬜ À faire |

---

## 18 septembre 2026

### Bloc 1 — Dossier 1, Questions 1 et 2 → intégrées au livrable Word

Les réponses rédigées dans `Bloc 1/exam_bloc_1_answer.md` ont été insérées dans le document officiel
`Bloc 1/Bloc 1_Concevoir et spécifier une solution digitale.docx`, sous les intitulés de questions existants.

Contenu de la **Question 1** : diagnostic de l'existant (parc de 10 salles, 1 700 places, files de 30 min),
démarcation objectifs économiques / enjeux métiers (compétence C1.1), 4 personas d'ingénierie
(Marc Dupont, Elena Rostova, Thomas Martin, Sophie Laurent), backlog initial en 4 EPICs et 12 User Stories (le .docx actuel est passé à 5 EPICs et 11 stories).

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

Backlog créé dans `grand-cinema-docs` (version initiale : **4 EPICs et 12 User Stories**, depuis remplacée par 5 EPICs et 14 stories : voir « Cohérence Bloc 1 ↔ Bloc 2 »), avec critères d'acceptation
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


### Cohérence Bloc 1 ↔ Bloc 2 (8 octobre, lecture seule)

Les deux .docx concordent : mêmes 5 EPICs, mêmes 5 profils, mêmes enjeux et même architecture AWS.
Le Bloc 1 compte **11 stories** ; le Bloc 2 en compte **14** (11 MVP, 3 V2) après découpage :
EPIC 2 passe de 2 à 4 (ajout de la story de compte 2.1 et du plan de salle V2 2.4, scission de « voir les places » 2.2 et « sélectionner et payer » 2.3), EPIC 5 de 4 à 5 (« suivre » 5.3 séparé de « analyser » 5.4).
L'écart est expliqué dans le Bloc 2.

Corrections faites le 8 octobre (vérifiées dans les .docx) :

- ✅ **Bloc 2** : notes de travail supprimées (« Points à valider avant de l'intégrer », « Point de vigilance pour la soutenance »).
- ✅ **Bloc 1, EPIC 2** : règle de gestion du blocage des places pendant le paiement (délai paramétrable, 10 minutes retenues) ajoutée ; story de compte et justification de l'authentification ajoutées.
- ✅ **Bloc 1, D2 Q3** : hypothèses de chiffrage (1 000 billets/jour, 50 % en ligne, billet à 10 €), tableau du coût total sur 3 ans pour A, B et C, paragraphe d'arbitrage face à B, grille Weezevent à jour (2,5 %, minimum 0,99 € TTC, août 2026). Résultat : la C se rentabilise contre A en 4,5 à 7 mois, mais coûte 35 à 55 k€ de plus que B sur 3 ans (justifié par la maîtrise technique, pas par le prix).
- ✅ **Schéma D3 Q2** : nouvelle version générée (voir tableau d'avancement).

Reste à faire :

- **Bloc 1** : insérer le nouveau schéma, nettoyer les paragraphes vides (une trentaine dans D2 Q3), retirer le préfixe « Point de vigilance à noter : » du dernier paragraphe de D2 Q3, décocher « Incorporer les polices » (fichier de 10,9 Mo).
- **Limite assumée** : le prix de la licence du plugin billetterie de la Famille B n'est pas confirmé (fourchette de 600 à 1 200 €/an couvrant hébergement et licences).

### Project GitHub aligné sur les copies (8-9 octobre)

Le Project n° 3 contenait encore le backlog initial (4 EPICs, 12 US, 97 points), en contradiction avec les copies : achat sans compte (US 2.3), tableau de bord en MVP (US 4.3), stories absentes des copies (jauge modulable, scan en moins de 2 s, chargement en moins d'1 s).

Méthode retenue : **conserver le Project et les numéros d'issues**, plutôt que tout supprimer et recréer (le lien du Project est cité dans les copies, et supprimer des issues ne remet pas les numéros à zéro).

- **13 issues modifiées** : EPICs #1 à #3, #4 devenu EPIC 5, US #5, #6, #8 à #12, #14, #16 ; #10 et #12 ont changé de parent.
- **3 issues fermées en « Not planned »** et retirées du Project : #7 (chargement < 1 s), #13 (anti-doublon, fusionné dans l'US 4.1), #15 (jauge modulable).
- **6 issues créées** : EPIC 4 (#17) et US 2.1, 2.4, 5.1, 5.3, 5.5 (#18 à #22).
- **Labels** conformes à la copie du Bloc 2 : `EPIC 1` à `EPIC 5`, `mvp` / `v2`, `profil: …`. Colonnes du Board : Backlog, Prêt, En cours, En test / revue, Terminé.
- **US 2.3 renommée** « Réserver un nombre de places et passer au paiement » : elle réserve une quantité de places ; le choix du siège reste en V2 (US 2.4).
- **Tâches techniques de l'US 2.3** (backend#1, backend#2, frontend#1) conservées et alignées sur le schéma d'architecture : blocage en base PostgreSQL, sans Redis.
- **Nouveau chiffrage : 89 points**, dont 63 MVP et 26 V2. Must have = 57 points (64 % du backlog). Vélocité de 10 à 13 points par sprint, soit 5 à 6 sprints de 2 semaines pour le MVP, cohérent avec les 685 à 935 heures du Bloc 1.

Vérifié sur GitHub le 9 octobre : 19 issues ouvertes (5 EPICs + 14 US), US 2.3 avec ses 3 tâches, EPIC 2 à `0 / 4`.

Piège rencontré : variables Git Bash vides (`$REPO_DOCS`, `$PROJECT`) après réouverture du terminal ; `gh --repo ""` se rabat alors sur le dépôt du dossier courant sans prévenir. Parade : `source ~/gc-env.sh` en début de session, puis `echo "$REPO_DOCS"`.

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
| Chiffrage du backlog | 89 story points (63 MVP, 26 V2), 5 à 6 sprints de 2 semaines pour le MVP | Vélocité de 10 à 13 points, 2 développeurs ; cohérent avec les 685 à 935 h du Bloc 1 (remplace 97 points / vélocité 25) |
| Mise en conformité du backlog GitHub | Modifier, fermer ou créer les issues ; garder le Project n° 3 | Lien cité dans les copies, tâches conservées, affinage visible dans l'historique |
| Blocage des places | En base PostgreSQL, dans une transaction, expiration à 10 minutes | Cohérent avec le schéma d'architecture (pas de Redis) |
| Titre de l'US 2.3 | « Réserver un nombre de places et passer au paiement » | Éviter la confusion avec le choix du siège (US 2.4, V2) |

---

## Prochaines actions

Terminé côté GitHub (confirmé les 8 et 9 octobre) :

1. ✅ Champ d'issue `Priority` retiré, Project 3 passé en **public**.
2. ✅ Accès vérifié en navigation privée (Project 3 et 3 dépôts).
3. ✅ Champs renseignés : Story Points, Priorite MoSCoW, Version (refaits le 9 octobre sur les 14 US).
4. ✅ Project personnel n° 4 (GLOOMQUEST) nettoyé.
5. ✅ Project aligné sur les copies (5 EPICs, 14 US, 89 points), vues créées, quelques tâches avancées.

Livrables, par priorité :

1. **Copie Bloc 2** : renommer l'US 2.3 (titre et description) pour qu'elle corresponde au Project. Corriger sur GitHub l'affichage du gras « En tant qu' » / « afin d' » dans 8 US (commandes `sed` + `gh issue edit`).
2. **Bloc 2** : Dossier 1 Q1 et Q2 sont rédigées. Reste : DoR/DoD (D1 Q3), stratégie de tests, critères de mise en production et plan B (D2), gestion des risques (D3).
3. **Bloc 3** : squelette backend/frontend fonctionnel (dépôts encore vides).
4. **Bloc 1** : insérer le nouveau schéma, nettoyer les paragraphes vides, retirer les polices incorporées, puis geler le document.
5. **Vidéos** (20 et 24 novembre) : trames de script, à planifier après le premier dépôt.

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
| `Réponses Bloc 1 2 & 3/Bloc 2/` | **Livrable Bloc 2** (.docx, en cours ; la copie .odt de 35 Mo ne sert pas au dépôt) |
| `TEST_CODES/typescript/` | Essais TypeScript |

---

<!--
MODÈLE POUR LA PROCHAINE ENTRÉE — copier-coller sous la dernière date

## JJ mois 2026

### <Bloc / Dossier / Sujet>
<Ce qui a été produit, en 3 à 5 lignes. Où ça vit. État : terminé / en cours / à reprendre.>

-->
