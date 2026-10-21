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

### Comptes (US 2.1)

Décision détaillée dans l'[ADR 0002](../adr/0002-authentication-jwt.md).

- Un compte par adresse e-mail (contrainte d'unicité en base), adresse enregistrée en minuscules et sans espaces. Deux inscriptions simultanées avec la même adresse : la base n'en accepte qu'une, l'autre reçoit `409`.
- **Mot de passe jamais stocké en clair** : seule son empreinte est enregistrée (colonne `PasswordHash`), calculée par le `PasswordHasher` d'ASP.NET Core (PBKDF2, HMAC-SHA512, 100 000 itérations, sel aléatoire par mot de passe). 12 caractères au moins (recommandation de la CNIL).
- En cas d'échec de connexion, le message ne dit pas si c'est l'adresse ou le mot de passe qui est faux (critère de l'US 2.1) : on ne révèle pas quelles adresses ont un compte.
- Tentatives limitées : 10 inscriptions ou connexions par minute et par adresse IP, réponse `429` au-delà.
- Session : jeton JWT signé (HMAC-SHA256), valable 60 minutes, envoyé dans l'en-tête `Authorization: Bearer`. Il contient l'identifiant (`sub`), l'adresse et le rôle de l'utilisateur. La clé de signature est un secret (user-secrets en développement, Parameter Store en production).

### Rôles et droits

Les quatre rôles du Bloc 1 (`UserRole` dans le code) :

| Action | Visiteur | Spectateur (`Customer`) | Caissier (`Cashier`) | Contrôleur (`TicketInspector`) | Responsable (`Manager`) |
|---|---|---|---|---|---|
| Consulter la programmation et les places restantes | Oui | Oui | Oui | Oui | Oui |
| Réserver et payer, voir ses billets | | Oui | | | |
| Consulter les réservations au guichet | | | Oui (lecture seule) | | Oui |
| Scanner et valider un billet | | | | Oui | |
| Gérer la programmation | | | | | Oui |
| Suivre les réservations | | | | | Oui |

- Chaque endpoint déclare qui peut l'appeler (politiques d'autorisation d'ASP.NET Core) ; un endpoint sans déclaration n'est accessible qu'aux utilisateurs connectés (politique de repli), jamais par oubli au public.
- Les droits sont vérifiés par l'API, jamais seulement par le front : masquer un bouton n'est pas une protection.
- Un spectateur n'accède qu'à ses propres réservations : l'identifiant de l'utilisateur vient de sa session, pas de l'URL ni du corps de la requête.
- Un compte créé par l'inscription publique a toujours le rôle `Customer` : les rôles du personnel ne s'obtiennent pas en s'inscrivant.

## Protection des données en transit et au repos

- **HTTPS** obligatoire dès que l'authentification existe. Sur la recette, CloudFront fournit le HTTPS entre le navigateur et l'application (adresse `*.cloudfront.net`) _(à venir, mise en ligne)_ ; le tronçon CloudFront → serveur reste en HTTP, et le serveur n'accepte que les adresses de CloudFront (liste de préfixes gérée par AWS). En production : HTTPS de bout en bout (nom de domaine, certificat, répartiteur de charge), critère bloquant de mise en production ([guide de déploiement](../deployment/deploy-guide.md#critères-de-mise-en-production)).
- Derrière CloudFront, l'API lit l'adresse du visiteur dans l'en-tête `X-Forwarded-For` (`UseForwardedHeaders`), pour que la limitation des tentatives de connexion s'applique par visiteur et non à tous à la fois _(à venir, mise en ligne)_.
- La base PostgreSQL n'est joignable que par l'API : aucun port publié sur Internet.
- Identifiants en GUID : impossibles à deviner ou à parcourir dans les URL.

## Secrets

- **Aucun secret dans les dépôts, les images Docker ni GitHub.**
- Développement : `.env` (ignoré par Git) et user-secrets .NET.
- Recette et production : AWS SSM Parameter Store, chiffré (mot de passe de la base, clé de signature des jetons, clés Stripe) ; lu au déploiement par le rôle du serveur. La clé Stripe de test reste en recette ; la clé réelle n'existe qu'en production.
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
| Contrôle d'accès défaillant | politiques par rôle vérifiées par l'API, accès refusé par défaut, accès limité à ses propres réservations, GUID | accès par défaut et GUID : fait ; rôles et réservations _(à venir, US 2.3 et 5.2)_ |
| Défaillances cryptographiques | mots de passe hachés et salés, HTTPS | hachage : fait ; HTTPS par CloudFront _(à venir, mise en ligne)_ |
| Injection (SQL) | requêtes EF Core paramétrées, aucun SQL construit par concaténation | Fait |
| Injection (XSS) | échappement automatique des affichages par Angular | _(à venir, front)_ |
| Conception non sécurisée | double vente empêchée par la base et par une transaction ([bug 0001](../quality/bugs/0001-double-booking.md)) | contrainte : faite, transaction _(à venir, US 2.3)_ |
| Mauvaise configuration | Swagger désactivé par défaut, aucune trace technique dans les réponses d'erreur, un seul port ouvert | Swagger : fait ; le reste _(à venir)_ |
| Composants vulnérables | versions figées, alertes Dependabot, analyse SonarCloud | versions : fait ; outils _(à venir)_ |
| Authentification défaillante | limitation des tentatives, message d'échec neutre | Fait |
| Journalisation insuffisante | journaux de l'API (connexions refusées, erreurs), sans mot de passe ni donnée personnelle superflue | _(à venir)_ |

## Vérification

- Tests automatisés des règles d'accès : un appel sans session reçoit `401`, un rôle insuffisant reçoit `403` _(à venir, US 2.1)_.
- Analyse SonarCloud à chaque Pull Request, points sensibles de sécurité examinés (voir [qualité du code](../quality/code-quality.md)).
- La [Definition of Done](https://github.com/Grand-Cinema-Studi/grand-cinema-docs/issues/24) impose, pour chaque story : mots de passe hachés, aucune donnée de carte stockée, données personnelles traitées selon le RGPD.
- Avant le premier festival, un test d'intrusion par un prestataire est recommandé (hors budget du MVP, Bloc 2).
