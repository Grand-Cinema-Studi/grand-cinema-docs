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
| Unitaire | un handler ou une règle métier, isolé : dépendances simulées, aucune base | xUnit, Moq, Shouldly, `FakeTimeProvider` | Fait : 50 tests (US 1.1 : 9 ; US 2.1 : 38 ; US 2.2 : 3) |
| Intégration (accès aux données) | un repository contre un vrai PostgreSQL, avec les vraies migrations | xUnit, Testcontainers (PostgreSQL 16 jetable) | Fait : 8 tests (US 1.1 : 2 ; US 2.1 : 5 ; US 2.2 : 1) |
| Intégration (API) | un endpoint de bout en bout dans l'API : routage, sérialisation, codes HTTP, Problem Details, authentification, limitation des tentatives, base réelle | `WebApplicationFactory`, Testcontainers | Fait : 20 tests (US 1.1 : 3 ; US 2.1 : 14 ; US 2.2 : 1 ; `/health` et document OpenAPI : 2) |
| Concurrence | deux requêtes simultanées sur la dernière place : une seule réussit | Testcontainers, tâches parallèles | _(à venir, US 2.3)_ — voir [bug 0001](bugs/0001-double-booking.md) |
| Composants front | un composant ou un service Angular isolé : réseau remplacé par `HttpTestingController`, service remplacé par une doublure | Vitest (jsdom), outils de test d'Angular | Fait : 78 tests (dépôt `grand-cinema-frontend`) |
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

- Une anomalie corrigée commence par un test qui la reproduit (rouge), puis le correctif (vert) : le test empêche qu'elle revienne.
- Les anomalies analysées en détail sont documentées dans [`bugs/`](bugs/).
