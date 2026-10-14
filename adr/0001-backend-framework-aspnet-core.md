# ADR 0001 – Framework backend : ASP.NET Core

- Statut : Accepté
- Concerne : backend
- Liens : [grand-cinema-backend](https://github.com/Grand-Cinema-Studi/grand-cinema-backend)

## Contexte

L'API du Grand Cinéma doit :

- servir la consultation de la programmation à un public large, sans compte ;
- vendre des places **sans double vente**, y compris quand plusieurs spectateurs visent la dernière place en même temps : transactions et verrous en base, contrainte d'unicité ;
- absorber des **pics courts et concentrés** pendant le Festival de Jazz (800 à 1 600 spectateurs par jour hors festival) ;
- tenir dans un **budget limité**, aussi bien en licences qu'en hébergement (petite machine AWS) ;
- être développée par une petite équipe, en TDD, avec une architecture modulaire et testable (Bloc 1 et stratégie de tests du Bloc 2) ;
- être documentée (OpenAPI / Swagger) et utiliser PostgreSQL, base retenue au Bloc 1.

Critères de comparaison imposés : performance, coût, scalabilité, facilité de développement.

## Décision

Nous utilisons **ASP.NET Core** (.NET 10, C#) en **Minimal API**, avec **EF Core** et le fournisseur **Npgsql** pour PostgreSQL.

## Comparaison

Alternatives étudiées : **NestJS** (Node.js, TypeScript) et **Spring Boot** (Java).

| Critère | ASP.NET Core | NestJS (Node.js) | Spring Boot (Java) |
|---|---|---|---|
| **Performance** | Compilé, multithread, entrées-sorties asynchrones (`async`/`await`) de bout en bout ; parmi les premiers frameworks généralistes des benchmarks TechEmpower | Très bon en entrées-sorties grâce à la boucle d'événements, mais un seul thread par processus : un calcul lourd bloque les autres requêtes | Très performant une fois la JVM chaude ; démarrage plus lent et consommation mémoire plus élevée |
| **Coût** | Gratuit et open source (licence MIT) ; outils gratuits (SDK, CLI, VS Code) ; empreinte mémoire modérée, compatible avec une petite instance | Gratuit et open source (MIT) ; empreinte mémoire faible | Gratuit et open source (Apache 2.0) ; la mémoire de la JVM pousse vers des instances plus grosses, donc plus chères |
| **Scalabilité** | API sans état : on ajoute des instances derrière un répartiteur de charge ; une instance utilise tous ses cœurs | Sans état également ; il faut un processus par cœur (mode cluster, PM2) pour exploiter une machine | Sans état également ; écosystème complet pour les gros systèmes, surdimensionné ici |
| **Facilité de développement** | Tout est inclus : injection de dépendances, configuration, secrets de développement, OpenAPI, health checks, `TimeProvider` pour tester les dates ; typage statique fort ; EF Core gère les migrations, les transactions et les types PostgreSQL ; tests avec xUnit | Même langage que le front Angular (TypeScript), structure inspirée d'Angular ; mais ORM et outils à choisir et assembler (TypeORM, Prisma…), typage effacé à l'exécution | Très complet et très répandu en entreprise ; plus verbeux et plus de configuration pour un projet de cette taille |

## Raisons du choix

1. **La règle la plus critique, la non-double vente, repose sur la base** : EF Core et Npgsql donnent un accès direct aux transactions, aux verrous de ligne et aux contraintes PostgreSQL, et les migrations versionnent le schéma avec le code.
2. **Testabilité** : l'injection de dépendances intégrée et `TimeProvider` permettent de tester chaque cas d'usage sans base ni date réelle, comme l'exige la stratégie TDD du Bloc 2.
3. **Budget** : aucune licence, et une API compacte qui tient sur la petite instance prévue pour le MVP.
4. **Moins de dépendances tierces** : documentation OpenAPI, configuration, secrets et health checks sont fournis par le framework. Moins de bibliothèques, c'est moins de risques de licence et de maintenance (voir « À surveiller »).

## Alternatives écartées

- **NestJS** : son principal atout, un seul langage pour le front et le back, ne compense pas l'assemblage d'outils tiers pour l'accès aux données et les migrations, sur la partie la plus risquée du projet (la réservation concurrente).
- **Spring Boot** : techniquement adapté, mais plus lourd à configurer et plus gourmand en mémoire, ce qui pèse sur le budget d'hébergement, sans bénéfice à l'échelle du Grand Cinéma.

## Conséquences

- Positives : un cadre complet et cohérent ; une architecture en couches (Clean Architecture) facile à mettre en place ; Swagger généré depuis le code ; le même conteneur Docker en local, en intégration continue et sur AWS.
- Négatives : deux langages dans le projet (C# pour l'API, TypeScript pour le front Angular) ; pas de partage direct des types entre front et back (à compenser en générant les types du front depuis le document OpenAPI si besoin).
- À surveiller : les licences de l'écosystème .NET. Plusieurs bibliothèques populaires sont passées sous licence commerciale (MediatR, AutoMapper, FluentAssertions 8) ; elles ne sont pas utilisées. Vérifier la licence de toute nouvelle dépendance avant de l'ajouter.

## Sources à joindre avant le dépôt

- Classement TechEmpower (dernier round publié) : position d'ASP.NET Core, de Node.js / NestJS et de Spring dans les tests JSON et requêtes base de données ;
- licences : pages officielles .NET (MIT), NestJS (MIT), Spring Boot (Apache 2.0) ;
- licences commerciales de MediatR, AutoMapper et FluentAssertions : pages des éditeurs, consultées au moment du dépôt.
