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
| `GET` | `/api/schedule?date=AAAA-MM-JJ` | public | films et séances d'un jour (aujourd'hui si la date est omise, toute date future acceptée) ; films triés par titre, séances par heure ; places restantes de chaque séance (`remainingSeats`) | 1.1, 2.2 |
| `GET` | `/api/movies` | public | films à l'affiche (au moins une séance à venir), triés par titre | 1.1 |
| `GET` | `/api/movies/{id}` | public | fiche d'un film : toutes ses séances à venir, sans limite de date (réservation jusqu'à un an à l'avance), regroupées par jour et triées par heure, avec les places restantes ; `404` (`/errors/movie-not-found`) si le film n'existe pas | 1.1, 2.2 |
| `POST` | `/api/users` | public, limité à 10 appels par minute et par IP | création d'un compte spectateur ; `acceptPrivacyPolicy: true` obligatoire (sinon `400`, champ `acceptPrivacyPolicy`), date et version de la politique enregistrées ; `201` avec un jeton d'accès, `409` si l'adresse est déjà utilisée | 2.1 |
| `POST` | `/api/tokens` | public, limité à 10 appels par minute et par IP | connexion ; `200` avec un jeton d'accès, `401` sans dire quelle donnée est fausse | 2.1 |
| `GET` | `/api/users/me` | connecté | profil de l'utilisateur identifié par le jeton | 2.1 |
| `DELETE` | `/api/users/me` | connecté | suppression du compte identifié par le jeton (droit à l'effacement) ; `204`, `404` si le compte n'existe plus ; réservations gardées, sans lien avec le compte | 2.1 |
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
  | Heure d'une séance | heure locale du cinéma (Europe/Paris), sans fuseau | `"2026-07-18T20:30:00"` |
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

## Codes de réponse

| Code | Quand | Exemple |
|---|---|---|
| `200 OK` | lecture ou action réussie, avec un contenu | programmation du jour, éventuellement vide |
| `201 Created` | ressource créée ; l'en-tête `Location` donne son adresse | places bloquées pour une réservation _(US 2.3)_ |
| `204 No Content` | action réussie, sans contenu à renvoyer | suppression de son compte (`DELETE /api/users/me`) |
| `400 Bad Request` | requête mal formée ou données invalides | date au mauvais format, nombre de places nul |
| `401 Unauthorized` | authentification absente ou invalide | réserver sans être connecté _(US 2.1)_ |
| `403 Forbidden` | connecté, mais rôle insuffisant | un spectateur qui modifie la programmation |
| `404 Not Found` | ressource inexistante | film inconnu (`/errors/movie-not-found`), séance inconnue |
| `409 Conflict` | la demande est valide mais contredit l'état actuel | plus assez de places restantes, place déjà prise par un autre spectateur _(US 2.3)_ |
| `500 Internal Server Error` | erreur imprévue | base injoignable |

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
