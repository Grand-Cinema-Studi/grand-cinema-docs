# Environnements et déploiement

Les éléments marqués _(à venir)_ sont décidés mais pas encore réalisés.

Ce document applique l'architecture du Bloc 1 et la stratégie des Blocs 2 (environnements, chaîne d'intégration, critères de mise en production). Ce qui n'est pas monté pour la mise en ligne du projet est écrit comme tel, avec la raison et le chemin vers la cible.

## Architecture cible (Bloc 1)

```text
Navigateur ── HTTPS ──► CloudFront (CDN)
                          ├─ fichiers du front ──────────► S3 (hébergement statique)  : Front-end (Angular)
                          └─ /api/* ──► répartiteur de charge ──► EC2 + Auto Scaling   : API Back-end
                                                                    ├─► RDS PostgreSQL
                                                                    ├─► S3 (billets et QR codes)
                                                                    └─► Stripe (paiement, PCI-DSS porté par Stripe)

CloudWatch   supervision : métriques, journaux, alertes
```

| Composant | Rôle | Dépôt / déploiement |
|---|---|---|
| Front-end (Angular) | application web (SPA) : programmation, compte, réservation, billets, écrans du personnel | dépôt `grand-cinema-frontend`, son propre pipeline : compilation, publication sur S3, invalidation du cache CloudFront |
| API Back-end (ASP.NET Core) | règles métier, sécurité, accès aux données | dépôt `grand-cinema-backend`, son propre pipeline : tests, image Docker, déploiement sur EC2 |
| PostgreSQL | utilisateurs, films, salles, places, séances, réservations | RDS en production |
| S3 (usage séparé du front) | billets dématérialisés et QR codes | US 3.2 _(à venir)_ |
| Stripe | paiement par carte ; aucune donnée de carte ne transite par l'application | US 3.1 _(à venir)_ |
| CloudWatch | supervision et alertes | voir « Supervision » |

Le front et l'API sont deux applications séparées, qui ne communiquent que par HTTP et JSON : un changement d'interface ne touche pas l'API, et inversement.

## Environnements (Bloc 2)

| Environnement | Rôle | Base de données | Paiement | État |
|---|---|---|---|---|
| Développement (poste) | écrire le code et les tests unitaires | PostgreSQL 16 en Docker Compose, jeux de données générés | Stripe simulé (doublure) | ✅ en place |
| Test (intégration continue) | exécuter les tests automatisés à chaque Pull Request | PostgreSQL jetable (Testcontainers), recréé à chaque exécution avec les mêmes jeux de données | Stripe simulé, puis mode test | ✅ tests d'intégration en local ; CI _(à venir)_ |
| Recette | tests de bout en bout, recette métier au fil du sprint | données réalistes (salles de 70 à 300 places, séances presque complètes, période de festival) | Stripe en mode test | **environnement de démonstration du projet** _(à venir)_ |
| Préproduction | valider la version complète avant la mise en production ; identique à la production, allumée seulement pour une release | données anonymisées, au volume de la production | Stripe en mode test | décrite, non montée |
| Production | service réel | données réelles | Stripe réel | décrite, non montée |

Aucune donnée réelle de client n'est utilisée en dehors de la production (RGPD).

## Mise en ligne retenue pour le projet : la recette

La démonstration du projet est l'environnement de **recette**. C'est une **version réduite** de l'architecture cible : le front est déployé comme dans la cible, l'API et la base tiennent sur un seul serveur.

```text
Navigateur ── HTTPS ──► CloudFront (adresse *.cloudfront.net, sans nom de domaine)
                          ├─ fichiers du front ──► S3 (privé, accès réservé à CloudFront)
                          └─ /api/*, /health, /swagger, /openapi/* ──► EC2, HTTP 80

EC2 t3.micro (Amazon Linux 2023 + Docker Compose), région Paris (eu-west-3)
   api      image du backend (GitHub Container Registry)
   migrate  applique les migrations, puis s'arrête
   db       PostgreSQL 16, volume Docker, aucun port publié

SSM Parameter Store   secrets et réglages, chiffrés
SSM Session Manager   accès au serveur sans SSH ni port d'administration ouvert
```

| | Cible (Bloc 1) | Recette du projet | Pourquoi l'écart |
|---|---|---|---|
| Front | S3 + CloudFront | S3 + CloudFront | aucun écart |
| API | EC2 avec Auto Scaling derrière un répartiteur de charge | un seul serveur EC2 | budget ; l'API est sans état (jeton JWT, blocage des places en base), donc on ajoute des instances sans changer le code |
| Base | RDS PostgreSQL (sauvegardes et mises à jour gérées) | PostgreSQL en conteneur sur le serveur | budget ; seule la chaîne de connexion change pour passer à RDS |
| HTTPS | de bout en bout | entre le navigateur et CloudFront ; HTTP entre CloudFront et le serveur | pas de nom de domaine ni de répartiteur de charge ; le serveur n'accepte que les adresses de CloudFront (liste de préfixes gérée par AWS) |
| Supervision | CloudWatch | alarmes CloudWatch de base _(à venir)_ | voir « Supervision » |
| Billets | S3 | à venir avec l'US 3.2 | story pas encore développée |

- **Une seule adresse** pour le front et l'API : CloudFront transmet `/api/*` au serveur (cache désactivé, en-tête `Authorization` et toutes les méthodes HTTP transmis). Le navigateur appelle `/api/...` sur le site qu'il affiche, donc **pas de CORS** à configurer.
- **Page d'accueil et routage Angular** : CloudFront renvoie `index.html` quand un chemin n'existe pas dans S3, pour que les routes de l'application fonctionnent au rechargement.
- **Migrations** appliquées par le déploiement, avant de démarrer la nouvelle version de l'API, jamais au démarrage de l'API.
- **Fuseau horaire** : `TZ=Europe/Paris` dans les conteneurs, pour que « aujourd'hui » soit le jour à Paris. Les séances sont enregistrées à l'heure du cinéma, les instants techniques (création, expiration du blocage) en UTC.
- **Passage à la cible** : l'image de l'API ne change pas ; on ajoute le répartiteur de charge et l'Auto Scaling, et la chaîne de connexion pointe vers RDS.
- **Repli** : si CloudFront n'était pas disponible avec le type de compte AWS utilisé, le front serait servi par un proxy sur le même serveur (HTTP, sans nom de domaine). C'est un écart à la cible, à dire explicitement.

Déployer, vérifier, revenir en arrière, sauvegarder : [guide de déploiement](deploy-guide.md). Les commandes de la première installation sont dans le journal du backend, [from_scratch.md](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/blob/dev/from_scratch.md).

## Secrets et configuration

| Environnement | Base de données | Chaîne de connexion (API) | Clés (jeton JWT, Stripe) |
|---|---|---|---|
| Développement | fichier `.env`, ignoré par Git | user-secrets .NET, hors du dépôt | user-secrets .NET ; Stripe simulé |
| Test (CI) | générés par Testcontainers | générée par Testcontainers | valeurs de test, jamais réutilisées ailleurs |
| Recette | SSM Parameter Store (chiffré) | variable d'environnement `ConnectionStrings__Default`, construite au déploiement | SSM Parameter Store ; **clé Stripe de test** |
| Production | SSM Parameter Store (chiffré) | idem | SSM Parameter Store ; **clé Stripe réelle**, contrôlée avant chaque mise en production |

Aucun secret n'est versionné, ni dans les dépôts, ni dans les images Docker, ni dans GitHub. Les clés de production sont stockées hors du code, et leur présence est un critère de mise en production (voir le guide).

Swagger est désactivé par défaut et s'active avec `Swagger__Enabled=true` (activé en développement et sur la recette pendant la correction).

## Intégration et déploiement continus _(à venir)_

Le front et l'API ont chacun leur dépôt et leur pipeline.

| Déclencheur | Étapes |
|---|---|
| Pull Request vers `dev` (branche d'intégration, `develop` du Git Flow) ou `main` | compilation, tests unitaires et d'intégration, mesure de la couverture (objectif d'au moins 80 % sur le code métier), analyse SonarCloud dont le Quality Gate doit être validé ; la fusion est bloquée si l'un de ces contrôles échoue ou si la revue de code n'est pas approuvée |
| Fusion dans `dev` | **déploiement automatique sur la recette** : API (image publiée dans GitHub Container Registry, serveur mis à jour par AWS Systems Manager), front (compilation, publication sur S3, invalidation du cache CloudFront) ; les tests de bout en bout tournent sur la recette |
| Avant une release | la version complète est déployée sur la préproduction (allumée pour l'occasion) : scan de sécurité (OWASP ZAP) et test de charge (JMeter) |
| Mise en production | **manuelle**, après validation du Product Owner (livraison continue), via une Pull Request `dev` → `main` ; elle est suivie d'un test de fumée |

L'accès de GitHub à AWS passe par OIDC : aucune clé AWS n'est stockée dans GitHub, et chaque rôle (API, front) ne peut faire que sa propre action.

## Supervision _(à venir)_

CloudWatch est la supervision de la cible. Pour la recette : alarmes sur l'état de l'instance et sur l'utilisation du processeur, alarme sur les erreurs 5xx de CloudFront. En production : alertes sur les erreurs, les temps de réponse et la saturation de la base (pas seulement des serveurs d'application), avec astreinte pendant le festival.

## Comptes de démonstration _(à venir)_

Un compte administrateur (`Manager`) et un compte spectateur (`Customer`) sont créés sur la recette ; leurs identifiants sont donnés dans les copies, pas dans les dépôts.
