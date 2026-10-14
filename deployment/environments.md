# Environnements et déploiement

Les éléments marqués _(à venir)_ sont décidés mais pas encore réalisés.

## Environnements

| Environnement | Où | Base de données | État |
|---|---|---|---|
| Développement | poste du développeur | PostgreSQL 16 en Docker Compose, données de démonstration | ✅ en place |
| Tests d'intégration | poste du développeur, puis GitHub Actions | PostgreSQL 16 jetable, démarré par Testcontainers pour chaque série de tests | ✅ en local ; CI _(à venir)_ |
| Production / démonstration | AWS, région Paris (`eu-west-3`) | PostgreSQL 16 en conteneur sur le serveur | _(à venir)_ |

Lancement en local : voir le `README.md` de chaque dépôt de code.

## Production : architecture retenue

Variante gratuite (compte AWS Free plan), toute l'application en conteneurs Docker sur un seul serveur :

```text
Internet ── HTTP 80 ──► EC2 t3.micro (Amazon Linux 2023 + Docker Compose)
                         proxy (Caddy, seul port ouvert : 80)
                           ├─ /api/*, /health, /swagger, /openapi/* ──► api   (image du backend)
                           └─ tout le reste                         ──► front (image du frontend Angular)
                         migrate (applique les migrations, puis s'arrête)
                         db (PostgreSQL 16, volume Docker, aucun port publié)

GitHub Container Registry   images du backend et du frontend
SSM Parameter Store         mot de passe PostgreSQL et réglages, chiffrés
SSM Session Manager         accès au serveur sans SSH ni port d'administration ouvert
```

- **Une seule adresse** pour le front et l'API : le navigateur appelle `/api/...` sur le site qu'il affiche, donc pas de configuration CORS.
- **Migrations** appliquées par le déploiement, avant de démarrer la nouvelle version de l'API, jamais au démarrage de l'API.
- **Fuseau horaire** : `TZ=Europe/Paris` dans les conteneurs, pour que « aujourd'hui » soit le jour à Paris.
- **Passage à l'architecture cible du Bloc 1** (RDS PostgreSQL géré, Auto Scaling) : l'image de l'API ne change pas, seule la chaîne de connexion change.

Déployer, vérifier, revenir en arrière, sauvegarder : [guide de déploiement](deploy-guide.md). Les commandes de la première installation (image Docker, serveur, scripts) sont dans le journal du backend, [from_scratch.md](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/blob/dev/from_scratch.md), étapes 9, 10 G et 11 G.

## Secrets et configuration

| Environnement | Base de données (Docker) | Chaîne de connexion (API) |
|---|---|---|
| Développement | fichier `.env`, ignoré par Git | user-secrets .NET, hors du dépôt |
| Tests d'intégration | générés par Testcontainers | générée par Testcontainers |
| Production | SSM Parameter Store (chiffré) | variable d'environnement `ConnectionStrings__Default`, construite au déploiement |

Aucun secret n'est versionné, ni dans les dépôts, ni dans les images Docker, ni dans GitHub.

Swagger est désactivé par défaut et s'active avec `Swagger__Enabled=true` (activé en développement, et sur l'environnement de démonstration pendant la correction).

## Intégration et déploiement continus _(à venir)_

| Déclencheur | Étapes |
|---|---|
| Pull Request vers `dev` ou `main` | compilation, tests unitaires et d'intégration, couverture, analyse SonarCloud |
| Fusion dans `main` | mêmes vérifications, construction et publication des images, déploiement sur le serveur |

## Comptes de démonstration _(à venir)_

Un compte administrateur (`Manager`) et un compte spectateur (`Customer`) seront créés sur l'environnement de démonstration ; leurs identifiants seront donnés dans les copies, pas dans les dépôts.
