# ADR 0003 – Hébergement : front sur S3 + CloudFront, API et base sur un serveur EC2 pour la recette

- Statut : Accepté
- Concerne : les deux
- Liens : [environnements](../deployment/environments.md), [guide de déploiement](../deployment/deploy-guide.md), journal du backend ([from_scratch.md](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/blob/dev/from_scratch.md), étapes 10 G et 11 G)

## Contexte

- Le Bloc 1 fixe l'architecture cible : AWS (`eu-west-3`), Front-end Angular en fichiers statiques sur S3 diffusés par CloudFront, API Back-end sur EC2 avec Auto Scaling, RDS PostgreSQL, S3 pour les billets, CloudWatch, Stripe.
- Le Bloc 2 fixe la livraison : un dépôt et un pipeline par application ; recette déployée automatiquement après fusion dans la branche d'intégration ; préproduction identique à la production ; mise en production manuelle après validation du Product Owner ; HTTPS actif comme critère bloquant.
- Le projet doit être en ligne pour la correction du Bloc 3, avec un budget nul (compte AWS Free plan).
- Une première version du journal du backend mettait tout sur un serveur (front en conteneur derrière un proxy Caddy, en HTTP) : écart avec les Blocs 1 et 2.

## Décision

1. L'environnement mis en ligne est la **recette** du Bloc 2. La préproduction et la production sont décrites, pas montées.
2. **Front-end** : fichiers Angular compilés dans un bucket S3 privé, diffusés par CloudFront (accès au bucket réservé à CloudFront), publiés par le pipeline du dépôt du front (S3, puis invalidation du cache).
3. **API Back-end et base** : un serveur EC2 `t3.micro` avec Docker Compose (API, migration, PostgreSQL 16 en conteneur), image dans GitHub Container Registry, secrets dans Parameter Store, déploiement par AWS Systems Manager, sans SSH.
4. **Une seule adresse** : CloudFront oriente `/api/*`, `/health`, `/swagger*` et `/openapi/*` vers le serveur (cache désactivé, en-tête `Authorization` transmis), le reste vers S3. HTTPS entre le navigateur et CloudFront ; le serveur n'accepte que les adresses de CloudFront.
5. Recette déployée automatiquement à chaque fusion dans `dev` ; production manuelle, après approbation du Product Owner (environnement GitHub `production`).

## Alternatives écartées

- **Tout sur un serveur derrière un proxy Caddy** (front en conteneur) : plus simple, mais front et back livrés par le même script, front hors de S3 + CloudFront (écart au Bloc 1), et HTTPS impossible sans nom de domaine.
- **Architecture cible complète** (répartiteur de charge, Auto Scaling, RDS) : payante au-delà des crédits, sans bénéfice pour une recette à faible trafic.
- **Front servi par l'API** (fichiers statiques dans ASP.NET Core) : couple les deux livraisons, contraire au Bloc 2.

## Conséquences

- Positives : front déployé comme dans la cible ; HTTPS sans nom de domaine ; pas de CORS ; front et back livrés séparément ; coût nul pendant le Free plan.
- Négatives : un seul serveur, sans Auto Scaling ; base en conteneur, sauvegardée par `pg_dump` ; HTTP entre CloudFront et le serveur (le jeton circule en clair sur ce tronçon) ; adresse `*.cloudfront.net`.
- Passage à la cible : ajouter un répartiteur de charge avec Auto Scaling et un certificat, faire pointer la chaîne de connexion vers RDS. Aucun changement de code.
- Sur l'ADR 0002 : le proxy Caddy cité dans son contexte est remplacé par CloudFront. La remarque sur `X-Forwarded-For` vaut toujours : derrière CloudFront, l'API lit l'adresse du visiteur dans cet en-tête (`UseForwardedHeaders`), sinon la limitation par adresse IP devient globale.
- À vérifier avant la mise en place : disponibilité de CloudFront avec le Free plan ; à défaut, appliquer le repli décrit dans [environnements](../deployment/environments.md) et le signaler dans la copie.
