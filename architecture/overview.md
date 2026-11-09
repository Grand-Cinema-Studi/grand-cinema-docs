# Vue d'ensemble de l'architecture

Les éléments marqués _(à venir)_ sont décidés mais pas encore réalisés.

## Composants

```mermaid
flowchart LR
    user["Spectateur, personnel du cinéma<br/>(navigateur, ordinateur ou smartphone)"]
    cdn["CloudFront<br/>CDN, HTTPS"]
    front["Front-end Angular<br/>fichiers statiques sur S3"]
    api["API Back-end<br/>ASP.NET Core sur EC2"]
    db[("PostgreSQL 16<br/>RDS en production")]
    stripe["Stripe<br/>paiement (mode test)"]
    watch["CloudWatch<br/>supervision (à venir)"]
    posters["Affiches des films<br/>bucket S3 privé (US 5.6)"]

    user -->|HTTPS| cdn
    cdn -->|"fichiers du front"| front
    cdn -->|"HTTP JSON /api/..."| api
    cdn -->|"/posters/..."| posters
    api -->|"ajout d'une affiche"| posters
    api -->|EF Core| db
    api -->|"API Stripe"| stripe
    api -.-> watch
```

| Composant | Rôle | Dépôt |
|---|---|---|
| CloudFront | CDN du Bloc 1 : HTTPS, une seule adresse ; oriente `/api/*` vers l'API, `/posters/*` vers le bucket des affiches et le reste vers les fichiers du front | configuration : [environnements](../deployment/environments.md) |
| Front-end (Angular) | application web (SPA) : programmation, compte, réservation, billets, écrans du personnel ; fichiers statiques sur S3 | [grand-cinema-frontend](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend) |
| API Back-end (ASP.NET Core) | règles métier, sécurité, accès aux données ; aucune page HTML ; conteneur Docker sur EC2 | [grand-cinema-backend](https://github.com/Grand-Cinema-Studi/grand-cinema-backend) |
| PostgreSQL | utilisateurs, films, salles, places, séances, réservations | schéma : [erd.md](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/blob/dev/docs/database/erd.md) |
| Affiches des films | images envoyées par le responsable (US 5.6) : bucket S3 privé, lu par CloudFront seule ; l'API ne peut qu'y ajouter un fichier | configuration : [environnements](../deployment/environments.md) ; sécurité : [affiches](../security/security.md#affiches-des-films-us-56) |
| Stripe | paiement par carte ; aucune donnée de carte ne transite par l'application ni n'est stockée ; conformité PCI-DSS portée par Stripe | service tiers |
| CloudWatch | supervision : métriques, journaux, alertes | AWS |

Billets : le Bloc 1 prévoyait un bucket S3 pour les billets et leurs QR codes. Il n'est pas créé (US 3.2) : le QR code n'est qu'une autre écriture du code du billet, que l'API renvoie et que le navigateur dessine. Il n'y a donc aucun fichier à stocker, à protéger ni à facturer.

Le front et l'API sont deux applications séparées qui ne communiquent que par HTTP et JSON : chacune a son dépôt et son pipeline, et se développe, se teste et se déploie seule (front : S3 + CloudFront ; API : EC2).

## Modules fonctionnels de l'API

Découpage issu du Bloc 1, réalisé progressivement user story par user story :

| Module | User stories | État |
|---|---|---|
| Programmation (consultation) | 1.1, 1.2 | Fait (`GET /api/schedule`, `GET /api/movies`, `GET /api/movies/{id}`) |
| Authentification | 2.1 | Fait (`POST /api/users`, `POST /api/tokens`, `GET /api/users/me`, `DELETE /api/users/me`) |
| Réservation | 2.2, 2.3 | Fait (places restantes ; `POST`, `GET` et `DELETE /api/reservations`) |
| Paiement | 3.1 | Fait (`POST /api/reservations/{id}/payment`, `POST /api/payments/stripe-webhook`), en mode test |
| Billets | 3.2, 4.1 | Fait pour l'US 3.2 (`GET /api/reservations/{id}/ticket`, `GET /api/users/me/tickets`) : code du billet tiré au paiement confirmé, QR code dessiné par le front ; contrôle à l'entrée fait pour l'US 4.1 (`/api/ticket-checks`, contrôleur seulement) : billet vérifié et marqué utilisé en une seule instruction SQL |
| Administration | 5.1, 5.2, 5.3, 5.6 | Fait (`/api/programming`, dont le catalogue des films `/api/programming/movies` ; `GET /api/reservation-tracking`), réservé au personnel |

## Architecture du backend

Clean Architecture en quatre projets ; les dépendances vont toujours vers le centre.

```mermaid
flowchart TB
    api["Api<br/>endpoints, injection de dépendances, Swagger"]
    infra["Infrastructure<br/>EF Core, PostgreSQL, repositories"]
    app["Application<br/>cas d'usage (handlers CQRS), DTO,<br/>interfaces des repositories"]
    domain["Domain<br/>entités, énumérations"]

    api --> app
    api --> infra
    infra --> app
    app --> domain
```

- **Domain** ne dépend de rien : ni du framework web, ni de la base.
- **Application** définit les interfaces dont elle a besoin (`IScreeningReadRepository`, …) ; **Infrastructure** les implémente. Les cas d'usage se testent donc sans base de données, avec des doublures.
- **CQRS** au niveau des opérations : chaque lecture est une requête (`IQueryHandler`), chaque modification une commande (`ICommandHandler`), traitée par son handler. Pas de bibliothèque, pas de bus de messages : un simple appel de méthode.
- **Api** ne contient aucune logique métier : un endpoint traduit la requête HTTP en requête ou commande CQRS, et renvoie le DTO en JSON.

## Points d'architecture structurants

- **Pas de double vente** : les places sont bloquées dans une transaction PostgreSQL pendant le paiement, et une contrainte d'unicité `(séance, place)` refuse en base toute seconde vente, même en cas de bug applicatif. Pas de Redis : la base suffit au volume du cinéma.
- **Blocage limité dans le temps** : places libérées si le paiement n'est pas validé au bout de 10 minutes (paramétrable), dans la transaction même du blocage suivant, sans tâche planifiée.
- **Heure du cinéma** : les séances sont enregistrées à l'heure locale du cinéma (Europe/Paris) ; les instants techniques (création, expiration) en UTC.
- **Paiement confirmé par Stripe seul** : la carte est saisie chez Stripe ; une notification signée, vérifiée par l'API, passe la réservation à « payée », sous le même verrou que les blocages.
- **Annuler ou archiver plutôt que supprimer** : une séance annulée garde ses ventes (`CancelledAt`, US 5.2), un film archivé ses séances, ses ventes et ses billets (`ArchivedAt`, US 5.6) ; un filtre ou une condition les retire des lectures. Seul le compte d'un utilisateur est vraiment effacé (droit à l'effacement), et ses réservations restent, sans lien avec lui. Le suivi des ventes et la comptabilité ne perdent donc rien.
- **Secrets hors du code** : aucun mot de passe ni clé dans les dépôts (voir [environnements](../deployment/environments.md)).

## Hébergement

| | Cible du Bloc 1 | Recette du projet (mise en ligne) |
|---|---|---|
| Fournisseur | AWS, région Paris (`eu-west-3`) | AWS, région Paris (`eu-west-3`), compte Free plan |
| Front-end | S3 + CloudFront | S3 + CloudFront |
| API Back-end | EC2 avec Auto Scaling pour les pics du festival | un serveur EC2, API en conteneur Docker |
| Base | RDS PostgreSQL (service géré) | PostgreSQL 16 en conteneur sur le serveur |
| HTTPS | de bout en bout | entre le navigateur et CloudFront ; HTTP entre CloudFront et le serveur, réservé à CloudFront |
| Raison de l'écart | — | budget nul ; l'API est sans état et son image Docker permet de passer à l'Auto Scaling et à RDS sans changer le code |

Détail : [deployment/environments.md](../deployment/environments.md) ; décision : [ADR 0003](../adr/0003-hosting-cloudfront-s3-ec2.md).

## Décisions liées

Les raisons de ces choix sont consignées dans les [ADR](../adr/README.md).
