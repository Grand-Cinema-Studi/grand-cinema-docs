# Guide de déploiement

Comment livrer une version, la vérifier, revenir en arrière et protéger les données, selon la stratégie du Bloc 2. L'architecture et les secrets sont décrits dans [environnements](environments.md) ; les commandes de la première installation sont dans le journal du backend ([from_scratch.md](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/blob/dev/from_scratch.md)).

Les éléments marqués _(à venir)_ sont décidés mais pas encore réalisés : aucune version n'est encore en ligne.

## Principe de livraison

| Événement | Ce qui se passe | Qui décide |
|---|---|---|
| Pull Request vers `dev` | intégration continue : tests, couverture, SonarCloud | automatique ; fusion bloquée si un contrôle échoue |
| Fusion dans `dev` | déploiement automatique sur la **recette** ; recette métier au fil du sprint | automatique ; validation en revue de sprint |
| Avant une release | version complète sur la **préproduction** : scan de sécurité, test de charge | l'équipe |
| Mise en production | manuelle, suivie d'un test de fumée | **le Product Owner décide**, l'équipe recommande |

Pour le projet, seule la recette est montée (voir [environnements](environments.md)). Les critères et procédures ci-dessous valent pour la préproduction et la production, qui reprennent les mêmes outils.

## Critères de mise en production

Une version ne part en production que si les critères suivants sont remplis. Chaque critère se vérifie par oui ou non, avec une preuve.

| Famille | Critère | Preuve | Poids |
|---|---|---|---|
| Transverse | la préproduction est identique à la production (services, configuration, domaine en HTTPS, fuseau horaire, volume de données) | fiche de configuration comparée | Bloquant |
| Fonctionnel | les parcours critiques passent en préproduction (réserver, payer en mode test, recevoir le billet, le scanner) | rapport des tests de bout en bout (Selenium) | Bloquant |
| | la recette métier est validée | procès-verbal signé par le Product Owner | Bloquant |
| | aucune anomalie bloquante ou majeure n'est ouverte | issues `bug` du Project | Bloquant |
| | les horaires s'affichent à l'heure de Paris et le blocage dure bien 10 minutes sur le serveur de préproduction | test de bout en bout sur une séance connue | Bloquant |
| Qualité du code | tous les tests automatisés passent, couverture du code métier d'au moins 80 %, Quality Gate SonarCloud validé | exécution GitHub Actions, rapport SonarCloud | Bloquant |
| Performance | avec 200 utilisateurs simultanés et un volume de données équivalent à un an d'exploitation : 95 % des requêtes en moins de 2 s, **zéro double vente** | rapport JMeter en préproduction | Double vente : bloquant ; temps de réponse : négociable |
| Sécurité | aucune faille de niveau élevé au scan, aucune dépendance vulnérable critique, HTTPS actif | rapport OWASP ZAP, alertes Dependabot | Bloquant |
| | les clés de production (Stripe, base de données) sont configurées et stockées hors du code | contrôle de la configuration de production | Bloquant |
| Exploitation | la migration de la base est rejouée en préproduction sur une copie anonymisée de la base de production, avec sa durée mesurée | journal de migration | Bloquant |
| | la sauvegarde automatique est active et une restauration a été testée | test de restauration daté | Bloquant |
| | le retour à la version précédente est écrit et testé ; la supervision et les alertes CloudWatch sont actives | procédure de retour arrière, tableau de bord CloudWatch | Bloquant |
| Légal | mentions légales, politique de confidentialité, gestion des cookies, conditions générales de vente ; paiement par Stripe, qui porte la conformité PCI-DSS | pages publiées | Bloquant |
| Organisationnel | caissiers et contrôleurs formés, contact support désigné pour les premiers jours | feuille de présence, planning support | Négociable |
| | guide utilisateur et documentation technique à jour (README, contrat d'API) | documents publiés | Négociable |

Chaque story livrée respecte en outre la [Definition of Done](https://github.com/Grand-Cinema-Studi/grand-cinema-docs/issues/24), qui inclut le Quality Gate SonarCloud.

## Si un critère n'est pas respecté

Un critère non respecté n'impose pas de tout bloquer, ni de tout livrer : on met en ligne ce qui est fiable, on protège le parcours de réservation, et on corrige le reste par itérations.

| Moment | Solution | Quand l'utiliser |
|---|---|---|
| Avant | corriger, retester, revalider en préproduction | correction rapide et sans risque |
| Avant | retirer une story de la version et livrer le reste (`git revert` du merge commit de la story) | le problème touche une fonctionnalité non indispensable au parcours |
| Avant | masquer une fonctionnalité par un interrupteur de configuration (feature flag) | le code est déployé mais la fonctionnalité n'est pas fiable |
| Avant | dérogation écrite : risque accepté, action corrective, date, signature du Product Owner | **critère négociable uniquement** |
| Avant | reporter la mise en ligne | critère bloquant non corrigeable à temps (exemple : double vente au test de charge) |
| Avant | contournement métier temporaire (vente au guichet ; vente en ligne arrêtée à 10 places restantes par séance) | éviter de pénaliser le cinéma pendant un report |
| Après | retour à la version précédente, décidé à l'avance | anomalie bloquante en production (exemple : paiements qui échouent) |
| Après | correctif urgent (hotfix, branche `hotfix/`), testé puis livré hors sprint | anomalie bloquante quand le retour arrière ferait perdre des données ou une correction importante |
| Après | correction au sprint suivant | anomalie majeure ou mineure |

Toute correction faite avant la mise en ligne repasse par les tests automatisés et par la préproduction. Un critère qui a manqué devient un contrôle permanent (test dans la CI, critère de la Definition of Done ou de cette liste).

## Calendrier de mise en ligne

- **Gel avant le festival** : aucune nouvelle fonctionnalité dans les 2 semaines qui précèdent le Festival de Jazz ; seules les corrections d'anomalies bloquantes passent.
- **Moment** : un jour de faible affluence, en semaine et le matin, avec l'équipe disponible ensuite.
- **Surveillance renforcée pendant 48 heures** après la mise en ligne, avec les alertes CloudWatch.
- **Seuil de retour arrière fixé à l'avance** : si le paiement ou la réservation échoue après la mise en ligne, retour à la version précédente.

## Déployer une version

### Le back-end

#### Automatiquement _(à venir)_

La fusion d'une Pull Request dans `dev` déclenche le workflow GitHub Actions du back-end :

1. il rejoue les tests ;
2. il construit l'image Docker de l'API et la publie dans GitHub Container Registry, étiquetée par le commit (`ghcr.io/grand-cinema-studi/grand-cinema-backend:<commit>`) ;
3. il demande au serveur (par AWS Systems Manager, sans SSH) d'exécuter `deploy/deploy.sh <commit>` ;
4. il vérifie que `/health` répond.

La mise en production est la même opération, lancée à la main après validation du Product Owner.

#### À la main

Depuis un terminal Session Manager ouvert sur le serveur (console AWS, EC2, « Se connecter »), avec le commit à déployer :

```bash
curl -fsSL https://raw.githubusercontent.com/Grand-Cinema-Studi/grand-cinema-backend/<commit>/deploy/deploy.sh -o /tmp/deploy.sh
sudo bash /tmp/deploy.sh <commit>
```

#### Ce que fait le script

1. Il télécharge le fichier Docker Compose **du commit déployé**.
2. Il lit les secrets dans Parameter Store et écrit le fichier `.env` du serveur, lisible par `root` seul.
3. Il télécharge l'image et ne recrée que les conteneurs qui changent (en général l'API) ; la base continue de tourner.
4. Le conteneur `migrate` applique les migrations ; **l'API ne démarre que si elles ont réussi**.
5. Il attend que `/health` réponde ; sinon, il affiche les journaux des migrations et de l'API, et échoue.

### Le front-end

#### Automatiquement _(à venir)_

La fusion d'une Pull Request dans `dev` du dépôt `grand-cinema-frontend` déclenche son workflow :

1. lint, tests, analyse SonarCloud et compilation de production ;
2. publication des fichiers dans le bucket S3 (`aws s3 sync dist/ s3://<bucket> --delete`) ;
3. invalidation du cache CloudFront (`aws cloudfront create-invalidation --distribution-id <id> --paths "/*"`), pour que les utilisateurs reçoivent la nouvelle version ;
4. contrôle que la page d'accueil répond.

Le rôle AWS du front ne peut écrire que dans ce bucket et invalider cette distribution ; il est distinct de celui du back-end.

#### À la main

Depuis le poste, avec un accès AWS personnel : compilation de production (`ng build --configuration production`), puis les deux commandes de publication et d'invalidation ci-dessus.

## Vérifier après déploiement (test de fumée)

```bash
curl https://<adresse>/health             # Healthy
curl https://<adresse>/api/schedule       # programmation du jour, datée du jour à Paris
```

Puis, dans un navigateur : la page d'accueil du front (la nouvelle version s'affiche, pas l'ancienne : cache CloudFront invalidé), une route Angular rechargée directement (`/schedule`), la connexion avec un compte de démonstration, et le parcours principal. Un appel d'API refusé (par exemple `GET /api/users/me` sans jeton) répond toujours `401` au format Problem Details, et non une page HTML. En production, un paiement de contrôle est effectué puis remboursé.

**Swagger pour la correction** : passer le paramètre `/grandcinema/recette/swagger-enabled` à `true` dans Parameter Store, relancer le déploiement du même commit (seule l'API redémarre), ouvrir `https://<adresse>/swagger`.

## Revenir en arrière

- **API** : relancer le script avec un commit précédent dont l'image existe (liste dans les Packages GitHub de l'organisation) :

  ```bash
  curl -fsSL https://raw.githubusercontent.com/Grand-Cinema-Studi/grand-cinema-backend/<ancien-commit>/deploy/deploy.sh -o /tmp/deploy.sh
  sudo bash /tmp/deploy.sh <ancien-commit>
  ```

- **Front** : republier le build précédent (relancer le workflow sur le commit précédent), ce qui invalide le cache CloudFront.

**Les migrations ne sont jamais annulées.** Une migration doit donc rester compatible avec la version précédente de l'API. Par exemple, pour renommer une colonne : une première version ajoute la nouvelle colonne et écrit dans les deux, une version suivante supprime l'ancienne.

## Sauvegarder et restaurer la base

### Sauvegarde

En recette : une tâche quotidienne (`/etc/cron.daily/grandcinema-backup`) exporte la base avec `pg_dump`, compressée, dans `/opt/grandcinema/backups` ; les sept dernières sont conservées. En production (cible) : sauvegardes automatiques de RDS.

Limite : en recette, ces sauvegardes sont sur le disque du serveur. Elles protègent d'une erreur de manipulation, pas de la perte du serveur. Avant toute opération risquée et avant la fin de la période gratuite d'AWS, la dernière sauvegarde est récupérée hors du serveur.

### Restauration

**À tester avant la mise en production** (critère bloquant), sur une copie :

```bash
cd /opt/grandcinema
sudo docker compose stop api
sudo docker compose exec -T db dropdb -U grandcinema grandcinema
sudo docker compose exec -T db createdb -U grandcinema grandcinema
gunzip -c backups/grandcinema-<AAAA-MM-JJ>.sql.gz | sudo docker compose exec -T db psql -U grandcinema -d grandcinema
sudo docker compose start api
```

Les données perdues au plus sont celles saisies depuis la dernière sauvegarde, soit une journée.

## Limites de l'hébergement retenu pour le projet

| Limite | Conséquence | Évolution (architecture cible du Bloc 1) |
|---|---|---|
| Un seul serveur pour l'API | s'il tombe, l'API est indisponible le temps de le relancer ; pas de montée en charge automatique | Auto Scaling derrière un répartiteur de charge |
| Base en conteneur, sauvegardes sur le serveur | jusqu'à une journée de données perdues ; aucune protection si le serveur est perdu | RDS : sauvegardes automatiques et restauration à un instant précis |
| HTTP entre CloudFront et le serveur | le jeton d'authentification circule en clair sur ce tronçon ; le serveur n'accepte que les adresses de CloudFront | nom de domaine, certificat et répartiteur de charge : HTTPS de bout en bout (critère de mise en production) |
| Adresse `*.cloudfront.net`, pas de nom de domaine | adresse peu lisible pour le public | nom de domaine du cinéma |
| Pas de préproduction montée | les critères propres à la préproduction (scan ZAP, test de charge JMeter) sont décrits mais pas exécutés sur un environnement identique à la production | préproduction allumée pour chaque release |
| Durée de la période gratuite d'AWS | l'environnement doit être migré ou arrêté à son terme | images sur GitHub, indépendantes d'AWS |

## Comptes de démonstration _(à venir)_

Un compte administrateur (`Manager`) et un compte spectateur (`Customer`) sont créés sur la recette. Leurs identifiants sont donnés dans les copies, jamais dans les dépôts.
