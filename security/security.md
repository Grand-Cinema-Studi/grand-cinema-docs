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
| Consulter la programmation et les places restantes | Oui | Oui | (renvoyé vers sa page) | (renvoyé vers sa page) | Oui |
| Réserver et payer, voir ses billets | | Oui | | | |
| Consulter les réservations au guichet | | | Oui (lecture seule) | | Oui |
| Scanner et valider un billet | | | | Oui | |
| Gérer la programmation | | | | | Oui |
| Suivre les réservations | | | Oui (lecture seule) | | Oui |

- Chaque endpoint déclare qui peut l'appeler (politiques d'autorisation d'ASP.NET Core) ; un endpoint sans déclaration n'est accessible qu'aux utilisateurs connectés (politique de repli), jamais par oubli au public.
- Les droits sont vérifiés par l'API, jamais seulement par le front : masquer un bouton n'est pas une protection.
- Un spectateur n'accède qu'à ses propres réservations : l'identifiant de l'utilisateur vient de sa session, pas de l'URL ni du corps de la requête.
- Un compte créé par l'inscription publique a toujours le rôle `Customer` : les rôles du personnel ne s'obtiennent pas en s'inscrivant.
- La programmation (`/api/programming…`, US 5.2) est réservée au rôle `Manager` par une politique d'autorisation posée sur tout le groupe d'endpoints (`RequireRole`, d'après le rôle du jeton) : un spectateur ou un caissier reçoit `403`, un visiteur `401`. Le front masque aussi le lien et la page, par confort seulement.
- L'achat (`POST /api/reservations`, première étape de tout achat) est réservé au rôle `Customer` par la politique `customer` (décision du Product Owner, 07/11/2026) : le caissier, le contrôleur et le responsable reçoivent `403`. Le paiement, l'annulation et la lecture d'une réservation sont réservés à son propriétaire : sans blocage possible, le personnel n'a rien à payer. Le front ne propose l'achat qu'à un visiteur ou un spectateur, et renvoie le caissier et le contrôleur de l'accueil et des films vers leur page (`/tracking`, `/ticket-check`) : chaque rôle n'a que les droits de son métier (moindre privilège).
- Le suivi des réservations (`GET /api/reservation-tracking`, US 5.3 et 5.1) est ouvert au responsable et au caissier par une politique nommée, `reservation-tracking` (`RequireRole(Manager, Cashier)`). Le caissier n'a ainsi qu'une lecture : aucun endpoint d'écriture ne lui est ouvert, et la programmation lui répond `403`. Un droit se donne endpoint par endpoint ; tout le reste reste fermé par défaut.

## Protection des données en transit et au repos

- **HTTPS** obligatoire dès que l'authentification existe. Sur la recette, CloudFront fournit le HTTPS entre le navigateur et l'application (adresse `*.cloudfront.net`) ; le tronçon CloudFront → serveur reste en HTTP, et le serveur n'accepte que les adresses de CloudFront (liste de préfixes gérée par AWS). En production : HTTPS de bout en bout (nom de domaine, certificat, répartiteur de charge), critère bloquant de mise en production ([guide de déploiement](../deployment/deploy-guide.md#critères-de-mise-en-production)).
- Derrière CloudFront, l'API lit l'adresse du visiteur dans l'en-tête `X-Forwarded-For` (`UseForwardedHeaders`), pour que la limitation des tentatives de connexion s'applique par visiteur et non à tous à la fois.
- La base PostgreSQL n'est joignable que par l'API : aucun port publié sur Internet.
- Identifiants en GUID : impossibles à deviner ou à parcourir dans les URL.

## Secrets

- **Aucun secret dans les dépôts, les images Docker ni GitHub.**
- Développement : `.env` (ignoré par Git) et user-secrets .NET.
- Recette et production : AWS SSM Parameter Store, chiffré (mot de passe de la base, clé de signature des jetons, clés Stripe) ; lu au déploiement par le rôle du serveur. La clé Stripe de test reste en recette ; la clé réelle n'existe qu'en production.
- Intégration continue : accès à AWS par OIDC, sans clé stockée dans GitHub ; jeton SonarCloud en secret du dépôt.

Détail par environnement : [environnements](../deployment/environments.md#secrets-et-configuration).

## Paiement (US 3.1)

- Le paiement est confié à **Stripe Checkout** : la carte est saisie sur la page hébergée par Stripe, et **aucune donnée de carte ne transite par l'API ni n'est stockée**. Le périmètre de conformité PCI DSS du projet reste minimal (questionnaire SAQ A).
- Une réservation n'est confirmée que par la notification de Stripe (`POST /api/payments/stripe-webhook`, `checkout.session.completed`), dont la signature HMAC-SHA256 est vérifiée avant tout traitement, avec une tolérance de 5 minutes contre le rejeu. Une notification non signée reçoit `400`. Le retour du navigateur sur `?payment=success` n'est qu'un affichage : il ne suffit pas à se déclarer « payé ».
- La confirmation est sans risque à répéter (une notification renvoyée par Stripe ne change rien) et prend le verrou de la séance : elle ne peut pas croiser la libération des places d'un blocage expiré. Un paiement sans place (arrivé après la libération, ou second paiement) est remboursé automatiquement.
- Le paiement exige l'acceptation des conditions générales de vente ; leur date et leur version sont enregistrées sur la réservation.
- Clés Stripe en mode test sur tous les environnements du projet : user-secrets en local, Parameter Store en recette (`stripe-secret-key`, `stripe-webhook-secret`).

## Billets (US 3.2 et 4.1, faits les 06 et 07/11/2026)

- Le QR code ne contient que le **code du billet** : 32 caractères hexadécimaux, 128 bits tirés par le générateur cryptographique de .NET (`RandomNumberGenerator`). Il ne se devine pas, et ne contient aucune donnée personnelle ni l'identifiant de la réservation, qui circule dans les adresses du site.
- Aucun billet sans paiement confirmé : le code est écrit par la même instruction `UPDATE` que le passage à `Confirmed`, et seulement si la réservation attendait son paiement ; une notification Stripe reçue deux fois ne le change pas. Un index unique interdit deux billets de même code.
- Un billet ne se lit que par son propriétaire : `404 ticket-not-found` pour un billet d'autrui, comme pour une réservation non payée, sans révéler lequel des deux.
- Le QR code est dessiné par le navigateur, dans le gabarit Angular : aucune image stockée, aucun HTML injecté, la protection d'Angular contre le XSS reste active. Le billet d'une séance annulée n'affiche pas de QR code.
- Contrôle à l'entrée (US 4.1) : réservé au rôle `TicketInspector` (politique `ticket-inspection`), conformément au tableau des droits. Le billet est vérifié et marqué utilisé (`TicketUsedAt`) par **une seule instruction** `UPDATE … WHERE "TicketUsedAt" IS NULL` : PostgreSQL verrouille la ligne, et de deux scans simultanés du même billet à deux portes, un seul laisse entrer. Testé sur un vrai PostgreSQL avec deux scans en parallèle.
- Un billet ne vaut que pour sa séance : le contrôleur choisit la séance qu'il contrôle, et un billet d'une autre séance est refusé. Un texte qui n'a pas la forme d'un code de billet (QR code d'autre chose) est refusé sans interroger la base.
- La caméra du contrôleur est lue dans le navigateur (`jsqr`) : aucune image ne quitte le téléphone, seul le texte lu part vers l'API. La réponse ne contient aucune donnée personnelle (film, heure, salle, places).

## Correspondance avec le Top 10 de l'OWASP

| Catégorie de risque | Mesure dans le projet | État |
|---|---|---|
| Contrôle d'accès défaillant | politiques par rôle vérifiées par l'API, accès refusé par défaut, accès limité à ses propres réservations, GUID | accès par défaut, GUID et réservations (celle d'un autre répond `404`) : fait ; rôle du responsable sur la programmation : fait (US 5.2, 04/11/2026) ; caissier en lecture seule sur le suivi des réservations : fait (US 5.1, 06/11/2026) ; contrôleur seul sur le contrôle des billets : fait (US 4.1, 07/11/2026) ; achat réservé au spectateur : fait (07/11/2026) |
| Défaillances cryptographiques | mots de passe hachés et salés, HTTPS | Fait (HTTPS par CloudFront sur la recette) |
| Injection (SQL) | requêtes EF Core paramétrées, aucun SQL construit par concaténation | Fait |
| Injection (XSS) | échappement automatique des affichages par Angular | Fait |
| Conception non sécurisée | double vente empêchée par la base et par une transaction ([bug 0001](../quality/bugs/0001-double-booking.md)) | Fait (US 2.3, bug 0001 corrigé le 30/10/2026) ; notification de paiement signée (US 3.1) |
| Mauvaise configuration | Swagger désactivé par défaut, aucune trace technique dans les réponses d'erreur, un seul port ouvert | Fait (un seul port ouvert, réservé à CloudFront) |
| Composants vulnérables | versions figées, alertes Dependabot, analyse SonarCloud | Fait (Dependabot et SonarCloud à chaque Pull Request) |
| Authentification défaillante | limitation des tentatives, message d'échec neutre | Fait |
| Journalisation insuffisante | journaux de l'API (connexions refusées, erreurs), sans mot de passe ni donnée personnelle superflue | _(à venir)_ |

## Vérification

- Tests automatisés des règles d'accès : un appel sans session reçoit `401`, un rôle insuffisant reçoit `403` : `401` testé (US 2.1 et 2.3) ; `403` testé pour un spectateur et un caissier sur la programmation (US 5.2), et pour un spectateur sur le suivi des réservations (US 5.3) ; `200` testé pour le caissier sur le suivi (US 5.1) ; `403` testé pour un spectateur, un caissier et un responsable sur le contrôle des billets (US 4.1), et pour le caissier, le contrôleur et le responsable sur l'achat (07/11/2026).
- Analyse SonarCloud à chaque Pull Request, points sensibles de sécurité examinés (voir [qualité du code](../quality/code-quality.md)).
- La [Definition of Done](https://github.com/Grand-Cinema-Studi/grand-cinema-docs/issues/24) impose, pour chaque story : mots de passe hachés, aucune donnée de carte stockée, données personnelles traitées selon le RGPD.
- Avant le premier festival, un test d'intrusion par un prestataire est recommandé (hors budget du MVP, Bloc 2).
