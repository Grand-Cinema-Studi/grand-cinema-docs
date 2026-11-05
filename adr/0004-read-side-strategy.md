# ADR 0004 – Stratégie du côté lecture (CQRS)

- Statut : Accepté
- Concerne : backend
- Date : 05/11/2026
- Liens : US 5.3 ; appliqué par la Pull Request [grand-cinema-backend#55](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/55) (`IReservationTrackingQueries`)

## Contexte

- CQRS appliqué au niveau des opérations : requêtes et commandes séparées, chacune avec son handler ; un seul modèle (entités du domaine) et une seule base PostgreSQL.
- Volumes faibles pour la programmation (quelques dizaines de séances par jour).
- Stratégie de tests du Bloc 2 : TDD, logique des handlers testée en unitaire avec Moq.
- L'US 5.3 introduit des lectures d'agrégats (nombre de réservations, places vendues, taux de remplissage).

## Décision

1. Écriture : entités et repository, pour protéger les règles métier (capacité, non-double vente).
2. Lecture simple d'une entité ou d'une petite liste : repository en lecture seule, nommé `I<Entity>ReadRepository`, placé dans `Abstractions/Persistence`, qui renvoie des entités ; la logique reste dans le handler et se teste en unitaire.
3. Lecture d'agrégats, de statistiques ou de gros volumes : projection SQL directe en DTO, interface nommée `I<Sujet>Queries`, placée dans `Features/<Feature>/Queries`, implémentée dans `Infrastructure/Persistence/Queries` ; testée en intégration avec PostgreSQL.
4. Passage d'une lecture du cas 2 au cas 3 uniquement sur un besoin constaté (écran lent, requête lourde), jamais par anticipation.

## Alternatives écartées

- Tout en repository : charger des centaines d'entités pour n'en garder que des totaux gaspille mémoire et requêtes (US 5.3).
- Tout en projection : la logique des handlers part dans le SQL et ne se teste plus qu'en intégration, contraire à la stratégie TDD du Bloc 2 ; deux mécanismes à maîtriser dès le départ, sans gain mesurable sur de petits volumes.
- Bases de lecture et d'écriture séparées (CQRS complet avec événements) : complexité et cohérence différée injustifiées pour ce projet.

## Conséquences

- Positives : chaque lecture utilise la technique adaptée ; les noms (`ReadRepository` ou `Queries`) indiquent l'approche ; règle connue de tous.
- Négatives : deux approches à connaître ; les projections exigent des tests d'intégration (base PostgreSQL de test).
- À surveiller : un écran de lecture simple qui devient lent est le signal de passer du cas 2 au cas 3.
