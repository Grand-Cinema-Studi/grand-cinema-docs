# Bug 0001 – Double réservation de la dernière place

- Statut : identifié à la conception ; reproduction et correctif développés avec l'US 2.3 _(à venir)_
- Gravité : bloquante (double vente)
- Concerne : backend, réservation (US 2.3)
- Liens : [US 2.3](https://github.com/Grand-Cinema-Studi/grand-cinema-docs/issues/9), [schéma de la base](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/blob/dev/docs/database/erd.md)

## Description

Deux spectateurs demandent en même temps la dernière place libre d'une séance. Si l'application vérifie d'abord qu'il reste une place, puis enregistre le blocage, les deux vérifications peuvent avoir lieu **avant** les deux enregistrements : chacun voit la place libre, et les deux blocages sont acceptés. La même place est vendue deux fois.

C'est une **situation de concurrence** (*race condition*) de type « vérifier puis agir » : chaque requête est correcte prise seule ; c'est leur entrelacement qui produit l'erreur.

| Instant | Requête A | Requête B |
|---|---|---|
| t1 | lit les places restantes : 1 | |
| t2 | | lit les places restantes : 1 |
| t3 | bloque la place | |
| t4 | | bloque la même place |
| Résultat | deux spectateurs pour un fauteuil | |

## Pourquoi c'est probable

- Pendant le Festival de Jazz, des centaines de spectateurs visent les mêmes séances au même moment (cas cité par l'énoncé).
- L'erreur ne se voit pas en développement : un seul utilisateur, aucune requête simultanée.
- Les tests unitaires ne la voient pas non plus : le repository y est simulé, il n'y a ni base réelle ni concurrence. C'est un risque relevé dès la stratégie de tests du Bloc 2.

## Reproduction

Un test d'intégration, contre un vrai PostgreSQL (Testcontainers) :

1. préparer une séance dont il ne reste qu'une place ;
2. lancer deux demandes de blocage **en parallèle**, synchronisées pour qu'elles partent au même instant ;
3. constater qu'avant correctif, les deux réussissent, et que la séance compte plus de places bloquées que sa salle n'en a.

Nom prévu : `When_two_users_book_the_last_seat_at_the_same_time_Then_only_one_succeeds`, étiqueté `US=2.3` et `Category=Integration`. Il reprend le critère d'acceptation de l'US 2.3 : « Quand deux utilisateurs visent la dernière place, alors un seul obtient le blocage ; l'autre reçoit un message explicite. »

## Correctif

Deux protections complémentaires.

**1. La base refuse toute seconde vente : contrainte d'unicité (déjà en place).**
`UNIQUE (ScreeningId, SeatId)` dans `ReservedSeats`, créée par la migration `InitialSchema`. Comme les places attribuées sont toujours des places réelles de la salle, deux réservations qui dépassent la capacité finissent forcément par viser une même place : la seconde insertion est refusée par PostgreSQL, même si le code a un défaut. C'est la dernière ligne de défense.

**2. Le gestionnaire de réservation sérialise les demandes : transaction et verrou.**
Le comptage et l'enregistrement se font dans une même transaction, qui commence par verrouiller la ligne de la séance :

```sql
SELECT "Id" FROM "Screenings" WHERE "Id" = @screeningId FOR UPDATE;
```

La seconde requête attend que la première ait terminé ; elle compte alors les places restantes à jour, constate qu'il n'en reste plus, et refuse. Le verrou ne porte que sur une séance : les réservations des autres séances ne sont pas ralenties.

**3. Une réponse claire.**
Le refus est un résultat métier attendu, pas une erreur technique : l'API répond `409 Conflict` au format Problem Details (voir les [conventions de l'API](../../api/api-conventions.md)), et le front affiche « Cette place vient d'être réservée par un autre spectateur ». Si malgré tout la contrainte d'unicité se déclenche, l'erreur PostgreSQL correspondante (`23505`) est traduite elle aussi en `409`.

Pourquoi les deux : la contrainte garantit l'intégrité quoi qu'il arrive, mais ne permet pas de bien répondre à l'utilisateur ; la transaction permet de bien répondre, mais dépend du code. Redis n'est pas nécessaire : la base suffit au volume du cinéma (voir l'[architecture](../../architecture/overview.md#points-darchitecture-structurants)).

## Validation

- Le test de reproduction passe : une seule demande réussit, l'autre reçoit `409`.
- Les tests de l'US 2.3 et des autres stories passent toujours (non-régression).
- Avant une mise en production réelle, un test de charge (JMeter, 200 utilisateurs simultanés sur une séance presque complète, prévu au Bloc 2) vérifie qu'aucune double vente n'apparaît sous charge.

Traçabilité, à la fusion : issue `bug`, Pull Request, commits du test (rouge) et du correctif (vert).

## Autres bugs potentiels relevés

Moins graves, à traiter avec leur user story :

| Risque | Conséquence | Traitement prévu |
|---|---|---|
| Un blocage expiré n'est pas libéré (paiement abandonné) | des places restent indisponibles alors que personne ne les a payées | libération à l'expiration des 10 minutes, testée avec une horloge simulée (US 2.3) |
| « Aujourd'hui » calculé dans le fuseau du serveur (souvent UTC) | entre minuit et 1 h ou 2 h (selon la saison), la programmation affichée est celle de la veille | heure de Paris imposée (`TZ=Europe/Paris` en production) et test du passage de minuit |
| Tri des titres selon la langue du serveur | « Élise » peut être classé après « Zorro » sur un serveur en anglais | comparaison explicite en français |
