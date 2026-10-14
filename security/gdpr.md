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

À la suppression d'un compte, ses réservations ne sont pas effacées (obligation comptable, statistiques) mais **anonymisées** : elles ne sont plus rattachées à une personne identifiable.

## Droits des personnes _(à venir)_

| Droit | Mise en œuvre |
|---|---|
| Information | page « Données personnelles » dans le front, accessible depuis la création de compte |
| Accès, portabilité | export des données du compte et des réservations, depuis l'espace personnel ou sur demande |
| Rectification | modification du nom et de l'adresse e-mail depuis l'espace personnel |
| Effacement | suppression du compte depuis l'espace personnel (avec anonymisation des réservations) |
| Contact | adresse de contact du cinéma, indiquée sur la page « Données personnelles » |

## Cookies et traceurs

- Seuls les cookies **strictement nécessaires** au fonctionnement (session, si ce mécanisme est retenu pour l'authentification) sont prévus : ils ne demandent pas de consentement.
- Aucun outil de mesure d'audience ni de publicité dans le MVP. S'il en était ajouté un, un bandeau de consentement deviendrait nécessaire.

## Sécurité des données

Les mesures techniques (hachage des mots de passe, HTTPS, base non exposée, secrets hors du code, droits par rôle) sont décrites dans la [sécurité](security.md).

## Données de test

Aucune donnée réelle de spectateur n'est utilisée hors production : les tests et la démonstration utilisent des données fictives (`seed-dev.sql`, données créées par les tests). C'est une règle de la stratégie de tests du Bloc 2.
