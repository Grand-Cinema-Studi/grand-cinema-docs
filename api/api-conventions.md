# Conventions de l'API

Le contrat entre le front et le back : forme des routes, des données et des erreurs. La liste exacte des endpoints et de leurs schémas est le document OpenAPI généré par le backend (Swagger) ; ce document fixe les règles que tous les endpoints suivent.

Les éléments marqués _(à venir)_ sont décidés mais pas encore réalisés.

## Documentation des endpoints (Swagger)

- ASP.NET Core génère le document OpenAPI à partir du code (`/openapi/v1.json`) ; Swagger UI l'affiche (`/swagger`). La documentation ne peut donc pas diverger du code.
- Chaque endpoint déclare un nom, une étiquette (la fonctionnalité), un résumé et une description. Le schéma de réponse est déduit du type de retour (`TypedResults.Ok<ScheduleDto>`).
- Swagger est désactivé par défaut et s'active par configuration (`Swagger:Enabled`, ou `Swagger__Enabled=true`) : activé en développement, et sur l'environnement de démonstration pendant la correction.

## Endpoints existants

| Méthode | Route | Accès | Rôle | User story |
|---|---|---|---|---|
| `GET` | `/api/schedule?date=AAAA-MM-JJ` | public | films et séances d'un jour (aujourd'hui si la date est omise, toute date future acceptée), séances pas encore commencées seulement ; films triés par titre, séances par heure ; places restantes de chaque séance (`remainingSeats`) | 1.1, 2.2 |
| `GET` | `/api/movies` | public | films à l'affiche (au moins une séance à venir), triés par titre | 1.1 |
| `GET` | `/api/movies/featured` | public | films à la une de l'accueil : au plus 4, ceux dont la prochaine séance est la plus proche (aujourd'hui, puis les jours et mois suivants ; à égalité, par titre) ; une séance commencée ou annulée ne compte pas | 1.1 |
| `GET` | `/api/movies/{id}` | public | fiche d'un film : toutes ses séances à venir, sans limite de date (réservation jusqu'à un an à l'avance), regroupées par jour et triées par heure, avec les places restantes ; `404` (`/errors/movie-not-found`) si le film n'existe pas | 1.1, 2.2 |
| `POST` | `/api/users` | public, limité à 10 appels par minute et par IP | création d'un compte spectateur ; `acceptPrivacyPolicy: true` obligatoire (sinon `400`, champ `acceptPrivacyPolicy`), date et version de la politique enregistrées ; `201` avec un jeton d'accès, `409` si l'adresse est déjà utilisée | 2.1 |
| `POST` | `/api/tokens` | public, limité à 10 appels par minute et par IP | connexion ; `200` avec un jeton d'accès, `401` sans dire quelle donnée est fausse | 2.1 |
| `GET` | `/api/users/me` | connecté | profil de l'utilisateur identifié par le jeton | 2.1 |
| `DELETE` | `/api/users/me` | connecté | suppression du compte identifié par le jeton (droit à l'effacement) ; `204`, `404` si le compte n'existe plus ; réservations gardées, sans lien avec le compte | 2.1 |
| `POST` | `/api/reservations` | connecté | blocage de 1 à 10 places d'une séance pendant 10 minutes (`PendingPayment`), places attribuées par le cinéma ; au plus 10 places par compte et par séance, bloquées ou payées, même en plusieurs blocages ; `201` avec `Location` ; `400` (`seatCount`), `404 screening-not-found`, `409 screening-started`, `409 seat-limit-reached` (le compte dépasserait 10 places sur la séance ; vérifié avant les places libres), `409 not-enough-seats` (y compris pour le second de deux clients sur la dernière place) | 2.3 |
| `GET` | `/api/reservations/{id}` | connecté, propriétaire | un blocage ou une réservation de l'utilisateur, avec son film (`movieId`), ses places et son statut (`PendingPayment`, `Expired` dès la fin des 10 minutes, `Cancelled` après une annulation, `Confirmed` après paiement) ; `404 reservation-not-found` pour un identifiant inconnu comme pour celle d'un autre | 2.3 |
| `GET` | `/api/reservations/{id}/ticket` | connecté, propriétaire | billet d'une réservation payée : `ticketCode` (32 caractères hexadécimaux, contenu du QR code dessiné par le front), `movieTitle`, `startsAt`, `roomName`, `seats`, `screeningCancelled` ; `404 ticket-not-found` pour une réservation non payée, inconnue ou d'un autre, sans distinction | 3.2 |
| `GET` | `/api/users/me/tickets` | connecté | billets de l'utilisateur : ses réservations payées, la séance la plus tardive d'abord (`reservationId`, `movieTitle`, `startsAt`, `roomName`, `seatCount`, `screeningCancelled`) ; une séance annulée après le paiement reste listée | 3.2 |
| `GET` | `/api/ticket-checks/screenings?date=AAAA-MM-JJ` | rôle `TicketInspector` | séances d'un jour que le contrôleur peut contrôler (aujourd'hui, à l'heure du cinéma, sans `date`), commencées comprises, annulées exclues, par heure puis salle | 4.1 |
| `POST` | `/api/ticket-checks` | rôle `TicketInspector` | `screeningId` et `ticketCode` lu dans le QR code ; `200` (film, heure, salle, places) et billet marqué utilisé, en une seule instruction SQL ; `409 ticket-already-used`, `409 ticket-for-another-screening`, `404 ticket-not-found` (code inconnu, ou QR code d'autre chose) | 4.1 |
| `POST` | `/api/reservations/{id}/payment` | connecté, propriétaire | page de paiement Stripe d'un blocage valable : `acceptTermsOfSale: true` obligatoire (sinon `400`, champ `acceptTermsOfSale`), date et version des conditions générales de vente enregistrées ; `200` avec `checkoutUrl` ; `404 reservation-not-found`, `409 reservation-paid`, `409 hold-expired` (10 minutes écoulées ou blocage annulé) | 3.1 |
| `POST` | `/api/payments/stripe-webhook` | public, réservé à Stripe : signature `Stripe-Signature` vérifiée avant tout | notification de Stripe ; `400` si elle n'est pas signée par Stripe ; un paiement abouti (`checkout.session.completed`) confirme la réservation une seule fois (`200`, y compris pour une notification répétée) ; un paiement arrivé après la libération des places, ou un second paiement, est remboursé | 3.1 |
| `DELETE` | `/api/reservations/{id}` | connecté, propriétaire | annulation d'un blocage avant paiement : statut `Cancelled` (la réservation est gardée), places remises en vente aussitôt ; `204`, y compris pour un blocage déjà expiré ou déjà annulé (rien ne change) ; `404 reservation-not-found` pour un identifiant inconnu comme pour celle d'un autre ; `409 reservation-confirmed` pour une réservation payée | 2.3 |
| `GET` | `/api/reservation-tracking?date=AAAA-MM-JJ` | rôle `Manager` ou `Cashier` (lecture seule) | suivi des réservations d'un jour, pour le responsable et le guichet : chaque séance (commencées comprises, annulées exclues) avec `soldOnline` (payées sur Stripe), `soldAtCounter`, `heldSeats` (paiements en cours), `capacity` et `remainingSeats` ; calculé par PostgreSQL (projection SQL, ADR 0004) | 5.3, 5.1 |
| `GET` | `/api/programming?date=AAAA-MM-JJ` | rôle `Manager` | programmation d'un jour pour le responsable : toutes les séances (commencées comprises) avec `takenSeats` (places bloquées ou vendues) et `capacity`, le catalogue des films (`movies`, avec `durationMinutes`) et les salles (`rooms`) | 5.2 |
| `POST` | `/api/programming/screenings` | rôle `Manager` | création d'une séance (`movieId`, `roomId`, `startsAt`, `price`) ; `201` avec `screeningId` et `Location` ; `400` (`startsAt` passé ou avec fuseau, `price` nul ou négatif), `404 movie-not-found`, `404 room-not-found`, `409 room-not-available` (salle occupée, nettoyage de 15 minutes compris) | 5.2 |
| `PUT` | `/api/programming/screenings/{id}` | rôle `Manager` | nouvelle date, heure et prix d'une séance (`startsAt`, `price`) ; ni la salle ni le film ; `204` ; `400`, `404 screening-not-found`, `409 screening-started`, `409 room-not-available`, `409 screening-has-reservations` (voir ci-dessous) | 5.2 |
| `DELETE` | `/api/programming/screenings/{id}` | rôle `Manager` | annulation d'une séance : elle disparaît de toutes les lectures, ses blocages passent à `Cancelled`, ses ventes sont gardées ; `204` ; `404 screening-not-found`, `409 screening-started`, `409 screening-has-reservations` | 5.2 |
| `GET` | `/health` | public | état de l'application, pour la supervision et le script de déploiement | — |
| `GET` | `/` | public | nom de l'API, pour un humain qui ouvre son adresse | — |

## Routes

- Préfixe `/api/` pour tout endpoint métier : CloudFront oriente ces chemins vers l'API, le reste vers les fichiers du front sur S3 (voir [environnements](../deployment/environments.md)).
- Ressources au pluriel, en anglais, en kebab-case : `/api/screenings/{id}`, `/api/reservations`.
- Le verbe HTTP porte l'action, pas l'URL : `POST /api/reservations`, et non `/api/create-reservation`.
- Les filtres passent en paramètres de requête : `?date=2026-07-18`.
- Les identifiants sont des GUID : impossibles à deviner ou à parcourir.

## Format des données

- JSON, noms de propriétés en camelCase (réglage par défaut d'ASP.NET Core).
- Dates et heures :

  | Donnée | Format | Exemple |
  |---|---|---|
  | Jour (`DateOnly`) | `AAAA-MM-JJ` | `"2026-07-18"` |
  | Heure d'une séance | heure locale du cinéma (Europe/Paris), sans fuseau ; envoyée avec un `Z` ou un décalage, elle est refusée (`400`) plutôt que convertie | `"2026-07-18T20:30:00"` |
  | Instant technique (création, expiration) | UTC, suffixe `Z` | `"2026-07-18T18:41:07Z"` |

  Une séance « à 20 h 30 » reste à 20 h 30 quel que soit le fuseau du serveur ou du navigateur ; un instant technique est absolu.
- Montants : nombre décimal en euros, deux décimales (`9.50`).
- Les réponses contiennent des DTO, jamais les entités : seules les données utiles à l'écran sortent de l'API.

Exemple, `GET /api/schedule?date=2026-07-18` :

`remainingSeats` : capacité de la salle moins les places vendues (`Confirmed`) et celles bloquées par un paiement en cours (`PendingPayment` non expiré) ; `0` : séance complète. Calculé à chaque appel, sans cache (US 2.2).

```json
{
  "date": "2026-07-18",
  "movies": [
    {
      "movieId": "6f1c…",
      "title": "Blue Note",
      "synopsis": "…",
      "durationMinutes": 136,
      "genre": "Documentary",
      "releaseDate": "2023-10-01",
      "posterUrl": "https://…/poster.jpg",
      "screenings": [
        { "screeningId": "a83e…", "roomName": "Salle 1", "startsAt": "2026-07-18T20:30:00", "price": 9.50, "remainingSeats": 42 }
      ]
    }
  ]
}
```

Exemple, `GET /api/movies/{id}` : les séances ont le même format que dans la programmation ; `days` contient un élément par jour qui a au moins une séance, dans l'ordre des dates.

```json
{
  "movieId": "2222…0005",
  "title": "Bird",
  "synopsis": "…",
  "durationMinutes": 161,
  "genre": "Drama",
  "releaseDate": "1988-09-30",
  "posterUrl": "https://…/poster.jpg",
  "days": [
    {
      "date": "2026-10-31",
      "screenings": [
        { "screeningId": "b41c…", "roomName": "Salle 4", "startsAt": "2026-10-31T19:30:00", "price": 9.50, "remainingSeats": 12 }
      ]
    },
    {
      "date": "2027-10-30",
      "screenings": [
        { "screeningId": "33333333-…", "roomName": "Salle 2", "startsAt": "2027-10-30T20:30:00", "price": 12.00, "remainingSeats": 120 }
      ]
    }
  ]
}
```

Exemple, `POST /api/reservations` avec `{ "screeningId": "…", "seatCount": 2 }`, réponse `201` avec l'en-tête `Location: /api/reservations/a410…` :

```json
{
  "reservationId": "a410…",
  "screeningId": "3333…",
  "movieId": "2222…0005",
  "status": "PendingPayment",
  "seats": ["A1", "A2"],
  "totalAmount": 24.00,
  "expiresAt": "2026-10-30T17:07:59.961Z"
}
```

`expiresAt` est un instant technique en UTC : le front en tire le compte à rebours affiché au spectateur. `movieId` lui permet de ramener le spectateur à la fiche du film, après une annulation ou quand les places ont été libérées.

Exemple, `POST /api/reservations/{id}/payment` avec `{ "acceptTermsOfSale": true }`, réponse `200` :

```json
{ "checkoutUrl": "https://checkout.stripe.com/c/pay/cs_test_…" }
```

Le front y envoie le spectateur : la carte est saisie sur la page de Stripe. Stripe le renvoie ensuite sur `/reservations/{id}?payment=success` (ou `/reservations/{id}` s'il renonce). Ce retour n'est qu'un affichage : seule la notification signée de Stripe passe la réservation à `Confirmed`.

## Codes de réponse

| Code | Quand | Exemple |
|---|---|---|
| `200 OK` | lecture ou action réussie, avec un contenu | programmation du jour, éventuellement vide |
| `201 Created` | ressource créée ; l'en-tête `Location` donne son adresse | places bloquées pour une réservation (`POST /api/reservations`) |
| `204 No Content` | action réussie, sans contenu à renvoyer | suppression de son compte (`DELETE /api/users/me`), annulation d'un blocage (`DELETE /api/reservations/{id}`) |
| `400 Bad Request` | requête mal formée ou données invalides | date au mauvais format, nombre de places nul, conditions générales de vente non acceptées, notification non signée par Stripe |
| `401 Unauthorized` | authentification absente ou invalide | réserver sans être connecté |
| `403 Forbidden` | connecté, mais rôle insuffisant | un spectateur qui modifie la programmation, un caissier qui ouvre la programmation, un spectateur ou un caissier qui contrôle un billet |
| `404 Not Found` | ressource inexistante | film inconnu (`/errors/movie-not-found`), séance inconnue ou annulée, salle inconnue (`room-not-found`), réservation sans billet car non payée, ou code scanné inconnu (`ticket-not-found`) |
| `409 Conflict` | la demande est valide mais contredit l'état actuel | plus assez de places restantes, y compris pour le second de deux spectateurs sur la dernière place ; séance déjà commencée ; paiement d'un blocage expiré (`hold-expired`) ou déjà payé (`reservation-paid`) ; plus de 10 places pour un même compte sur une séance (`seat-limit-reached`) ; annulation en ligne d'une réservation payée (`reservation-confirmed`) ; salle déjà occupée à cette heure (`room-not-available`) ; séance dont des places sont prises, tant que la demande n'est pas confirmée (`screening-has-reservations`) ; billet déjà utilisé (`ticket-already-used`) ou d'une autre séance (`ticket-for-another-screening`) |
| `500 Internal Server Error` | erreur imprévue | base injoignable |

**Confirmation d'une demande** (US 5.2) : quand des places d'une séance sont bloquées ou vendues, sa modification ou son annulation répond `409 screening-has-reservations` ; le front affiche le nombre de places et demande confirmation, puis renvoie la même demande avec `?confirmed=true`. L'avertissement est porté par l'API : aucun client ne peut le sauter.

Une liste vide n'est pas une erreur : une journée sans séance renvoie `200` avec `"movies": []`, et c'est le front qui affiche le message « aucune séance ».

## Format des erreurs

Toutes les erreurs suivent le format standard **Problem Details** ([RFC 9457](https://www.rfc-editor.org/rfc/rfc9457)), intégré à ASP.NET Core (`AddProblemDetails`) :

```json
{
  "type": "/errors/not-enough-seats",
  "title": "Not enough seats left",
  "status": 409,
  "detail": "Only 2 seats are left for this screening.",
  "instance": "/api/reservations"
}
```

- Le front affiche un message en français choisi d'après `type` ; il ne montre jamais `detail` brut à l'utilisateur.
- Erreurs de validation : `400`, avec un champ `errors` qui liste les champs en cause.
- Erreur imprévue (`500`) : réponse générique, sans trace technique ni message d'exception ; le détail part dans les journaux du serveur.
- Une erreur métier attendue (places insuffisantes, place déjà prise) n'est pas une exception non gérée : le handler renvoie un résultat d'échec, que l'endpoint traduit en code HTTP.

## Authentification

Les endpoints publics (programmation, films à l'affiche, fiche d'un film, places restantes, inscription, connexion) ne demandent rien. Les autres exigent un utilisateur connecté, et selon le cas un rôle : le front envoie le jeton reçu à l'inscription ou à la connexion dans l'en-tête `Authorization: Bearer <jeton>`. Sans jeton valide, la réponse est `401` ; avec un rôle insuffisant, `403`. L'accès est refusé par défaut, y compris pour une route qui n'existe pas : un visiteur sans jeton reçoit alors `401`, et non `404`, ce qui l'empêche de sonder les routes de l'API. Mécanisme et règles : [sécurité](../security/security.md#authentification-et-rôles).

## Faire évoluer le contrat

- Ajouter un champ ou un endpoint ne casse rien : c'est une évolution mineure.
- Supprimer ou renommer un champ, changer un type ou une route casse le front : la Pull Request du backend le signale (`!` dans le message de commit), et la Pull Request correspondante du front est fusionnée en même temps. Voir [versioning](../development/versioning.md#version-de-lapi).
