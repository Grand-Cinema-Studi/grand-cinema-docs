# Qualité du code

Les contrôles automatiques de la qualité du code, et ce qu'on en fait.

Les éléments marqués _(à venir)_ sont décidés mais pas encore réalisés.

## Outil retenu : SonarCloud

SonarCloud (désormais « SonarQube Cloud ») est la version hébergée de SonarQube (outil cité dans la stratégie de tests du Bloc 2). En place sur le backend depuis le 25/10/2026 ; frontend _(à venir)_.

| Critère | Pourquoi SonarCloud |
|---|---|
| Coût | gratuit pour un dépôt public (plan Free) |
| Langages | C# et TypeScript : un seul outil pour le backend et le frontend |
| Intégration | résultat affiché dans chaque Pull Request GitHub ; bloque la fusion |
| Contenu | bugs, vulnérabilités, points sensibles de sécurité (*security hotspots*), mauvaises pratiques (*code smells*), duplication, couverture des tests |
| Installation | aucun serveur à héberger, contrairement à SonarQube |

## Branchement

L'analyse automatique de SonarCloud ne prend pas en charge le C# : elle est désactivée, et l'analyse est lancée par l'intégration continue (GitHub Actions, `.github/workflows/ci.yml`, job `build-and-test`), avec le scanner .NET.

1. Organisation GitHub `Grand-Cinema-Studi` importée dans SonarCloud (organisation `grand-cinema-studi`), projet `Grand-Cinema-Studi_grand-cinema-backend` ; analyse automatique désactivée.
2. Jeton d'analyse du projet enregistré comme secret du dépôt GitHub (`SONAR_TOKEN`) : jamais dans le code.
3. Ordre des étapes du job :

   ```bash
   dotnet restore
   dotnet format --verify-no-changes --no-restore      # avant le scanner : seulement les règles du projet
   dotnet tool install --global dotnet-sonarscanner
   dotnet sonarscanner begin /k:"Grand-Cinema-Studi_grand-cinema-backend" /o:"grand-cinema-studi" \
     /d:sonar.host.url="https://sonarcloud.io" /d:sonar.token="$SONAR_TOKEN" \
     /d:sonar.qualitygate.wait=true \
     /d:sonar.cs.opencover.reportsPaths="TestResults/**/coverage.opencover.xml" \
     /d:sonar.exclusions="**/Migrations/**,**/*.sql" \
     /d:sonar.coverage.exclusions="**/Program.cs,**/Migrations/**,**/Persistence/Configurations/**,**/DependencyInjection.cs,scripts/**"
   dotnet build --no-restore --configuration Release
   dotnet test --no-build --configuration Release --settings coverlet.runsettings --collect:"XPlat Code Coverage" --results-directory TestResults
   dotnet sonarscanner end /d:sonar.token="$SONAR_TOKEN"
   ```

   - Coverlet produit la couverture au format OpenCover, que SonarCloud lit pour le C# ; ses réglages (formats, propriétés automatiques non comptées) sont dans `coverlet.runsettings`, partagé avec les postes ([stratégie de tests](testing-strategy.md#couverture-de-code)) ; `--results-directory` réunit les rapports des projets de tests au même endroit.
   - `dotnet format` passe **avant** `sonarscanner begin` : le scanner ajoute ses propres analyseurs au build, et `dotnet format` les prendrait pour des écarts du projet (échec rencontré sur la règle S6966).
   - Exclusions de l'analyse : le code généré (`**/Migrations/**`) et les scripts SQL de démonstration (`**/*.sql`), que SonarCloud prenait pour du PL/SQL Oracle alors qu'ils sont écrits pour PostgreSQL. Exclusions de la couverture : le code sans logique (`Program.cs`, configurations EF Core, enregistrement des dépendances) et les scripts d'outillage lancés à la main (`scripts/**`).
   - `sonar.qualitygate.wait=true` : le job échoue quand le Quality Gate est rouge.
4. Déclenchement : chaque Pull Request vers `dev` ou `main`, et chaque push sur `dev`.
5. **Branche principale SonarCloud : `dev`.** Le plan Free n'analyse que la branche principale et les Pull Requests ; le premier push sur `dev` échouait (« Organization is not allowed to access data from non main branches »). `dev` étant la branche d'intégration et celle déployée en recette, c'est elle qui sert de référence au nouveau code ; les Pull Requests vers `main` restent analysées.
6. Contrôles `build-and-test` et « SonarCloud Code Analysis » obligatoires dans le ruleset `protect-main-and-dev` (voir le [workflow Git](../development/git-workflow.md#protections)) : la fusion est techniquement impossible tant qu'ils sont rouges (critère de la Definition of Done du Bloc 2).

Même démarche pour le frontend (dépôt `grand-cinema-frontend`) : action `SonarSource/sonarqube-scan-action` épinglée sur un commit, réglages dans `sonar-project.properties`, couverture LCOV produite par Vitest, branche principale `dev`, analyse automatique désactivée, mêmes contrôles obligatoires dans les rulesets. Première analyse (27/10/2026) : 0 problème, notes A, 98,4 % de couverture.

## Quality gate

Une Pull Request ne peut être fusionnée que si le *quality gate* passe. On garde celui par défaut de SonarCloud (« Sonar way »), qui juge le **nouveau code** :

| Condition sur le nouveau code | Seuil |
|---|---|
| Couverture par les tests | au moins 80 % |
| Lignes dupliquées | au plus 3 % |
| Fiabilité (bugs) | note A |
| Sécurité (vulnérabilités) | note A |
| Points sensibles de sécurité | tous examinés |
| Maintenabilité | note A |

Juger le nouveau code plutôt que le total évite deux écueils : bloquer le projet pour une dette ancienne, ou laisser la qualité se dégrader parce que la moyenne reste bonne. Le seuil de 80 % est celui de la [stratégie de tests](testing-strategy.md#couverture-de-code).

## Traitement des problèmes relevés

- **Bug ou vulnérabilité** : corrigé avant la fusion, avec un test qui le reproduit quand le code est testable (un script de déploiement se vérifie à l'exécution).
- **Point sensible de sécurité** : examiné, puis corrigé ou marqué « sûr » avec la raison.
- **Mauvaise pratique** : corrigée si elle gêne la lecture ou l'évolution ; sinon, acceptée ou désactivée avec la raison écrite. Un problème n'est jamais masqué sans explication.

Chaque correction issue d'une analyse est consignée ci-dessous (règle, fichier, problème, décision, Pull Request), pour montrer la démarche de bout en bout.

## Journal des analyses

### Première analyse complète de `dev` (25/10/2026)

| Mesure (tout le code) | Avant | Après correction |
|---|---|---|
| Problèmes | 9 (1 sécurité, 4 fiabilité, 4 maintenabilité) | 0 |
| Note de sécurité | C | A |
| Notes de fiabilité et de maintenabilité | A | A |
| Points sensibles de sécurité | 0 | 0 |
| Duplication | 0,0 % | 0,0 % |
| Couverture | 67,6 % | 72,3 % |
| Avertissement d'analyse | scripts SQL analysés comme du PL/SQL | aucun |

| # | Règle | Fichier | Problème | Décision et correction | Pull Request |
|---|---|---|---|---|---|
| 1 | Shell : utiliser `[[` plutôt que `[` (fiabilité, élevée) | `scripts/reset-demo-screenings.sh`, lignes 18, 29 et 35 | `[` est une commande externe : une variable vide ou contenant des espaces peut fausser le test | **Corrigé** : `[[ … ]]`, syntaxe interne de bash, qui ne découpe pas les variables (3 occurrences) | `fix/sonarcloud-first-report` |
| 2 | Sécurité : HTTPS non imposé, redirection possible vers un site non sûr (moyenne) | `deploy/deploy.sh`, ligne 19 | `curl -L` suivrait une redirection vers une adresse `http://` : un attaquant placé sur le réseau pourrait fournir un faux fichier Compose, exécuté en `root` | **Corrigé** : `curl --proto '=https' --tlsv1.2` (aucun autre protocole, même après redirection ; versions anciennes de TLS refusées) | `fix/sonarcloud-first-report` |
| 3 | S6966 : attendre `RunAsync` (fiabilité, moyenne) | `src/GrandCinema.Api/Program.cs`, dernière ligne | `app.Run()` bloque le thread en attendant la fin de l'application | **Corrigé** : `await app.RunAsync();` | `fix/sonarcloud-first-report` |
| 4 | CA1861 : tableau constant passé en argument (maintenabilité, moyenne) | 4 tests : `RegisterUserHandlerTests`, `GetMoviesShowingHandlerTests`, `GetScheduleHandlerTests`, `MovieReadRepositoryTests` | la règle demande un champ `static readonly` pour ne pas recréer le tableau à chaque appel | **Désactivé pour les tests seulement** (`.editorconfig`, section `[tests/**.cs]`) : dans un test, exécuté une fois, le gain est nul, et un champ éloignerait la valeur attendue de l'assertion. Le code de production reste soumis à la règle | `fix/sonarcloud-first-report` |
| 5 | Avertissement : dictionnaire de données PL/SQL non configuré | `docs/database/*.sql` | scripts PostgreSQL analysés avec les règles d'Oracle | **Exclus de l'analyse** (`**/*.sql`) : scripts de données de démonstration, vérifiés à chaque chargement (`ON_ERROR_STOP`) | `fix/sonarcloud-first-report` |

Couverture : 72,3 % sur tout le code, sous l'objectif de 80 % de la stratégie de tests, qui porte sur le code métier (domaine et cas d'usage). Les endpoints HTTP n'avaient pas encore de tests d'API : comblé le 28/10/2026 (voir « Tests de l'API »).

### Pull Request du déploiement continu (25/10/2026)

SonarCloud analyse aussi les workflows GitHub Actions. Sur la Pull Request `chore/continuous-deployment`, le Quality Gate a échoué (« C Security Rating on New Code ») : la fusion a été bloquée jusqu'à la correction.

| # | Règle | Fichier | Problème | Décision et correction | Pull Request |
|---|---|---|---|---|---|
| 6 | Sécurité : épingler une dépendance sur l'empreinte complète d'un commit (échec) | `.github/workflows/deploy.yml`, `docker/login-action` et `aws-actions/configure-aws-credentials` | une étiquette comme `@v3` est mobile : déplacée vers un autre code par son propriétaire ou par un attaquant ayant pris son dépôt, elle s'exécuterait avec les permissions du job (publication d'images, accès AWS) | **Corrigé** : actions tierces épinglées sur l'empreinte du commit, version en commentaire (`@81c6f93… # v4.7.0`) ; Dependabot met à jour l'empreinte et le commentaire | `chore/continuous-deployment` |
| 7 | Sécurité : permission d'écriture déclarée pour tout le workflow (avertissement, 2 occurrences) | `.github/workflows/deploy.yml`, `id-token: write` et `packages: write` | le job de tests recevait des droits dont il n'a pas besoin | **Corrigé** : `permissions: {}` pour le workflow, droits déclarés par job (moindre privilège) ; la dernière étape passe aussi l'adresse de l'API par `env:` plutôt que par une expression dans le script (injection de script) | `chore/continuous-deployment` |

### Pull Request de publication du script TMDB

Le script qui récupère les affiches, dates de sortie et durées sur TMDB (`scripts/fetch-tmdb-posters.mjs`, Node) était conservé hors du dépôt ; il a été publié, puisqu'il ne contient aucun secret (le jeton est lu dans `.env`, ignoré par Git). Le Quality Gate a bloqué la fusion jusqu'aux deux corrections.

| # | Règle | Fichier | Problème | Décision et correction | Pull Request |
|---|---|---|---|---|---|
| 8 | Couverture du nouveau code inférieure à 80 % (Quality Gate) | `scripts/fetch-tmdb-posters.mjs` | 0 % : script d'outillage, lancé à la main une fois par jeu de données, sans test | **Exclu de la couverture** (`scripts/**` dans `sonar.coverage.exclusions`) : il ne fait pas partie de l'application livrée et son résultat (`seed-posters.sql`, 125 films sur 125) est vérifié à chaque chargement ; il reste analysé pour ses problèmes | publication du script TMDB |
| 9 | Expression régulière exposée à un temps d'exécution excessif (retour arrière) | `scripts/fetch-tmdb-posters.mjs`, lecture des lignes `-- tmdb:` | une seule expression avec plusieurs parties variables peut essayer un très grand nombre de découpages sur une ligne mal formée | **Corrigé** : lecture réécrite en étapes simples (`parseMovieLine` : repérer la balise, puis l'année entre parenthèses), vérifiée sur les mêmes 125 résultats | publication du script TMDB |

### Couverture du code métier (25/10/2026)

Branche `test/business-code-coverage`. Mesure par couche avec ReportGenerator, avec les exclusions de SonarCloud : 87,7 % au départ (domaine 51 %, uniquement des propriétés), **98,5 % des lignes et 97,7 % des branches** à l'arrivée ; cas d'usage à **100 %**. Détail de la démarche dans la [stratégie de tests](testing-strategy.md#mesure-du-25102026).

| Décision | Pourquoi |
|---|---|
| Propriétés automatiques non comptées (`SkipAutoProps`, `coverlet.runsettings` partagé par les postes et la CI) | aucune logique à tester ; les compter pousserait à des tests sans assertion utile |
| Tests ajoutés sur les règles non vérifiées (connexion vide, longueurs d'inscription, configuration JWT), chacun cassé volontairement avant d'être gardé | la couverture sert à trouver les règles non testées, pas à atteindre un chiffre |
| `Error.Forbidden` supprimée (méthode, valeur `ErrorType.Forbidden` et traduction en 403) | jamais appelée : un test sur du code mort ne prouve rien ; le 403 vient de l'autorisation d'ASP.NET Core |
| Dossier `Features/Movies/DTOS` renommé `DTOs` dans Git | homogène avec les autres fonctionnalités ; Windows ne voit pas la différence de casse, Linux (CI, serveur) oui |

La couche API n'avait pas encore de tests : SonarCloud la comptait dans le total, d'où un chiffre global inférieur à celui du code métier. Comblé le 28/10/2026 (ci-dessous).

### Tests de l'API (28/10/2026)

Branches `test/api-endpoints` (15 tests d'API, `WebApplicationFactory` et Testcontainers, détail dans la [stratégie de tests](testing-strategy.md#mesure-du-28102026)) puis `test/serializable-theory-data`. Aucun code de production modifié.

| Mesure (tout le code, branche `dev`) | Avant | Après |
|---|---|---|
| Couverture | 77,6 % | **97,2 %** |
| Problèmes | 1 (maintenabilité) | 0 |
| Notes de sécurité, fiabilité, maintenabilité | A | A |
| Points sensibles de sécurité | 0 | 0 |

| # | Règle | Fichier | Problème | Décision et correction | Pull Request |
|---|---|---|---|---|---|
| 10 | xUnit1045 : donnée de théorie non sérialisable (maintenabilité, moyenne) | `tests/GrandCinema.Application.Tests/Features/Auth/Commands/RegisterUserHandlerTests.cs` | la théorie recevait un `RegisterUserCommand` : ses 5 cas s'exécutaient, mais le Test Explorer les affichait comme un seul test | **Corrigé** : la théorie reçoit des chaînes (`TheoryData<string, string, string, string, string>`) et construit la commande dans le test ; 5 cas listés séparément | `test/serializable-theory-data` |

La Pull Request des tests d'API affichait « 0,0 % Coverage on New Code » avec un Quality Gate vert : elle n'ajoute que du code de tests, qui n'est pas lui-même mesuré ; il n'y a donc aucune ligne nouvelle à couvrir. Pour une comparaison locale, ReportGenerator exclut aussi le code généré sous `obj/`, que SonarCloud n'analyse pas.

## Autres contrôles

| Contrôle | Ce qu'il apporte | État |
|---|---|---|
| Compilateur C#, nullable activé | erreurs de type et risques de `null` détectés à la compilation | Fait |
| Revue de chaque Pull Request | relecture du diff selon les [standards de code](../development/coding-standards.md#revue-de-code) | Fait |
| `dotnet format --verify-no-changes` | formatage identique sur tous les postes, vérifié par la CI à chaque Pull Request ; mise en forme initiale de tout le code dans un commit `style:` séparé | Fait (25/10/2026) |
| `.editorconfig` | réglages des règles par dossier (CA1861 désactivée dans les tests, avec la raison) | Fait (25/10/2026) |
| Dependabot | alerte, et propose une mise à jour, quand une dépendance (paquet NuGet, action GitHub, image Docker) a une faille connue ou une nouvelle version | Fait (25/10/2026) |
