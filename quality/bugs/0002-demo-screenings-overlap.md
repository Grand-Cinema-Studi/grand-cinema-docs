# Bug 0002 – Deux films dans la même salle à la même heure (données de démonstration de la recette)

- Statut : **corrigé le 08/11/2026** (script `deploy/reset-demo-data.sh`, journal du backend, étape 27 bis) ; vérifié sur la recette
- Gravité : mineure (données de démonstration seulement ; l'API refuse ce cas à toute création ou modification de séance), mais visible du public et du jury sur la recette
- Concerne : backend, données de démonstration (`docs/database/seed-dev.sql`) et procédure de rechargement sur la recette
- Trouvé : en recette, le 08/11/2026, en consultant la programmation du jour
- Liens : issue `bug` du dépôt backend #… _(à reporter)_, [procès-verbal de recette](../acceptance-report.md#anomalies-relevées-pendant-la-recette)

## Description

Sur la recette, la programmation du 08/11/2026 montrait **Alien** et **Ratatouille** à 22 h 30 en Salle 1. Une salle ne peut accueillir qu'une séance à la fois : l'API le garantit (`409 room-not-available`, US 5.2), nettoyage de 15 minutes compris. Ces séances n'avaient pourtant pas été créées par l'API, mais par le script des données de démonstration.

## Cause

`seed-dev.sql` calcule le programme de démonstration **à partir du jour où il est chargé** : chaque ligne place un film dans une salle et un créneau, du jour `CURRENT_DATE + premier jour` au jour `CURRENT_DATE + dernier jour`. L'identifiant d'une séance est tiré de son film, de sa salle et de sa **date exacte** (`md5(film || salle || date et heure)`), et l'insertion ignore les identifiants déjà présents (`ON CONFLICT DO NOTHING`).

| Chargement | Le 08/11 à 22 h 30 en Salle 1 | Identifiant |
|---|---|---|
| un premier jour | Ratatouille | tiré de « Ratatouille, Salle 1, 08/11 22:30 » |
| un autre jour (programme décalé d'autant) | Alien | tiré de « Alien, Salle 1, 08/11 22:30 » : différent, donc inséré |

Rechargé le **même** jour, le script ne crée aucun doublon. Rechargé **un autre jour**, tout le programme glisse : l'ancien reste, le nouveau s'ajoute, et des films différents se retrouvent dans le même créneau.

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
- Le journal du backend ne recharge plus jamais `seed-dev.sql` seul : chaque procédure renvoie au script.

## Validation

- 08/11/2026 : script lancé sur la recette ; requête de contrôle à 0 ; le 08/11 à 22 h 30, la Salle 1 n'a plus qu'un film ; programmation, connexion des quatre comptes et réservation vérifiées.
- Une séance créée à la main dans l'écran Programmation peut tomber sur un créneau du nouveau programme : le script le signale (contrôle supérieur à 0) ; elle est alors annulée dans l'écran Programmation.

## Pour aller plus loin

Comme la contrainte d'unicité du bug 0001, la base pourrait refuser elle-même deux séances qui se chevauchent dans une salle : une contrainte d'exclusion PostgreSQL (`EXCLUDE USING gist (RoomId WITH =, période WITH &&)`), qui suppose de stocker la fin de chaque séance. Notée dans les améliorations (V2) : l'API et le script de contrôle suffisent au volume du projet.
