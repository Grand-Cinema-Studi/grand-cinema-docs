# grand-cinema-docs

Documentation transverse du projet **« Le Grand Cinéma »**, plateforme web de réservation de billets réalisée dans le cadre du titre professionnel **Concepteur Développeur d'Applications (CDA, niveau 6)** — Bachelor Développeur d'Application, Studi.

Ce dépôt réunit ce qui concerne **l'application entière**, front et back ensemble : l'architecture, les décisions techniques et leur justification, les règles de développement, la qualité, la sécurité et le déploiement.

---

## 🎬 Contexte du projet

« Le Grand Cinéma » est un cinéma historique (50+ ans, **10 salles modulables, 1 700 places**) dont le fonctionnement est encore entièrement physique. Lors des pics d'affluence — notamment pendant le **Festival de Jazz** — les files d'attente dépassent 30 minutes, des spectateurs abandonnent leur achat et le remplissage des salles est mal optimisé.

**Objectif business** : +30 % de fréquentation sur 12 mois grâce à une plateforme de réservation en ligne.

**Profils utilisateurs identifiés** : Spectateur régulier · Festivalier / Touriste · Caissier · Contrôleur · Responsable Programmation / Direction.

## 🗂️ Contenu

```
grand-cinema-docs/
├── README.md
├── architecture/              🏗️ architecture
│   ├── overview.md            vue d'ensemble : composants, couches du backend, hébergement
│   ├── glossary.md            vocabulaire métier (français) ↔ noms du code (anglais)
│   └── diagrams/              index des schémas et exports en image
├── adr/                       🧭 décisions techniques, une par fichier, numérotées
├── development/               🛠️ développer
│   ├── dev-setup.md           préparer un poste et lancer l'application en local
│   ├── git-workflow.md        branches, protections, Pull Requests, messages de commit
│   ├── coding-standards.md    structure, nommage, écriture du code, revue
│   └── versioning.md          ce qui est versionné, numéros de version, tags
├── api/                       🔌 contrat front ↔ back
│   └── api-conventions.md     routes, format des données, codes HTTP, erreurs, Swagger
├── quality/                   ✅ qualité et tests
│   ├── testing-strategy.md    niveaux de tests, TDD, couverture, anomalies
│   ├── code-quality.md        SonarCloud et quality gate
│   └── bugs/                  analyses d'anomalies : identification, reproduction, correctif, validation
├── security/                  🛡️ sécurité
│   ├── security.md            authentification, rôles, secrets, paiement, Top 10 de l'OWASP
│   └── gdpr.md                données personnelles, conservation, droits (RGPD)
└── deployment/                🚀 mise en ligne
    ├── environments.md        environnements, architecture de production, secrets, CI/CD
    └── deploy-guide.md        déployer, vérifier, revenir en arrière, sauvegarder
```

Les éléments marqués _(à venir)_ dans les documents sont décidés mais pas encore réalisés.

### 🏗️ Architecture et décisions

| Document | Pour répondre à |
|---|---|
| [Vue d'ensemble de l'architecture](architecture/overview.md) | Comment l'application est-elle découpée, et où tourne-t-elle ? |
| [Glossaire](architecture/glossary.md) | Quel nom porte, dans le code, chaque notion métier ? |
| [Schémas](architecture/diagrams/README.md) | Où trouver chaque schéma, et comment l'exporter ? |
| [Décisions d'architecture (ADR)](adr/README.md) | Pourquoi ce choix plutôt qu'un autre ? |

### 🛠️ Développement

| Document | Pour répondre à |
|---|---|
| [Poste de développement](development/dev-setup.md) | Comment lancer toute l'application sur mon poste ? |
| [Workflow Git](development/git-workflow.md) | Quelle branche créer, comment l'intégrer, comment écrire mes commits ? |
| [Standards de code](development/coding-standards.md) | Où ranger mon code, comment le nommer, que vérifie la revue ? |
| [Versioning](development/versioning.md) | Qu'est-ce qui est versionné, et comment une version est-elle numérotée ? |

### 🔌 API

| Document | Pour répondre à |
|---|---|
| [Conventions de l'API](api/api-conventions.md) | Quelle forme ont les routes, les données et les erreurs échangées entre le front et le back ? |

### ✅ Qualité

| Document | Pour répondre à |
|---|---|
| [Stratégie de tests](quality/testing-strategy.md) | Qu'est-ce qui est testé, à quel niveau, et comment le prouver ? |
| [Qualité du code](quality/code-quality.md) | Quels contrôles automatiques, et quels seuils bloquent une fusion ? |
| [Bug 0001 – Double réservation](quality/bugs/0001-double-booking.md) | Comment la même place pourrait-elle être vendue deux fois, et comment l'empêcher ? |

### 🛡️ Sécurité

| Document | Pour répondre à |
|---|---|
| [Sécurité](security/security.md) | Qui peut faire quoi, et comment l'application est-elle protégée ? |
| [Données personnelles (RGPD)](security/gdpr.md) | Quelles données sont traitées, pourquoi, et combien de temps ? |

### 🚀 Déploiement

| Document | Pour répondre à |
|---|---|
| [Environnements](deployment/environments.md) | Où tourne l'application, et comment est-elle configurée ? |
| [Guide de déploiement](deployment/deploy-guide.md) | Comment mettre en ligne une version, revenir en arrière, restaurer la base ? |

## 📍 Où se trouve quelle documentation

| Documentation | Emplacement | Raison |
|---|---|---|
| Architecture globale, décisions techniques (ADR), règles communes (Git, code, tests, sécurité), déploiement | **ce dépôt** | elles concernent le front et le back ensemble, et se lisent au même endroit |
| Installation, lancement, tests d'un composant | `README.md` de son dépôt | elles changent avec le code, dans la même Pull Request |
| Schéma de la base de données, script SQL | `grand-cinema-backend/docs/database` | générés depuis les migrations : ils doivent rester à côté d'elles |
| Liste des endpoints et de leurs schémas | Swagger / OpenAPI, généré par le backend | toujours fidèle au code ; les règles communes sont dans [api-conventions.md](api/api-conventions.md) |
| Convention de nommage des branches | `CONTRIBUTING.md` de chaque dépôt de code | documentée dans le dépôt lui-même, lue au moment de contribuer ; le détail est dans [git-workflow.md](development/git-workflow.md) |

Règle : une Pull Request du front ou du back qui change l'architecture ou prend une décision technique s'accompagne d'une Pull Request dans ce dépôt (nouvel ADR ou document mis à jour), citée dans sa description.

## 🔗 Liens

| Ressource | Lien |
|---|---|
| Organisation GitHub | https://github.com/Grand-Cinema-Studi |
| Dépôt backend (ASP.NET Core, PostgreSQL) | https://github.com/Grand-Cinema-Studi/grand-cinema-backend |
| Dépôt frontend (Angular) | https://github.com/Grand-Cinema-Studi/grand-cinema-frontend |
| Suivi du projet (Project GitHub n° 3) | https://github.com/orgs/Grand-Cinema-Studi/projects/3 |
| Application déployée | _À venir_ |

## ✍️ Contribuer

Même fonctionnement que les dépôts de code ([workflow Git](development/git-workflow.md)) : une branche `docs/<sujet>` par modification, une Pull Request vers `main`, des messages de commit au format [Conventional Commits](https://www.conventionalcommits.org/fr/) (`docs(adr): …`, `docs(architecture): …`).

Les documents sont rédigés en français ; les noms de fichiers sont en anglais, comme dans le code.

## ✍️ Auteur

**Slimane Smahi** — Bachelor Développeur d'Application, Studi
