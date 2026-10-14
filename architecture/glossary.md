# Glossaire

Tout ce qui est nommé dans le code est en **anglais** : projets, classes, propriétés, tables et colonnes, noms des tests, branches et commits. Le français reste réservé à la documentation, aux textes affichés à l'utilisateur et aux données d'exemple. Ce glossaire fait le lien entre le vocabulaire métier (cahier des charges, user stories, modèle de données du Bloc 1) et le code.

## Notions métier

| Métier (français) | Code (anglais) | Remarque |
|---|---|---|
| Programmation (films et horaires d'un jour) | `Schedule` | `GET /api/schedule` |
| Films à l'affiche (au moins une séance à venir) | `MoviesShowing` | |
| Film | `Movie` | table `Movies` |
| Salle | `Room` | table `Rooms` ; `Capacity` = nombre maximal de places |
| Place | `Seat` | table `Seats` ; repérée par un rang (`Row`) et un numéro (`Number`) |
| Séance | `Screening` | table `Screenings` ; `StartsAt` à l'heure locale du cinéma |
| Réservation | `Reservation` | table `Reservations` |
| Place réservée | `ReservedSeat` | table `ReservedSeats` ; une ligne par place d'une réservation |
| Utilisateur | `User` | table `Users` |
| Places restantes | `RemainingSeats` | US 2.2, à venir |

## Rôles des utilisateurs (`UserRole`)

| Métier | Code |
|---|---|
| Spectateur (régulier, festivalier, touriste) | `Customer` |
| Caissier | `Cashier` |
| Contrôleur | `TicketInspector` |
| Responsable Programmation / Direction | `Manager` |

## Statuts d'une réservation (`ReservationStatus`)

| Métier | Code | Signification |
|---|---|---|
| En attente de paiement | `PendingPayment` | places bloquées pendant le paiement, jusqu'à l'expiration (10 minutes) |
| Confirmée | `Confirmed` | paiement validé, places vendues |
| Expirée | `Expired` | paiement non validé à temps, places libérées |
| Annulée | `Cancelled` | réservation annulée, places libérées |

## Vocabulaire technique

| Terme | Sens dans ce projet |
|---|---|
| Dépôt | dépôt Git (`grand-cinema-backend`, …) |
| Repository | pattern d'accès aux données : interface définie par l'Application (`IScreeningReadRepository`), implémentée par l'Infrastructure |
| Requête (query) | opération qui lit sans rien modifier, traitée par un `IQueryHandler` |
| Commande (command) | opération qui modifie des données, traitée par un `ICommandHandler` |
| Handler | classe qui traite une requête ou une commande : un cas d'usage |
| Entité | objet du modèle métier (`Screening`, …), propre au backend |
| DTO | objet de transfert : ce que l'API renvoie au front (`ScheduleDto`, …) |
| Migration | évolution versionnée du schéma de la base, générée par EF Core |
| Pull Request | demande d'intégration d'une branche sur GitHub ; équivalent de la merge request de GitLab |
| `dev` | branche d'intégration des dépôts de code, équivalente à la branche `develop` du Git Flow |
| ADR | Architecture Decision Record : fiche qui consigne une décision technique (voir [adr](../adr/README.md)) |
