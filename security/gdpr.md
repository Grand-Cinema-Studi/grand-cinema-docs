# Données personnelles (RGPD)

Quelles données personnelles l'application traite, pourquoi, combien de temps, et comment les droits des personnes sont respectés.

Les éléments marqués _(à venir)_ sont décidés mais pas encore réalisés. Les durées de conservation sont des propositions, à valider par le responsable du traitement.

## Rôles

| Rôle RGPD | Qui |
|---|---|
| Responsable du traitement | Le Grand Cinéma (sa direction) |
| Sous-traitant : hébergement | AWS, région Paris (`eu-west-3`) : les données restent dans l'Union européenne |
| Sous-traitant : paiement | Stripe, qui traite les données de carte sous sa propre conformité (PCI DSS) |
| Hébergement du code | GitHub : le code seulement, aucune donnée de spectateur |

## Données traitées

| Données | Où | Finalité | Base légale |
|---|---|---|---|
| Adresse e-mail, prénom, nom | table `Users` | créer le compte, se connecter, retrouver ses billets | exécution du contrat |
| Empreinte du mot de passe | table `Users` | authentification ; le mot de passe lui-même n'est jamais stocké | exécution du contrat |
| Rôle, date de création du compte | table `Users` | droits d'accès, gestion des comptes | exécution du contrat |
| Date et version de la politique de confidentialité acceptée à l'inscription | table `Users` (`PrivacyPolicyAcceptedAt`, `PrivacyPolicyVersion`) | prouver que la personne a été informée (article 13, principe de responsabilité) | obligation légale |
| Réservations : séance, places, montant, statut, dates | tables `Reservations`, `ReservedSeats` | vendre et contrôler les billets | exécution du contrat |
| Identifiant du paiement chez Stripe _(à venir)_ | table `Reservations` | rapprocher une réservation de son paiement | exécution du contrat, obligation comptable |
| Statistiques de fréquentation | calculées à partir des réservations | programmer les séances, anticiper les pics | intérêt légitime ; données agrégées, sans profil individuel |

**Minimisation** : aucune autre donnée n'est demandée (ni date de naissance, ni téléphone, ni adresse postale). Les données de carte bancaire ne passent jamais par l'application.

## Durées de conservation (proposition)

| Données | Durée | À l'issue |
|---|---|---|
| Compte | tant que le compte est actif ; 3 ans sans connexion | suppression du compte |
| Réservations et paiements | durée légale de conservation des pièces comptables (10 ans) | suppression |
| Blocages expirés ou annulés | les places sont libérées immédiatement ; la réservation reste avec son statut pour l'historique | anonymisée avec le compte |
| Journaux techniques | quelques semaines | suppression automatique |

À la suppression d'un compte, ses réservations ne sont pas effacées (obligation comptable, statistiques) mais **anonymisées** : elles ne sont plus rattachées à une personne identifiable. C'est la base qui le garantit : la clé étrangère `Reservations.UserId` est en `ON DELETE SET NULL` (migration `DetachReservationsOfDeletedAccounts`), vérifiée par un test d'intégration sur PostgreSQL.

## Droits des personnes

| Droit | Mise en œuvre |
|---|---|
| Information | page « Données personnelles » dans le front, accessible par un lien à côté de la case à cocher de l'inscription _(front à venir)_. L'API refuse l'inscription sans `acceptPrivacyPolicy: true` et enregistre la date et la version acceptées (fait côté API, 28/10/2026). La case atteste l'information ; la base légale du compte reste l'exécution du contrat |
| Accès, portabilité | profil consultable (`GET /api/users/me`) ; export des données du compte et des réservations, sur demande _(à venir)_ |
| Rectification | modification du nom et de l'adresse e-mail depuis l'espace personnel _(à venir)_ |
| Effacement | `DELETE /api/users/me` : suppression du compte, réservations gardées et détachées (fait côté API, 28/10/2026) ; bouton dans l'espace personnel, avec confirmation _(front à venir)_ |
| Contact | adresse de contact du cinéma, indiquée sur la page « Données personnelles » _(à venir)_ |

## Cookies et traceurs

- Seuls les cookies **strictement nécessaires** au fonctionnement (session, si ce mécanisme est retenu pour l'authentification) sont prévus : ils ne demandent pas de consentement.
- Aucun outil de mesure d'audience ni de publicité dans le MVP. S'il en était ajouté un, un bandeau de consentement deviendrait nécessaire.

## Sécurité des données

Les mesures techniques (hachage des mots de passe, HTTPS, base non exposée, secrets hors du code, droits par rôle) sont décrites dans la [sécurité](security.md).

## Données de test

Aucune donnée réelle de spectateur n'est utilisée hors production : les tests et la démonstration utilisent des données fictives (`seed-dev.sql`, données créées par les tests). C'est une règle de la stratégie de tests du Bloc 2.
