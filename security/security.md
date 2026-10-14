# Sécurité

Les mesures de sécurité de l'application, ce qu'elles protègent, et leur état.

Les éléments marqués _(à venir)_ sont décidés mais pas encore réalisés.

## Ce qu'il faut protéger

| Bien | Risque principal |
|---|---|
| Les comptes des spectateurs | vol de mot de passe, accès au compte d'un autre |
| Les réservations et les billets | double vente, billet falsifié ou utilisé deux fois |
| Le paiement | vol de données de carte, paiement validé sans avoir été payé |
| La programmation | modification par une personne non autorisée |
| Les données personnelles | fuite, conservation excessive (voir [RGPD](gdpr.md)) |
| Le serveur et la base | intrusion, perte de données |

## Authentification et rôles

### Comptes _(à venir, US 2.1)_

- Un compte par adresse e-mail (contrainte d'unicité en base), adresse enregistrée en minuscules.
- **Mot de passe jamais stocké en clair** : seule son empreinte est enregistrée (colonne `PasswordHash`), calculée par un algorithme lent et salé conçu pour les mots de passe (PBKDF2 avec le `PasswordHasher` d'ASP.NET Core, ou Argon2). Le choix sera consigné dans un ADR.
- En cas d'échec de connexion, le message ne dit pas si c'est l'adresse ou le mot de passe qui est faux (critère de l'US 2.1) : on ne révèle pas quelles adresses ont un compte.
- Tentatives de connexion limitées (limitation de débit d'ASP.NET Core) contre les attaques par force brute.
- Mécanisme de session (jeton JWT ou cookie `HttpOnly`) : décidé avec l'US 2.1 et consigné dans le même ADR.

### Rôles et droits

Les quatre rôles du Bloc 1 (`UserRole` dans le code) :

| Action | Visiteur | Spectateur (`Customer`) | Caissier (`Cashier`) | Contrôleur (`TicketInspector`) | Responsable (`Manager`) |
|---|---|---|---|---|---|
| Consulter la programmation et les places restantes | ✅ | ✅ | ✅ | ✅ | ✅ |
| Réserver et payer, voir ses billets | | ✅ | | | |
| Consulter les réservations au guichet | | | ✅ (lecture seule) | | ✅ |
| Scanner et valider un billet | | | | ✅ | |
| Gérer la programmation | | | | | ✅ |
| Suivre les réservations | | | | | ✅ |

- Chaque endpoint déclare qui peut l'appeler (politiques d'autorisation d'ASP.NET Core) _(à venir)_ ; un endpoint sans déclaration n'est accessible qu'aux utilisateurs connectés, jamais par oubli au public.
- Les droits sont vérifiés par l'API, jamais seulement par le front : masquer un bouton n'est pas une protection.
- Un spectateur n'accède qu'à ses propres réservations : l'identifiant de l'utilisateur vient de sa session, pas de l'URL ni du corps de la requête.
- Un compte créé par l'inscription publique a toujours le rôle `Customer` : les rôles du personnel ne s'obtiennent pas en s'inscrivant.

## Protection des données en transit et au repos

- **HTTPS** obligatoire dès que l'authentification existe. L'environnement de démonstration est d'abord en HTTP ; un nom de domaine et un certificat Let's Encrypt (obtenu et renouvelé seul par le proxy Caddy) sont ajoutés avant l'US 2.1 _(à venir)_.
- La base PostgreSQL n'est joignable que par l'API : aucun port publié sur Internet.
- Identifiants en GUID : impossibles à deviner ou à parcourir dans les URL.

## Secrets

- **Aucun secret dans les dépôts, les images Docker ni GitHub.**
- Développement : `.env` (ignoré par Git) et user-secrets .NET.
- Production : AWS SSM Parameter Store, chiffré ; lu au déploiement par le rôle du serveur.
- Intégration continue : accès à AWS par OIDC, sans clé stockée dans GitHub ; jeton SonarCloud en secret du dépôt _(à venir)_.

Détail par environnement : [environnements](../deployment/environments.md#secrets-et-configuration).

## Paiement _(à venir, US 3.1)_

- Le paiement est confié à **Stripe** : la saisie de la carte se fait sur un formulaire fourni par Stripe, et **aucune donnée de carte ne transite par l'API ni n'est stockée**. Le périmètre de conformité PCI DSS du projet reste minimal.
- Une réservation n'est confirmée que sur notification de Stripe (webhook), dont la signature est vérifiée : un appel du navigateur ne suffit pas à se déclarer « payé ».
- Clés Stripe en mode test sur tous les environnements du projet.

## Billets _(à venir, US 3.2 et 4.1)_

- Le QR code contient un identifiant impossible à deviner, jamais des données lisibles.
- Un billet ne peut être validé qu'une fois : le scan enregistre son utilisation.

## Correspondance avec le Top 10 de l'OWASP

| Catégorie de risque | Mesure dans le projet | État |
|---|---|---|
| Contrôle d'accès défaillant | politiques par rôle vérifiées par l'API, accès limité à ses propres réservations, GUID | _(à venir, US 2.1)_ |
| Défaillances cryptographiques | mots de passe hachés et salés, HTTPS | _(à venir, US 2.1)_ |
| Injection (SQL) | requêtes EF Core paramétrées, aucun SQL construit par concaténation | ✅ |
| Injection (XSS) | échappement automatique des affichages par Angular | _(à venir, front)_ |
| Conception non sécurisée | double vente empêchée par la base et par une transaction ([bug 0001](../quality/bugs/0001-double-booking.md)) | contrainte ✅, transaction _(à venir, US 2.3)_ |
| Mauvaise configuration | Swagger désactivé par défaut, aucune trace technique dans les réponses d'erreur, un seul port ouvert | ✅ Swagger ; le reste _(à venir)_ |
| Composants vulnérables | versions figées, alertes Dependabot, analyse SonarCloud | versions ✅ ; outils _(à venir)_ |
| Authentification défaillante | limitation des tentatives, message d'échec neutre | _(à venir, US 2.1)_ |
| Journalisation insuffisante | journaux de l'API (connexions refusées, erreurs), sans mot de passe ni donnée personnelle superflue | _(à venir)_ |

## Vérification

- Tests automatisés des règles d'accès : un appel sans session reçoit `401`, un rôle insuffisant reçoit `403` _(à venir, US 2.1)_.
- Analyse SonarCloud à chaque Pull Request, points sensibles de sécurité examinés (voir [qualité du code](../quality/code-quality.md)).
- La [Definition of Done](https://github.com/Grand-Cinema-Studi/grand-cinema-docs/issues/24) impose, pour chaque story : mots de passe hachés, aucune donnée de carte stockée, données personnelles traitées selon le RGPD.
- Avant le premier festival, un test d'intrusion par un prestataire est recommandé (hors budget du MVP, Bloc 2).
