# Standards de code

Règles d'écriture du code, appliquées en revue de Pull Request. Elles servent un objectif : qu'un développeur qui arrive sur le projet retrouve partout la même structure et les mêmes noms.

Les éléments marqués _(à venir)_ sont décidés mais pas encore réalisés.

## Langue

- **Tout ce qui est nommé dans le code est en anglais** : projets, espaces de noms, dossiers, classes, méthodes, tests, tables et colonnes, branches, messages de commit. L'anglais est la langue de l'écosystème (bibliothèques, documentation, messages d'erreur), et une seule langue évite les noms hybrides comme `GetSeancesByDate`.
- **Le français** est réservé aux textes affichés à l'utilisateur, aux données d'exemple et à la documentation.
- Le lien entre le vocabulaire métier et les noms du code est tenu dans le [glossaire](../architecture/glossary.md).

## Backend (C#)

### Structure

| Couche | Rangement | Exemple |
|---|---|---|
| Domain | par type d'élément | `Entities/Screening.cs`, `Enums/ReservationStatus.cs` |
| Application | par fonctionnalité, puis `Queries`, `Commands`, `DTOs` | `Features/Screenings/Queries/GetScheduleHandler.cs` |
| Application, contrats partagés | `Abstractions/` | `Abstractions/Persistence/IScreeningReadRepository.cs` |
| Infrastructure | par responsabilité technique | `Persistence/Configurations/`, `Persistence/Repositories/` |
| Api | par la même fonctionnalité que l'Application | `Endpoints/Screenings/ScheduleEndpoints.cs` |

- Ajouter une user story ne touche, en principe, qu'un dossier de fonctionnalité par couche.
- Pas de dossier vide créé « pour plus tard » : `Commands/` apparaît avec la première commande.
- Les dépendances vont vers le centre (voir l'[architecture](../architecture/overview.md#architecture-du-backend)) : une classe de Domain ou d'Application ne référence jamais EF Core ni ASP.NET Core.

### Nommage

| Élément | Règle | Exemple |
|---|---|---|
| Types, méthodes, propriétés | PascalCase | `GetScheduleHandler`, `StartsAt` |
| Paramètres, variables locales | camelCase | `cancellationToken`, `screenings` |
| Champs privés | `_camelCase` | `_timeProvider` |
| Interfaces | préfixe `I` | `IQueryHandler` |
| Méthodes asynchrones | suffixe `Async` | `GetScreeningsAsync` ; exception : `Handle`, imposé par les interfaces CQRS |
| Requête, commande et leur handler | `<Verbe><Objet>Query` / `Command` / `Handler` | `GetScheduleQuery`, `GetScheduleHandler` |
| Repository | `I<Entité>ReadRepository` en lecture seule, `I<Entité>Repository` sinon | `IScreeningReadRepository` |
| DTO | suffixe `Dto` | `ScheduleDto` |
| Tests | `When_<situation>_Then_<résultat attendu>` | `When_no_screening_exists_Then_the_movie_list_is_empty` |

### Écriture

- **Nullable activé** dans tous les projets : le compilateur signale les risques de `null`.
- Classes `sealed` par défaut (handlers, configurations, repositories) : une classe n'est ouverte à l'héritage que si c'est voulu.
- DTO en `record class` avec des propriétés `required` et `init` : immuables, et un champ oublié est une erreur de compilation.
- Conversion entité → DTO écrite à la main, dans des méthodes d'extension (`DTOs/Mappings/`), sans AutoMapper : explicite et vérifiée à la compilation.
- Dépendances reçues par le constructeur (injection de dépendances), enregistrées par couche (`AddApplication`, `AddInfrastructure`).
- Date et heure courantes lues par `TimeProvider`, jamais par `DateTime.Now` : les tests peuvent fixer l'heure.
- Endpoints : réponses typées (`TypedResults`), et chaque endpoint porte un nom, une étiquette, un résumé et une description pour Swagger (voir les [conventions de l'API](../api/api-conventions.md)).
- Toute méthode asynchrone accepte et transmet un `CancellationToken`.
- Aucun secret, mot de passe ni adresse de serveur dans le code (voir la [sécurité](../security/security.md)).

### Commentaires

Le code doit se lire sans commentaire : noms explicites, méthodes courtes. Un commentaire explique **pourquoi**, jamais ce que fait la ligne suivante. Les commentaires du code sont en anglais.

## Frontend (Angular) _(à venir)_

- Guide de style officiel d'Angular : un composant par fichier, noms en kebab-case (`schedule-page.component.ts`), rangement par fonctionnalité.
- TypeScript en mode strict.
- Appels à l'API regroupés dans des services, typés à partir du contrat OpenAPI.

## Formatage et analyse automatique

| Outil | Rôle | État |
|---|---|---|
| Compilateur C# (avertissements, nullable) | erreurs de typage, `null` possible | Fait |
| `.editorconfig` et `dotnet format --verify-no-changes` | style et formatage identiques sur tous les postes, vérifiés en intégration continue | _(à venir)_ |
| ESLint et Prettier | équivalent côté front | _(à venir)_ |
| SonarCloud | bugs, vulnérabilités, duplication, couverture | _(à venir)_, voir [qualité du code](../quality/code-quality.md) |

La [Definition of Done](https://github.com/Grand-Cinema-Studi/grand-cinema-docs/issues/24) exige qu'aucun de ces outils ne remonte d'erreur avant la fusion.

## Revue de code

Chaque Pull Request est relue avant fusion (voir le [workflow Git](git-workflow.md#pull-requests)). Points vérifiés :

- chaque critère d'acceptation de la story a son test, et les tests passent ;
- les règles de ce document sont respectées ;
- pas de logique métier dans les endpoints ni dans les repositories ;
- pas de code mort, de code commenté ni de `TODO` sans issue ;
- la documentation concernée est à jour (README, Swagger, ADR si une décision a été prise).
