# Versioning

Ce qui est versionné dans le projet, et comment une version est identifiée.

Les éléments marqués _(à venir)_ sont décidés mais pas encore réalisés.

## Ce qui est versionné, et où

| Élément | Où | Pourquoi |
|---|---|---|
| Code source, tests | Git, un dépôt par composant | historique complet, revue par Pull Request |
| Version du SDK .NET | `global.json` du backend | même compilation sur tous les postes et en intégration continue |
| Outils .NET (`dotnet-ef`) | `dotnet-tools.json` du backend | même version d'outil partout, restaurée par `dotnet tool restore` |
| Versions des bibliothèques | fichiers `.csproj` (backend), `package-lock.json` (frontend) | versions exactes, pas de mise à jour silencieuse |
| Schéma de la base | migrations EF Core, et `schema.sql` généré depuis elles | chaque évolution du schéma est datée, relue et rejouable sur toute base |
| Version de PostgreSQL | image `postgres:16` dans Docker Compose | même moteur en local, en test et en production |
| Images Docker de l'application | GitHub Container Registry (`ghcr.io/grand-cinema-studi/grand-cinema-backend:<commit>`), une image par commit fusionné dans `dev` | ce qui tourne en production est identifié exactement, et l'on peut revenir à une image précédente |
| Décisions techniques | ADR numérotés, jamais modifiés une fois acceptés | l'historique des décisions reste lisible |

## Numéro de version de l'application

[Versionnage sémantique](https://semver.org/lang/fr/) : `MAJEUR.MINEUR.CORRECTIF`.

| Partie | Augmente quand | Exemple |
|---|---|---|
| MAJEUR | un changement casse le contrat de l'API (champ supprimé ou renommé, route modifiée) | `2.0.0` |
| MINEUR | une fonctionnalité est ajoutée sans casser l'existant (une user story livrée) | `1.1.0` |
| CORRECTIF | une anomalie est corrigée | `1.1.1` |

- Tant que le MVP n'est pas complet, les versions sont en `0.x` : le contrat de l'API peut encore évoluer.
- `1.0.0` correspond au MVP complet : le parcours consultation → réservation → paiement → billet → contrôle.
- Le type des commits ([Conventional Commits](git-workflow.md#messages-de-commit)) indique la partie à augmenter : `fix` → correctif, `feat` → mineur, `!` ou `BREAKING CHANGE` → majeur.

## Tags et notes de version

Aucun tag n'a été posé pendant le développement du MVP. Le premier est **`v1.0.0`**, posé le 11/11/2026 sur les versions gelées le 10/11/2026 (backend `6dae1da`, frontend `6f25625`) : le MVP complet, accepté après la recette consignée dans le [procès-verbal de recette](../quality/acceptance-report.md) (52 critères conformes). Le dépôt de documentation porte le même tag, sur l'état de la documentation livrée avec cette version. Ensuite, chaque fusion de `dev` dans `main` est une version, livrée selon la procédure ci-dessous.

## Procédure de release

Elle se fait dans le backend, puis dans le frontend, puis dans le dépôt de documentation. Exemple avec `v1.0.0`.

**Conditions préalables**

- Le code est gelé : plus aucune fusion dans `dev` depuis la recette.
- La recette est faite sur les commits de `dev` à livrer, et le Product Owner a accepté la version dans le [procès-verbal de recette](../quality/acceptance-report.md).
- Les commits de `dev` sont ceux du procès-verbal :

  ```bash
  git fetch && git rev-parse --short origin/dev
  ```

**1. Pull Request `dev` → `main`** (backend, puis frontend)

- GitHub : **Pull requests**, **New pull request**, base `main`, compare `dev`. Titre `release: v1.0.0 (MVP)` ; la description résume la version (user stories, recette, tests, SonarCloud).
- Contrôles obligatoires (ruleset `protect-main-and-dev`) : l'intégration continue, SonarCloud, et `main-source-branch`, qui n'accepte vers `main` que `dev` ou une branche `hotfix/…` du même dépôt.
- Fusion par **commit de fusion** (« Create a merge commit »), jamais par « Squash » : `main` garde l'historique des Pull Requests des user stories.
- La fusion dans `main` ne déploie rien : la recette se déploie depuis `dev` ([environnements](../deployment/environments.md)) ; au moment de la release, elle tourne déjà exactement sur ces commits.

**2. Tag annoté et release GitHub**

```bash
git switch main && git pull
git log -1 --oneline                 # Merge pull request #… from …/dev
git tag -a v1.0.0 -m "v1.0.0 – MVP"
git push origin v1.0.0
gh release create v1.0.0 --title "v1.0.0 – MVP" --generate-notes --verify-tag
git switch dev
```

- Le tag est **annoté** (`-a`) : il porte un auteur, une date et un message, contrairement à un tag léger.
- `--generate-notes` liste les Pull Requests fusionnées depuis la release précédente (pour la première, depuis le début du projet) ; `--verify-tag` refuse de créer la release si le tag n'a pas été poussé.
- Sans l'outil `gh` : **Releases**, **Draft a new release**, choisir le tag, **Generate release notes**, **Publish release**.

**3. Dépôt de documentation**

Il travaille directement sur `main` : la documentation de la version y est commitée, puis le même tag est posé (`v1.0.0`, message « documentation of the MVP ») et une release est créée de la même façon.

**4. Vérifications**

- Chaque dépôt affiche la release, marquée **Latest**.
- Sur le serveur, l'image de l'API en service porte toujours le commit livré : `sudo docker ps --format '{{.Names}}  {{.Image}}'`.
- Test de fumée sur la recette : accueil, connexion des quatre comptes, Swagger, une affiche servie par `/posters/`.

**Correctif après une release**

Une anomalie bloquante trouvée après la release est corrigée sur une branche `hotfix/…` partie de `main`, fusionnée par Pull Request dans `main`, puis reportée dans `dev` par une Pull Request `main` → `dev` ([workflow Git](git-workflow.md)), et livrée sous le numéro de correctif suivant (`v1.0.1`), avec la même procédure.

Le backend et le frontend ont chacun leur numéro de version : ils sont livrés indépendamment, tant que le contrat de l'API est respecté.

## Mises à jour des dépendances

Dependabot propose chaque semaine les mises à jour des dépendances : paquets NuGet, actions GitHub et images Docker pour le backend ; paquets npm et actions GitHub pour le frontend. Ses Pull Requests visent `dev` (`target-branch: dev`), jamais `main`.

**Quand il s'est mis en marche.** Dependabot lit sa configuration (`.github/dependabot.yml`) sur la branche par défaut du dépôt, `main`. Jusqu'à la première release, `main` ne contenait que le commit initial : la configuration, présente sur `dev` depuis le 20/10/2026 (frontend) et le 25/10/2026 (backend), n'était pas lue, et aucune mise à jour n'a été proposée pendant le développement. Dependabot a démarré à la fusion de `dev` dans `main` de la release v1.0.0 (backend : Pull Request #80, 11/11/2026).

**Ses premières propositions, après le gel (backend).** Arrivées après le gel du code du 10/11/2026, elles ne sont pas intégrées à la v1.0.0 :

| Pull Request | Mise à jour | Décision |
|---|---|---|
| #81 | image `dotnet/aspnet` 10.0 → 11.0 | version majeure : décision à part (ADR, essais), pas de fusion automatique |
| #82 | image `dotnet/sdk` 10.0 → 11.0 | même décision que #81 |
| #83 | groupe `github-actions`, 4 mises à jour | cycle de maintenance, version `v1.0.1` |
| #84 | groupe `nuget-minor-and-patch`, 12 mises à jour (versions mineures et correctifs) | cycle de maintenance, version `v1.0.1` |

Leur intégration continue échoue au démarrage (30 à 40 secondes) ; la cause reste à analyser. Le fichier n'écarte les versions majeures que pour NuGet, d'où #81 et #82 : la même règle est à ajouter pour les images Docker.

**Ce qui bloque une livraison, et ce qui ne la bloque pas**

| Où l'intégration continue est rouge | Conséquence |
|---|---|
| sur `main` ou sur `dev` : du code livré ou livrable | **bloquant** : on ne livre pas tant qu'elle n'est pas verte |
| sur une Pull Request non fusionnée, de Dependabot ou d'une branche de travail | **non bloquant** : le code livré n'a pas changé ; la proposition attend d'être corrigée ou refusée |

**En exploitation**, ces propositions sont traitées régulièrement (chaque semaine) : c'est de la maintenance, pas un prérequis à la mise en production.

- Correctifs et versions mineures : intégration continue réparée si besoin, fusion dans `dev`, recette, puis version de correctif (`v1.0.1`) selon la procédure de release.
- Version majeure (.NET 11, nouvelle version majeure d'Angular) : un projet à part, avec son ADR, planifié et testé.
- Faille de sécurité signalée sur une dépendance utilisée : traitée en priorité, par une branche `hotfix/…` si la version livrée est exposée.

## Version de l'API

- Les routes ne portent pas de numéro de version (`/api/schedule`) : l'API n'a qu'un client, le front du projet, livré en même temps qu'elle.
- Le document OpenAPI est publié sous le nom `v1` (`/openapi/v1.json`).
- Si un changement incompatible devenait nécessaire alors que d'autres clients existent (application mobile, partenaires), la nouvelle version serait exposée sous `/api/v2/...`, l'ancienne étant maintenue le temps de la migration.
