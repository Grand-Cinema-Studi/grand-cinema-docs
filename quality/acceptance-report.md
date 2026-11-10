# Procès-verbal de recette du MVP

Ce document trace la recette du MVP : chaque critère d'acceptation des 12 user stories est rejoué sur l'environnement de recette, avec les comptes de démonstration, puis validé ou refusé par le Product Owner. C'est la preuve du point de la [Definition of Done](https://github.com/Grand-Cinema-Studi/grand-cinema-docs/issues/24) « déployée sur l'environnement de recette et validée par le Product Owner », et du plan de recette de la [stratégie de tests](testing-strategy.md) exécuté.

Les tests automatisés (236 au backend, 183 au frontend) prouvent les règles à chaque Pull Request. Cette recette vérifie autre chose : le produit déployé, tel qu'un utilisateur le voit, de bout en bout.

## Cadre de la recette

| Élément | Valeur |
|---|---|
| Environnement | recette AWS (CloudFront, EC2, PostgreSQL), voir [environnements](../deployment/environments.md) |
| Adresse | https://d3oaw7zyh0jmbo.cloudfront.net |
| Documentation de l'API | https://d3oaw7zyh0jmbo.cloudfront.net/swagger (activé sur la recette pendant la correction, désactivé par défaut : voir [environnements](../deployment/environments.md)) |
| Version du backend | branche `dev`, commit [`6dae1da`](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/commit/6dae1da) |
| Version du frontend | branche `dev`, commit [`6f25625`](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/commit/6f25625) |
| Données | données de démonstration remises à zéro par `deploy/reset-demo-data.sh` le 10/11/2026, commit `6dae1da` du backend (jamais `seed-dev.sql` seul : voir le [bug 0002](bugs/0002-demo-screenings-overlap.md)) |
| Paiement | Stripe en mode test : carte acceptée `4242 4242 4242 4242`, carte refusée `4000 0000 0000 0002` (date future, code quelconque) |
| Appareils | un ordinateur (Chrome, 1440 px) et un téléphone (navigateur mobile) |
| Recette menée par | Slimane Smahi |
| Dates | Le 10 novembre 2026 |
| Historique | première passe complète le 09/11/2026 sur le frontend `9b0def1` : 52 critères conformes. Rejouée entièrement le 10/11/2026, sur la version gelée, après les améliorations faites avant le gel (afficher et confirmer le mot de passe, [F #33](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/33) ; formulaire d'un film dans une fenêtre, [F #35](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/35)). La passe du 09/11 reste lisible dans l'historique Git de ce document |

## Comptes utilisés

Les mots de passe sont dans le README du backend (« Données de démonstration »).

| Compte | Rôle | Sert à recetter |
|---|---|---|
| _(aucun)_ | visiteur | US 1.1, 1.2, 2.2, l'invitation à se connecter de l'US 2.3 |
| compte créé pendant la recette | spectateur | US 2.1 (création, refus, suppression) |
| `client@grand-cinema.test` | spectateur (`Customer`) | US 2.3, 3.1, 3.2 |
| `manager@grand-cinema.test` | responsable (`Manager`) | US 5.2, 5.3, 5.6 |
| `cashier@grand-cinema.test` | caissier (`Cashier`) | US 5.1 |
| `inspector@grand-cinema.test` | contrôleur (`TicketInspector`) | US 4.1 |

## Comment remplir

- Une ligne par critère d'acceptation, dans l'ordre du backlog ; la colonne « Scénario » dit quoi faire, la colonne « Attendu » ce qu'on doit voir.
- **Résultat** : « conforme », « non conforme » ou « conforme avec réserve » (réserve expliquée en note). Pour un « non conforme », une issue `bug` est ouverte dans le dépôt de la cause et notée dans « Anomalies ».
- **Preuve** : _manuelle_ (rejouée à l'écran). _base_ (requête SQL sur la recette) ; _auto_ (le critère n'est pas rejouable à la main de façon fiable : le test automatisé cité en est la preuve, et sa dernière exécution verte en CI est vérifiée).
- **PR** : la ou les Pull Requests qui ont livré le critère (B = backend, F = frontend).

## Epic 1 – Consultation et programmation

### US 1.1 – Consulter les films et les horaires

| N° | Critère | Scénario | Attendu | Compte | Preuve | Date | Résultat | PR |
|---|---|---|---|---|---|---|---|---|
| 1.1-1 | Je vois les films avec leurs séances (jour, heure, salle) | Ouvrir l'accueil, puis « Films à l'affiche » et la fiche d'un film | Chaque film montre ses séances : jour, heure, salle | visiteur | manuelle | 10/11/2026 | Conforme | [B #27](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/27), [B #43](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/43), [F #2](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/2), [F #9](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/9) |
| 1.1-2 | Une date choisie n'affiche que ses séances | Choisir demain avec le sélecteur de date | Seules les séances de demain s'affichent | visiteur | manuelle | 10/11/2026 | Conforme | [F #2](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/2) |
| 1.1-3 | Un message explicite si aucune séance ce jour | Choisir une date sans séance (dans 2 mois, hors festival) | Un message « aucune séance » s'affiche | visiteur | manuelle | 10/11/2026 | Conforme | [F #2](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/2) |

### US 1.2 – Consulter sans compte

| N° | Critère | Scénario | Attendu | Compte | Preuve | Date | Résultat | PR |
|---|---|---|---|---|---|---|---|---|
| 1.2-1 | Programmation accessible sans connexion | Fenêtre de navigation privée, ouvrir l'adresse de recette | La programmation s'affiche, sans demande de connexion | visiteur | manuelle | 10/11/2026 | Conforme | [F #2](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/2) |
| 1.2-2 | Accès à la réservation depuis une séance | Choisir une séance, puis « Réserver » | Le formulaire de réservation s'ouvre (la connexion n'est demandée qu'au moment de réserver, US 2.3) | visiteur | manuelle | 10/11/2026 | Conforme | [F #9](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/9) |
| 1.2-3 | Pages lisibles sur smartphone | Sur le téléphone : accueil, films, fiche d'un film, menu de l'en-tête | Lisibles sans zoom ni défilement horizontal | visiteur | manuelle | 10/11/2026 | Conforme | [F #17](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/17) |
| 1.2-4 | Bandeau des cookies ; refuser n'empêche pas de consulter | Navigation privée : « Tout refuser », puis parcourir la programmation ; « Cookies » en bas de page | Le bandeau disparaît, la programmation reste accessible ; « Cookies » le rouvre | visiteur | manuelle | 10/11/2026 | Conforme | [F #23](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/23) |

## Epic 2 – Réservation et choix des places en temps réel

### US 2.1 – Créer un compte et me connecter

| N° | Critère | Scénario | Attendu | Compte | Preuve | Date | Résultat | PR |
|---|---|---|---|---|---|---|---|---|
| 2.1-1 | Informations valides : compte créé et connecté | « Connexion », « Créer un compte », adresse `recette-<date>@example.com`, mot de passe saisi deux fois (« Confirmer le mot de passe ») ; l'œil affiche puis masque chaque saisie | Compte créé, prénom affiché dans l'en-tête | nouveau compte | manuelle | 10/11/2026 | Conforme | [B #21](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/21), [F #6](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/6), [F #33](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/33) |
| 2.1-2 | E-mail déjà utilisé : refus explicite | Se déconnecter, recréer un compte avec la même adresse | Création refusée, message « adresse déjà utilisée » | visiteur | manuelle | 10/11/2026 | Conforme | [B #21](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/21) |
| 2.1-3 | Identifiants incorrects : message qui ne dit pas lequel | Connexion avec un bon e-mail et un mauvais mot de passe, puis un e-mail inconnu | Le même message dans les deux cas | visiteur | manuelle | 10/11/2026 | Conforme | [B #21](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/21) |
| 2.1-4 | Mot de passe jamais stocké en clair | `SELECT "PasswordHash" FROM "Users" WHERE "Email" = 'recette-<date>@example.com';` sur le serveur | Une empreinte `AQAAAAIAAYag…`, jamais le mot de passe | — | base | 10/11/2026 | Conforme | [B #21](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/21) |
| 2.1-5 | Création conditionnée à la politique de confidentialité, liée | Formulaire de création : laisser la case décochée, puis suivre le lien | Création impossible sans la case ; le lien ouvre la politique | visiteur | manuelle | 10/11/2026 | Conforme | [B #40](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/40), [F #6](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/6) |
| 2.1-6 | Suppression du compte et de ses données personnelles | « Mon compte », « Supprimer mon compte », confirmer ; puis se reconnecter | Compte supprimé, reconnexion refusée ; les réservations payées restent, sans donnée personnelle | nouveau compte | manuelle et base | 10/11/2026 | Conforme | [B #40](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/40), [F #6](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/6) |

### US 2.2 – Voir les places restantes en temps réel

| N° | Critère | Scénario | Attendu | Compte | Preuve | Date | Résultat | PR |
|---|---|---|---|---|---|---|---|---|
| 2.2-1 | Chaque séance affiche ses places restantes | Accueil et fiche d'un film | Un nombre de places restantes sur chaque séance | visiteur | manuelle | 10/11/2026 | Conforme | [B #42](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/42), [F #8](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/8) |
| 2.2-2 | Places vendues ou en cours de paiement non comptées | Noter le nombre d'une séance ; bloquer 2 places (US 2.3) ; recharger dans une autre fenêtre | Le nombre a baissé de 2 | `client@` | manuelle | 10/11/2026 | Conforme | [B #42](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/42) |
| 2.2-3 | Séance complète : affichée « complète », réservation impossible | Une séance de 19 h 30 des 7 prochains jours (complète dans le seed) | « Complet », pas de bouton « Réserver » | visiteur | manuelle | 10/11/2026 | Conforme | [B #42](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/42), [F #8](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/8) |

### US 2.3 – Réserver un nombre de places et passer au paiement

| N° | Critère | Scénario | Attendu | Compte | Preuve | Date | Résultat | PR |
|---|---|---|---|---|---|---|---|---|
| 2.3-1 | Non connecté : invité à se connecter ou créer un compte | Visiteur : « Réserver » sur une séance | Page de connexion, puis retour à la réservation après connexion | visiteur | manuelle | 10/11/2026 | Conforme | [F #10](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/10) |
| 2.3-2 | Places bloquées 10 minutes au passage au paiement | Choisir 2 places, réserver | Page du blocage, compte à rebours de 10 minutes | `client@` | manuelle | 10/11/2026 | Conforme | [B #44](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/44), [B #45](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/45), [F #11](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/11) |
| 2.3-3 | Aucune réservation enregistrée avant paiement confirmé | Pendant le blocage : « Mon compte », « Mes billets » ; sur le serveur, `SELECT "Status" …` | Aucun billet ; statut `PendingPayment`, pas `Confirmed` | `client@` | manuelle et base | 10/11/2026 | Conforme | [B #44](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/44) |
| 2.3-4 | Dernière place visée par deux utilisateurs : un seul l'obtient | Non rejouable de façon fiable à la main (deux clics à la même milliseconde) | Test d'intégration de la [bug 0001](bugs/0001-double-booking.md), vert sur la dernière CI de `dev` | — | auto | 10/11/2026 | Conforme | [B #44](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/44) |
| 2.3-5 | 10 minutes sans paiement : places libérées, compteur à jour, message | Bloquer 1 place, attendre 10 minutes sans payer | Message d'expiration ; le nombre de places restantes remonte | `client@` | manuelle | 10/11/2026 | Conforme | [B #45](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/45), [F #12](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/12) |
| 2.3-6 | Plus de places qu'il n'en reste : refus avec message | Sur une séance presque pleine, demander plus que le reste | Refus, message « plus assez de places » | `client@` | manuelle | 10/11/2026 | Conforme | [B #44](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/44) |
| 2.3-7 | 10 places au plus | Essayer 11 places ; puis bloquer 10 places dans un onglet et 1 dans un autre | Impossible au-delà de 10, même avec plusieurs onglets | `client@` | manuelle | 10/11/2026 | Conforme | [B #46](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/46), [F #10](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/10) |

## Epic 3 – Paiement et billetterie dématérialisée

### US 3.1 – Payer ma réservation en ligne

| N° | Critère | Scénario | Attendu | Compte | Preuve | Date | Résultat | PR |
|---|---|---|---|---|---|---|---|---|
| 3.1-1 | Carte valide : paiement confirmé, réservation enregistrée, billet affiché, places vendues | Bloquer 2 places, accepter les CGV, « Payer », carte `4242…` | Retour sur le site, « Paiement confirmé », le billet s'affiche ; places restantes −2 | `client@` | manuelle | 10/11/2026 | Conforme | [B #47](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/47), [F #13](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/13) |
| 3.1-2 | Carte refusée : message, places bloquées jusqu'à la fin des 10 minutes, nouvel essai possible | Payer avec `4000 0000 0000 0002`, puis réessayer avec `4242…` avant la fin | Refus affiché par Stripe ; le second essai aboutit | `client@` | manuelle | 10/11/2026 | Conforme | [B #47](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/47) |
| 3.1-3 | Aucune donnée de carte stockée | Sur le serveur, la table `Reservations` : seules les colonnes de la référence Stripe (`PaymentReference`) | Aucune colonne ni valeur de carte | — | base | 10/11/2026 | Conforme | [B #47](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/47) |
| 3.1-4 | Paiement seulement après acceptation des CGV, liées | Laisser la case des CGV décochée ; suivre le lien | « Payer » impossible ; le lien ouvre les CGV | `client@` | manuelle | 10/11/2026 | Conforme | [B #47](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/47), [F #13](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/13) |

### US 3.2 – Recevoir un billet dématérialisé

| N° | Critère | Scénario | Attendu | Compte | Preuve | Date | Résultat | PR |
|---|---|---|---|---|---|---|---|---|
| 3.2-1 | Paiement confirmé : billet avec QR code unique, film, séance, salle, nombre de places | Après 3.1-1 : la page de la réservation | Film, date et heure, salle, « 2 places : … », QR code | `client@` | manuelle | 10/11/2026 | Conforme | [B #63](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/63), [F #24](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/24) |
| 3.2-2 | Billet accessible depuis mon compte | « Mon compte », « Mes billets », « Voir le billet » | La réservation payée est listée et rouvre le billet | `client@` | manuelle | 10/11/2026 | Conforme | [B #63](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/63), [F #24](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/24) |
| 3.2-3 | Paiement échoué : aucun billet | Après un paiement refusé (3.1-2, avant le second essai) : « Mes billets » | Aucun billet pour cette réservation | `client@` | manuelle | 10/11/2026 | Conforme | [B #63](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/63) |

## Epic 4 – Contrôle d'accès en salle

### US 4.1 – Scanner un billet

| N° | Critère | Scénario | Attendu | Compte | Preuve | Date | Résultat | PR |
|---|---|---|---|---|---|---|---|---|
| 4.1-1 | Billet valide pour la séance : accès autorisé, billet marqué « utilisé » | Le billet de 3.2-1 affiché sur l'ordinateur ; sur le téléphone, « Contrôle », la séance du billet, scanner | Bandeau vert « Accès autorisé », film et places | `inspector@` | manuelle | 10/11/2026 | Conforme | [B #64](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/64), [F #26](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/26) |
| 4.1-2 | Billet déjà utilisé : refus explicite | Scanner le même billet après 4 secondes | Bandeau rouge « Billet déjà utilisé. » | `inspector@` | manuelle | 10/11/2026 | Conforme | [B #64](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/64) |
| 4.1-3 | Autre séance, QR code inconnu ou illisible : refus explicite | Choisir une autre séance et scanner ; scanner un QR code quelconque ; saisir `bonjour` | « Ce billet est pour une autre séance. » ; « QR code inconnu ou illisible. » | `inspector@` | manuelle | 10/11/2026 | Conforme | [B #64](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/64), [F #26](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/26) |
| 4.1-4 | Retour visuel immédiat | Pendant 4.1-1 à 4.1-3 | Le bandeau change aussitôt après chaque scan | `inspector@` | manuelle | 10/11/2026 | Conforme | [F #26](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/26) |

## Epic 5 – Administration et reporting

### US 5.1 – Consulter les réservations en ligne (caissier)

| N° | Critère | Scénario | Attendu | Compte | Preuve | Date | Résultat | PR |
|---|---|---|---|---|---|---|---|---|
| 5.1-1 | Par séance, places vendues en ligne et restantes | « Réservations », aujourd'hui | Pour chaque séance : vendues en ligne, restantes ; la séance de 3.1-1 compte 2 places en ligne | `cashier@` | manuelle | 10/11/2026 | Conforme | [B #62](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/62), [F #22](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/22) |
| 5.1-2 | Accès en lecture seule | Parcourir la page | Aucun bouton d'action ; l'API ne lui ouvre aucune écriture | `cashier@` | manuelle | 10/11/2026 | Conforme | [B #62](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/62) |
| 5.1-3 | Pas d'accès à la programmation ni à l'analyse | En-tête ; puis ouvrir `/programming` à la main | Pas de lien « Programmation » ; `/programming` ramène au suivi des réservations (sa page, depuis le 07/11/2026) | `cashier@` | manuelle | 10/11/2026 | Conforme | [B #62](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/62), [F #22](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/22) |

### US 5.2 – Gérer la programmation

| N° | Critère | Scénario | Attendu | Compte | Preuve | Date | Résultat | PR |
|---|---|---|---|---|---|---|---|---|
| 5.2-1 | Séance créée (film, salle, date, heure) visible en consultation publique | « Programmation », ajouter une séance demain à 10 h ; ouvrir l'accueil à la date de demain | La séance apparaît dans la programmation publique | `manager@` | manuelle | 10/11/2026 | Conforme | [B #51](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/51), [F #16](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/16) |
| 5.2-2 | Séance modifiée ou supprimée : consultation publique à jour | Déplacer la séance à 11 h, puis l'annuler | L'accueil montre 11 h, puis plus de séance | `manager@` | manuelle | 10/11/2026 | Conforme | [B #51](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/51), [F #16](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/16) |
| 5.2-3 | Séance avec réservations : avertissement avant de valider | Modifier la séance de 3.1-1 | Avertissement avec le nombre de places prises, confirmation demandée | `manager@` | manuelle | 10/11/2026 | Conforme | [B #51](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/51), [F #16](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/16) |
| 5.2-4 | Seuls les profils autorisés | Avec `client@`, `cashier@` et `inspector@` : ouvrir `/programming` | `client@` revient à l'accueil ; `cashier@` et `inspector@` reviennent à leur page ; `403` de l'API dans tous les cas | `client@`, `cashier@`, `inspector@` | manuelle | 10/11/2026 | Conforme | [B #51](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/51) |

### US 5.3 – Suivre les réservations

| N° | Critère | Scénario | Attendu | Compte | Preuve | Date | Résultat | PR |
|---|---|---|---|---|---|---|---|---|
| 5.3-1 | Par séance, places vendues et capacité de la salle | « Réservations », aujourd'hui, puis une autre date | Vendues (en ligne, au guichet), capacité, taux de remplissage | `manager@` | manuelle | 10/11/2026 | Conforme | [B #55](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/55), [F #20](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/20) |
| 5.3-2 | Données conservées pour l'analyse future (US 5.4) | Sur le serveur : les réservations payées et expirées des jours passés sont toujours en base, avec leurs dates (`CreatedAt`, `PaidAt`) | Les lignes existent ; rien n'est supprimé à la fin d'une séance | — | base | 10/11/2026 | Conforme | [B #55](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/55) |

### US 5.6 – Gérer le catalogue des films (ajoutée au MVP le 07/11/2026)

Préparer deux images : une affiche JPEG ou PNG de moins de 2 Mo, et une image de plus de 2 Mo (ou un fichier texte renommé en `.jpg`).

| N° | Critère | Scénario | Attendu | Compte | Preuve | Date | Résultat | PR |
|---|---|---|---|---|---|---|---|---|
| 5.6-1 | Film ajouté (titre, synopsis, durée, genre, date de sortie, affiche) : proposé dans le formulaire des séances | « Programmation », « Ajouter un film au catalogue » (en haut de la page : la fenêtre s'ouvre) : titre `Recette <date>`, genre choisi dans la liste, la petite affiche ; puis « Nouvelle séance » | « … est ajouté au catalogue. » ; le film est déjà choisi dans « Nouvelle séance » | `manager@` | manuelle | 10/11/2026 | Conforme | [B #67](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/67), [F #30](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/30) , [F #35](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/35) |
| 5.6-2 | Une séance programmée : la fiche, affiche comprise, est publique | Programmer le film demain à 10 h ; en navigation privée, l'accueil à la date de demain, puis la fiche du film | Le film, son affiche et sa séance s'affichent ; l'affiche est servie par la recette (`/posters/…`) | `manager@`, visiteur | manuelle | 10/11/2026 | Conforme | [B #67](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/67), [F #30](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/30) |
| 5.6-3 | Fiche modifiée : consultation publique à jour ; durée figée tant qu'une séance n'est pas terminée | « Modifier un film du catalogue » (en haut de la page), choisir le film de 5.6-1 dans la fenêtre : changer le titre et l'affiche, « Enregistrer le film » ; puis changer sa durée | La fiche publique montre le nouveau titre et la nouvelle affiche ; le changement de durée est refusé : « Une séance de ce film n'est pas terminée… » | `manager@`, visiteur | manuelle | 10/11/2026 | Conforme | [B #73](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/73), [F #31](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/31) , [F #35](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/35) |
| 5.6-4 | Affiche JPEG, PNG ou WebP de 2 Mo au plus, sinon refus explicite | Ajouter un film avec l'image de plus de 2 Mo, puis avec le fichier texte renommé en `.jpg` (Swagger pour ce second cas, si le front le bloque) | Plus de 2 Mo : refus avant tout envoi ; fichier qui n'est pas une image : `400` sur `poster`, message affiché ; aucun film ajouté | `manager@` | manuelle | 10/11/2026 | Conforme | [B #67](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/67), [F #30](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/30) |
| 5.6-5 | Seul le responsable accède à cette fonction | Avec le jeton de `client@`, `cashier@` puis `inspector@` : `POST /api/programming/movies` (Swagger) | `403` pour les trois ; aucun lien « Programmation » dans leur en-tête | `client@`, `cashier@`, `inspector@` | manuelle | 10/11/2026 | Conforme | [B #67](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/67) |
| 5.6-6 | Film archivé : plus proposé à la programmation ; séances passées, ventes et billets gardés | Annuler la séance de 5.6-2 ; « Modifier un film du catalogue », le film d'essai, « Archiver ce film », « Confirmer l'archivage » ; puis, sur le serveur, `SELECT "Title", "ArchivedAt" FROM "Movies" WHERE "ArchivedAt" IS NOT NULL;` | « … est archivé… » ; le film n'est plus dans les listes de la page ; la ligne existe toujours, avec sa date d'archivage | `manager@` | manuelle et base | 10/11/2026 | Conforme | [B #73](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/73), [F #31](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/31) , [F #35](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/35) |

## Ajustements des droits par rôle (07/11/2026)

Hors critères des user stories : chaque rôle n'ouvre que les pages de son métier, et seul le spectateur achète (tableau des droits de la [sécurité](../security/security.md)).

| N° | Règle | Scénario | Attendu | Compte | Preuve | Date | Résultat | PR |
|---|---|---|---|---|---|---|---|---|
| A-1 | Seul le spectateur achète | Avec le jeton de `cashier@`, `inspector@` puis `manager@` : `POST /api/reservations` (Swagger ou `curl`) | `403` pour les trois ; `201` pour `client@` | `cashier@`, `inspector@`, `manager@`, `client@` | manuelle | 10/11/2026 | Conforme | [B #66](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/66) |
| A-2 | Le caissier et le contrôleur n'ouvrent que leurs pages | Après la connexion ; puis « Grand Cinéma », `/` et `/movies` | Arrivée et retour sur « Réservations » (`cashier@`), « Contrôle » (`inspector@`) ; « Mon compte » et les pages légales s'ouvrent | `cashier@`, `inspector@` | manuelle | 10/11/2026 | Conforme | [F #29](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/29) |
| A-3 | Achat proposé seulement au visiteur et au spectateur | En-tête (ordinateur et téléphone) ; fiche d'un film, choisir une séance | Pas de « Acheter un billet » pour le personnel ; le responsable voit « La réservation en ligne est réservée aux spectateurs. » | `manager@`, `cashier@`, `inspector@` | manuelle | 10/11/2026 | Conforme | [F #29](https://github.com/Grand-Cinema-Studi/grand-cinema-frontend/pull/29) |

## Anomalies relevées pendant la recette

Chaque anomalie est une issue `bug` du dépôt de la cause, rattachée au Project ; elle est corrigée en TDD (test qui la reproduit d'abord), puis le critère est rejoué.

| Critère | Issue | Gravité | Correctif | Rejoué le | Résultat |
|---|---|---|---|---|---|
| 1.1-1, 5.2-1 (données de démonstration) : Alien et Ratatouille à 22 h 30 en Salle 1 le 08/11/2026 | [B #79](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/issues/79) | Mineure | Séances de démonstration supprimées avant chaque rechargement, contrôle des chevauchements : `deploy/reset-demo-data.sh` ([bug 0002](bugs/0002-demo-screenings-overlap.md)) ; [B #68](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/68), [B #69](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/69), [B #71](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/pull/71) | 08/11/2026 | Conforme |

## Synthèse

| | Critères | Conforme | Conforme avec réserve | Non conforme |
|---|---|---|---|---|
| Epic 1 (US 1.1, 1.2) | 7 | 7 | 0 | 0 |
| Epic 2 (US 2.1, 2.2, 2.3) | 16 | 16 | 0 | 0 |
| Epic 3 (US 3.1, 3.2) | 7 | 7 | 0 | 0 |
| Epic 4 (US 4.1) | 4 | 4 | 0 | 0 |
| Epic 5 (US 5.1, 5.2, 5.3, 5.6) | 15 | 15 | 0 | 0 |
| Ajustements des droits | 3 | 3 | 0 | 0 |
| **Total** | **52** | **52** | **0** | **0** |

## Décision du Product Owner

- [x] MVP **accepté** : tous les critères sont conformes, ou leurs réserves sont acceptées ; la version peut être fusionnée dans `main` et étiquetée `v1.0.0` ([versioning](../development/versioning.md)).
- [ ] MVP **refusé** : les anomalies listées ci-dessus doivent être corrigées et rejouées avant la release.

Réserves acceptées : aucune

Date : `10 novembre 2026` · Product Owner : Slimane Smahi (rôle tenu dans le cadre du projet)
