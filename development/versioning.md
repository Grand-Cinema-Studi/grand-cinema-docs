# Versioning

Ce qui est versionné dans le projet, et comment une version est identifiée.

Les éléments marqués _(à venir)_ sont décidés mais pas encore réalisés.

## Ce qui est versionné, et où

| Élément | Où | Pourquoi |
|---|---|---|
| Code source, tests | Git, un dépôt par composant | historique complet, revue par Pull Request |
| Version du SDK .NET | `global.json` du backend | même compilation sur tous les postes et en intégration continue |
| Outils .NET (`dotnet-ef`) | `dotnet-tools.json` du backend | même version d'outil partout, restaurée par `dotnet tool restore` |
| Versions des bibliothèques | fichiers `.csproj` (backend), `package-lock.json` (frontend, _à venir_) | versions exactes, pas de mise à jour silencieuse |
| Schéma de la base | migrations EF Core, et `schema.sql` généré depuis elles | chaque évolution du schéma est datée, relue et rejouable sur toute base |
| Version de PostgreSQL | image `postgres:16` dans Docker Compose | même moteur en local, en test et en production |
| Images Docker de l'application | GitHub Container Registry, une image par commit _(à venir)_ | ce qui tourne en production est identifié exactement, et l'on peut revenir à une image précédente |
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

## Tags et notes de version _(à venir)_

À chaque fusion de `dev` dans `main` :

1. un tag Git annoté `v<version>` est posé sur le commit de fusion :

   ```bash
   git switch main && git pull
   git tag -a v0.2.0 -m "v0.2.0"
   git push origin v0.2.0
   ```

2. une release GitHub est créée depuis ce tag, avec la liste des changements produite à partir des messages de commit et des Pull Requests (`gh release create v0.2.0 --generate-notes`).

Le backend et le frontend ont chacun leur numéro de version : ils sont livrés indépendamment, tant que le contrat de l'API est respecté.

## Version de l'API

- Les routes ne portent pas de numéro de version (`/api/schedule`) : l'API n'a qu'un client, le front du projet, livré en même temps qu'elle.
- Le document OpenAPI est publié sous le nom `v1` (`/openapi/v1.json`).
- Si un changement incompatible devenait nécessaire alors que d'autres clients existent (application mobile, partenaires), la nouvelle version serait exposée sous `/api/v2/...`, l'ancienne étant maintenue le temps de la migration.
