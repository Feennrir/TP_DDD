# Scénarios de test de domaine

Ce document présente les scénarios de test au format BDD (Given/When/Then) pour valider le respect des invariants métier définis dans le fichier `invariants.md`. Chaque invariant dispose d'un scénario "happy path" (règle respectée) et d'un scénario "sad path" (règle violée).

---

## Agrégat : Reservation

### Invariant 1 : Unicité temporelle des places

#### Scénario 1 – Happy path

**Given** une place P1 (rang J, siège 12) pour la séance S1 du 15 juin à 20h00  
**And** cette place P1 est dans l'état "Disponible"  
**And** aucune réservation active n'a verrouillé cette place  
**When** un spectateur crée une nouvelle réservation R1 et ajoute la place P1 à son panier  
**Then** la réservation R1 est créée avec le statut "En cours"  
**And** la place P1 passe à l'état "Bloquée"  
**And** la place P1 est associée exclusivement à la réservation R1  
**And** aucune autre réservation ne peut sélectionner cette même place tant que R1 est active

#### Scénario 2 – Sad path

**Given** une place P1 (rang J, siège 12) pour la séance S1 du 15 juin à 20h00  
**And** une réservation R1 existante avec le statut "En cours" a déjà verrouillé la place P1  
**And** la réservation R1 n'a pas encore expiré (créée il y a 5 minutes)  
**When** un second spectateur tente de créer une réservation R2 en ajoutant la même place P1  
**Then** le système rejette l'opération avec l'erreur "Place déjà réservée"  
**And** aucune réservation R2 n'est créée  
**And** la place P1 reste bloquée par la réservation R1  
**And** le spectateur est informé que la place n'est plus disponible

---

### Invariant 2 : Cohérence du cycle de vie limité

#### Scénario 1 – Happy path

**Given** une réservation R1 créée à 14h00 avec le statut "En cours"  
**And** la réservation R1 contient 3 places (P1, P2, P3) toutes bloquées  
**And** la date d'expiration de R1 est fixée à 14h10 (10 minutes après création)  
**And** l'heure actuelle du système est 14h08  
**When** le spectateur valide le paiement de sa réservation R1  
**Then** la réservation R1 passe au statut "Confirmée"  
**And** les 3 places (P1, P2, P3) passent à l'état "Vendue"  
**And** la date d'expiration n'est plus applicable car la réservation est confirmée  
**And** le système génère les billets correspondants

#### Scénario 2 – Sad path

**Given** une réservation R1 créée à 14h00 avec le statut "En cours"  
**And** la réservation R1 contient 2 places (P1, P2) toutes bloquées  
**And** la date d'expiration de R1 est fixée à 14h10  
**And** le spectateur n'a pas effectué de paiement  
**When** l'heure système atteint 14h10 et le processus d'expiration automatique s'exécute  
**Then** la réservation R1 passe au statut "Expirée"  
**And** les 2 places (P1, P2) redeviennent "Disponibles" dans l'inventaire  
**And** la réservation R1 ne peut plus être confirmée ou modifiée  
**And** les places sont à nouveau proposables à d'autres spectateurs

---

### Invariant 3 : Intégrité du montant total

#### Scénario 1 – Happy path

**Given** une réservation R1 en cours de création  
**And** le spectateur ajoute une place P1 avec un tarif "Plein tarif" à 50 EUR  
**And** le spectateur ajoute une place P2 avec un tarif "Réduit étudiant" à 30 EUR  
**And** le spectateur ajoute une place P3 avec un tarif "Enfant" à 20 EUR  
**And** aucune réduction globale n'est appliquée à la commande  
**When** le système calcule le montant total de la réservation R1  
**Then** le montant total est égal à 100 EUR (50 + 30 + 20)  
**And** le montant est strictement positif  
**And** ce montant est utilisé pour initier la transaction de paiement

#### Scénario 2 – Sad path

**Given** une réservation R1 en cours avec 2 places déjà ajoutées  
**And** le montant actuel de R1 est de 80 EUR (2 places à 40 EUR chacune)  
**And** le système détecte une corruption de données suite à une manipulation incorrecte  
**When** un calcul erroné tente de définir le montant total à -10 EUR (valeur négative)  
**Then** le système rejette l'opération avec l'erreur "Montant invalide : valeur négative interdite"  
**And** le montant total de la réservation reste inchangé à 80 EUR  
**And** la réservation ne peut pas progresser tant que le montant n'est pas valide et positif  
**And** un log d'erreur est créé pour investigation

---

## Agrégat : PlanDeSalle (Configuration Séance)

### Invariant 4 : Respect de la Jauge physique

#### Scénario 1 – Happy path

**Given** une séance S1 pour le spectacle "Carmen" le 20 mai à 20h00  
**And** la zone "Balcon" de cette séance a une jauge maximale de 100 places  
**And** 95 places sont déjà vendues dans cette zone  
**And** 5 places restent disponibles  
**When** un spectateur sélectionne 3 places dans la zone "Balcon"  
**Then** les 3 places sont ajoutées à sa réservation  
**And** le compteur de places vendues/bloquées passe à 98  
**And** le nombre total de places vendues/bloquées (98) reste inférieur ou égal à la jauge (100)  
**And** 2 places restent encore disponibles dans cette zone

#### Scénario 2 – Sad path

**Given** une séance S1 pour le spectacle "Carmen" le 20 mai à 20h00  
**And** la zone "Fosse" de cette séance a une jauge maximale de 50 places  
**And** 48 places sont déjà vendues ou bloquées dans cette zone  
**And** 2 places seulement restent disponibles  
**When** un spectateur tente de sélectionner 5 places dans la zone "Fosse"  
**Then** le système rejette l'opération avec l'erreur "Jauge dépassée : seulement 2 places disponibles"  
**And** aucune place n'est ajoutée à la réservation  
**And** le compteur reste à 48 places vendues/bloquées  
**And** le spectateur est informé du nombre exact de places encore disponibles (2)

---

### Invariant 5 : Unicité de l'emplacement

#### Scénario 1 – Happy path

**Given** une séance S1 pour le concert du 10 juillet à 21h00  
**And** le plan de salle de la séance contient une zone "Orchestre"  
**And** aucune place avec l'emplacement "Rang A, Siège 5" n'existe encore dans cette zone  
**When** le gestionnaire configure le plan de salle et crée une place P1 avec l'emplacement "Rang A, Siège 5"  
**Then** la place P1 est créée avec succès  
**And** l'identifiant unique (Rang A, Siège 5) est enregistré dans le système  
**And** cette place devient disponible pour la vente  
**And** aucune autre place ne peut porter le même identifiant dans cette séance

#### Scénario 2 – Sad path

**Given** une séance S1 pour le concert du 10 juillet à 21h00  
**And** le plan de salle contient déjà une place P1 avec l'emplacement "Rang B, Siège 10" dans la zone "Orchestre"  
**When** le gestionnaire tente de créer une nouvelle place P2 avec le même emplacement "Rang B, Siège 10" dans la même zone  
**Then** le système rejette l'opération avec l'erreur "Emplacement dupliqué : Rang B, Siège 10 existe déjà"  
**And** aucune place P2 n'est créée  
**And** seule la place P1 originale reste dans le système  
**And** le gestionnaire est informé de la violation d'unicité

---

### Invariant 6 : Appartenance exclusive

#### Scénario 1 – Happy path

**Given** un plan de salle "Opéra Garnier - Configuration Standard"  
**And** une zone Z1 "Balcon Central" appartenant à ce plan de salle  
**And** une place P1 (Rang F, Siège 8) non encore affectée  
**When** le gestionnaire associe la place P1 à la zone Z1  
**Then** la place P1 appartient exclusivement à la zone Z1  
**And** la zone Z1 appartient exclusivement au plan de salle "Opéra Garnier - Configuration Standard"  
**And** la hiérarchie Place → Zone → PlanDeSalle est cohérente et complète  
**And** il est possible de calculer le taux de remplissage de la zone Z1

#### Scénario 2 – Sad path

**Given** un plan de salle P1 "Théâtre Municipal"  
**And** une zone Z1 "Parterre" appartenant au plan P1  
**And** une place PL1 (Rang C, Siège 3) appartenant déjà à la zone Z1  
**And** un second plan de salle P2 "Configuration Festival" existe dans le système  
**And** une zone Z2 "Zone VIP" appartenant au plan P2  
**When** le gestionnaire tente d'affecter la place PL1 simultanément à la zone Z2 (sans la retirer de Z1)  
**Then** le système rejette l'opération avec l'erreur "Appartenance multiple interdite"  
**And** la place PL1 reste uniquement associée à la zone Z1  
**And** la structure hiérarchique n'est pas corrompue  
**And** les calculs d'inventaire restent fiables
