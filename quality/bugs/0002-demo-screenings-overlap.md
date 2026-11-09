# Bug 0002 – Deux films dans la même salle à la même heure (données de démonstration de la recette)

- Statut : **corrigé le 08/11/2026** (scripts `deploy/reset-demo-data.sh` et `docs/database/seed-remove-overlaps.sql`, journal du backend, étape 27 bis)  ; vérifié sur la recette le 08/11/2026
- Gravité : mineure (données de démonstration seulement ; l'API refuse ce cas à toute création ou modification de séance), mais visible du public et du jury sur la recette
- Concerne : backend, données de démonstration (`docs/database/seed-dev.sql`) et procédure de rechargement sur la recette
- Trouvé : en recette, le 08/11/2026, en consultant la programmation du jour
- Liens : issue `bug` [#79](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/issues/79) du dépôt backend (créée le 09/11/2026, après le correctif, puis fermée), Pull Requests #68, #69 et #71 du backend, [procès-verbal de recette](../acceptance-report.md#anomalies-relevées-pendant-la-recette)

## Description

Sur la recette, la programmation du 08/11/2026 montrait **Alien** et **Ratatouille** à 22 h 30 en Salle 1. Une salle ne peut accueillir qu'une séance à la fois : l'API le garantit (`409 room-not-available`, US 5.2), nettoyage de 15 minutes compris. Ces séances n'avaient pourtant pas été créées par l'API, mais par le script des données de démonstration.

## Cause

`seed-dev.sql` calcule le programme de démonstration **à partir du jour où il est chargé** : chaque ligne place un film dans une salle et un créneau, du jour `CURRENT_DATE + premier jour` au jour `CURRENT_DATE + dernier jour`. L'identifiant d'une séance est tiré de son film, de sa salle et de sa **date exacte** (`md5(film || salle || date et heure)`), et l'insertion ignore les identifiants déjà présents (`ON CONFLICT DO NOTHING`).

| Chargement | Le 08/11 à 22 h 30 en Salle 1 | Identifiant |
|---|---|---|
| un premier jour | Ratatouille | tiré de « Ratatouille, Salle 1, 08/11 22:30 » |
| un autre jour (programme décalé d'autant) | Alien | tiré de « Alien, Salle 1, 08/11 22:30 » : différent, donc inséré |

Rechargé le **même** jour, le script ne crée aucun doublon. Rechargé **un autre jour**, tout le programme glisse : l'ancien reste, le nouveau s'ajoute, et des films différents se retrouvent dans le même créneau.

### Seconde cause, révélée par le premier lancement du correctif (08/11/2026)

Le contrôle final du script a encore trouvé **10 chevauchements**, tous entre séances de démonstration, en Salle 9 : Amadeus à 19 h 30, puis Seven ou Whiplash à 22 h 30, du 26/11 au 05/12. `seed-dev.sql` donne à Amadeus 160 minutes, et son programme laisse 3 heures entre deux créneaux d'une salle ; mais `seed-posters.sql`, appliqué ensuite, remplace les durées par celles de TMDB : **180 minutes**. 19 h 30 + 180 + 15 minutes de nettoyage = 22 h 45, après le début de la séance suivante. Un simple déplacement d'Amadeus dans le programme ne suffit pas : la salle et les créneaux sont tous occupés sur cette période (vérifié par une simulation du programme).

Sur le poste du développeur, `scripts/reset-demo-screenings.sh` supprime d'abord les séances de démonstration (identifiants en `33333333-`) : le problème n'y apparaît pas. Sur la recette, la procédure du journal rechargeait `seed-dev.sql` **seul**, à chaque nouveau compte de démonstration (étapes 10g, 15j, 23, 25, 27), donc à des jours différents.

## Pourquoi il n'a pas été vu plus tôt

- Le script avait été vérifié « rejouable deux fois sans erreur, aucun chevauchement » : vrai le jour du test, pas d'un jour à l'autre.
- Les tests automatisés ne lisent pas ces données : chaque test crée les siennes dans une base jetable.
- Les données de démonstration sont écrites en SQL, **sans passer par l'API** : la règle de la salle libre, testée dans l'API, ne s'applique pas à elles.

Leçon : des données insérées hors de l'application doivent être vérifiées par la même règle que l'application.

## Reproduction

Sur la base de la recette, la requête de contrôle (même règle que l'API : durée du film plus 15 minutes) :

```sql
SELECT count(*)
FROM "Screenings" s1
JOIN "Movies" m1 ON m1."Id" = s1."MovieId"
JOIN "Screenings" s2 ON s2."RoomId" = s1."RoomId" AND s2."Id" > s1."Id" AND s2."CancelledAt" IS NULL
JOIN "Movies" m2 ON m2."Id" = s2."MovieId"
WHERE s1."CancelledAt" IS NULL
  AND s1."StartsAt" < s2."StartsAt" + make_interval(mins => m2."DurationMinutes" + 15)
  AND s2."StartsAt" < s1."StartsAt" + make_interval(mins => m1."DurationMinutes" + 15);
```

Avant le correctif : un résultat supérieur à 0 (dont Alien et Ratatouille, Salle 1, 08/11 22 h 30). Après : 0.

## Correctif

- **`deploy/reset-demo-data.sh`** (dépôt backend), lancé sur le serveur par Session Manager : il fait sur la recette ce que `reset-demo-screenings.sh` fait sur le poste. En **une seule transaction**, il supprime les séances de démonstration et leurs réservations, puis recharge `seed-dev.sql` et `seed-posters.sql` de la branche demandée (`dev` par défaut). Il se termine par la requête de contrôle ci-dessus et **échoue** s'il reste un chevauchement.
- Ce qu'il garde : les comptes, les films (y compris ceux ajoutés par le responsable, US 5.6) et les séances créées dans l'écran Programmation. Ce qu'il efface : les réservations faites sur les séances de démonstration (achats d'essai compris).
- **`docs/database/seed-remove-overlaps.sql`** (seconde cause), appliqué après `seed-posters.sql` par les deux scripts de remise à zéro (poste et recette) : il applique la règle de l'API aux données de démonstration, une fois les durées de TMDB connues, et supprime la **plus tardive** de deux séances de démonstration qui se chevauchent, avec sa vente de démonstration. Il ne supprime jamais une séance créée dans l'écran Programmation. Vérifié sur PostgreSQL 16 avec `schema.sql` : 10 chevauchements après `seed-posters.sql`, 0 après, 2 547 séances gardées ; rejoué une seconde fois, toujours 0.
- Le journal du backend ne recharge plus jamais `seed-dev.sql` seul : chaque procédure renvoie au script.

## Validation

- 08/11/2026, premier lancement sur la recette : le 08/11 à 22 h 30, la Salle 1 n'a plus qu'un film ; programmation, connexion des quatre comptes et réservation vérifiées. Mais le contrôle final trouve 10 chevauchements entre séances de démonstration (seconde cause, ci-dessus) : le script échoue comme prévu, au lieu de laisser une recette incohérente.
- Avec `seed-remove-overlaps.sql` : 0 chevauchement sur PostgreSQL 16 (08/11/2026). Sur la recette, le 08/11/2026, script et données du commit `d97767f` de `dev` : 127 films, 2 104 séances à venir (les 10 séances de 22 h 30 en Salle 9 qui chevauchaient Amadeus sont retirées), « no room holds two screenings at the same time ».
- Une séance créée à la main dans l'écran Programmation peut tomber sur un créneau du nouveau programme : le script le signale (contrôle supérieur à 0) ; elle est alors annulée dans l'écran Programmation.

## Pour aller plus loin

Comme la contrainte d'unicité du bug 0001, la base pourrait refuser elle-même deux séances qui se chevauchent dans une salle : une contrainte d'exclusion PostgreSQL (`EXCLUDE USING gist (RoomId WITH =, période WITH &&)`), qui suppose de stocker la fin de chaque séance. Notée dans les améliorations (V2) : l'API et le script de contrôle suffisent au volume du projet.
