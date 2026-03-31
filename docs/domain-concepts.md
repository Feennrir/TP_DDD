# Entités et Objets Valeur – Vue conceptuelle

## Entités

### Entité 1 : Reservation

**Description**  
Représente l'engagement temporaire ou définitif d'un spectateur pour une ou plusieurs places. Elle gère le cycle de vie complet, du blocage initial dans le panier jusqu'à la validation finale après paiement.

**Attributs**

| Attribut | Type métier | Description |
|----------|-------------|-------------|
| ReservationId | Identifiant unique | Identifiant immuable permettant de tracer de façon unique une réservation dans le système. Il garantit l'unicité et la traçabilité de chaque engagement client. |
| Statut | État de réservation | Représente l'étape actuelle dans le cycle de vie de la réservation (ex: Temporaire, EnAttenteDePaiement, Confirmee, Annulee). Il conditionne les actions possibles sur la réservation et son impact sur l'inventaire. |
| DateCreation | Horodatage | Date et heure exactes de la création de la réservation. Elle sert de référence temporelle pour calculer l'expiration du blocage temporaire et pour l'audit des transactions. |
| ListePlaces | Collection de références de places | Ensemble des places associées à cette réservation. Chaque référence pointe vers une entité Place spécifique pour une séance donnée. |
| Coordonnees | Coordonnées de l'acheteur | Informations de contact de la personne effectuant la réservation (nom, prénom, email). Ces données permettent l'envoi de confirmations et la communication avec le client. |
| MontantTotal | Prix total | Somme des prix de toutes les places incluses dans la réservation, calculée selon les tarifs applicables. Ce montant est immuable une fois la réservation validée. |

**Invariants métier**

1. **Règle de durée de vie du panier** : Une réservation avec le statut "Temporaire" ne peut exister plus de 10 minutes sans être confirmée par un paiement. Au-delà de cette durée, elle doit être automatiquement annulée et les places libérées dans l'inventaire.

2. **Cohérence du statut** : Une réservation ne peut passer au statut "Confirmee" que si le paiement a été validé par le système de paiement. Une réservation confirmée ne peut jamais revenir à un statut antérieur (Temporaire ou EnAttenteDePaiement).

3. **Intégrité des places** : Toutes les places référencées dans ListePlaces doivent appartenir à la même séance et être dans un état compatible avec le statut de la réservation (Bloquée si Temporaire, Vendue si Confirmee).

---

### Entité 2 : Place

**Description**  
Unité minimale d'inventaire représentant le droit pour une personne d'assister à une séance. Elle possède un état qui évolue selon les interactions des spectateurs et garantit qu'une même place physique ne peut être vendue deux fois.

**Attributs**

| Attribut | Type métier | Description |
|----------|-------------|-------------|
| PlaceId | Identifiant unique | Identifiant immuable permettant de distinguer chaque place d'inventaire de façon unique. Il combine souvent la référence à la séance et à l'emplacement physique. |
| StatutPlace | État de disponibilité | Indique l'état actuel de la place dans le processus de vente (Disponible, Bloquee, Vendue, Indisponible). Cet état détermine si la place peut être sélectionnée par un client. |
| Emplacement | Localisation physique | Définit la position exacte de la place dans le lieu (rang, numéro de siège). Peut être null pour les zones en placement libre où aucun siège n'est assigné. |
| ZoneReference | Référence à une zone | Identifie la zone du plan de salle à laquelle appartient cette place. La zone détermine les caractéristiques communes comme la catégorie tarifaire ou le mode de placement (libre/numéroté). |
| TarifApplique | Prix associé | Montant tarifaire applicable à cette place selon sa catégorie et les règles commerciales en vigueur. Ce prix peut varier selon les profils clients (plein tarif, réduit, etc.). |
| SeanceReference | Référence à une séance | Identifie de manière univoque la séance pour laquelle cette place donne accès. Une place n'a de sens que dans le contexte d'une séance spécifique. |

**Invariants métier**

1. **Unicité de vente** : Une place ne peut être associée qu'à une seule réservation confirmée à la fois. Si StatutPlace = "Vendue", il doit exister exactement une réservation avec statut "Confirmee" qui référence cette place.

2. **Cohérence temporelle du blocage** : Une place avec StatutPlace = "Bloquee" doit être associée à une réservation temporaire active. Si la réservation expire ou est annulée, le statut doit automatiquement revenir à "Disponible".

3. **Respect de la jauge** : Le nombre total de places en état "Bloquee" ou "Vendue" pour une zone donnée ne peut jamais dépasser la jauge maximale définie pour cette zone. Cette règle garantit le respect des contraintes de sécurité et de capacité.

---

## Objet Valeur

### Objet Valeur : Prix

**Propriétés**

| Propriété | Type | Description |
|-----------|------|-------------|
| Montant | Nombre décimal | Valeur numérique du prix, exprimée avec deux décimales pour représenter les centimes. Doit toujours être positif ou nul. |
| Devise | Code devise (ISO 4217) | Code international identifiant la monnaie utilisée (ex: EUR, USD, GBP). Garantit la cohérence des transactions dans un contexte multidevise. |

**Explication de l'immuabilité**

Un Prix est un objet valeur car il représente un concept mesuré uniquement par sa valeur intrinsèque, sans identité propre. Deux prix de "50 EUR" sont strictement équivalents et interchangeables, peu importe où et quand ils sont créés.

Cette immuabilité simplifie également le raisonnement sur l'état du système : on sait qu'un Prix stocké dans une entité Reservation ou CommandeClient ne changera jamais, ce qui facilite l'audit, la traçabilité et la reproductibilité des calculs financiers.
