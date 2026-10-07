# Planning Fast-Track — Bloc 1 : Conception & Spécification
**Sujet d'examen officiel :** Le Grand Cinéma (Plateforme web de réservation et billetterie dématérialisée)  
**Titre Professionnel :** Concepteur Développeur d'Applications (CDA — Niveau 6)  
**Période d'exécution :** Du lundi 7 septembre au samedi 12 septembre 2026 (1 semaine chrono)  
**Volume horaire total :** 18 heures (6 créneaux sanctuarisés de 3 h)  
**Format de rendu imposé :** Fichier `Examen_B1_copiearendre_NOM_Prenom.pdf` + liens publics logiciels + schéma d'architecture obligatoire.

---

## 1. Vue Synthétique du Sprint Éclair (18 h)

| Session | Date & Horaires | Dossier ciblé | Livrables opérationnels à produire |
| :--- | :--- | :--- | :--- |
| **Session 1** | Lundi 07/09 (14h-17h) | **Dossier 1 — Q1** | Cadrage des objectifs business, personas & Backlog d'EPICs (Trello) |
| **Session 2** | Mardi 08/09 (14h-17h) | **Dossier 1 — Q2** | Définition théorique du MVP, matrice d'arbitrage MVP vs V2/V3 |
| **Session 3** | Mercredi 09/09 (14h-17h) | **Dossier 2 — Q1 & Q2** | État de l'art des solutions hors langages & grille d'arbitrage retombées |
| **Session 4** | Jeudi 10/09 (14h-17h) | **Dossier 2 — Q3** | Chiffrage financier TCO (CAPEX/OPEX, licences, hébergement, délais) |
| **Session 5** | Vendredi 11/09 (14h-17h) | **Dossier 3 — Q1 & Q2** | Choix d'hébergement sous contrainte budgétaire + Schéma d'architecture obligatoire |
| **Session 6** | Samedi 12/09 (09h-12h) | **Dossier 3 — Q3 & Rendu** | Justification de l'évolutivité, intégration dans la copie officielle & export PDF |

---

## 2. Déroulé Opérationnel Détaillé Session par Session

### Session 1 — Lundi 07/09/2026 (14h00 - 17h00 | 3 h)
* **Dossier traité :** Dossier 1 — La demande du client (Question 1)
* **Compétence :** C1.1 — Spécifier la demande du client au regard des objectifs
* **Actions concrètes :**
  1. Formalisation des objectifs stratégiques : hausse de fréquentation (+30% sur 12 mois), réduction du temps d'attente (< 5 min au lieu de > 30 min), optimisation du remplissage des 10 salles (1 700 places).
  2. Identification des types d'utilisateurs (Spectateur régulier, Festivalier / Touriste, Caissier / Contrôleur, Responsable Programmation / Direction).
  3. Création du tableau de bord Trello public et rédaction structurée des EPICs (Consultation & Programmation, Réservation & Choix des places en temps réel, Paiement & Billetterie dématérialisée, Administration & Reporting).

---

### Session 2 — Mardi 08/09/2026 (14h00 - 17h00 | 3 h)
* **Dossier traité :** Dossier 1 — La demande du client (Question 2)
* **Compétence :** C1.1 — Définition et cadrage du MVP
* **Actions concrètes :**
  1. Définition académique et professionnelle du concept de MVP (Minimum Viable Product).
  2. Sélection rigoureuse du périmètre fonctionnel MVP pour "Le Grand Cinéma" (consultation catalogue, réservation avec jauge temps réel, paiement Stripe, génération QR code).
  3. Justification des exclusions du MVP reportées en V2/V3 (choix précis du siège sur plan 2D/3D, programmes fidélité, réservations groupes pour festivals) pour respecter le critère d'évolutivité du jury sans figer l'architecture.

---

### Session 3 — Mercredi 09/09/2026 (14h00 - 17h00 | 3 h)
* **Dossier traité :** Dossier 2 — Les solutions (Questions 1 & 2)
* **Compétence :** C1.2 — État de l'art et faisabilité technico-économique
* **Actions concrètes :**
  1. Benchmark comparatif de solutions du marché (hors langages) :
     * Billetteries SaaS propriétaires (ex. Weezevent, Billetweb).
     * CMS e-commerce / extensions (WordPress/WooCommerce billetterie).
     * Architecture sur-mesure combinant Headless CMS / BaaS (Strapi / Supabase / Firebase) et passerelles de paiement spécialisées (Stripe Checkout).
  2. Grille d'évaluation multicritères (Coût, Délais, Autonomie, Gestion des pics du Festival de Jazz, Commission par billet).
  3. Détermination et justification argumentée de la solution retenue face aux contraintes du Grand Cinéma.

---

### Session 4 — Jeudi 10/09/2026 (14h00 - 17h00 | 3 h)
* **Dossier traité :** Dossier 2 — Les solutions (Question 3)
* **Compétence :** C1.2 — Évaluation financière et analyse d'impact
* **Actions concrètes :**
  1. Élaboration du modèle de coût complet (TCO sur 1 an et 3 ans) :
     * Coûts d'implémentation initiale (jours/hommes de développement).
     * Coûts récurrents (abonnements SaaS, commissions de paiement, hébergement cloud, nom de domaine/SSL).
  2. Analyse comparative des conséquences d'arbitrage en termes de : Coût vs Délai de mise en production vs Pérennité face aux montées en charge.
  3. Formalisation des tableaux chiffrés avec la nomenclature imposée par Studi (séparateur de milliers).

---

### Session 5 — Vendredi 11/09/2026 (14h00 - 17h00 | 3 h)
* **Dossier traité :** Dossier 3 — L'architecture logicielle (Questions 1 & 2)
* **Compétence :** C1.3 — Architecture logicielle et dimensionnement
* **Actions concrètes :**
  1. Justification du choix d'hébergement : arbitrage Cloud PaaS / Serverless vs Serveur dédié/interne, tenant compte du budget limité et du pic d'affluence (+30% et fortes concentrations horaires lors du festival).
  2. Conception graphique du Schéma d'Architecture Cible (**Livrable éliminatoire obligatoire**) :
     * Front-End Web Client & Admin (SPA/PWA responsive).
     * Passerelle API / Services Backend.
     * Gestion du cache et des réservations concurrentes (Redis / File d'attente pour éviter le surbooking).
     * Base de données relationnelle persistante et stockage sécurisé des billets (S3 / Cloud Storage).

---

### Session 6 — Samedi 12/09/2026 (09h00 - 12h00 | 3 h)
* **Dossier traité :** Dossier 3 (Question 3) + Finalisation & Packaging du Rendu
* **Compétence :** C1.3 — Évolutivité & Contrôle qualité du livrable
* **Actions concrètes :**
  1. Rédaction de la justification d'architecture axée sur la scalabilité horizontale, la découplabilité des composants et la maintenabilité.
  2. Remplissage rigoureux de la copie officielle `Examen_B1_copiearendre_NOM_Prenom`.
  3. Vérification des conformités de la grille : nomenclature univoque, schéma inséré en haute résolution, lien Trello public testé en navigation privée sans compte.
  4. Exportation du fichier final en PDF prêt pour le premier dépôt.
  * **CLÔTURE DU BLOC 1 LE SAMEDI 12/09 À 12H00 — GAIN : 5 SEMAINES D'AVANCE POUR LE CODE.**
