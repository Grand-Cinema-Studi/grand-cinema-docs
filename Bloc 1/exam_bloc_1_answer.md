# Dossier 1 : La demande du client

---

## Question 1 : Spécification de la demande au regard des objectifs du projet, identification des utilisateurs et description de leurs besoins

---

### 1. Contexte d'exploitation et diagnostic de l'existant

L'établissement **« Le Grand Cinéma »** constitue un pilier de la vie culturelle locale depuis plus de 50 ans[cite: 5]. Il dispose d'un parc de 10 salles représentant une capacité totale de **1 700 places**, réparties selon la typologie suivante[cite: 5] :

| Catégorie de salle | Nombre de salles | Capacité unitaire | Capacité cumulée | Spécificités et usages |
| :--- | :---: | :---: | :---: | :--- |
| **Premium / IMAX** | 2 | 300 places | 600 places | Grands écrans, son immersif, blockbusters et grands concerts[cite: 5]. |
| **Standard** | 4 | 180 places | 720 places | Exploitation généraliste, programmation grand public et films d'auteur[cite: 5]. |
| **VIP** | 2 | 120 places | 240 places | Confort supérieur, tarification premium, service personnalisé[cite: 5]. |
| **Événementielle / modulable** | 2 | 70 places | 140 places | Espace adaptable (projections privées, débats, masterclasses Jazz)[cite: 5]. |
| **TOTAL** | **10 salles** | — | **1 700 places** | **Parc modulable sous contrainte d'optimisation d'occupation**[cite: 5] |

#### Le constat opérationnel et financier
En exploitation courante, l'établissement accueille :
* Entre **800 et 1 200 spectateurs par jour** en semaine du lundi au vendredi[cite: 5] ;
* Entre **1 200 et 1 600 spectateurs par jour** le week-end[cite: 5].

Pendant le **Festival de Jazz**, événement d'envergure internationale accueillant des milliers de visiteurs, la fréquentation globale enregistre une hausse significative de **+30 %**[cite: 5]. Toutefois, ce flux additionnel ne se répartit pas de manière linéaire : il se caractérise par des **pics de réservation massifs et concentrés sur des créneaux horaires très restreints** (sorties de concerts, débuts de soirées, séances thématiques)[cite: 5].

Le cinéma fonctionne exclusivement avec une billetterie physique au guichet[cite: 5]. Ce modèle génère une rupture opérationnelle critique en période de forte affluence[cite: 5] :
1. **Saturation physique :** Des files d'attente ininterrompues excédant **30 minutes** se forment dans le hall[cite: 5].
2. **Perte sèche de chiffre d'affaires :** Face à cette friction, de nombreux spectateurs renoncent à l'achat (coût d'attrition élevé)[cite: 5].
3. **Sous-remplissage paradoxal :** Malgré une forte demande dans le hall, certaines séances démarrent avec des sièges inoccupés en raison du débit d'encaissement limité du guichet physique[cite: 5].
4. **Dégradation de l'image de marque :** L'expérience client globale est altérée, pénalisant la fidélisation des résidents et privant l'établissement des retombées positives de la clientèle touristique[cite: 5].

Face à ces blocages, la direction a acté la conception d'une **application web de réservation et de billetterie dématérialisée accessible 24 h/24 et 7 j/7**[cite: 5].

---

### 2. Démarcation des objectifs économiques et des enjeux métiers (Compétence C1.1)

Afin de satisfaire rigoureusement aux critères d'évaluation du référentiel CDA, la spécification distingue expressément les impératifs de rentabilité financière des nécessités d'excellence opérationnelle[cite: 5].

┌─────────────────────────────────────────────────────────────────────────┐
│                       ALIGNEMENT STRATÉGIQUE C1.1                       │
└────────────────────────────────────┬────────────────────────────────────┘
│
┌───────────────────────────┴───────────────────────────┐
▼                                                       ▼
┌──────────────────────────────────┐   ┌──────────────────────────────────┐
│       OBJECTIFS ÉCONOMIQUES      │   │      ENJEUX MÉTIERS / OPÉR.      │
│     (Rentabilité & Croissance)   │   │     (Processus & Expérience)     │
├──────────────────────────────────┤   ├──────────────────────────────────┤
│ • +30 % de CA sur 12 mois        │   │ • Temps d'accès réduit (< 2 min) │
│ • Éradication du taux d'abandon  │   │ • Fluidification du hall         │
│ • Optimisation du panier moyen   │   │ • Pilotage des jauges (1 700 pl) │
│ • Captation du flux touristique  │   │ • Concurrence transactionnelle   │
│ • Rentabilité sous budget limité │   │ • Disponibilité temps réel 24/7  │
└──────────────────────────────────┘   └──────────────────────────────────┘


#### A. Les objectifs économiques du projet
1. **Augmentation de 30 % de la fréquentation et du chiffre d'affaires sur 12 mois :** Capter le surcroît de demande induit par le Festival de Jazz tout en augmentant la fréquence de visite annuelle des résidents[cite: 5].
2. **Suppression du coût d'attrition lié à l'attente :** Convertir l'intégralité des intentions d'achat en supprimant le goulet d'étranglement du guichet physique responsable des abandons de vente[cite: 5].
3. **Maximisation du taux de remplissage (*Yield Management*) :** Améliorer l'occupation des 1 700 places réparties sur les 10 salles, notamment sur les séances creuses en semaine (800 à 1 200 spectateurs) et sur les salles à haute contribution de marge (Premium/IMAX et VIP)[cite: 5].
4. **Pénétration du segment touristique international :** Permettre l'achat à distance avant l'arrivée physique sur les lieux, via des paiements dématérialisés conformes aux standards internationaux[cite: 5].
5. **Optimisation du coût d'acquisition et d'exploitation :** Respecter l'enveloppe budgétaire limitée allouée au projet en concevant une infrastructure au coût d'hébergement ajusté, sans investissement lourd en matériel guichet supplémentaire[cite: 5].

#### B. Les enjeux métiers et opérationnels
1. **Éradication des files d'attente à l'entrée :** Remplacer une transaction physique de 3 à 5 minutes par le scan d'un e-billet dématérialisé en moins de 2 secondes, réduisant le temps d'attente moyen de plus de 30 minutes à moins de 2 minutes par personne[cite: 5].
2. **Désengorgement et sécurisation des espaces physiques :** Fluidifier la circulation dans le hall d'accueil pour respecter les normes de sécurité des Établissements Recevant du Public (ERP)[cite: 5].
3. **Gouvernance dynamique de la programmation :** Fournir à la direction un back-office unifié pour configurer l'affectation des films aux 10 salles (en tenant compte de la modularité des salles événementielles de 70 places)[cite: 5].
4. **Résilience et intégrité transactionnelle lors des pics de charge :** Garantir que lors des réservations simultanées massives du festival, le système empêche rigoureusement tout surbooking grâce à une gestion synchrone des verrous de réservation[cite: 5].
5. **Autonomie des équipes opérationnelles :** Doter le personnel de salle d'un outil de validation des accès fiable, utilisable sur terminal mobile, fonctionnant même en cas de micro-coupure réseau locale[cite: 5].

---

### 3. Typologie des utilisateurs (Personas d'ingénierie)

Pour adresser l'ensemble des cas d'usage, quatre profils types ont été modélisés :

┌────────────────────────────────────────────────────────────────────────┐
│                   ÉCOSYSTÈME DES PERSONAS IDENTIFIÉS                   │
├───────────────────────────────────┬────────────────────────────────────┤
│ 1. MARC DUPONT (42 ans)           │ 2. ELENA ROSTOVA (28 ans)          │
│ • Profil : Spectateur Résident    │ • Profil : Festivalière / Touriste │
│ • Contexte : Habitué local        │ • Contexte : Étrangère, mobile-only│
│ • Attente : Réservation express   │ • Attente : Parcours instantané    │
├───────────────────────────────────┼────────────────────────────────────┤
│ 3. THOMAS MARTIN (24 ans)         │ 4. SOPHIE LAURENT (51 ans)         │
│ • Profil : Agent d'Accueil        │ • Profil : Directrice Exploitation │
│ • Contexte : Contrôle des portes  │ • Contexte : Gestionnaire du parc  │
│ • Attente : Scan < 2 secondes     │ • Attente : Monitoring des jauges  │
└───────────────────────────────────┴────────────────────────────────────┘


#### Persona 1 : Le Spectateur Régulier (« Marc Dupont », 42 ans)
* **Description :** Cadre résidant dans l'agglomération, amateur de cinéma, vient 2 à 3 fois par mois en famille ou entre amis, principalement le week-end[cite: 5].
* **Comportement numérique :** Utilise un ordinateur portable au bureau et un smartphone au quotidien ; familier des plateformes e-commerce.
* **Problèmes rencontrés :** Frustré de devoir faire la queue 30 minutes le samedi soir ou de trouver une séance complète une fois arrivé au guichet[cite: 5].
* **Besoins opérationnels :**
  * Consulter les horaires et les fiches descriptives des films depuis son domicile[cite: 5].
  * Réserver ses places avec confirmation instantanée[cite: 5].
  * Visualiser la disponibilité des places en temps réel[cite: 5].
  * Conserver ses billets dématérialisés dans son smartphone[cite: 5].

#### Persona 2 : La Festivalière / Touriste (« Elena Rostova », 28 ans)
* **Description :** Touriste étrangère présente dans la ville pour 4 jours lors du Festival de Jazz ; dispose d'un planning rythmé par les concerts[cite: 5].
* **Comportement numérique :** Usage exclusif du smartphone (*mobile-first*), consulte le web via son réseau 4G/5G en itinérance ; ne maîtrise pas parfaitement la langue locale.
* **Problèmes rencontrés :** Ne dispose pas du temps nécessaire pour se rendre au cinéma avant la séance pour faire la queue[cite: 5] ; craint les processus d'inscription longs.
* **Besoins opérationnels :**
  * Tunnel d'achat direct sans obligation de créer un compte lourd (*Guest Checkout*).
  * Sélection rapide de la séance avec indication claire du format de salle (Premium/IMAX, VIP)[cite: 5].
  * Paiement immédiat sécurisé par carte bancaire internationale ou portefeuille mobile (Apple Pay, Google Pay).
  * Obtention instantanée d'un e-billet avec QR code accessible hors-ligne[cite: 5, 6].

#### Persona 3 : L'Agent d'Accueil et de Contrôle (« Thomas Martin », 24 ans)
* **Description :** Salarié opérationnel du cinéma, posté aux portes d'accès des 10 salles lors des entractes et démarrages de séances[cite: 5].
* **Contexte de travail :** Doit faire entrer jusqu'à 600 personnes en moins de 15 minutes sur les 2 salles Premium/IMAX[cite: 5].
* **Problèmes rencontrés :** Fatigue et stress causés par la gestion manuelle de billets papier, les risques d'usurpation et les réclamations des clients excédés par l'attente[cite: 5].
* **Besoins opérationnels :**
  * Application web mobile de scan dédiée, réactive et simple d'utilisation[cite: 5].
  * Validation unitaire d'un billet en moins de 2 secondes via lecture optique du QR code[cite: 5].
  * Détection immédiate des anomalies (billet invalide, séance erronée, billet déjà utilisé).
  * Indicateur en temps réel du nombre de personnes déjà admises par rapport à la jauge globale de la salle.

#### Persona 4 : La Directrice d'Exploitation (« Sophie Laurent », 51 ans)
* **Description :** Responsable de la gestion financière, de la programmation des 10 salles et des équipes du Grand Cinéma[cite: 5].
* **Contexte de travail :** Travaille sur poste fixe et tablette ; doit arbitrer l'allocation des salles selon l'actualité culturelle et le Festival de Jazz[cite: 5].
* **Problèmes rencontrés :** Manque total de visibilité sur les préventes ; incapacité d'anticiper la programmation faute de métriques en temps réel[cite: 5].
* **Besoins opérationnels :**
  * Tableau de bord synthétique affichant les taux d'occupation des 1 700 places en direct[cite: 5].
  * Interface d'administration pour créer, modifier ou annuler des séances et affecter les films aux salles[cite: 5].
  * Paramétrage des jauges pour les salles modulables (70 à 140 places)[cite: 5].
  * Outils d'analyse statistique de la fréquentation pour mesurer l'atteinte des +30 % de croissance[cite: 5].

---

### 4. Backlog d'EPICs et structuration du tableau Trello

En application des méthodes agiles d'ingénierie logicielle (référentiel CDA), les besoins métiers sont formalisés sous la forme d'un backlog hiérarchisé découpé en **4 EPICs majeurs**[cite: 6].

> **Lien vers l'espace de gestion agile (Trello public) :**  
> `https://trello.com/b/GrandCinema-CDA-Backlog` *(Lien accessible publiquement en lecture sans authentification)*[cite: 5, 6]

┌────────────────────────────────────────────────────────────────────────┐
│               STRUCTURE DU TABLEAU DE BORD AGILE TRELLO                │
├───────────────────┬───────────────────┬────────────────────────────────┤
│ EPIC 1 : CATALOGUE│ EPIC 2 : TUNNEL   │ EPIC 3 : BILLETS & CONTRÔLE    │
│ • US 1.1 Filtrage │ • US 2.1 Jauge live│ • US 3.1 Génération QR code    │
│ • US 1.2 Fiche film│ • US 2.2 Verrou TTL│ • US 3.2 Scan agent PWA       │
│ • US 1.3 Horaires │ • US 2.3 Paiement │ • US 3.3 Contrôle anti-doublon │
├───────────────────┴───────────────────┴────────────────────────────────┤
│ EPIC 4 : ADMINISTRATION & PILOTAGE                                     │
│ • US 4.1 Gestion planning/salles • US 4.2 Jauges modulables • US 4.3 KPI│
└────────────────────────────────────────────────────────────────────────┘


#### EPIC 1 : Consultation et Découverte du Catalogue (Parcours Client)
* **Description :** Fournir une interface web responsive performante permettant à tout visiteur de consulter l'offre cinématographique et événementielle du Grand Cinéma sans friction[cite: 5].
* **Valeur métier & économique :** Maximiser la conversion visiteur-acheteur, mettre en avant les séances à forte rentabilité (IMAX, VIP) et exposer l'offre spécifique au Festival de Jazz[cite: 5].
* **User Stories associées :**
  * *US 1.1 :* En tant que spectateur, je souhaite filtrer la programmation par date, par genre et par type de salle (IMAX, Standard, VIP, Événementielle) afin de trouver rapidement la séance adaptée à mes envies[cite: 5].
  * *US 1.2 :* En tant que festivalier, je souhaite consulter la fiche détaillée d'un film (synopsis, durée, bande-annonce, classification) afin de valider mon choix culturel[cite: 5].
  * *US 1.3 :* En tant qu'utilisateur mobile, je souhaite consulter la grille horaire avec un temps de chargement inférieur à 1 seconde, même sur un réseau cellulaire encombré[cite: 5].

#### EPIC 2 : Réservation et Gestion des Jauges en Temps Réel (Flux Transactionnel)
* **Description :** Permettre la sélection et l'achat sécurisé de places avec synchronisation instantanée des disponibilités pour éliminer tout risque de surréservation lors des pics d'affluence[cite: 5, 6].
* **Valeur métier & économique :** Supprimer l'attente physique de 30 minutes, sécuriser l'encaissement 24 h/24 et fiabiliser la concurrence d'accès sur les 1 700 places du parc[cite: 5].
* **User Stories associées :**
  * *US 2.1 :* En tant que client, je souhaite visualiser en temps réel le nombre exact de places encore disponibles sur la séance choisie pour éviter les déplacements inutiles[cite: 5].
  * *US 2.2 :* En tant que client, je souhaite que mes places soient temporairement réservées (panier avec compte à rebours de 10 minutes) pendant la phase de paiement afin d'empêcher qu'un autre internaute ne finalise la commande à ma place[cite: 5, 6].
  * *US 2.3 :* En tant que festivalier, je souhaite payer via une interface sécurisée (CB, Apple Pay/Google Pay) sans obligation de création de compte lourd (*Guest Checkout*)[cite: 5, 6].

#### EPIC 3 : Billetterie Dématérialisée et Contrôle d'Accès (Opérations & Sécurité)
* **Description :** Émettre automatiquement des titres d'accès dématérialisés infalsifiables et fournir aux agents d'accueil un dispositif rapide de validation aux portes des 10 salles[cite: 5].
* **Valeur métier & économique :** Réduire le temps d'accès unitaire en salle à moins de 2 secondes, éliminer les coûts de consommables papier et éradiquer la fraude[cite: 5].
* **User Stories associées :**
  * *US 3.1 :* En tant que client ayant payé, je souhaite recevoir immédiatement mon e-billet par email et pouvoir le télécharger sous forme de PDF / afficher son QR code sur mon smartphone[cite: 5].
  * *US 3.2 :* En tant qu'agent d'accueil, je souhaite scanner le QR code d'un spectateur depuis mon terminal mobile pour valider son accès en moins de 2 secondes[cite: 5].
  * *US 3.3 :* En tant qu'agent d'accueil, je souhaite que le système m'alerte visuellement et de façon sonore si un billet a déjà été scanné, afin d'interdire les réutilisations frauduleuses.

#### EPIC 4 : Administration, Programmation et Pilotage Métier (Back-Office Direction)
* **Description :** Offrir à l'équipe de direction une interface centralisée pour planifier la programmation, configurer la modularité des salles et monitorer les indicateurs de performance commerciale[cite: 5].
* **Valeur métier & économique :** Piloter l'objectif des +30 % de croissance annuelle, optimiser l'occupation des 1 700 places et adapter l'exploitation en direct face aux affluences du Festival de Jazz[cite: 5].
* **User Stories associées :**
  * *US 4.1 :* En tant que directrice d'exploitation, je souhaite créer, modifier et planifier des séances en affectant un film, une salle et un tarif afin de construire le planning hebdomadaire[cite: 5].
  * *US 4.2 :* En tant que gestionnaire, je souhaite moduler la capacité des salles événementielles (ex. fixer la jauge à 70 places ou la fusionner à 140 places) selon le format de l'événement programmé[cite: 5].
  * *US 4.3 :* En tant que responsable financière, je souhaite visualiser un tableau de bord analytique présentant le taux d'occupation par salle, le panier moyen et l'évolution globale de la fréquentation[cite: 5].

---
---

## Question 2 : Cadrage du Minimum Viable Product (MVP), justifications d'arbitrage et trajectoire d'évolutivité

---

### 1. Définition académique et démarche stratégique du MVP

Le concept de **Minimum Viable Product (MVP)**, formalisé par Frank Robinson en 2001 et théorisé par Eric Ries au sein de la méthodologie *Lean Startup*, désigne la version opérationnelle d'un produit logiciel qui permet de collecter le maximum d'enseignements validés sur ses utilisateurs avec le minimum d'investissement initial en temps de développement et en ressources financières.

Dans le cadre d'une démarche d'ingénierie logicielle de niveau 6, le MVP ne doit en aucun cas être assimilé à :
* Un **Proof of Concept (POC)**, dont l'objet est purement d'éprouver la faisabilité technologique en laboratoire ;
* Un **prototype jetable**, dénué de standards de qualité industrielle ;
* Une **version dégradée ou instable**, qui altérerait l'image de l'établissement[cite: 5].

┌────────────────────────────────────────────────────────────────────────┐
│                   APPROCHE QUALITATIVE DU SOCLE MVP                    │
├───────────────────────────────────┬────────────────────────────────────┤
│ CE QUE LE MVP EST                 │ CE QUE LE MVP N'EST PAS            │
├───────────────────────────────────┼────────────────────────────────────┤
│ • Une application de production   │ • Un prototype jetable (POC)       │
│ • Un système transactionnel stable│ • Un projet inachevé ou instable   │
│ • Une réponse au ROI immédiat     │ • Une simple liste de souhaits     │
│ • Un socle architectural ouvert   │ • Une architecture rigide / figée  │
└───────────────────────────────────┴────────────────────────────────────┘


Pour Le Grand Cinéma, le MVP représente le **socle transactionnel minimal requis pour résoudre immédiatement le point de rupture économique** (éradication de la file de 30 minutes, captation des ventes du Festival de Jazz, respect de l'enveloppe budgétaire limitée)[cite: 5]. Il s'inscrit au cœur de la boucle itérative *« Construire - Mesurer - Apprendre »* :

 ┌──────────────────────────────────────────────────────────┐
 │                 BOUCLE LEAN STARTUP DU MVP               │
 └─────────────────────────────┬────────────────────────────┘
                               │
                ┌──────────────▼─────────────┐
                │        CONSTRUIRE          │
                │   (Socle transactionnel    │
                │      billetterie V1)       │
                └──────────────┬─────────────┘
                               │
                               ▼
┌──────────────────────────┐               ┌──────────────────────────┐
│        APPRENDRE         │               │         MESURER          │
│ (Ajustement ergonomie,   │◄──────────────┤ (Taux de conversion,     │
│  préparation V2 plan 2D) │               │  taux d'occupation, flux)│
└──────────────────────────┘               └──────────────────────────┘


---

### 2. Périmètre fonctionnel retenu pour le MVP (Version 1)

Pour répondre aux contraintes du cahier des charges tout en respectant l'enveloppe budgétaire limitée[cite: 5], le MVP se concentre sur **l'axe critique de vente et de contrôle d'accès**[cite: 5, 6] :

1. **Consultation de la programmation (Front-office Web Responsive) :**
   * Affichage de la liste des séances ordonnée par date et par salle[cite: 5].
   * Fiche synthétique du film (titre, durée, classification, résumé court)[cite: 5].
   * Affichage transparent des tarifs applicables[cite: 5].
2. **Gestion de jauge globale en temps réel :**
   * Calcul dynamique et affichage du solde de places disponibles par séance (ex. *« 42 places restantes sur 300 »*)[cite: 5].
   * Mécanisme de réservation avec verrouillage temporaire (panier avec TTL de 10 minutes) pour sécuriser l'intégrité de l'inventaire lors des montées en charge[cite: 5, 6].
3. **Paiement sécurisé et émission dématérialisée :**
   * Intégration d'une passerelle de paiement tierce conforme PCI-DSS (ex. Stripe Checkout / carte bancaire)[cite: 6].
   * Parcours client simplifié avec option d'achat express sans compte (*Guest Checkout*)[cite: 5, 6].
   * Émission automatique d'un e-billet dématérialisé avec QR code cryptographique unique téléchargeable en PDF ou affichable sur smartphone[cite: 5, 6].
4. **Contrôle d'accès opérationnel :**
   * Application web mobile (PWA) sécurisée pour les agents d'accueil, exploitant la caméra du terminal pour scanner et valider les billets en moins de 2 secondes[cite: 5].
   * Marqueur d'état immédiat du billet (*« Valide »*, *« Déjà utilisé »*, *« Séance incorrecte »*).
5. **Back-office d'exploitation basique :**
   * Interface d'authentification pour l'équipe de direction[cite: 5].
   * Formulaire de création / modification des séances (choix du film, de la salle parmi les 10, de l'horaire et de la jauge)[cite: 5].
   * Tableau de bord récapitulatif du nombre de places vendues par séance[cite: 5].

---

### 3. Matrice d'arbitrage fonctionnel et trajectoire d'évolutivité (V1 / V2 / V3)

Afin de garantir que la solution ne soit pas figée et réponde aux exigences d'évolutivité du critère C1.1[cite: 5], le périmètre est ventilé par version avec justifications objectives des reports[cite: 6] :

┌────────────────────────────────────────────────────────────────────────┐
│               TRAJECTOIRE D'ÉVOLUTIVITÉ APPLICATIVE                    │
├───────────────────┬───────────────────┬────────────────────────────────┤
│      MVP (V1)     │    VERSION 2      │          VERSION 3             │
│   (SOCLE VITAL)   │ (CONFORT & USAGES)│    (OPTIMISATION AVANCÉE)      │
├───────────────────┼───────────────────┼────────────────────────────────┤
│ • Catalogue web   │ • Plan 2D sièges  │ • Yield management dynamique   │
│ • Jauge temps réel│ • Comptes clients │ • API billetterie externe      │
│ • Tunnel d'achat  │ • Programme fidel.│   (Office tourisme / Festival) │
│ • QR code & scan  │ • Pass Festival   │ • Bornes physiques connectées  │
│ • Admin basique   │ • PWA spectateur  │ • Analyse prédictive IA        │
└───────────────────┴───────────────────┴────────────────────────────────┘


| Fonctionnalité | Version | Niveau de criticité | Justification technique et économique de l'arbitrage |
| :--- | :---: | :---: | :--- |
| **Affichage du catalogue & séances** | **MVP (V1)** | **Indispensable** | Prérequis absolu pour exposer l'offre du cinéma et permettre l'acte d'achat à distance[cite: 5]. |
| **Jauge de disponibilité en temps réel** | **MVP (V1)** | **Critique** | Résout le problème des séances mal optimisées et empêche le surbooking en cas de forte affluence[cite: 5]. |
| **Panier avec verrouillage TTL (10 min)** | **MVP (V1)** | **Critique** | Empêche les conflits de concurrence transactionnelle (*race conditions*) lors des pics du festival[cite: 5, 6]. |
| **Paiement sécurisé dématérialisé** | **MVP (V1)** | **Indispensable** | Élimine la file d'attente physique de 30 minutes et sécurise l'encaissement direct[cite: 5]. |
| **Génération du e-billet avec QR code** | **MVP (V1)** | **Indispensable** | Permet la dématérialisation complète du titre d'accès du spectateur[cite: 5]. |
| **Application de contrôle d'accès agent** | **MVP (V1)** | **Indispensable** | Évite de déplacer le bouchon de 30 minutes du guichet de vente vers la porte d'entrée de la salle[cite: 5]. |
| **Back-office basique (séances & jauges)**| **MVP (V1)** | **Indispensable** | Permet à l'exploitant de gérer les 10 salles modulables de manière autonome[cite: 5]. |
| **Plan de salle interactif 2D (siège précis)**| **V2** | **Reporté** | **Complexité excessive :** Les salles sont modulables (notamment les salles de 70 places)[cite: 5]. Modéliser un plan interactif dynamique ferait exploser le budget et les délais de développement[cite: 5, 6]. La jauge globale répond à 100 % du besoin d'encaissement immédiat[cite: 6]. |
| **Espace compte client & Historique** | **V2** | **Reporté** | Priorité accordée à la friction minimale d'achat pour les festivaliers (*Guest Checkout*)[cite: 5, 6]. |
| **Pass multi-séances & Réservations groupes** | **V2** | **Reporté** | Nécessite des règles de contingentement complexes qui retarderaient le lancement du socle critique[cite: 5, 6]. |
| **Plan interactif 3D immersif** | **V3** | **Exclu** | Coût de modélisation prohibitif, surconsommation de bande passante mobile, valeur ajoutée marginale pour un cinéma de centre-ville[cite: 5]. |
| **Yield Management automatisé** | **V3** | **Reporté** | L'optimisation algorithmique des tarifs selon l'offre et la demande nécessite d'accumuler un historique d'au moins 6 à 12 mois de ventes en production[cite: 5]. |
| **API Billetterie Partenaires** | **V3** | **Reporté** | Interconnexion avec l'Office de Tourisme et la plateforme officielle du Festival de Jazz ; dépendances contractuelles et techniques tierces à traiter une fois le système éprouvé[cite: 5]. |

---

### 4. Anticipation architecturale : concevoir une solution non figée (Compétence C1.1)

Pour satisfaire au critère d'évaluation C1.1 (*« intégrer les évolutions probables, afin de ne pas produire une solution figée par ses choix techniques et architecturaux initiaux »*)[cite: 5], l'ingénierie du MVP intègre en amont les fondations des versions ultérieures :

┌────────────────────────────────────────────────────────────────────────┐
│              DÉCOUPLAGE ARCHITECTURAL EXTENSIBLE (API-FIRST)           │
└───────────────────────────────────┬────────────────────────────────────┘
│
┌────────────────────┴────────────────────┐
▼                                         ▼
┌─────────────────────────────┐           ┌─────────────────────────────┐
│    CANAUX CLIENTS MVP       │           │   ÉVOLUTIONS V2 / V3        │
│ • Web Responsive (React/Vue)│           │ • App Mobile Native (PWA V2)│
│ • PWA Scan Agent d'Accueil  │           │ • API Partenaires (Tourisme)│
└──────────────┬──────────────┘           └──────────────┬──────────────┘
│                                         │
└────────────────────┬────────────────────┘
│ Requêtes JSON / HTTPS
▼
┌────────────────────────────────────────────────────────────────────────┐
│                   API BACKEND DÉCOUPLÉE (RESTful)                      │
│        (Authentification JWT, Gestion des Paniers, Moteur d'Achat)     │
└───────────────────────────────────┬────────────────────────────────────┘
│
┌──────────────────────────┴──────────────────────────┐
▼                                                     ▼
┌──────────────────────────────────┐         ┌──────────────────────────────────┐
│    COUCHE CACHE / CONCURRENCE    │         │       BASE RELATIONNELLE         │
│ • Redis (TTL 10 min paniers)     │         │ • Tables modulables              │
│ • Gestion des verrous de jauge   │         │ • Entité Siege prévue pour la V2 │
└──────────────────────────────────┘         └──────────────────────────────────┘


#### A. Modélisation de données extensible (SGBD Relationnel)
Bien que le MVP ne propose pas le choix du siège sur plan 2D, le schéma relationnel de la base de données est conçu dès l'origine pour l'accueillir sans restructuration destructrice :
* La table `Reservation` est liée à la table `Seance` par une table de liaison `Billet`.
* Une table `Siege` (champs : `id`, `salle_id`, `rangee`, `numero`, `type_fauteuil`) est modélisée dès la phase de conception.
* En **V1 (MVP)**, l'attribut `siege_id` de la table `Billet` est simplement conservé à l'état `NULL` (placement libre avec décrémentation de la jauge globale)[cite: 6].
* En **V2**, l'activation du plan interactif nécessitera uniquement d'instancier les enregistrements de la table `Siege` et d'assigner l'identifiant lors de la réservation, sans aucune altération du modèle physique de données existant ni perte d'historique[cite: 6].

#### B. Architecture découplée en micro-services logiques (*API-First*)
* La solution sépare formellement le Frontend (Application Web Single Page responsive) du Backend (API RESTful)[cite: 6].
* Cette séparation garantit que l'exposition future de points d'accès pour des tiers (API Office de Tourisme en V3) ou le développement d'une application mobile grand public (V2) consommeront exactement les mêmes services métier (authentification, calcul de jauge, génération de billets) sans réécriture de code applicatif[cite: 6].

#### C. Isolation de la brique de verrouillage de concurrence
* La réservation temporaire (TTL de 10 minutes) est découplée de la base transactionnelle principale via un cache clé-valeur rapide en mémoire vive (Redis)[cite: 6].
* Cette architecture protège le système contre les goulots d'étranglement lors des pics du Festival de Jazz et permet de brancher ultérieurement un algorithme d'ajustement dynamique des prix (Yield Management V3) directement sur les flux de consultation sans ralentir l'encaissement[cite: 5, 6].