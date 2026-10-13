# Notes pour l'oral — Points de vigilance Blocs 1, 2 et 3

Le Grand Cinéma · préparation de la soutenance et des vidéos de réponse aux formateurs.
Format : pour chaque point, **la question probable du jury**, puis **la réponse à donner**.
Ce fichier n'est pas à déposer : il sert uniquement à préparer l'oral.

---

## BLOC 1 — Concevoir et spécifier une solution digitale

### Dossier 1, Question 1 — Demande, utilisateurs, EPICs

**5 profils et non 4**
- Question probable : « Pourquoi distinguer le caissier du contrôleur ? »
- Réponse : ils n'ont ni le même poste ni les mêmes droits. Le caissier consulte les réservations en ligne au guichet (lecture seule). Le contrôleur scanne les billets à l'entrée des salles. Aucun des deux n'accède à la programmation ni à l'analyse, réservées au Responsable Programmation. Cette séparation structure directement les droits d'accès de l'application.

**Délai de blocage de 10 minutes (règle de gestion de l'EPIC 2)**
- Question probable : « D'où viennent les 10 minutes ? »
- Réponse : ce n'est pas une mesure, c'est un choix de départ. C'est un paramètre métier ajustable, à affiner après observation des durées de paiement réelles. L'important est le principe : pas de réservation sans paiement, et des places libérées automatiquement si le paiement n'aboutit pas.

**Story de compte utilisateur**
- Question probable : « Pourquoi obliger à créer un compte pour réserver ? Un achat sans compte aurait réduit l'abandon de panier. »
- Réponse : la consultation reste libre, sans compte. Le compte sert à rattacher chaque billet à un utilisateur identifié et à lui permettre de retrouver ses billets. Un achat invité est une évolution possible, à arbitrer selon le taux d'abandon mesuré après la mise en ligne (KPI prévu).
- À vérifier avant l'oral : l'entité Utilisateur doit figurer dans le schéma d'architecture (c'est fait : RDS « Utilisateurs, films, séances, réservations » et module Authentification) et dans le MCD s'il y en a un.

### Dossier 1, Question 2 — MVP

- Question probable : « Pourquoi reporter le plan de salle en V2 alors que l'énoncé parle de visualiser les places en temps réel ? »
- Réponse : le besoin a été scindé. Le comptage des places restantes et le verrou anti-surbooking sont dans le MVP, car ils sont indispensables pour vendre sans double vente. Le plan de salle graphique est un confort : il est reporté en V2 pour tenir le budget.
- Question probable : « Comment anticiper les pics sans l'outil d'analyse ? »
- Réponse : les données de réservation sont collectées dès la V1 (suivi des réservations par séance). L'outil d'analyse et les alertes de seuil viennent en V2, sur des données déjà disponibles.

### Dossier 2, Questions 1 et 2 — Les solutions

- Question probable : « Weezevent gère des quotas de places. Pourquoi dire que l'anti-surbooking n'est pas garanti ? »
- Réponse : un SaaS gère un stock de billets, mais pas un verrou temporaire paramétrable pendant le paiement, ni une montée en charge maîtrisée pendant le festival. Surtout, on ne contrôle ni le code ni les évolutions : impossible de développer les fonctionnalités V2. Être prudent : dire « pas garanti » et « non maîtrisé », pas « impossible ».

### Dossier 2, Question 3 — Les coûts

**Hypothèses de chiffrage**
- Question probable : « D'où sortent 1 000 billets par jour, 50 % en ligne et 10 € le billet ? »
- Réponse : l'énoncé donne 800 à 1 600 spectateurs par jour, j'ai pris une valeur centrale. Les 50 % en ligne et le prix de 10 € sont des hypothèses, annoncées comme telles dans la copie, car l'énoncé ne les donne pas. La méthode reste valable avec d'autres valeurs : le seuil de rentabilité face à A est d'environ 85 000 billets en ligne par an (≈ 230 par jour).

**La Famille C coûte plus cher que la B**
- Question probable : « Vous retenez la solution la plus chère sur 3 ans ? »
- Réponse : oui, et c'est assumé dans la copie. Face à A, la C est rentabilisée en 4,5 à 7 mois. Face à B, elle coûte 35 à 55 k€ de plus sur 3 ans : c'est le prix de la maîtrise technique (verrou anti-surbooking, tenue des pics, évolutivité V2, pas de dépendance aux plugins). Si le client ne peut pas financer 40 à 62 k€ au départ, la B est l'option de repli.

**Prix non confirmés**
- La licence du plugin billetterie de la Famille B n'est pas chiffrée précisément (prix non trouvé). Elle est couverte par la fourchette de 600 à 1 200 €/an. Le dire franchement si la question vient.
- La grille Weezevent (2,5 %, minimum 0,99 € TTC par billet) vient d'un comparatif consulté en août 2026. Revérifier sur le site de l'éditeur avant l'oral.
- Le TJM de 435 €/jour et les fourchettes de design doivent pouvoir être sourcés : garder les sources sous la main.

### Dossier 3, Question 1 — Hébergement

**Pourquoi AWS plutôt qu'OVHcloud (note méthodologique retirée de la copie)**
- OVHcloud a été évalué comme alternative pour la Famille C. À coût équivalent, il offre une tarification plus simple (VPS forfaitaire à environ 8 €/mois), mais moins de services managés.
- AWS est retenu pour la maturité de ses services managés : Auto Scaling natif pour absorber les pics du festival, RDS, écosystème de développeurs. Contrepartie : une tarification fragmentée, à surveiller avec des alertes budgétaires.

**Dimensionnement**
- Question probable : « Une instance t3.micro suffit-elle pour 1 600 spectateurs par jour et le festival ? »
- Réponse : c'est un point de départ pour le MVP, pas une cible figée. L'Auto Scaling ajoute des instances pendant les pics, et CloudWatch déclenche les alertes. Le dimensionnement réel se valide par des tests de charge avant le festival.

**Hausse des tarifs des hébergeurs**
- La copie cite une hausse « jusqu'à +45 % chez OVHcloud sur certaines gammes » en 2026. Avoir la source sous la main, sinon reformuler à l'oral sans chiffre.

### Dossier 3, Questions 2 et 3 — Schéma et justification de l'architecture

- Question probable : « Vous parlez d'architecture modulaire et découplée, mais le front et l'API tournent sur la même couche EC2. »
- Réponse : le découplage est logique avant d'être physique. Le front et l'API sont deux blocs distincts, et l'API est découpée en modules (Authentification, Réservation, Paiement, Billets, Administration). On peut les séparer physiquement plus tard sans refonte. Des microservices dès le MVP seraient de la sur-ingénierie pour le budget et l'équipe.
- Question probable : « Pourquoi Stripe est-il en dehors d'AWS ? »
- Réponse : c'est un service tiers, appelé par le module Paiement de l'API. Aucune donnée de carte n'est stockée par l'application, ce qui limite le périmètre de conformité.

---

## BLOC 2 — Gestion, pilotage et testing

### Dossier 1, Question 1 — Approche agile

**Durée du projet**
- Question probable : « En combien de temps ? »
- Réponse : par la méthode. Sprints courts, mise en ligne dès que le parcours complet est validé selon la DoD. Ne pas avancer de durée que je ne peux pas étayer. Renvoyer à l'estimation de charge du Bloc 1 (Dossier 2, Question 3), qui est chiffrée et vérifiable.

**Disponibilité du Product Owner**
- Question probable : « Qui est le Product Owner, et sera-t-il disponible ? »
- Réponse : la direction du cinéma, appuyée par le Responsable Programmation. Sa disponibilité est listée dans les limites assumées de la copie : sans elle, l'agilité ne fonctionne pas.

### Dossier 1, Question 2 — User Stories

**GitHub au lieu de Trello**
- Question probable : « L'énoncé demande Trello. Pourquoi GitHub ? »
- Réponse : GitHub Projects offre les mêmes fonctions (colonnes, étiquettes, cartes) et ajoute la traçabilité du besoin jusqu'au code : chaque story est liée aux tâches techniques des dépôts backend et frontend. Le Project est public et consultable sans compte.

**14 stories contre 11 dans le Bloc 1**
- Question probable : « Pourquoi le nombre de stories a-t-il changé ? »
- Réponse : c'est l'affinage du backlog. La story de compte a été ajoutée, « visualiser et réserver » a été découpée en stories réalisables dans un sprint, et « suivre et analyser » a été scindée entre MVP (suivre) et V2 (analyser). L'écart est expliqué dans la copie.

**Réserver un nombre de places (US 2.3) n'est pas choisir son siège (US 2.4)**
- Question probable : « Le choix des places n'était-il pas reporté en V2 ? »
- Réponse : ce qui est reporté en V2, c'est le **choix du siège** sur un plan de salle (US 2.4). Le MVP permet de réserver une **quantité** de places pour une séance (US 2.3), avec le blocage de 10 minutes. Sans cette story, le MVP n'aurait plus de réservation, alors que le Bloc 1 le définit comme le parcours complet consultation → réservation → paiement → billet → contrôle.
- Vocabulaire à ne pas mélanger : le **compteur** de places restantes (US 2.2) et le **compte à rebours** de 10 minutes pendant le paiement (US 2.3).

**Règle de l'US 5.2 (avertissement quand une séance a déjà des réservations)**
- C'est un choix de conception, pas une exigence de l'énoncé. Le défendre ainsi : éviter de supprimer ou modifier une séance déjà vendue sans que le responsable en mesure l'impact sur les spectateurs. Si je ne suis pas à l'aise pour le défendre, le retirer de la copie.

**Cohérence avec le Project GitHub**
- Le Project est aligné sur la copie depuis le 9 octobre : 5 EPICs, 14 stories, mêmes titres, critères et étiquettes (EPIC, mvp / v2, profil). Vérifier une dernière fois, après avoir renommé l'US 2.3 dans la copie.
- Question probable : « On voit dans l'historique des issues que les titres ont changé. Pourquoi ? »
- Réponse : le premier backlog a été créé avant la rédaction des copies. Il a été affiné ensuite : stories reformulées, trois stories retirées (fermées en « Not planned » avec un commentaire qui explique pourquoi), cinq ajoutées. C'est le fonctionnement normal d'un backlog agile, et l'historique en garde la trace.

**Chiffrage du backlog**
- Question probable : « Comment avez-vous chiffré, et combien de temps pour le MVP ? »
- Réponse : 89 points au total, dont 63 pour le MVP et 26 pour la V2. Avec deux développeurs et une vélocité de 10 à 13 points par sprint de 2 semaines, le MVP représente 5 à 6 sprints, soit 10 à 13 semaines. C'est cohérent avec le chiffrage horaire du Bloc 1 (685 à 935 heures de développement).
- Question probable : « Pourquoi l'US 2.3 vaut-elle 13 points ? »
- Réponse : elle porte le verrou anti-surbooking, l'expiration à 10 minutes et la concurrence sur la dernière place. C'est la story la plus risquée techniquement.
- Question probable : « Plus de 60 % de Must have, ce n'est pas trop ? »
- Réponse : 57 points sur 89, soit 64 % du backlog complet. Sur le seul MVP, la part est plus élevée, ce qui est attendu : le MVP a été défini au Bloc 1 comme le parcours minimal indispensable. Les trois stories V2 sont en « Won't have » pour cette version, ce qui veut dire « pas maintenant », pas « jamais ».

**Blocage des places en base, sans Redis**
- Question probable : « Comment garantissez-vous qu'une place n'est pas vendue deux fois ? »
- Réponse : le blocage est enregistré en base PostgreSQL, dans une transaction qui verrouille la séance, avec une expiration à 10 minutes. C'est cohérent avec le schéma d'architecture, et suffisant pour le volume du Grand Cinéma. Redis serait une optimisation possible si la charge l'exigeait, mais il faudrait alors l'ajouter au schéma.

### Dossier 1, Question 3 et Dossiers 2 et 3

À compléter une fois les réponses rédigées (DoR et DoD, stratégie de tests, critères de mise en production, plan B, gestion des risques).

---

## BLOC 3 — Développer une solution digitale

Rappel du critère commun à toutes les questions du Bloc 3 : « interactions avec l'évaluateur démontrant l'emploi correct des terminologies métiers et une ouverture d'esprit constructive à la critique ». Face à une objection : reconnaître la limite, expliquer le compromis choisi et dire ce que je ferais autrement avec plus de temps ou de volume. Ne jamais défendre un choix par « c'est comme ça qu'on fait ».

### À régler avant le 1ᵉʳ dépôt (20 octobre)

**La branche `dev` au lieu de `develop` (choix assumé)**
- L'énoncé demande mot pour mot « une branche develop » (Dossier 1, Question 1) et « une merge request vers develop » (Dossier 2, Question 6). Le dépôt garde `dev` : seul le nom est abrégé, le rôle est identique.
- À faire : rendre la correspondance visible pour le correcteur, au lieu d'attendre la question :
  - dans la copie (Dossier 1, Question 1) et dans `CONTRIBUTING.md` : « `dev` est la branche d'intégration, équivalente à la branche `develop` demandée par l'énoncé » ;
  - dans la copie (Dossier 2, Question 6) : « Pull Request vers `dev` (branche develop) ».
- Question probable : « L'énoncé demande une branche develop. Où est-elle ? »
- Réponse : « `dev` joue exactement le rôle de `develop` : branche d'intégration protégée, cible de toutes les Pull Requests, et `main` n'est alimentée que depuis elle. Seul le nom est abrégé, et je l'ai documenté dans la convention de branches. »

**« Pour chaque user story définie dans le bloc 2 » (Dossier 2, Question 3)**
- L'énoncé demande toutes les stories ; le Bloc 3 en développe une sélection : US 1.1, 2.1, 2.2, 2.3, 5.2 et 5.3 (paiement et billet QR en bonus si le temps le permet).
- Question probable : « Pourquoi ne pas avoir développé toutes les user stories ? »
- Réponse : le backlog du Bloc 2 sépare le MVP de la V2 (US 2.4, 5.4 et 5.5 en « Won't have » pour cette version). Dans le temps de l'épreuve, j'ai priorisé le cœur du parcours (consulter, se connecter, voir les places restantes, réserver sans double vente, programmer, suivre) et la story la plus risquée techniquement (US 2.3). Les autres stories du MVP sont planifiées dans les sprints suivants, avec la même méthode (branche, TDD, Pull Request).
- À vérifier avant l'oral : la liste exacte des stories du MVP non développées, pour la citer sans hésiter.

**État du dépôt backend au 13 octobre**
- Fusionnées dans `dev` : PR #9 (contrats CQRS), PR #10 (US 1.1 : handler de la programmation et ses 5 tests), PR #11 (journal), PR #12 (EF Core, PostgreSQL en Docker, outil `dotnet-ef`).
- Pas encore fait : repository réel, endpoint, Swagger (`Program.cs` est encore le modèle « Hello World »), modèle de données complet et migration.

**L'US 1.1 n'est pas encore une fonctionnalité complète (Dossier 2, Question 6)**
- L'énoncé demande une fonctionnalité complète : branche, développement, merge request et documentation Swagger. Aujourd'hui, l'US 1.1 n'a que sa logique métier : il manque le repository PostgreSQL, l'endpoint et Swagger.
- À faire avant le 20 octobre : terminer l'US 1.1 de bout en bout (c'est la plus simple à présenter comme fonctionnalité complète).
- Si la question vient : « L'US est livrée en deux Pull Requests : la logique métier testée d'abord (PR #10), puis l'infrastructure, l'endpoint et Swagger une fois la base en place. »

**Le tri des films par titre n'a pas de test**
- Le handler trie les films par titre (`OrderBy(m => m.Title)`), mais ni test ni critère d'acceptation ne l'exige : du code de production sans test, contraire à la règle TDD annoncée.
- Décision : garder le tri (sans lui, l'ordre des films dépend de PostgreSQL et peut changer d'un appel à l'autre), mais le rendre officiel. Détail dans `from_scratch.md`, étape 5e bis :
  - ajouter le critère à l'issue de l'US 1.1 (« Les films sont affichés par ordre alphabétique de titre ») ;
  - écrire le test `When_several_movies_exist_Then_they_are_sorted_by_title` ;
  - vérifier qu'il échoue en retirant temporairement le tri, puis le rétablir ; le noter dans le message de commit.
- Question probable : « Quel test exige que les films soient triés par titre ? »
- Réponse : « J'ai repéré du code de production sans test. J'ai fait valider la règle, je l'ai ajoutée aux critères d'acceptation de l'US 1.1, puis j'ai écrit le test, en vérifiant qu'il échouait bien sans le tri. »
- Question probable : « Pourquoi par titre et pas par heure de séance ? »
- Réponse : c'est une décision du Product Owner : l'ordre alphabétique est simple et prévisible pour le spectateur ; l'ordre de la première séance serait une alternative utile au guichet. Le critère écrit fixe le choix, et le test le protège.
- Piège à connaître : `OrderBy` sur du texte dépend de la langue du serveur (« Élise » peut passer après « Zorro » sur un serveur en anglais). Correctif possible : comparer explicitement en français. Même famille de problème que le fuseau horaire.

**`.env.example` absent du dépôt**
- Le journal prévoit un `.env.example` versionné (noms des variables sans valeur), mais il n'est pas dans le dépôt. Sans lui, quelqu'un qui clone le projet ne sait pas quelles variables créer (`POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`).
- À faire : l'ajouter, ou ne pas l'annoncer à l'oral.

**Le code et le modèle de données ne disent pas encore la même chose**
- L'entité `Movie` a une date de sortie (`ReleaseDate`) absente du modèle prévu, et rend le synopsis et l'affiche obligatoires.
- À faire : aligner le modèle de données (étape 7), le MCD de la copie et le tableau de correspondance. Le jury compare le schéma et le code.

**Petites coquilles visibles**
- La branche de la PR #10 s'appelle `feature/us-1-1-view-shedule` (« shedule » sans c). Réponse si besoin : « Coquille dans le nom ; la branche est fusionnée et supprimée, je ne réécris pas l'historique pour ça. »
- Le fichier de mapping s'appelle `ScreenMappings.cs` alors que l'entité est `Screening` : renommer en `ScreeningMappings.cs` pour rester cohérent.

### Dossier 1, Question 1 — Dépôt Git, branches et convention

**Pull Request et merge request**
- Question probable : « L'énoncé parle de merge request, où sont-elles ? »
- Réponse : c'est la même chose. « Merge request » est le terme de GitLab, « Pull Request » celui de GitHub. Chaque branche est intégrée par une Pull Request vers la branche d'intégration.

**Protection des branches**
- Question probable : « Qu'est-ce qui empêche de pousser directement sur main ? »
- Réponse : un ruleset GitHub actif sur `main` et sur la branche d'intégration : Pull Request obligatoire, suppression interdite, force push bloqué, aucune exception (bypass list vide). Testé : un push direct est refusé.

**Zéro approbation requise**
- Question probable : « Une Pull Request sans relecteur, ça sert à quoi ? »
- Réponse : GitHub interdit d'approuver sa propre Pull Request ; en projet solo, une approbation obligatoire bloquerait toute fusion. La Pull Request garde son intérêt : relecture du diff (onglet Files changed), trace de chaque changement, et bientôt les vérifications automatiques obligatoires (compilation et tests en intégration continue). En équipe, je mettrais au moins une approbation.

**Mode de fusion**
- Question probable : « Pourquoi des commits de fusion plutôt qu'un historique linéaire ? »
- Réponse : le merge commit garde chaque branche visible dans le graphe Git, ce qui prouve le workflow par branche demandé par l'énoncé. Je connais les alternatives : squash (un commit par Pull Request) et rebase (historique linéaire, mais commits recréés et lien avec la branche perdu).

**Convention de nommage**
- Préfixes `feature/`, `fix/`, `chore/`, `docs/`, `test/`, `hotfix/` ; branche de fonctionnalité = préfixe + numéro de story + intitulé en anglais (exemple : `feature/us-1-1-view-schedule`, sur le modèle `feature/login-user` de l'énoncé) ; messages de commit au format Conventional Commits (`feat`, `fix`, `test`, `refactor`, `docs`, `chore`).

**Les commits qui retirent des fichiers `obj/` et `bin/`**
- Question probable : « Pourquoi un commit qui supprime des centaines de fichiers de compilation ? »
- Réponse : le `.gitignore` .NET avait été oublié au départ, donc des fichiers générés ont été versionnés. Corrigé en les retirant du suivi (`git rm --cached`, sans les supprimer du disque) et en ajoutant le `.gitignore` standard. Leçon retenue : `dotnet new gitignore` avant le premier commit. Assumer l'erreur et la correction, c'est une démarche normale.

**`global.json`**
- Question probable : « À quoi sert ce fichier ? »
- Réponse : il fixe la version du SDK .NET pour le dépôt. Je travaille sur deux ordinateurs (Windows et Linux) : même SDK partout, donc même compilation, et la version est écrite dans le dépôt.

### Dossier 1, Question 2 — Base de données

**Sept tables au lieu de cinq**
- Question probable : « L'énoncé cite cinq entités, pourquoi sept tables ? »
- Réponse : les cinq demandées sont couvertes (Users, Movies, Screenings, Reservations, Seats). J'ajoute Rooms (la capacité et les places dépendent de la salle) et ReservedSeats (le lien entre une réservation et ses places). C'est la normalisation en troisième forme normale.

**Code et tables en anglais, MCD en français**
- Question probable : « Votre MCD dit Séance, votre code dit Screening. »
- Réponse : tout le code est en anglais, langue de l'écosystème .NET, pour éviter les noms hybrides. Un tableau de correspondance relie le vocabulaire du Bloc 1 à celui du code (Film → Movie, Salle → Room, Place → Seat, Séance → Screening, Place réservée → ReservedSeat, Programmation → Schedule).
- À faire : mettre ce tableau dans la copie du Bloc 3, à côté du schéma.

**Contrainte d'unicité sur séance et place**
- Question probable : « Comment la base empêche-t-elle de vendre deux fois la même place ? »
- Réponse : une contrainte UNIQUE sur (ScreeningId, SeatId) dans ReservedSeats. Même si le code applicatif a un bug, la base refuse la deuxième ligne. C'est la dernière ligne de défense ; la première est la transaction (voir Dossier 3, Question 3).

**Une place doit appartenir à la salle de la séance**
- Question probable : « Qu'est-ce qui empêche de réserver une place d'une autre salle ? »
- Réponse : aucune clé étrangère ne peut l'exprimer simplement. Le contrôle est fait dans le handler de réservation et couvert par un test. Savoir le dire avant que le jury le trouve.

**Libération des places**
- Question probable : « À l'expiration, pourquoi supprimer les lignes de ReservedSeats plutôt que les marquer ? »
- Réponse : la contrainte d'unicité doit pouvoir accepter une nouvelle réservation de la même place. L'historique n'est pas perdu : la réservation reste dans Reservations avec le statut Expired ou Cancelled.

**Identifiants GUID**
- Question probable : « Pourquoi des GUID plutôt que des entiers ? »
- Réponse : on ne peut pas deviner ni parcourir les identifiants dans les URL (par exemple les réservations des autres), et ils peuvent être créés par l'application avant l'enregistrement. Contrepartie connue : index plus volumineux, sans impact à ce volume.

**Script SQL**
- Question probable : « Avez-vous écrit le SQL à la main ? »
- Réponse : il est généré depuis les migrations EF Core (`dotnet ef migrations script`), donc toujours fidèle au code. Le schéma entité-relation est versionné dans `docs/database`.

### Dossier 2, Questions 1 et 4 — Bonnes pratiques

**Clean Architecture**
- Question probable : « Pourquoi cette architecture pour un projet de cette taille ? »
- Réponse : le métier (Domain, Application) ne dépend ni de la base ni du framework web : les dépendances vont toujours vers le centre. Résultat : les cas d'usage se testent sans base, et l'on peut changer de base ou d'hébergeur sans toucher au métier. C'est la traduction de l'architecture modulaire du Bloc 1.

**Le dossier `Abstractions`**
- Question probable : « À quoi sert ce dossier ? »
- Réponse : il contient les contrats dont les cas d'usage ont besoin. Application les définit, Infrastructure les implémente : c'est l'inversion de dépendances (le « D » de SOLID). Grâce à ça, mes handlers ne connaissent pas PostgreSQL et je les teste en TDD avec des mocks.

**Le dossier `Abstractions/Cqrs`**
- Question probable : « Vous utilisez un broker de messages ? »
- Réponse : non. Requêtes et commandes sont des « messages » au sens CQRS, mais elles sont traitées dans le même processus, par un simple appel de méthode. C'est pour éviter cette confusion que le dossier s'appelle `Cqrs` et non `Messaging`.
- Attention : ne pas parler de « messagerie » ou de « bus ».

**Deux dossiers `Persistence`**
- `Application/Abstractions/Persistence` : les interfaces (ce dont le métier a besoin). `Infrastructure/Persistence` : DbContext, configurations EF Core, migrations, implémentations des repositories (comment on le fait avec PostgreSQL). Ne pas les confondre à l'oral.

**Le terme « repository »**
- Question probable : « Pourquoi parler de repository ? »
- Réponse : c'est le pattern de Martin Fowler et d'Eric Evans : le métier manipule une sorte de collection d'objets sans savoir comment ils sont stockés. `IScreeningReadRepository` renvoie des entités `Screening` ; « Read » indique qu'il ne fait que lire.
- Vocabulaire : « dépôt » pour Git, « repository » pour le pattern. Ne pas mélanger.
- Question probable : « EF Core est déjà un repository, pourquoi en rajouter un ? »
- Réponse : pour que la couche Application ne dépende pas d'EF Core : les handlers se testent avec un mock de l'interface, et le SQL reste regroupé dans Infrastructure. Alternative connue : une interface qui expose directement les DbSet, plus simple mais qui lie l'Application à EF Core.

**Organisation par fonctionnalité**
- Question probable : « Pourquoi `Features/Screenings/Queries`, `Commands`, `DTOs` ? »
- Réponse : tout ce qui concerne un cas d'usage est au même endroit, et ajouter une story ne touche qu'un dossier. Les interfaces partagées restent dans `Abstractions`, pour qu'une fonctionnalité ne dépende pas d'une autre. Pas de dossier vide : `Commands/` apparaît avec la première commande.

**DTO plutôt qu'entités**
- Question probable : « Pourquoi ne pas renvoyer directement les entités au front ? »
- Réponse : le DTO ne contient que ce que l'écran affiche, n'expose pas le modèle interne et évite les références circulaires à la sérialisation JSON.

**Mapping manuel des entités vers les DTO**
- Question probable : « Pourquoi ne pas utiliser AutoMapper ? »
- Réponse : la conversion est écrite à la main, dans des méthodes d'extension rangées dans `DTOs/Mappings`. Elle est explicite, vérifiée à la compilation (une propriété oubliée ou renommée se voit tout de suite), sans réflexion et facile à déboguer. AutoMapper serait une dépendance de plus pour quelques lignes de code ; il est aussi passé sous licence commerciale (même éditeur que MediatR).
- À vérifier avant l'oral : les conditions de licence actuelles d'AutoMapper.

**Sécurité**
- Aucun secret dans le dépôt public : nom, utilisateur et mot de passe de la base dans un `.env` non versionné, lu par Docker Compose ; chaîne de connexion dans les user-secrets .NET (le projet Api porte un `UserSecretsId`). Le `.env.example` est encore à ajouter.
- Question probable : « Pourquoi le mot de passe est-il à deux endroits, `.env` et user-secrets ? »
- Réponse : chaque outil a son rôle. Docker Compose lit le `.env` pour configurer le conteneur PostgreSQL ; ASP.NET Core ne lit pas ce fichier nativement. Les user-secrets sont intégrés à .NET, stockés hors du dépôt (dans le dossier personnel), donc impossibles à commiter par erreur, et lus aussi par `dotnet ef` pour les migrations. En production, ni l'un ni l'autre : variables d'environnement ou gestionnaire de secrets AWS. Inconvénient accepté : le mot de passe est saisi deux fois, sur chaque machine.
- Mots de passe stockés hachés (colonne PasswordHash), jamais en clair.
- À compléter avec l'US 2.1 : authentification, rôles et protection des endpoints.

**Gestion des erreurs**
- À compléter quand elle sera en place (format de réponse d'erreur de l'API, codes HTTP, journalisation). Prévoir au minimum le cas de la place déjà prise : réponse claire au lieu d'une erreur technique.

**Versioning**
- Branches protégées, une branche par story, Pull Requests, Conventional Commits, SDK figé par `global.json`, outil `dotnet-ef` figé par `dotnet-tools.json`, migrations de base versionnées.

**PostgreSQL dans Docker**
- Question probable : « Pourquoi Docker pour la base ? »
- Réponse : même moteur que la production prévue au Bloc 1 (RDS PostgreSQL), version figée (`postgres:16`), rien à installer sur les deux ordinateurs, un volume qui conserve les données et une vérification de santé (`pg_isready`) qui indique quand la base est prête.

**`dotnet-ef` en outil local**
- Question probable : « Pourquoi un fichier `dotnet-tools.json` ? »
- Réponse : `dotnet-ef` gère les migrations. Installé localement au dépôt, sa version exacte est écrite dans ce manifeste commité : toute machine (l'autre ordinateur, la future intégration continue) récupère la même version avec `dotnet tool restore`. Elle doit correspondre à la version d'EF Core du projet (10.0.12), ce qu'une installation globale ne garantit pas.

**Licences des bibliothèques**
- Question probable : « Pourquoi pas MediatR ? »
- Réponse : MediatR est passé sous licence commerciale en 2025, et mon besoin est couvert par deux interfaces (`IQueryHandler`, `ICommandHandler`). Même raison pour Shouldly à la place de FluentAssertions (version 8 commerciale).
- À vérifier avant l'oral : les conditions de licence actuelles de MediatR et FluentAssertions.

**« Aujourd'hui » et le fuseau horaire**
- Question probable : « Si un spectateur consulte la programmation à 23 h 30 et que le serveur est en UTC, quel jour voit-il ? »
- Réponse : risque réel de décalage d'un jour. La date du jour vient d'un `TimeProvider`, remplacé dans les tests par une horloge simulée réglée sur le fuseau Europe/Paris (test du cycle 4). En production, `TimeProvider.System` prend le fuseau du serveur, souvent UTC sur AWS : il faudra convertir l'heure courante en heure de Paris explicitement. Point identifié, à traiter au déploiement.
- Limite du test actuel : il simule midi à Paris, donc le même jour qu'en UTC. Le cas qui piège (par exemple 22 h 30 UTC en été, soit 0 h 30 le lendemain à Paris) n'est pas testé : un test à ajouter quand le fuseau sera géré.

**ADR 0001 – stratégie de lecture**
- Les choix d'architecture sont consignés dans `docs/adr` (contexte, décision, alternatives écartées, conséquences). Le citer quand une question porte sur « pourquoi ce choix ».

### Dossier 2, Questions 2 et 5 — Choix du framework

- À compléter : comparaison d'ASP.NET Core (backend) avec au moins une alternative sur les quatre critères de l'énoncé (performance, coût, scalabilité, facilité de développement), et éventuellement Angular côté front. Chaque critère doit être sourcé ou chiffré, pas seulement affirmé.

### Dossier 2, Questions 3 et 6 — Développement des user stories

**Pourquoi commencer par l'US 1.1**
- Question probable : « Pourquoi cette story en premier ? »
- Réponse : c'est la plus petite tranche complète, sans dépendance à l'authentification ni au paiement. Elle valide l'architecture et l'outillage de tests avant la story la plus risquée (US 2.3).

**TDD**
- Question probable : « Comment prouvez-vous que vous avez fait du TDD ? »
- Réponse : par l'historique Git de la PR #10. Chaque critère d'acceptation du Bloc 2 correspond à un test nommé `When_<situation>_Then_<résultat>`, étiqueté avec le numéro de la story (5 tests). L'historique montre :
  - cycle 1 (regroupement par film) : un commit `red`, un commit `green`, un commit `refactor` ;
  - cycle 4 (date du jour par défaut) : un commit `red`, puis un commit `green` ;
  - cycles 2, 3 et 5 (date choisie, aucune séance, tri par heure) : un seul commit `test(...)` chacun, car le test est passé au vert dès son écriture.
- Question probable : « Les tests des cycles 2, 3 et 5 n'ont jamais été rouges. Est-ce encore du TDD ? »
- Réponse : « Pas au sens strict, et je l'assume. Le code minimal du cycle 1 transmettait déjà la date au repository et renvoyait une liste vide sans séance : les tests 2 et 3 confirment ces comportements, je les garde comme spécification des critères d'acceptation. Pour le tri par heure, le comportement existait avant son test : il a été écrit trop tôt, alors qu'un refactor ne doit pas ajouter de comportement. Ce que j'en retiens : quand un nouveau test passe immédiatement, je casse volontairement le code pour vérifier que le test échoue bien, puis je le rétablis. »
- À vérifier avant l'oral : dans quel commit le tri par heure est apparu (probablement le refactor du cycle 1), pour répondre précisément.
- Question probable : « Vos tests n'utilisent pas de base : comment savez-vous que le filtre par jour fonctionne ? »
- Réponse : les tests unitaires vérifient la logique du handler, avec un mock du repository. Le filtre SQL est vérifié par un test d'intégration sur une vraie base PostgreSQL.

**Parcours d'une requête**
- Question probable : « Décrivez le parcours d'une requête dans votre application. »
- Réponse en cinq temps : l'endpoint (Api) transforme la requête HTTP en `GetScheduleQuery` ; l'injection de dépendances fournit le handler ; le handler (Application) appelle `IScreeningReadRepository.GetScreeningsAsync` ; le repository (Infrastructure) interroge PostgreSQL via EF Core et renvoie des entités ; le handler construit un `ScheduleDto`, que l'endpoint renvoie en JSON (HTTP 200). Schéma complet dans `from_scratch.md`.

**Est-ce vraiment du CQRS ?**
- Question probable : « Votre lecture passe par un repository qui charge des entités. Est-ce vraiment du CQRS ? »
- Réponse : « J'applique le CQRS au niveau des opérations : requêtes et commandes séparées, chacune avec son handler, et un repository en lecture seule. Je n'ai pas séparé les modèles de lecture et d'écriture, parce que les volumes sont faibles et que cela me permet de tester la logique de la programmation en unitaire, conformément à la stratégie TDD du Bloc 2. Si un écran devenait lourd, je passerais ce côté lecture en projection SQL directe vers les DTO, avec une interface `IScheduleQueries`, testée en intégration. »
- Les trois niveaux à connaître : séparer les opérations (mon cas), séparer les modèles (lecture par projection SQL directe en DTO), séparer les bases (démesuré ici).
- Attention : ne pas dire « CQRS complet ».

**Repository ou projection : le mélange**
- Question probable : « Pour le suivi des réservations, vous faites autrement. Ce n'est pas incohérent ? »
- Réponse : le suivi (US 5.3) calcule des agrégats : il vaut mieux les faire calculer par PostgreSQL que charger des centaines d'entités pour n'en garder que des totaux. Le mélange suit une règle écrite dans l'ADR 0001 : écriture par entités et repository ; lecture simple par repository ; lecture d'agrégats par projection. Le nom indique l'approche (`ReadRepository` ou `Queries`). Ce qui doit être uniforme, c'est la règle, pas la technique.

**Swagger**
- À faire : au 13 octobre, `Program.cs` est encore le modèle « Hello World », sans endpoint ni Swagger. Prévoir la documentation de l'API générée et accessible, et montrer l'endpoint de l'US développée en entier pour la Question 6.

### Dossier 3 — Qualité, tests et bug

**Question 1 — Outil de qualité (SonarCloud)**
- À compléter une fois branché : quality gate, problèmes relevés, corrections apportées. SonarCloud est gratuit pour un dépôt public.

**Question 2 — Tests unitaires et couverture**
- Question probable : « Quel taux de couverture visez-vous ? »
- Réponse : pas un chiffre pour le principe. Ce qui compte, c'est que la logique métier (handlers et domaine) soit couverte ; un taux élevé obtenu sur du code sans logique ne prouve rien. Couverture mesurée avec Coverlet, capture du rapport dans la copie.
- Outils à savoir justifier : xUnit, Moq (doublures), Shouldly (assertions lisibles), horloge simulée (`FakeTimeProvider`) pour maîtriser la date.
- À montrer : `GetScheduleHandler` couvert à 100 %, dans un rapport HTML généré avec ReportGenerator à partir du fichier de Coverlet (ou directement dans VS Code avec C# Dev Kit).
- Limite à savoir dire : un pourcentage élevé prouve que le code a été exécuté, pas que les assertions vérifient les bons résultats.

**Question 3 — Bug potentiel : la double réservation**
- Question probable : « Comment l'avez-vous identifié, reproduit, corrigé et validé ? »
- Réponse :
  - identifié : deux spectateurs réservent en même temps la dernière place d'une séance ; chacun voit la place libre, les deux réservations passent ;
  - reproduit : un test d'intégration lance deux réservations en parallèle et constate, avant correctif, que les deux sont acceptées ;
  - corrigé : transaction avec verrou de ligne sur la séance pendant la vérification et l'enregistrement, plus la contrainte d'unicité (ScreeningId, SeatId) en filet de sécurité ;
  - validé : le même test passe, une seule réservation réussit, l'autre reçoit un message clair (conflit) au lieu d'une erreur technique.
- Question probable : « Pourquoi la contrainte et la transaction ? »
- Réponse : la contrainte garantit l'intégrité même en cas d'erreur de code ; la transaction permet de gérer le cas proprement et de répondre clairement à l'utilisateur.
- Cohérence avec le Bloc 2 : même réponse que « blocage des places en base, sans Redis ».
- À compléter avec le vrai test, les commits et les captures.

### Vocabulaire à employer correctement

- Pull Request = merge request (GitHub et GitLab).
- Dépôt = Git ; repository = pattern d'accès aux données.
- Requête (query) : lit sans modifier ; commande (command) : modifie ; handler : traite l'une ou l'autre.
- Entité (modèle métier) et DTO (données envoyées à l'écran).
- Test unitaire (sans base, avec doublures) et test d'intégration (avec une vraie base).
- Doublure, ou mock : objet qui simule une dépendance dans un test.
- Inversion de dépendances, injection de dépendances : la première est le principe (Application définit l'interface), la seconde le mécanisme (le conteneur fournit l'implémentation).
