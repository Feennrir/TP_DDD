# Observabilité

## Correlation ID

Le Correlation ID est un identifiant unique qui permet de relier tous les appels et événements d'un même parcours métier, de la création de réservation jusqu'à la génération du billet. Il est créé à l'entrée du système, dès la première requête HTTP côté API Réservation & Inventaire, puis ajouté dans les en-têtes sortants vers le contexte Paiement. Le même identifiant est aussi propagé dans les événements publiés sur le bus (exemple : `PaiementTraite`, `ReservationConfirmee`) afin de tracer le flux asynchrone. Chaque contexte le recopie dans ses logs structurés pour reconstruire une transaction de bout en bout. En cas d'incident, il devient la clé principale de diagnostic inter-contextes.

## Métriques métier

Les métriques suivantes sont exposables sur `/metrics`.

| Nom de la métrique | Description | Type (counter / gauge / histogram) |
|---|---|---|
| `billetterie_reservations_creees_total` | Nombre total de réservations créées (statut initial EN_COURS). | counter |
| `billetterie_reservations_en_cours` | Nombre courant de réservations encore actives et non expirées. | gauge |
| `billetterie_delai_confirmation_reservation_seconds` | Durée entre création de réservation et confirmation après paiement, utile pour détecter une lenteur du tunnel d'achat. | histogram |

## Logging structuré

Exemple de log JSON :

```json
{
  "timestamp": "2026-04-27T14:25:03.412Z",
  "level": "INFO",
  "service": "reservation-inventaire",
  "boundedContext": "ReservationInventaire",
  "eventType": "ReservationConfirmee",
  "correlationId": "corr-8d7e4a19-9f0c-4f20-8f18-1a5c4b9f02de",
  "reservationId": "RES-ABC123",
  "transactionId": "TRX-789456",
  "statutReservation": "CONFIRMEE",
  "places": 2,
  "montant": 85.0,
  "devise": "EUR",
  "durationMs": 842,
  "message": "Reservation confirmée après validation du paiement"
}
```