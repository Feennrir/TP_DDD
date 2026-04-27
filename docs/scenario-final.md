# Scénario complet inter-contextes

## Contexte métier

Sophie souhaite acheter deux places pour un concert. Le parcours complet couvre la réservation temporaire, le paiement, la confirmation de la vente puis la génération des billets et des QR codes.

## Contextes impliqués

| Bounded Context | Rôle dans le scénario |
|---|---|
| Contexte Evenements | Fournit la séance, la capacité et les données de mise en vente |
| Contexte Réservation & Inventaire | Crée la réservation, bloque les places et confirme la vente |
| Contexte Paiement | Traite la transaction et publie le résultat du paiement |
| Contexte Billetterie & Accès | Génère les billets et les QR codes uniques |

## Narration détaillée

Sophie sélectionne deux places numérotées sur l’interface de billetterie.
L’application appelle l’API de Réservation & Inventaire pour créer une réservation temporaire.
Le contexte Réservation vérifie que les places sont encore disponibles et appartiennent à la même séance.
L’agrégat Reservation est créé avec le statut EN_COURS et une date d’expiration à 10 minutes.
Les places passent à l’état BLOQUEE pour empêcher toute double réservation.
Le montant total est calculé à partir des prix unitaires des places.
Un événement métier ReservationCreee est produit pour tracer l’ouverture du panier.
Le client demande ensuite l’initiation du paiement depuis la réservation créée.
Le contexte Réservation transmet un contrat JSON au contexte Paiement via son ACL.
Le contexte Paiement enregistre la transaction et renvoie un état EN_ATTENTE ou ACCEPTEE selon le prestataire.
Lorsque le paiement est validé, le contexte Paiement publie l’événement PaiementTraite.
Le contexte Réservation & Inventaire consomme cet événement et confirme la réservation.
Les places associées à la réservation passent à l’état VENDUE.
Un événement ReservationConfirmee est émis pour notifier les contextes downstream.
Le contexte Billetterie & Accès consomme cet événement sans interroger directement le contexte Paiement.
Il crée un billet par place confirmée, en associant chaque billet à la réservation concernée.
Pour chaque billet, un QR code unique est généré afin de garantir un contrôle d’accès sans ambiguïté.
Les billets sont persistés puis mis à disposition du client via l’espace personnel ou par email.
Si le paiement avait échoué ou expiré, la réservation aurait été annulée et les places relibérées.
Le scénario se termine lorsque Sophie reçoit ses billets et que la vente est définitivement enregistrée.

## Liste des événements déclenchés

1. ReservationCreee
2. PaiementTraite
3. ReservationConfirmee
4. BilletGenere
5. ConfirmationEnvoyee

## Rappel des invariants concernés

- Une place ne peut être vendue qu’une seule fois au même instant.
- Une réservation temporaire expire automatiquement au bout de 10 minutes.
- Le montant total de la réservation doit correspondre à la somme des places réservées.
- Une réservation ne peut devenir CONFIRMEE qu’après paiement validé.
- Chaque billet doit posséder un QR code unique.
- Chaque billet doit rester lié à une réservation confirmée.