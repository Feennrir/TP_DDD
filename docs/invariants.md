# Agrégats et invariants

## 1. Agrégat : Reservation

Cet agrégat est le cœur du processus transactionnel. Il est responsable de garantir qu'une commande est constituée de places valides, attribuées à un acheteur, et qu'elle respecte les délais impartis avant de devenir définitive.

*   **Racine de l'agrégat :** `Reservation` (Entité)
*   **Entités/OV internes :**
    *   `LigneReservation` (Entité locale : associe une Place à un Tarif au sein de la réservation)
    *   `Coordonnees` (Objet Valeur : contact de l'acheteur)
    *   `MontantTotal` (Objet Valeur : somme des prix)
    *   `StatutReservation` (Objet Valeur : EnCours, Confirmée, Annulée)
    *   `Expiration` (Objet Valeur : timestamp de fin du blocage temporaire)

### Tableau des invariants

| Invariant | Description métier | Conséquence si non respecté |
| :--- | :--- | :--- |
| **Unicité temporelle des places** | Une réservation ne peut contenir des places qui sont déjà verrouillées dans une autre réservation active (statut "En cours" ou "Confirmée") sur le même créneau horaire. Le système doit garantir l'exclusivité du blocage. | Risque majeur de **double réservation** (surbooking). Deux clients paieraient pour le même siège, entraînant des litiges et une perte de confiance critique. |
| **Cohérence du cycle de vie limité** | Une réservation au statut "En cours" doit posséder une date d'expiration valide (ex: 10 minutes). Si le paiement n'est pas validé avant cette date, la réservation doit transiter vers un état "Expirée" et libérer ses lignes. | Les places resteraient bloquées indéfiniment sans paiement ("stock mort"), empêchant d'autres clients de les acheter et faussant le taux de remplissage réel. |
| **Intégrité du montant total** | Le montant total de la réservation doit toujours être strictement égal à la somme des prix des lignes de réservation, moins les éventuelles réductions appliquées globalement. Il ne peut jamais être négatif. | Incohérence comptable et financière. L'entreprise pourrait perdre de l'argent ou facturer le mauvais montant au client lors de l'appel au service de paiement. |

---

## 2. Agrégat : PlanDeSalle (Configuration Séance)

Cet agrégat modélise l'offre disponible pour une séance donnée. Il structure l'inventaire physique et logique, garantissant que la vente respecte les capacités d'accueil du lieu.

*   **Racine de l'agrégat :** `Seance` (Entité)
*   **Entités/OV internes :**
    *   `Zone` (Entité : subdivision de la salle)
    *   `Jauge` (Objet Valeur : capacité max)
    *   `Place` (Entité : siège unitaire)
    *   `CategorieDePlace` (Objet Valeur : classification tarifaire)

### Tableau des invariants

| Invariant | Description métier | Conséquence si non respecté |
| :--- | :--- | :--- |
| **Respect de la Jauge physique** | Le nombre total de places mises en vente (numérotées ou en placement libre) pour une zone donnée ne doit jamais excéder la capacité maximale (Jauge) définie par les règles de sécurité du lieu. | Violation des normes de sécurité incendie et légales. En cas de contrôle ou d'incident, la responsabilité pénale des organisateurs serait engagée. |
| **Unicité de l'emplacement** | Au sein d'une même séance, un couple (Rang, Numéro) identifiant une place numérotée doit être absolument unique. Il est impossible d'avoir deux sièges physiques portant le même identifiant. | Le système proposerait la vente de "places fantômes" ou dupliquées. Les spectateurs se retrouveraient à plusieurs pour un seul siège physique dans la salle. |
| **Appartenance exclusive** | Une Place doit appartenir à une et une seule Zone, et une Zone doit appartenir à un et un seul Plan de Salle. Il ne peut y avoir d'orphelins ou de partages transversaux dans la structure hiérarchique. | Corruption de la structure de l'inventaire. Impossible de calculer correctement les taux de remplissage par zone ou d'appliquer des règles de tarification ciblées. |

---

## 3. Agrégat : Billet

Cet agrégat représente le titre d'accès sécurisé généré après confirmation d'une réservation. Il garantit l'unicité et la traçabilité des accès aux événements.

*   **Racine de l'agrégat :** `Billet` (Entité)
*   **Entités/OV internes :**
    *   `QRCode` (Objet Valeur : code unique sécurisé)
    *   `ReservationReference` (Objet Valeur : lien vers la réservation source)
    *   `StatutBillet` (Objet Valeur : Actif, Utilisé, Annulé)
    *   `InfosSpectacle` (Objet Valeur : détails de la séance)

### Tableau des invariants

| Invariant | Description métier | Conséquence si non respecté |
| :--- | :--- | :--- |
| **Unicité du QR Code** | Chaque billet généré doit posséder un QR Code absolument unique dans l'ensemble du système. Aucun doublon ne peut exister, même pour des événements différents ou des périodes distinctes. | Risque de **fraude massive** et de perte de contrôle des entrées. Plusieurs personnes pourraient utiliser le même QR Code pour accéder à l'événement, rendant le système de sécurité inefficace. |
| **Lien Réservation-Billet** | Un billet ne peut être généré que si et seulement si il existe une réservation confirmée (statut "Confirmée") dont le paiement a été validé. Le billet doit maintenir une référence immuable vers cette réservation. | Génération de billets frauduleux ou invalides. Des spectateurs pourraient entrer sans avoir payé, causant des pertes financières et des problèmes de conformité comptable. |

---

## 4. Agrégat : Transaction

Cet agrégat gère le processus de paiement et assure la cohérence financière entre la réservation et le règlement effectif.

*   **Racine de l'agrégat :** `Transaction` (Entité)
*   **Entités/OV internes :**
    *   `MontantTransaction` (Objet Valeur : montant payé)
    *   `ReservationReference` (Objet Valeur : lien vers la réservation)
    *   `StatutTransaction` (Objet Valeur : EnCours, Validée, Échouée, Annulée)
    *   `IdentifiantExterne` (Objet Valeur : référence du prestataire de paiement)

### Tableau des invariants

| Invariant | Description métier | Conséquence si non respecté |
| :--- | :--- | :--- |
| **Cohérence montant paiement** | Le montant de la transaction doit être strictement égal au montant total de la réservation associée au moment de l'initiation du paiement. Toute divergence doit être rejetée avant l'envoi vers le prestataire. | Incohérence comptable critique. L'entreprise pourrait facturer un montant incorrect (trop ou pas assez), entraînant des litiges clients, des pertes financières ou des problèmes d'audit. |
| **Unicité de confirmation** | Une réservation ne peut être associée qu'à une seule transaction confirmée (statut "Validée"). Il est impossible de confirmer le paiement d'une même réservation plusieurs fois. | Risque de **double facturation** du client ou de génération multiple de billets pour une seule commande. Cela causerait des conflits lors du contrôle d'accès et des complications de remboursement. |

---

## Schéma UML des agrégats (complet)

*(Note : Ce schéma illustre les frontières de tous les agrégats. Les racines sont marquées par le stéréotype <<Root>>)*

```mermaid
classDiagram
    namespace Agregat_Reservation {
        class Reservation {
            <<Root>>
            +ReservationId id
            +statut
            +dateExpiration
            +calculerTotal()
            +ajouterPlace()
        }
        class LigneReservation {
            +quantite
            +tarifApplique
        }
        class Coordonnees {
            <<Value Object>>
            +email
            +nom
        }
    }

    namespace Agregat_Seance {
        class Seance {
            <<Root>>
            +SeanceId id
            +dateDebut
            +estComplete()
        }
        class Zone {
            +nom
            +typePlacement
        }
        class Jauge {
            <<Value Object>>
            +capaciteMax
        }
        class Place {
            +PlaceId id
            +numero
            +rang
            +etat
        }
    }

    namespace Agregat_Billet {
        class Billet {
            <<Root>>
            +BilletId id
            +statut
            +validerAcces()
        }
        class QRCode {
            <<Value Object>>
            +codeUnique
            +dateGeneration
        }
        class ReservationRef {
            <<Value Object>>
            +reservationId
        }
    }

    namespace Agregat_Transaction {
        class Transaction {
            <<Root>>
            +TransactionId id
            +statut
            +valider()
        }
        class MontantTransaction {
            <<Value Object>>
            +montant
            +devise
        }
        class IdentifiantExterne {
            <<Value Object>>
            +referenceStripe
        }
    }

    Reservation "1" *-- "1..*" LigneReservation : contient
    Reservation "1" --> "1" Coordonnees
    
    Seance "1" *-- "1..*" Zone : composée de
    Zone "1" --> "1" Jauge
    Zone "1" *-- "0..*" Place : contient

    Billet "1" --> "1" QRCode : possède
    Billet "1" --> "1" ReservationRef : issu de

    Transaction "1" --> "1" MontantTransaction : pour
    Transaction "1" --> "1" ReservationRef : concerne
    Transaction "1" --> "1" IdentifiantExterne : identifié par
```
