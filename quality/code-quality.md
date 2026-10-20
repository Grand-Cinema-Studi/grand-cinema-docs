# Qualité du code

Les contrôles automatiques de la qualité du code, et ce qu'on en fait.

Les éléments marqués _(à venir)_ sont décidés mais pas encore réalisés.

## Outil retenu : SonarCloud _(à venir)_

SonarCloud est la version hébergée de SonarQube (outil cité dans la stratégie de tests du Bloc 2).

| Critère | Pourquoi SonarCloud |
|---|---|
| Coût | gratuit pour un dépôt public |
| Langages | C# et TypeScript : un seul outil pour le backend et le frontend |
| Intégration | résultat affiché dans chaque Pull Request GitHub ; peut bloquer la fusion |
| Contenu | bugs, vulnérabilités, points sensibles de sécurité (*security hotspots*), mauvaises pratiques (*code smells*), duplication, couverture des tests |
| Installation | aucun serveur à héberger, contrairement à SonarQube |

## Branchement

L'analyse automatique de SonarCloud ne prend pas en charge le C# : l'analyse est lancée par l'intégration continue (GitHub Actions), avec le scanner .NET.

1. Importer l'organisation GitHub `Grand-Cinema-Studi` dans SonarCloud, puis le dépôt backend ; désactiver l'analyse automatique.
2. Créer un jeton SonarCloud et l'enregistrer comme secret du dépôt GitHub (`SONAR_TOKEN`) : jamais dans le code.
3. Dans le workflow d'intégration continue, encadrer la compilation et les tests par le scanner :

   ```bash
   dotnet tool install --global dotnet-sonarscanner
   dotnet sonarscanner begin /k:"<clé-du-projet>" /o:"grand-cinema-studi" \
     /d:sonar.token="$SONAR_TOKEN" \
     /d:sonar.cs.opencover.reportsPaths="tests/**/TestResults/**/coverage.opencover.xml"
   dotnet build --configuration Release
   dotnet test --configuration Release --no-build --collect:"XPlat Code Coverage;Format=opencover"
   dotnet sonarscanner end /d:sonar.token="$SONAR_TOKEN"
   ```

   Coverlet produit la couverture au format OpenCover, que SonarCloud lit pour le C#.
4. Exclure de l'analyse le code généré (`sonar.exclusions` : `**/Migrations/**`), et de la couverture le code sans logique (`sonar.coverage.exclusions` : `**/Program.cs`, `**/Migrations/**`, `**/Persistence/Configurations/**`, `**/DependencyInjection.cs`). Ajouter `sonar.qualitygate.wait=true` : l'intégration continue échoue quand le Quality Gate est rouge, donc la fusion est bloquée (critère de la Definition of Done du Bloc 2).
5. Rendre la vérification SonarCloud obligatoire dans le ruleset de `main` et `dev` (voir le [workflow Git](../development/git-workflow.md#protections)).

Même démarche pour le frontend, avec le scanner adapté et la couverture au format LCOV.

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

- **Bug ou vulnérabilité** : corrigé avant la fusion, avec un test qui le reproduit.
- **Point sensible de sécurité** : examiné, puis corrigé ou marqué « sûr » avec la raison.
- **Mauvaise pratique** : corrigée si elle gêne la lecture ou l'évolution ; sinon, marquée « acceptée » avec la raison. Un problème n'est jamais masqué sans explication.

À partir de la première analyse, chaque correction qui en découle est consignée dans ce document (règle SonarCloud, fichier, problème, correction, Pull Request), pour montrer la démarche de bout en bout.

## Autres contrôles

| Contrôle | Ce qu'il apporte | État |
|---|---|---|
| Compilateur C#, nullable activé | erreurs de type et risques de `null` détectés à la compilation | ✅ |
| Revue de chaque Pull Request | relecture du diff selon les [standards de code](../development/coding-standards.md#revue-de-code) | ✅ |
| `dotnet format --verify-no-changes` | formatage identique sur tous les postes | _(à venir)_ |
| Dependabot | alerte, et propose une mise à jour, quand une bibliothèque utilisée a une faille connue | _(à venir)_ |
