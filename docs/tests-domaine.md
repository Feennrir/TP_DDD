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

---

## Agrégat : Billet (Contexte Billetterie & Accès)

### Invariant 7 : Unicité du QR Code

#### Scénario 1 – Happy path

**Given** une réservation R1 confirmée avec paiement validé  
**And** la réservation R1 contient 2 places (P1, P2) pour la séance du 20 juin à 20h00  
**And** aucun billet n'a encore été généré pour cette réservation  
**When** le système génère les billets pour la réservation R1  
**Then** un billet B1 est créé pour la place P1 avec un QR Code unique QR_ABC123  
**And** un billet B2 est créé pour la place P2 avec un QR Code unique QR_DEF456  
**And** les deux QR Codes (QR_ABC123 et QR_DEF456) sont différents  
**And** aucun autre billet dans le système ne possède ces QR Codes  
**And** les billets sont marqués avec le statut "Actif"

#### Scénario 2 – Sad path

**Given** un billet B1 existant avec le QR Code QR_XYZ789 pour un événement E1  
**And** une nouvelle réservation R2 confirmée pour un événement E2 différent  
**And** le système tente de générer un billet B2 pour la réservation R2  
**When** le générateur de QR Code produit accidentellement le même code QR_XYZ789 (collision)  
**Then** le système détecte la duplication avec l'erreur "QR Code déjà existant"  
**And** aucun billet B2 n'est créé tant que l'unicité n'est pas garantie  
**And** le système régénère un nouveau QR Code unique  
**And** un log d'alerte est créé pour investigation de la collision

---

### Invariant 8 : Lien Réservation-Billet

#### Scénario 1 – Happy path

**Given** une réservation R1 avec le statut "Confirmée"  
**And** le paiement de la réservation R1 a été validé par le prestataire (Transaction T1 validée)  
**And** la réservation R1 contient 3 places pour la séance du 25 juin à 19h00  
**When** le système déclenche la génération des billets pour la réservation R1  
**Then** 3 billets (B1, B2, B3) sont créés avec succès  
**And** chaque billet possède une référence immuable vers la réservation R1  
**And** tous les billets sont marqués avec le statut "Actif"  
**And** les billets sont envoyés par email au client avec les QR Codes

#### Scénario 2 – Sad path

**Given** une réservation R1 avec le statut "En cours" (non confirmée)  
**And** aucun paiement n'a été validé pour cette réservation  
**And** le délai d'expiration n'est pas encore dépassé  
**When** un utilisateur malveillant tente de déclencher manuellement la génération de billets pour R1  
**Then** le système rejette l'opération avec l'erreur "Réservation non confirmée : génération de billet impossible"  
**And** aucun billet n'est créé  
**And** aucun QR Code n'est généré  
**And** un log de sécurité est créé pour tracer la tentative frauduleuse

---

## Agrégat : Transaction (Contexte Paiement)

### Invariant 9 : Cohérence montant paiement

#### Scénario 1 – Happy path

**Given** une réservation R1 avec un montant total de 150 EUR (3 places à 50 EUR)  
**And** le spectateur a validé son panier et initié le paiement  
**And** le système calcule le montant à transmettre au prestataire de paiement  
**When** le système crée une transaction T1 pour la réservation R1  
**Then** la transaction T1 est créée avec un montant de 150 EUR  
**And** le montant de la transaction (150 EUR) est strictement égal au montant de la réservation  
**And** la transaction T1 est envoyée au prestataire Stripe avec ce montant  
**And** la référence de la réservation R1 est stockée dans la transaction T1

#### Scénario 2 – Sad path

**Given** une réservation R1 avec un montant total de 120 EUR  
**And** le système a calculé correctement le montant initial  
**And** une erreur de synchronisation survient dans le système  
**When** le système tente de créer une transaction T1 avec un montant incorrect de 180 EUR (différent de la réservation)  
**Then** le système rejette l'opération avec l'erreur "Montant incohérent : transaction=180 EUR, réservation=120 EUR"  
**And** aucune transaction n'est créée  
**And** aucun appel au prestataire de paiement n'est effectué  
**And** le spectateur est invité à réessayer  
**And** un log d'erreur critique est créé pour investigation

---

### Invariant 10 : Unicité de confirmation

#### Scénario 1 – Happy path

**Given** une réservation R1 avec le statut "En cours"  
**And** une transaction T1 initiée pour R1 avec le statut "En cours"  
**And** le prestataire de paiement Stripe valide la transaction T1  
**When** le système reçoit la notification de validation du paiement  
**Then** la transaction T1 passe au statut "Validée"  
**And** la réservation R1 passe au statut "Confirmée"  
**And** la transaction T1 est marquée comme la transaction confirmée unique pour R1  
**And** le système déclenche la génération des billets

#### Scénario 2 – Sad path

**Given** une réservation R1 avec le statut "Confirmée"  
**And** une transaction T1 validée et confirmée pour la réservation R1  
**And** les billets ont déjà été générés et envoyés au client  
**When** le système reçoit une seconde notification de paiement (webhook en double ou tentative frauduleuse) pour la même réservation R1  
**Then** le système rejette l'opération avec l'erreur "Réservation déjà confirmée : paiement multiple interdit"  
**And** aucune nouvelle transaction n'est créée  
**And** la réservation R1 reste inchangée avec la transaction T1 originale  
**And** aucun billet supplémentaire n'est généré  
**And** un log de sécurité est créé pour tracer la tentative de double paiement

