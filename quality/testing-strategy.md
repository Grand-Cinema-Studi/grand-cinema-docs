# Stratégie de tests

Comment l'application est testée, à quel niveau, avec quels outils, et comment on le prouve. Cette stratégie applique au code celle définie au Bloc 2.

Les éléments marqués _(à venir)_ sont décidés mais pas encore réalisés.

## Principes

- **Tester en continu**, à chaque commit et à chaque Pull Request, et non en fin de projet : une anomalie trouvée tard coûte plus cher.
- **Pyramide des tests** : beaucoup de tests unitaires rapides, moins de tests d'intégration, peu de tests de bout en bout. Les niveaux reprennent les quatre niveaux de l'ISTQB du Bloc 2 : unitaire, intégration, système, acceptation.
- **TDD sur le code métier** (rouge, vert, refactor) : le test est écrit avant le code qu'il vérifie. C'est là qu'une erreur coûte le plus : programmation, places restantes, blocage et expiration, validation d'un billet.
- **Un critère d'acceptation = au moins un test.** Le nom du test reprend le « Quand / Alors » du critère, et le test porte le numéro de sa story : on retrouve tous les tests d'une story, et la story de chaque test.
- **Non-régression** : tous les tests automatisés sont rejoués à chaque Pull Request ; une nouvelle story ne doit jamais en casser une autre.

## Niveaux de tests

| Niveau | Ce qui est vérifié | Outils | État |
|---|---|---|---|
| Unitaire | un handler ou une règle métier, isolé : dépendances simulées, aucune base | xUnit, Moq, Shouldly, `FakeTimeProvider` | Fait : 102 tests (US 1.1 : 14 ; US 2.1 : 38 ; US 2.2 : 3 ; US 2.3 : 14, dont 2 règles du domaine ; US 3.1 : 16, dont 7 sur l'adaptateur Stripe ; US 5.2 : 16, dont 2 règles du domaine ; US 5.3 : 1) |
| Intégration (accès aux données) | un repository contre un vrai PostgreSQL, avec les vraies migrations | xUnit, Testcontainers (PostgreSQL 16 jetable) | Fait : 28 tests (US 1.1 : 5 ; US 2.1 : 5 ; US 2.2 : 1 ; US 2.3 : 8 ; US 3.1 : 4 ; US 5.2 : 4 ; US 5.3 : 1) |
| Intégration (API) | un endpoint de bout en bout dans l'API : routage, sérialisation, codes HTTP, Problem Details, authentification, limitation des tentatives, base réelle | `WebApplicationFactory`, Testcontainers | Fait : 45 tests (US 1.1 : 6 ; US 2.1 : 14 ; US 2.2 : 1 ; US 2.3 : 8 ; US 3.1 : 4 ; US 5.2 : 8 ; US 5.3 : 2 ; `/health` et document OpenAPI : 2) |
| Concurrence | deux requêtes simultanées sur la dernière place : une seule réussit ; dix requêtes simultanées sur dix places : dix places différentes (preuve du verrou) | Testcontainers, tâches parallèles (`Task.WhenAll`) | Fait (30/10/2026) — voir [bug 0001](bugs/0001-double-booking.md) ; deux onglets du même compte qui bloquent 10 places au même instant : un seul réussit (01/11/2026) |
| Composants front | un composant ou un service Angular isolé : réseau remplacé par `HttpTestingController`, service remplacé par une doublure | Vitest (jsdom), outils de test d'Angular | Fait : 136 tests (dépôt `grand-cinema-frontend`) |
| Système / bout en bout | le parcours complet dans un navigateur : consulter, réserver, payer en mode test, recevoir le billet, le scanner | Selenium (Bloc 2), automatisé sur la recette, limité aux parcours critiques | _(à venir, avec le front)_ |
| Performance / charge | la tenue des pics du festival : 200 utilisateurs simultanés sur une séance, 95 % des requêtes en moins de 2 s, zéro double vente | JMeter | en préproduction, avant chaque mise en production et avant le festival _(à venir ; préproduction non montée pour le projet)_ |
| Sécurité | failles courantes (injection SQL, XSS, CSRF, force brute, QR code falsifié ou réutilisé), dépendances vulnérables | SonarCloud et Dependabot à chaque Pull Request ; scan OWASP ZAP en préproduction, à chaque release | SonarCloud et Dependabot : fait (25/10/2026) ; OWASP ZAP _(à venir)_ |
| Compatibilité / ergonomie | pages lisibles sur smartphone et ordinateur ; scan sur les vrais terminaux des contrôleurs | tests manuels guidés par une checklist | _(à venir, en recette, à chaque sprint)_ |
| Acceptation (recette) | la conformité au besoin, vue par les métiers (exemple : le Responsable Programmation crée une séance et la retrouve dans la programmation publique) | cahier de recette (OK / KO), procès-verbal | _(à venir, au fil du sprint ; validation en revue de sprint)_ |

## Conventions d'écriture

- Nom : `When_<situation>_Then_<résultat attendu>`, par exemple `When_no_screening_exists_Then_the_movie_list_is_empty`.
- Étiquettes xUnit :
  - `[Trait("US", "1.1")]` : la story vérifiée ;
  - `[Trait("Category", "Integration")]` : le test a besoin de Docker.
- Structure Arrange / Act / Assert, marquée par trois commentaires.
- Assertions avec Shouldly (`result.Movies.ShouldBeEmpty()`) : lisibles, et messages d'échec explicites.
- Le temps est simulé (`FakeTimeProvider`) : un test sur « aujourd'hui » ou sur l'expiration à 10 minutes s'exécute en quelques millisecondes, et donne le même résultat à toute heure.
- Les tests d'intégration démarrent leur propre PostgreSQL dans un conteneur jetable, y appliquent les migrations, puis le détruisent : ils ne dépendent ni de la base de développement, ni de l'ordre d'exécution.
- Un test qui passe au vert dès son écriture est cassé volontairement (en retirant le code qu'il vérifie) pour s'assurer qu'il sait échouer, puis le code est rétabli.
- Tests d'API : la vraie API démarre en mémoire (`WebApplicationFactory`, environnement `Testing`) sur un PostgreSQL jetable partagé par tous les tests d'API ; chaque test crée ses propres données (adresse, salle et film au nom unique) et n'affirme que sur elles. Les réponses sont lues en JSON brut, avec les noms que reçoit le front, et non désérialisées dans les DTO de l'API : renommer une propriété casse le test comme elle casserait le front. Chaque client simule un visiteur distinct (`X-Forwarded-For`), comme derrière CloudFront.

## Commandes

```bash
dotnet test                                        # tous les tests (Docker doit tourner)
dotnet test --filter "Category!=Integration"       # tests unitaires seuls
dotnet test --filter "US=1.1"                      # tests d'une user story
dotnet test tests/GrandCinema.Api.Tests           # tests d'API seuls (Docker doit tourner)
dotnet test --settings coverlet.runsettings --collect:"XPlat Code Coverage" --results-directory TestResults   # avec mesure de la couverture
```

## Preuve du TDD

L'historique Git montre l'ordre test puis code : pour chaque cycle, un commit `red` (le test, qui échoue), un commit `green` (le code minimal), et si besoin un commit `refactor`. Exemple sur l'US 1.1 :

```text
test(us-1.1): red - group screenings by movie
feat(us-1.1): green - group screenings by movie
refactor(us-1.1): extract movie schedule mapping
```

Les Pull Requests fusionnées en merge commit conservent ces commits (voir le [workflow Git](../development/git-workflow.md#pull-requests)).

Exception assumée : pour l'US 2.1, le code et les tests ont été écrits ensemble, en un seul passage (journal du backend, étape 12). Chaque test a été cassé volontairement pour vérifier qu'il sait échouer. Le cycle rouge, vert, refactor commit par commit reprend à partir de l'US 2.2.

US 2.2 (places restantes) : le test d'API est écrit en premier et reste rouge jusqu'à la fin (boucle externe) ; viennent ensuite un cycle unitaire (la capacité moins les places prises) et un cycle d'intégration sur PostgreSQL (le comptage des places vendues ou bloquées), chacun avec son commit `red` puis `green`. Le cas limite de l'expiration (`>` et non `>=`) est vérifié par sabotage.

US 2.3 (bloquer des places pendant 10 minutes) : 5 tests d'API écrits en premier, dont la reproduction du bug 0001, puis quatre cycles, chacun avec son commit `red` puis `green` : la règle du domaine (durée de 10 minutes, prix figé, expiration), premiers tests de `GrandCinema.Domain.Tests` ; le handler de blocage (1 à 10 places, séance inconnue ou commencée, places insuffisantes, horloge UTC) ; la lecture d'un blocage par son seul propriétaire ; le repository contre PostgreSQL (ordre des places, refus sans blocage partiel, libération des blocages expirés, dix blocages simultanés). Sabotages : `<` au lieu de `<=` sur l'expiration et sur l'heure de début, statut non recalculé, verrou retiré, ordre des rangées. Le test d'API « sans jeton, `401` » était vert dès son écriture (l'accès est refusé par défaut, même à une route inexistante) : il a été vu rouge en rendant l'endpoint public. Total : 104 tests (63 unitaires, 14 d'intégration, 27 d'API).

US 2.3, annulation d'un blocage avant paiement (`DELETE /api/reservations/{id}`, 31/10/2026) : 2 tests d'API écrits en premier (le propriétaire annule et les places reviennent aussitôt ; un autre client reçoit `404`), et un test d'API existant complété (`movieId` dans la réponse). Leur rouge a été un `405` : l'adresse existait déjà pour `GET`, pas pour `DELETE`. Puis trois cycles, chacun avec son commit `red` puis `green` : le film dans la réponse ; le handler (propriétaire seul, refus d'une réservation payée) ; le repository contre PostgreSQL (statut `Cancelled`, places supprimées et aussitôt revendables, réservation payée jamais annulée). Le cycle d'intégration a aussi révélé que la lecture d'un blocage ne chargeait pas sa séance, ce que le test unitaire, qui la fournit lui-même, ne pouvait pas voir. Sabotages : condition sur le propriétaire retirée du handler, condition `Status = 'PendingPayment'` retirée de la requête d'annulation. Total : 111 tests (66 unitaires, 16 d'intégration, 29 d'API).

US 2.3, au plus 10 places par compte et par séance, bloquées ou payées (01/11/2026) : défaut trouvé en recette locale, la limite de 1 à 10 ne valant que pour un blocage, plusieurs onglets permettaient d'en cumuler. Même démarche que le bug 0001 : le défaut est d'abord reproduit par un test d'API (deux blocages de 10 puis de 1 place, le second attendu en `409 seat-limit-reached`, reçu en `201`). Le contrat du repository change ensuite en refactor, tests verts (`HoldResult` : le blocage ou la raison du refus, au lieu d'un blocage ou `null`). Puis deux cycles, chacun avec son commit `red` puis `green` : le handler traduit le nouveau refus ; le repository compte les places du compte sous le verrou de la séance (limite exacte avec des places payées et bloquées ; deux onglets simultanés, un seul réussit). Total : 115 tests (67 unitaires, 18 d'intégration, 30 d'API).

US 3.1, paiement Stripe en mode test (03/11/2026) : 4 tests d'API écrits en premier, avec Stripe remplacé par une doublure (`FakePaymentGateway`) : page de paiement, conditions générales de vente refusées, notification signée qui confirme, notification forgée refusée. Leur rouge : `404` pour la page de paiement, `401` pour la notification (route inexistante, refus par défaut) ; c'est pour obtenir un vrai rouge que la notification forgée est refusée en `400` et non en `401`. Puis quatre cycles, chacun avec son commit `red` puis `green` : la page de paiement dans le handler (conditions, propriétaire, réservation payée, blocage expiré) ; la notification dans le handler (signature, confirmation, remboursement, autres événements) ; la confirmation en base sous le verrou de la séance (confirmée, refusée après libération des places, notification répétée, second paiement) ; la vérification de la signature avec la vraie bibliothèque Stripe, sans réseau (HMAC-SHA256 calculé par le test). Sabotages : expiration ignorée, signature ignorée, statut non vérifié en base, mauvais secret. Le Quality Gate a refusé la première version (75,8 % de couverture du nouveau code) : l'adaptateur Stripe a reçu 5 tests sans réseau, un faux gestionnaire HTTP répondant à la place de Stripe (montant en centimes, réservation, adresses de retour, durée de la page, remboursement, configuration), chacun vu rouge par sabotage. Total : 139 tests (83 unitaires, 22 d'intégration, 34 d'API).

Correctif de la programmation du jour (US 1.1, 03/11/2026, Pull Request #49 du backend) : défaut trouvé en recette, un film dont la seule séance du matin était passée restait dans la programmation du jour et dans le carrousel de l'accueil, alors que le blocage refusait déjà une séance commencée. Gravité mineure (affichage, aucune vente possible), corrigé aussitôt. Deux tests unitaires écrits d'abord, avec l'horloge simulée : les séances commencées disparaissent, et leur film avec elles s'il n'en a pas d'autre (une séance qui commence à l'instant compte comme commencée) ; « commencée » se juge à l'heure de Paris. Commit `red`, puis `green` : la règle du blocage (`StartsAt > maintenant`) appliquée au handler. Sabotages : `>=` au lieu de `>`, heure UTC au lieu de l'heure locale. Pas de test d'API : la requête SQL ne change pas, et l'API de test tourne sur l'horloge réelle. Côté front, un ajustement de libellés (Pull Request #14), sans règle nouvelle : tests adaptés et un test ajouté pour le message des autres jours. Total : 141 tests (85 unitaires, 22 d'intégration, 34 d'API) ; front : 117.

US 5.2, gestion de la programmation (04/11/2026) : 7 tests d'API écrits en premier (création visible dans la programmation publique, déplacement, annulation, avertissement puis confirmation, `403` pour un spectateur et un caissier, lecture d'un jour), avec un compte de n'importe quel rôle créé par le test et son jeton signé par l'API. Puis quatre cycles, chacun avec son commit `red` puis `green` : la règle du domaine (deux séances ne partagent jamais une salle, nettoyage compris), la création, la modification et l'annulation avec l'avertissement (suivies d'un refactor, tests verts, qui supprime un doublon), la lecture d'un jour et le repository contre PostgreSQL. Ce dernier cycle a montré l'intérêt des tests d'intégration : le filtre global qui cache les séances annulées faisait disparaître, par une jointure interne, la réservation payée d'un spectateur ; le test est passé au rouge, et `IgnoreQueryFilters()` l'a corrigé. Sabotages : `<=` au lieu de `<` et nettoyage retiré dans la règle de chevauchement, confirmation ignorée, séance comparée à elle-même, filtre global retiré, rôle retiré du groupe d'endpoints. L'essai manuel dans Swagger a trouvé un `500` (une heure envoyée avec `Z`) : reproduit par un test d'API et un test unitaire, corrigé par un `400`. Total : 169 tests (101 unitaires, 26 d'intégration, 42 d'API).

Correctif du carrousel de l'accueil (US 1.1, 05/11/2026, Pull Request #53 du backend) : une régression du correctif précédent. Les films à la une étaient les premiers de la programmation du jour ; depuis que celle-ci cache les séances commencées, elle est vide le soir, et le carrousel disparaissait. Les tests de non-régression couvraient la programmation, pas « le carrousel le soir ». Correction : `GET /api/movies/featured`, les 4 films dont la prochaine séance est la plus proche. Un test d'intégration (séances en 2040 : ce soir avant demain, séance commencée ou annulée exclue, limite respectée) et un test d'API écrits d'abord ; sabotage : tri par titre au lieu de la prochaine séance. Côté front, le test « les 4 premiers films du jour » est remplacé par « sans aucune séance aujourd'hui, le carrousel affiche quand même les films de l'API ». Total : 171 tests (101 unitaires, 27 d'intégration, 43 d'API) ; front : 132.

US 5.3, suivi des réservations (05/11/2026, Pull Request #55 du backend) : première lecture par projection SQL, selon l'[ADR 0004](../adr/0004-read-side-strategy.md). 2 tests d'API écrits en premier (comptes justes pour le responsable, `403` pour un spectateur), puis un cycle `red` / `green` : le handler, testé en unitaire (jour du cinéma à 0 h 30 heure de Paris, instant UTC transmis pour l'expiration des blocages), et la projection, testée contre PostgreSQL, comme l'impose l'ADR (ventes en ligne et au guichet, blocage valable et blocage expiré, séance annulée et lendemain exclus). Sabotages : référence de paiement ignorée, expiration ignorée, heure locale au lieu de l'instant UTC, rôle retiré. Total : 175 tests (102 unitaires, 28 d'intégration, 45 d'API) ; front : 136.

Fiche d'un film (`GET /api/movies/{id}`, US 1.1) : même démarche, avec 2 tests d'API écrits en premier, un cycle unitaire sur le handler (film inconnu, séances lues à partir de l'heure locale et places comptées à l'instant UTC, regroupement par jour) et un cycle d'intégration sur les deux lectures (le film par son identifiant, ses séances à venir). Deux sabotages : jours non triés, et `>` à la place de `>=` sur le début de la séance. Le total passe à 85 tests (53 unitaires, 10 d'intégration, 22 d'API).

Ce que la boucle externe a attrapé : le code de l'erreur avait été écrit `Movie-not-found`, avec une majuscule. Les tests unitaires restaient verts, car ils comparent l'objet erreur avec lui-même ; seul le test d'API, qui lit le `type` réellement reçu par le front, l'a détecté. Le rouge initial a aussi montré qu'une route inexistante répond `401`, et non `404`, à un visiteur sans jeton : l'accès est refusé par défaut, y compris hors des routes déclarées.

Les 15 tests d'API ont été écrits après les endpoints qu'ils vérifient : ce sont des tests de contrat. Chacun a été vu rouge en sabotant le code qu'il protège (paramètre `date` ignoré, adresse du compte créé, conflit traduit en `400`, profil ouvert à tous, signature du jeton jamais vérifiée, `UseForwardedHeaders` retiré, adresse de serveur dans le document OpenAPI), puis le sabotage a été annulé. À partir de l'US 2.2, le test d'API d'un endpoint est écrit avant lui.

## Couverture de code

- Mesurée par **Coverlet** (`coverlet.collector`, dans les projets de tests). Les réglages sont dans **`coverlet.runsettings`**, à la racine du dépôt backend, lu à l'identique par les postes et par l'intégration continue : les chiffres sont les mêmes partout.
  - Formats produits : **OpenCover** (lu par SonarCloud) et **Cobertura** (lu par ReportGenerator).
  - **`SkipAutoProps`** : les propriétés automatiques (`public string Title { get; set; }`) ne sont pas comptées. Elles sont écrites par le compilateur et ne contiennent aucune logique ; les compter pousserait à écrire des tests qui affectent puis relisent une valeur, sans rien vérifier.
- Rapport par couche et par classe avec **ReportGenerator**, avec les mêmes exclusions que SonarCloud :

  ```bash
  dotnet tool install --global dotnet-reportgenerator-globaltool
  rm -rf TestResults
  dotnet test --settings coverlet.runsettings --collect:"XPlat Code Coverage" --results-directory TestResults
  reportgenerator -reports:"TestResults/**/coverage.cobertura.xml" -targetdir:"TestResults/coverage-report" \
    -reporttypes:"TextSummary;Html" \
    -filefilters:"-**/Program.cs;-**/Migrations/**;-**/Persistence/Configurations/**;-**/DependencyInjection.cs;-**/obj/**"
  cat TestResults/coverage-report/Summary.txt     # TestResults/ est ignoré par Git
  ```

- Envoyée à SonarCloud à chaque analyse, voir [qualité du code](code-quality.md).
- **Objectif : au moins 80 % sur le code métier** (domaine et cas d'usage), comme fixé au Bloc 2. Le chiffre n'est pas une fin en soi : un pourcentage élevé prouve que le code a été exécuté, pas que les assertions vérifient les bons résultats. Ce qui compte, c'est que chaque règle et ses cas limites soient testés, et que chaque test ait été vu échouer.
- Le code sans logique (configuration EF Core, `Program.cs`, enregistrement des dépendances, migrations générées) n'est pas compté (paramètre `sonar.coverage.exclusions`, et `-filefilters` de ReportGenerator).
- Le code produit à la compilation par les générateurs de source d'ASP.NET Core (fichiers sous `obj/`, par exemple `Microsoft.AspNetCore.OpenApi.Generated`) n'est pas compté non plus (`-**/obj/**`) : SonarCloud ne l'analyse pas, et le compter faisait tomber la couche API de 99,3 % à 55,5 % sans qu'aucune ligne écrite à la main ne soit en cause.

### Mesure du 25/10/2026

| Couche | Lignes | Lecture |
|---|---|---|
| Domaine | aucune ligne à couvrir | entités sans règle pour l'instant (propriétés seules). Le projet `GrandCinema.Domain.Tests` est gardé, vide : il accueillera les premières règles du domaine (blocage et expiration des places, US 2.3) |
| Cas d'usage (`GrandCinema.Application`) | **100 %** | toutes les classes à 100 % |
| Infrastructure | 96,1 % | seul `GrandCinemaDbContext` (69 %) : ses ensembles des places et des réservations, pas encore utilisés (US 2.2 et 2.3) |
| **Total mesuré** | **98,5 % des lignes, 97,7 % des branches** (43 sur 44) | 47 tests |

Démarche suivie pour y arriver, à partir de 87,7 % (et 51 % sur le domaine, dû aux seules propriétés) :

1. Mesure par couche et par classe, avant d'écrire un test, pour viser les règles réellement non vérifiées.
2. `SkipAutoProps` (ci-dessus).
3. Tests ajoutés sur des règles non vérifiées, chacun cassé volontairement pour prouver qu'il échoue sans la règle :
   - connexion : adresse vide ou blanche, mot de passe vide, refusés **sans interroger la base** (vérifié par `Verify(..., Times.Never)` : sans lui, le résultat serait le même et le test ne prouverait rien) ;
   - inscription : chaque champ invalide seul (dont le nom de famille), les longueurs maximales dépassées (adresse 256, mot de passe 128, prénom et nom 100) et acceptées exactement à la limite (un `>=` à la place du `>` est détecté) ;
   - configuration JWT : valeur manquante nommée dans l'erreur, clé de signature de 31 octets refusée et de 32 acceptée, durée de 60 minutes par défaut.
4. Code mort supprimé plutôt que testé : `Error.Forbidden`, jamais appelée (le refus 403 d'un utilisateur sans le bon rôle vient de l'autorisation d'ASP.NET Core, avant le handler).

Reste alors : la couche API, sans tests ; comblé le 28/10/2026 (ci-dessous).

### Mesure du 28/10/2026

Après les tests d'API (journal du backend, étape 13) :

| Couche | Lignes | Lecture |
|---|---|---|
| API (`GrandCinema.Api`) | **99,3 %** | absente des mesures précédentes (aucun test ne la chargeait) ; toutes les classes à 100 % sauf `ErrorMappings` (93,7 %) : la traduction d'une erreur inconnue en `500`, qu'aucun cas métier ne produit |
| Cas d'usage (`GrandCinema.Application`) | **100 %** | inchangé |
| Infrastructure | 96,1 % | inchangé (`GrandCinemaDbContext`, voir ci-dessus) |
| **Total mesuré** | **98,8 % des lignes**, 90,1 % des branches (64 sur 71) | 62 tests : 43 unitaires, 4 d'intégration, 15 d'API |

Le taux de branches baisse parce que celles de l'API sont désormais comptées. **SonarCloud, branche `dev` : 97,2 %** sur tout le code (77,6 % avant), 0 problème, notes A.

## Environnements de test

| Environnement | Tests exécutés | Données |
|---|---|---|
| Poste du développeur | unitaires à chaque modification, tous avant de pousser | base de développement + données de démonstration ; base jetable pour l'intégration |
| Intégration continue (GitHub Actions) | tous les tests automatisés, formatage, couverture, analyse SonarCloud, à chaque Pull Request ; fusion bloquée si un contrôle est rouge | bases jetables, recréées à chaque exécution |
| Recette (AWS) | après chaque déploiement automatique (fusion dans `dev`) : test de fumée `/health` par CloudFront, automatique ; programmation et connexion vérifiées à la main ; tests de bout en bout _(à venir)_ ; recette métier au fil du sprint | données réalistes de démonstration, jamais de données réelles |
| Préproduction (décrite, non montée pour le projet) | avant chaque release : scan OWASP ZAP, test de charge JMeter, migration rejouée | données anonymisées, au volume de la production |

Ces environnements sont ceux du Bloc 2. Pour le projet, seule la recette est en ligne : c'est l'adresse donnée au jury ; la préproduction et la production sont décrites dans [environnements](../deployment/environments.md).

## Anomalies

- Chaque anomalie est une issue GitHub étiquetée `bug`, rattachée au Project, avec sa gravité :

  | Gravité | Définition | Traitement |
  |---|---|---|
  | Bloquante | empêche le parcours principal ou met en danger les données, sans contournement (double vente, faille de sécurité) | correction immédiate ; aucune mise en production tant qu'elle est ouverte |
  | Majeure | fonction importante dégradée, avec contournement | corrigée dans le sprint ; la story ne peut pas passer « Terminé » |
  | Mineure | défaut d'affichage ou de confort | ajoutée au backlog, priorisée par le Product Owner |

- L'issue est ouverte dans le dépôt où se trouve la cause du bug (backend ou frontend) : le correctif, sa Pull Request et l'issue sont au même endroit, et la Pull Request la ferme avec `Fixes #<numéro>`. Les US et les epics, elles, sont dans le dépôt de documentation. Exemples : les issues #57 à #61 du backend (double vente de la dernière place, limite de 10 places contournée avec plusieurs onglets, séances déjà commencées proposées, erreur 500 sur une heure avec fuseau, carrousel vide le soir).
- Une anomalie corrigée commence par un test qui la reproduit (rouge), puis le correctif (vert) : le test empêche qu'elle revienne.
- Les anomalies analysées en détail sont documentées dans [`bugs/`](bugs/).
