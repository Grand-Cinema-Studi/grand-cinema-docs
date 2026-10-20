# Workflow Git

Règles communes aux dépôts du projet : branches, protections, Pull Requests et messages de commit. Le `CONTRIBUTING.md` de chaque dépôt de code reprend la convention de nommage des branches, pour qu'elle soit documentée dans le dépôt lui-même ; ce document en donne le détail et les raisons.

Les éléments marqués _(à venir)_ sont décidés mais pas encore réalisés.

## Branches

Modèle inspiré du Git Flow, simplifié pour une équipe réduite :

| Branche | Rôle | Créée depuis | Fusionnée dans |
|---|---|---|---|
| `main` | production : chaque fusion est une version livrable | — | — |
| `dev` | intégration : reçoit toutes les Pull Requests de travail | `main` | `main` |
| `feature/us-<n>-<intitulé>` | une user story | `dev` | `dev` |
| `fix/<intitulé>` | correction d'une anomalie trouvée en cours de développement | `dev` | `dev` |
| `test/<intitulé>` | ajout ou correction de tests seuls | `dev` | `dev` |
| `chore/<intitulé>` | outillage, configuration, dépendances, CI | `dev` | `dev` |
| `docs/<intitulé>` | documentation seule | `dev` | `dev` |
| `hotfix/<intitulé>` | correction urgente de la production | `main` | `main`, puis `dev` |

**`dev` est la branche `develop` du Git Flow** demandée par l'énoncé : même rôle (branche d'intégration protégée, cible de toutes les Pull Requests, seule source de `main` avec les hotfix). Seul le nom est abrégé ; la branche n'est pas renommée.

Pas de branche `release/` : avec une seule version en cours à la fois, la Pull Request `dev` → `main` joue ce rôle.

### Nommage

- Tout en minuscules, mots séparés par des tirets, en anglais comme le code.
- Branche de user story : préfixe + numéro de la story + intitulé court. Exemples : `feature/us-1-1-view-schedule`, `feature/us-2-3-reserve-places` (sur le modèle `feature/login-user` de l'énoncé).
- Une branche = une user story (ou une tâche) = une Pull Request. Une story trop grosse pour une seule Pull Request est livrée en plusieurs branches successives, chacune laissant `dev` compilable et testé (l'US 1.1 l'a été en quatre : logique métier, repository, endpoint, correctif de tri).

## Protections

Dans les dépôts de code, un ruleset GitHub (`protect-main-and-dev`) protège `main` et `dev` :

| Règle | Effet |
|---|---|
| Pull Request obligatoire | aucun push direct ; tout changement est relu et tracé |
| Suppression interdite | les branches stables ne peuvent pas disparaître |
| Force push bloqué | l'historique publié ne peut pas être réécrit |
| Liste d'exceptions (bypass) vide | le propriétaire du dépôt est soumis aux mêmes règles |
| Approbations requises : 0 | GitHub interdit d'approuver sa propre Pull Request : en projet solo, une approbation obligatoire bloquerait toute fusion. En équipe, au moins une approbation serait exigée |
| Vérifications obligatoires (compilation, tests, analyse de qualité) | _(à venir, avec l'intégration continue)_ : une Pull Request rouge ne pourra plus être fusionnée |

La règle a été testée : un push direct sur `dev` est refusé.

Le dépôt de documentation n'a que `main`, sur laquelle arrivent les Pull Requests `docs/<sujet>`.

## Pull Requests

1. Créer la branche depuis `dev` à jour : `git switch dev && git pull && git switch -c feature/us-<n>-<intitulé>`.
2. Commiter par petites étapes (voir les messages ci-dessous), pousser la branche.
3. Ouvrir la Pull Request vers `dev`. Sa description donne :
   - la story ou la tâche concernée (`Closes #<issue>` pour fermer l'issue à la fusion) ;
   - ce qui change, et pourquoi ;
   - comment c'est testé ;
   - la checklist de la [Definition of Done](https://github.com/Grand-Cinema-Studi/grand-cinema-docs/issues/24) applicable ;
   - le lien vers la Pull Request de documentation, si la modification change l'architecture ou prend une décision (nouvel ADR).
4. Relire le diff (onglet *Files changed*) avant de fusionner, et attendre que l'intégration continue soit verte _(à venir)_.
5. Fusionner en **merge commit**, puis supprimer la branche.

**Pourquoi le merge commit** plutôt que le squash ou le rebase : chaque branche reste visible dans le graphe Git, avec tous ses commits. C'est la trace du travail par branche demandé par l'énoncé, et des cycles TDD (commits rouge, vert, refactor). Le squash réduirait la branche à un commit ; le rebase recréerait les commits et perdrait le lien avec la branche.

« Pull Request » (GitHub) et « merge request » (GitLab, terme de l'énoncé) désignent la même chose.

### Livraison en production

Une Pull Request `dev` → `main`, quand un incrément est terminé et que les [critères de mise en production](../deployment/deploy-guide.md#critères-de-mise-en-production) sont remplis. Sa fusion donne lieu à un tag de version (voir [versioning](versioning.md)) ; le déploiement en production attend ensuite l'approbation du Product Owner : c'est la mise en production manuelle du Bloc 2 _(à venir)_.

### Déploiement de la recette

Chaque fusion dans `dev` déploie automatiquement la recette (Bloc 2), front et back chacun par son pipeline _(à venir)_ : voir le [guide de déploiement](../deployment/deploy-guide.md).

### Correctif urgent

Branche `hotfix/<intitulé>` depuis `main`, Pull Request vers `main`, puis report dans `dev` (Pull Request `main` → `dev`) pour que le correctif ne soit pas perdu à la livraison suivante.

## Messages de commit

Format [Conventional Commits](https://www.conventionalcommits.org/fr/) : `<type>(<portée>): <description>`, en anglais, à l'impératif, sans point final.

| Type | Usage | Exemple |
|---|---|---|
| `feat` | nouveau comportement | `feat(us-1.1): add schedule endpoint and Swagger` |
| `fix` | correction d'une anomalie | `fix(us-2.3): reject a seat already reserved` |
| `test` | ajout ou modification de tests | `test(us-1.1): no screening - movie list is empty` |
| `refactor` | restructuration sans changement de comportement | `refactor(us-1.1): extract movie schedule mapping` |
| `docs` | documentation | `docs(adr): add backend framework decision` |
| `chore` | outillage, configuration, dépendances | `chore: add EF Core, PostgreSQL and docker compose` |
| `ci` | intégration et déploiement continus | `ci: run tests on pull requests` |

- **Portée** : le numéro de la story (`us-1.1`) ou la partie concernée (`db`, `deploy`, `adr`).
- **Cycles TDD** : les commits d'un cycle sont marqués `red`, `green` ou `refactor` dans la description (`test(us-1.1): red - group screenings by movie`, puis `feat(us-1.1): green - group screenings by movie`). L'historique montre ainsi l'ordre test puis code.
- **Changement incompatible** : `!` après le type (`feat(api)!: …`) et une ligne `BREAKING CHANGE:` dans le corps du message. Il impose une version majeure (voir [versioning](versioning.md)).

Ces messages servent ensuite à produire la liste des changements de chaque version.

## Historique publié

L'historique de `main` et `dev` n'est jamais réécrit, même pour corriger une coquille dans un nom de branche ou un message de commit : la correction se fait par un nouveau commit. Seule une branche de travail non encore fusionnée peut être réorganisée.
