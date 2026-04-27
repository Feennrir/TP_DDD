# Contrats d'échange entre contexts

Ce document formalise les échanges entre les 4 bounded contexts du projet :
- Contexte Evenements
- Contexte Réservation & Inventaire
- Contexte Paiement
- Contexte Billetterie & Accès

Objectif : disposer d'au moins un contrat explicite pour chaque contexte.

---

## Contrat 1 : Publier une séance prête à la vente

- **Context source** : Contexte Evenements
- **Context cible** : Contexte Réservation & Inventaire
- **Type d'échange** : Event asynchrone

### Schéma du message

```json
{
  "eventName": "EvenementConfigure",
  "eventSchema": {
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "type": "object",
    "additionalProperties": false,
    "required": [
      "eventId",
      "seanceId",
      "evenementId",
      "dateHeureSeance",
      "capaciteTotale",
      "datePublication"
    ],
    "properties": {
      "eventId": {
        "type": "string",
        "pattern": "^EVT-[0-9]{8}-[0-9]{3}$",
        "example": "EVT-20260427-010"
      },
      "seanceId": {
        "type": "string",
        "pattern": "^SEA-[A-Z0-9]+$",
        "example": "SEA-20260615-S1"
      },
      "evenementId": {
        "type": "string",
        "pattern": "^EVE-[A-Z0-9]+$",
        "example": "EVE-FEST-001"
      },
      "dateHeureSeance": {
        "type": "string",
        "format": "date-time",
        "example": "2026-06-15T20:00:00Z"
      },
      "capaciteTotale": {
        "type": "integer",
        "minimum": 1,
        "example": 1200
      },
      "datePublication": {
        "type": "string",
        "format": "date-time",
        "example": "2026-04-27T10:00:00Z"
      }
    }
  }
}
```

### Règles du contrat

- Une séance n'est vendable que si `EvenementConfigure` est reçu côté Réservation & Inventaire.
- Le contexte cible initialise l'inventaire des places à partir de ce message.

---

## Contrat 2 : Démarrer un paiement depuis une réservation

- **Context source** : Contexte Réservation & Inventaire
- **Context cible** : Contexte Paiement
- **Type d'échange** : REST synchrone

### Schéma du message

```json
{
  "contractName": "DemarrerPaiement",
  "transport": {
    "protocol": "REST",
    "method": "POST",
    "path": "/paiements"
  },
  "requestSchema": {
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "type": "object",
    "additionalProperties": false,
    "required": ["reservationId", "montant", "devise", "client", "dateExpirationReservation"],
    "properties": {
      "reservationId": {
        "type": "string",
        "pattern": "^RES-[A-Z0-9]+$",
        "example": "RES-ABC123"
      },
      "montant": {
        "type": "number",
        "minimum": 0,
        "multipleOf": 0.01,
        "example": 85.0
      },
      "devise": {
        "type": "string",
        "enum": ["EUR"],
        "example": "EUR"
      },
      "client": {
        "type": "object",
        "additionalProperties": false,
        "required": ["nom", "email"],
        "properties": {
          "nom": {
            "type": "string",
            "minLength": 1,
            "example": "Dupont Sophie"
          },
          "email": {
            "type": "string",
            "format": "email",
            "example": "sophie.dupont@example.com"
          }
        }
      },
      "dateExpirationReservation": {
        "type": "string",
        "format": "date-time",
        "example": "2026-04-27T14:33:45Z"
      },
      "retourUrl": {
        "type": "string",
        "format": "uri",
        "example": "https://billetterie.example.com/paiement/retour"
      }
    }
  },
  "responseSchema": {
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "type": "object",
    "additionalProperties": false,
    "required": ["transactionId", "reservationId", "statut", "dateTransaction"],
    "properties": {
      "transactionId": {
        "type": "string",
        "pattern": "^TRX-[A-Z0-9]+$",
        "example": "TRX-789456"
      },
      "reservationId": {
        "type": "string",
        "pattern": "^RES-[A-Z0-9]+$",
        "example": "RES-ABC123"
      },
      "statut": {
        "type": "string",
        "enum": ["EN_ATTENTE", "ACCEPTEE", "REFUSEE"],
        "example": "EN_ATTENTE"
      },
      "redirectUrl": {
        "type": "string",
        "format": "uri",
        "example": "https://prestataire-paiement.example.com/session/TRX-789456"
      },
      "messageErreur": {
        "type": "string",
        "example": "Montant invalide ou session expirée"
      },
      "dateTransaction": {
        "type": "string",
        "format": "date-time",
        "example": "2026-04-27T14:24:10Z"
      }
    }
  }
}
```

### Exemple de requête JSON

```json
{
  "reservationId": "RES-ABC123",
  "montant": 85.0,
  "devise": "EUR",
  "client": {
    "nom": "Dupont Sophie",
    "email": "sophie.dupont@example.com"
  },
  "dateExpirationReservation": "2026-04-27T14:33:45Z",
  "retourUrl": "https://billetterie.example.com/paiement/retour"
}
```

### Règles du contrat

- Le contexte Réservation & Inventaire ne transmet que les données nécessaires au paiement.
- Le montant envoyé doit correspondre exactement au montant total de la réservation.
- Le contexte Paiement ne modifie pas l'agrégat Réservation directement ; il retourne uniquement un état de transaction.

---

## Contrat 3 : Publier le résultat d'un paiement

- **Context source** : Contexte Paiement
- **Context cible** : Contexte Réservation & Inventaire
- **Type d'échange** : Event asynchrone

### Schéma du message

```json
{
  "eventName": "PaiementTraite",
  "eventSchema": {
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "type": "object",
    "additionalProperties": false,
    "required": [
      "eventId",
      "transactionId",
      "reservationId",
      "statut",
      "montant",
      "devise",
      "dateEvenement"
    ],
    "properties": {
      "eventId": {
        "type": "string",
        "pattern": "^EVT-[0-9]{8}-[0-9]{3}$",
        "example": "EVT-20260427-001"
      },
      "transactionId": {
        "type": "string",
        "pattern": "^TRX-[A-Z0-9]+$",
        "example": "TRX-789456"
      },
      "reservationId": {
        "type": "string",
        "pattern": "^RES-[A-Z0-9]+$",
        "example": "RES-ABC123"
      },
      "statut": {
        "type": "string",
        "enum": ["ACCEPTE", "REFUSE", "ANNULE"],
        "example": "ACCEPTE"
      },
      "montant": {
        "type": "number",
        "minimum": 0,
        "multipleOf": 0.01,
        "example": 85.0
      },
      "devise": {
        "type": "string",
        "enum": ["EUR"],
        "example": "EUR"
      },
      "motifRefus": {
        "type": "string",
        "example": "Carte expirée"
      },
      "dateEvenement": {
        "type": "string",
        "format": "date-time",
        "example": "2026-04-27T14:23:45Z"
      }
    },
    "allOf": [
      {
        "if": {
          "properties": {
            "statut": {
              "const": "REFUSE"
            }
          }
        },
        "then": {
          "required": ["motifRefus"]
        }
      }
    ]
  }
}
```

### Exemple d'événement JSON

```json
{
  "eventId": "EVT-20260427-001",
  "transactionId": "TRX-789456",
  "reservationId": "RES-ABC123",
  "statut": "ACCEPTE",
  "montant": 85.0,
  "devise": "EUR",
  "dateEvenement": "2026-04-27T14:23:45Z"
}
```

### Règles du contrat

- L'événement est émis uniquement après décision finale du prestataire de paiement.
- Le contexte Réservation & Inventaire consomme l'événement pour confirmer ou annuler la réservation.
- En cas de paiement accepté, le flux métier peut ensuite déclencher la génération des billets par le contexte Billetterie & Accès via ses propres mécanismes internes.

---

## Contrat 4 : Déclencher la génération de billets

- **Context source** : Contexte Réservation & Inventaire
- **Context cible** : Contexte Billetterie & Accès
- **Type d'échange** : Event asynchrone

### Schéma du message

```json
{
  "eventName": "ReservationConfirmee",
  "eventSchema": {
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "type": "object",
    "additionalProperties": false,
    "required": [
      "eventId",
      "reservationId",
      "dateConfirmation",
      "places",
      "coordonnees"
    ],
    "properties": {
      "eventId": {
        "type": "string",
        "pattern": "^EVT-[0-9]{8}-[0-9]{3}$",
        "example": "EVT-20260427-099"
      },
      "reservationId": {
        "type": "string",
        "pattern": "^RES-[A-Z0-9]+$",
        "example": "RES-ABC123"
      },
      "dateConfirmation": {
        "type": "string",
        "format": "date-time",
        "example": "2026-04-27T14:25:00Z"
      },
      "coordonnees": {
        "type": "object",
        "additionalProperties": false,
        "required": ["nom", "email"],
        "properties": {
          "nom": {
            "type": "string",
            "example": "Dupont Sophie"
          },
          "email": {
            "type": "string",
            "format": "email",
            "example": "sophie.dupont@example.com"
          }
        }
      },
      "places": {
        "type": "array",
        "minItems": 1,
        "items": {
          "type": "object",
          "additionalProperties": false,
          "required": ["placeId", "rang", "numero"],
          "properties": {
            "placeId": {
              "type": "string",
              "pattern": "^PLC-[A-Z0-9]+$",
              "example": "PLC-001"
            },
            "rang": {
              "type": "string",
              "example": "J"
            },
            "numero": {
              "type": "integer",
              "minimum": 1,
              "example": 12
            }
          }
        }
      }
    }
  }
}
```

### Exemple d'événement JSON

```json
{
  "eventId": "EVT-20260427-099",
  "reservationId": "RES-ABC123",
  "dateConfirmation": "2026-04-27T14:25:00Z",
  "coordonnees": {
    "nom": "Dupont Sophie",
    "email": "sophie.dupont@example.com"
  },
  "places": [
    {
      "placeId": "PLC-001",
      "rang": "J",
      "numero": 12
    },
    {
      "placeId": "PLC-002",
      "rang": "J",
      "numero": 13
    }
  ]
}
```

### Règles du contrat

- Le contexte Billetterie & Accès ne génère jamais de billet sans événement `ReservationConfirmee`.
- Chaque place reçue doit produire un billet avec QR code unique.
