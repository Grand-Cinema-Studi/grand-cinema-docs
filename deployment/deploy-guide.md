# Guide de déploiement

Comment mettre en ligne une version, la vérifier, revenir en arrière et protéger les données. L'architecture de production et la gestion des secrets sont décrites dans [environnements](environments.md) ; les commandes complètes de la première installation sont dans le journal du backend ([from_scratch.md](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/blob/dev/from_scratch.md), étapes 9, 10 G et 11 G).

Les éléments marqués _(à venir)_ sont décidés mais pas encore réalisés : aucune version n'est encore en ligne.

## Critères de mise en production

Une version ne part en production que si tous ces critères sont remplis :

| Critère | Vérifié par |
|---|---|
| Tous les tests automatisés passent | intégration continue, sur la Pull Request `dev` → `main` |
| Le quality gate SonarCloud passe | intégration continue |
| Aucune anomalie bloquante ou majeure n'est ouverte | issues `bug` du Project |
| Les stories livrées respectent la Definition of Done | checklist dans chaque Pull Request |
| La recette est validée par le Product Owner | revue de sprint |
| Les migrations de base sont compatibles avec la version précédente de l'API | relecture de la Pull Request (voir « Revenir en arrière ») |
| Une sauvegarde récente de la base existe | `/opt/grandcinema/backups` sur le serveur |

Ces critères reprennent la stratégie de tests du Bloc 2 : tests au vert, aucune anomalie bloquante ou majeure, recette validée.

### Si un critère n'est pas rempli

La version ne part pas. Selon le cas :

| Situation | Réponse |
|---|---|
| Un test ou le quality gate échoue | correction sur une branche `fix/`, nouvelle Pull Request ; la fusion reste bloquée d'ici là |
| Une anomalie bloquante est ouverte sur une story | la story est retirée de la livraison (sa fusion dans `dev` est annulée par `git revert` du merge commit) et revient au sprint suivant ; le reste est livré |
| La recette n'est pas validée | livraison reportée à la revue suivante, avec la liste des écarts |
| Le problème apparaît après la mise en ligne | retour à la version précédente (voir plus bas), puis correction par une branche `hotfix/` |

## Déployer une version

### Automatiquement _(à venir)_

La fusion d'une Pull Request dans `main` déclenche le workflow GitHub Actions **Deploy** :

1. il rejoue les tests ;
2. il construit l'image Docker de l'API et la publie dans GitHub Container Registry, étiquetée par le commit (`ghcr.io/grand-cinema-studi/grand-cinema-backend:<commit>`) ;
3. il demande au serveur (par AWS Systems Manager, sans SSH) d'exécuter `deploy/deploy.sh <commit>` ;
4. il vérifie que `/health` répond.

L'accès de GitHub à AWS passe par OIDC : aucune clé AWS n'est stockée dans GitHub, et le rôle utilisé ne peut que lancer cette commande sur ce serveur.

### À la main

Depuis un terminal Session Manager ouvert sur le serveur (console AWS, EC2, « Se connecter »), avec le commit à déployer :

```bash
curl -fsSL https://raw.githubusercontent.com/Grand-Cinema-Studi/grand-cinema-backend/<commit>/deploy/deploy.sh -o /tmp/deploy.sh
sudo bash /tmp/deploy.sh <commit>
```

### Ce que fait le script

1. Il télécharge le fichier Docker Compose et la configuration du proxy **du commit déployé**.
2. Il lit les secrets dans Parameter Store et écrit le fichier `.env` du serveur, lisible par `root` seul.
3. Il télécharge les images et ne recrée que les conteneurs qui changent (en général l'API) ; la base, le front et le proxy continuent de tourner.
4. Le conteneur `migrate` applique les migrations ; **l'API ne démarre que si elles ont réussi**.
5. Il attend que `/health` réponde ; sinon, il affiche les journaux des migrations et de l'API, et échoue.

## Vérifier après déploiement

```bash
curl http://<adresse>/health              # Healthy
curl http://<adresse>/api/schedule        # programmation du jour, datée du jour à Paris
```

Puis, dans un navigateur : la page d'accueil du front, et le parcours principal.

**Swagger pour la correction** : passer le paramètre `/grandcinema/prod/swagger-enabled` à `true` dans Parameter Store, relancer le déploiement du même commit (seule l'API redémarre), ouvrir `http://<adresse>/swagger`.

## Revenir en arrière

Relancer le script avec un commit précédent dont l'image existe (liste dans les Packages GitHub de l'organisation) :

```bash
curl -fsSL https://raw.githubusercontent.com/Grand-Cinema-Studi/grand-cinema-backend/<ancien-commit>/deploy/deploy.sh -o /tmp/deploy.sh
sudo bash /tmp/deploy.sh <ancien-commit>
```

**Les migrations ne sont jamais annulées.** Une migration doit donc rester compatible avec la version précédente de l'API. Par exemple, pour renommer une colonne : une première version ajoute la nouvelle colonne et écrit dans les deux, une version suivante supprime l'ancienne.

## Sauvegarder et restaurer la base

### Sauvegarde

Une tâche quotidienne (`/etc/cron.daily/grandcinema-backup`) exporte la base avec `pg_dump`, compressée, dans `/opt/grandcinema/backups` ; les sept dernières sont conservées.

Limite : ces sauvegardes sont sur le disque du serveur. Elles protègent d'une erreur de manipulation, pas de la perte du serveur. Avant toute opération risquée et avant la fin de la période gratuite d'AWS, la dernière sauvegarde est récupérée hors du serveur.

### Restauration

À tester avant la mise en ligne, sur une copie :

```bash
cd /opt/grandcinema
sudo docker compose stop api
sudo docker compose exec -T db dropdb -U grandcinema grandcinema
sudo docker compose exec -T db createdb -U grandcinema grandcinema
gunzip -c backups/grandcinema-<AAAA-MM-JJ>.sql.gz | sudo docker compose exec -T db psql -U grandcinema -d grandcinema
sudo docker compose start api
```

Les données perdues au plus sont celles saisies depuis la dernière sauvegarde, soit une journée.

## Limites de l'hébergement retenu

| Limite | Conséquence | Évolution prévue |
|---|---|---|
| Un seul serveur | s'il tombe, l'application est indisponible le temps de le relancer | Auto Scaling et base RDS de l'architecture cible du Bloc 1 |
| Sauvegardes quotidiennes, sur le serveur | jusqu'à une journée de données perdues ; aucune protection si le serveur est perdu | sauvegardes automatiques et restauration à un instant précis avec RDS |
| HTTP tant qu'il n'y a pas de nom de domaine | données en clair sur le réseau | HTTPS avant l'authentification (voir la [sécurité](../security/security.md#protection-des-données-en-transit-et-au-repos)) |
| Durée de la période gratuite d'AWS | l'environnement doit être migré ou arrêté à son terme | images sur GitHub, indépendantes d'AWS ; passage à RDS sans changer le code |

## Comptes de démonstration _(à venir)_

Un compte administrateur (`Manager`) et un compte spectateur (`Customer`) sont créés sur l'environnement de démonstration. Leurs identifiants sont donnés dans les copies, jamais dans les dépôts.
