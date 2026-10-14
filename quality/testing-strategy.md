# Stratégie de tests

Comment l'application est testée, à quel niveau, avec quels outils, et comment on le prouve. Cette stratégie applique au code celle définie au Bloc 2.

Les éléments marqués _(à venir)_ sont décidés mais pas encore réalisés.

## Principes

- **Tester en continu**, à chaque commit et à chaque Pull Request, et non en fin de projet : une anomalie trouvée tard coûte plus cher.
- **Pyramide des tests** : beaucoup de tests unitaires rapides, moins de tests d'intégration, peu de tests de bout en bout.
- **TDD sur le code métier** (rouge, vert, refactor) : le test est écrit avant le code qu'il vérifie. C'est là qu'une erreur coûte le plus : programmation, places restantes, blocage et expiration, validation d'un billet.
- **Un critère d'acceptation = au moins un test.** Le nom du test reprend le « Quand / Alors » du critère, et le test porte le numéro de sa story : on retrouve tous les tests d'une story, et la story de chaque test.
- **Non-régression** : tous les tests automatisés sont rejoués à chaque Pull Request ; une nouvelle story ne doit jamais en casser une autre.

## Niveaux de tests

| Niveau | Ce qui est vérifié | Outils | État |
|---|---|---|---|
| Unitaire | un handler ou une règle métier, isolé : dépendances simulées, aucune base | xUnit, Moq, Shouldly, `FakeTimeProvider` | ✅ 6 tests (US 1.1) |
| Intégration (accès aux données) | un repository contre un vrai PostgreSQL, avec les vraies migrations | xUnit, Testcontainers (PostgreSQL 16 jetable) | ✅ 1 test (US 1.1) |
| Intégration (API) | un endpoint de bout en bout dans l'API : routage, sérialisation, codes HTTP, base réelle | `WebApplicationFactory`, Testcontainers | _(à venir)_ |
| Concurrence | deux requêtes simultanées sur la dernière place : une seule réussit | Testcontainers, tâches parallèles | _(à venir, US 2.3)_ — voir [bug 0001](bugs/0001-double-booking.md) |
| Composants front | un composant Angular isolé | outils de test d'Angular CLI | _(à venir)_ |
| Bout en bout | le parcours complet dans un navigateur | à choisir avec le front (Selenium au Bloc 2) | _(à venir)_ |
| Charge | la tenue des pics du festival | JMeter | hors périmètre de l'épreuve ; prévu avant une mise en production réelle |
| Sécurité | failles courantes, dépendances vulnérables | SonarCloud, Dependabot, OWASP ZAP | _(à venir)_ |

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

## Commandes

```bash
dotnet test                                        # tous les tests (Docker doit tourner)
dotnet test --filter "Category!=Integration"       # tests unitaires seuls
dotnet test --filter "US=1.1"                      # tests d'une user story
dotnet test --collect:"XPlat Code Coverage"        # avec mesure de la couverture
```

## Preuve du TDD

L'historique Git montre l'ordre test puis code : pour chaque cycle, un commit `red` (le test, qui échoue), un commit `green` (le code minimal), et si besoin un commit `refactor`. Exemple sur l'US 1.1 :

```text
test(us-1.1): red - group screenings by movie
feat(us-1.1): green - group screenings by movie
refactor(us-1.1): extract movie schedule mapping
```

Les Pull Requests fusionnées en merge commit conservent ces commits (voir le [workflow Git](../development/git-workflow.md#pull-requests)).

## Couverture de code

- Mesurée par **Coverlet** (`coverlet.collector`, déjà dans les projets de tests), qui produit un fichier `coverage.cobertura.xml` par projet de tests.
- Rapport HTML lisible avec **ReportGenerator** :

  ```bash
  dotnet tool install -g dotnet-reportgenerator-globaltool
  reportgenerator -reports:"tests/**/TestResults/**/coverage.cobertura.xml" -targetdir:coverage-report -reporttypes:Html
  ```

- Envoyée à SonarCloud à chaque analyse _(à venir)_, voir [qualité du code](code-quality.md).
- **Objectif : au moins 80 % sur le code métier** (handlers et domaine), comme fixé au Bloc 2. Le chiffre n'est pas une fin en soi : un pourcentage élevé prouve que le code a été exécuté, pas que les assertions vérifient les bons résultats. Ce qui compte, c'est que chaque règle et ses cas limites soient testés.
- Le code sans logique (configuration EF Core, `Program.cs`, migrations générées) n'est pas compté dans l'objectif.

## Environnements de test

| Environnement | Tests exécutés | Données |
|---|---|---|
| Poste du développeur | unitaires à chaque modification, tous avant de pousser | base de développement + données de démonstration ; base jetable pour l'intégration |
| Intégration continue (GitHub Actions) _(à venir)_ | tous les tests automatisés, couverture, analyse SonarCloud, à chaque Pull Request | bases jetables, recréées à chaque exécution |
| Démonstration (AWS) _(à venir)_ | vérification après déploiement (`/health`, programmation), recette manuelle | données de démonstration, jamais de données réelles |

Le Bloc 2 prévoit un environnement de recette distinct de la production ; dans la variante gratuite retenue pour le projet, l'environnement de démonstration joue les deux rôles (voir [environnements](../deployment/environments.md)).

## Anomalies

- Chaque anomalie est une issue GitHub étiquetée `bug`, rattachée au Project, avec sa gravité :

  | Gravité | Définition | Traitement |
  |---|---|---|
  | Bloquante | empêche le parcours principal ou met en danger les données, sans contournement (double vente, faille de sécurité) | correction immédiate ; aucune mise en production tant qu'elle est ouverte |
  | Majeure | fonction importante dégradée, avec contournement | corrigée dans le sprint ; la story ne peut pas passer « Terminé » |
  | Mineure | défaut d'affichage ou de confort | ajoutée au backlog, priorisée par le Product Owner |

- Une anomalie corrigée commence par un test qui la reproduit (rouge), puis le correctif (vert) : le test empêche qu'elle revienne.
- Les anomalies analysées en détail sont documentées dans [`bugs/`](bugs/).
