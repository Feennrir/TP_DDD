# Exemples d'API REST

Ce document présente des exemples d'endpoints REST pour exposer les opérations métier du système de billetterie. Tous les noms respectent l'Ubiquitous Language et le modèle de domaine défini.

---

## Endpoint 1 : Créer une réservation

### Méthode et URL

```
POST /api/reservations
```

### Description

Cet endpoint permet de créer une nouvelle réservation en bloquant temporairement une ou plusieurs places pour un spectateur. L'opération initialise l'agrégat `Reservation` avec le statut `EnCours` et active un mécanisme d'expiration automatique de 10 minutes.

**Règles métier appliquées** :
- Vérification de la disponibilité des places (unicité temporelle)
- Calcul du montant total selon les prix des places
- Création du blocage temporaire avec expiration

### Exemple de requête

```json
{
  "coordonnees": {
    "nom": "Dupont Sophie",
    "email": "sophie.dupont@example.com"
  },
  "places": [
    {
      "placeId": "PLC-001",
      "prix": {
        "montant": 50.00,
        "devise": "EUR"
      }
    },
    {
      "placeId": "PLC-002",
      "prix": {
        "montant": 35.00,
        "devise": "EUR"
      }
    }
  ]
}
```

### Exemple de réponse

**Cas de succès (HTTP 201 Created)** :

```json
{
  "reservationId": "RES-ABC123",
  "statut": "EN_COURS",
  "dateCreation": "2026-06-15T14:23:45Z",
  "coordonnees": {
    "nom": "Dupont",
    "email": "sophie.dupont@example.com"
  },
  "places": [
    {
      "placeId": "PLC-001",
      "etat": "BLOQUEE",
      "prix": {
        "montant": 50.00,
        "devise": "EUR"
      },
      "emplacement": {
        "rang": "J",
        "numero": 12
      }
    },
    {
      "placeId": "PLC-002",
      "etat": "BLOQUEE",
      "prix": {
        "montant": 35.00,
        "devise": "EUR"
      },
      "emplacement": {
        "rang": "J",
        "numero": 13
      }
    }
  ],
  "montantTotal": {
    "montant": 85.00,
    "devise": "EUR"
  },
  "dateExpiration": "2026-06-15T14:33:45Z",
  "_links": {
    "self": {
      "href": "/api/reservations/RES-ABC123"
    }
  }
}
```

**Cas d'erreur - Place déjà réservée (HTTP 409 Conflict)** :

```json
{
  "type": "PLACE_NON_DISPONIBLE",
  "title": "Place déjà réservée",
  "status": 409,
  "detail": "La place PLC-001 a déjà été réservée par un autre spectateur",
  "placeId": "PLC-001",
  "timestamp": "2026-06-15T14:23:45Z"
}
```

---

## Endpoint 2 : Consulter une réservation

### Méthode et URL

```
GET /api/reservations/{reservationId}
```

### Description

Cet endpoint permet de récupérer l'état complet d'une réservation existante à partir de son identifiant unique. Cette consultation est nécessaire pour afficher le panier du spectateur ou pour vérifier le statut avant le paiement.

**Informations retournées** :
- Identifiant et statut de la réservation
- Coordonnées de l'acheteur
- Liste des places bloquées avec leurs prix et emplacements
- Montant total
- Date d'expiration (si statut EN_COURS)

### Exemple de requête

```
GET /api/reservations/RES-ABC123
```

### Exemple de réponse

**Cas de succès (HTTP 200 OK)** :

```json
{
  "reservationId": "RES-ABC123",
  "statut": "EN_COURS",
  "dateCreation": "2026-06-15T14:23:45Z",
  "dateExpiration": "2026-06-15T14:33:45Z",
  "coordonnees": {
    "nom": "Dupont Sophie",
    "email": "sophie.dupont@example.com"
  },
  "places": [
    {
      "placeId": "PLC-001",
      "etat": "BLOQUEE",
      "prix": {
        "montant": 50.00,
        "devise": "EUR"
      },
      "emplacement": {
        "rang": "J",
        "numero": 12
      }
    },
    {
      "placeId": "PLC-002",
      "etat": "BLOQUEE",
      "prix": {
        "montant": 35.00,
        "devise": "EUR"
      },
      "emplacement": {
        "rang": "J",
        "numero": 13
      }
    }
  ],
  "montantTotal": {
    "montant": 85.00,
    "devise": "EUR"
  },
  "_links": {
    "self": {
      "href": "/api/reservations/RES-ABC123"
    },
    "confirmer": {
      "href": "/api/reservations/RES-ABC123/paiement",
      "method": "POST"
    },
    "annuler": {
      "href": "/api/reservations/RES-ABC123",
      "method": "DELETE"
    }
  }
}
```

**Cas de réservation confirmée (HTTP 200 OK)** :

```json
{
  "reservationId": "RES-ABC123",
  "statut": "CONFIRMEE",
  "dateCreation": "2026-06-15T14:23:45Z",
  "coordonnees": {
    "nom": "Dupont Sophie",
    "email": "sophie.dupont@example.com"
  },
  "places": [
    {
      "placeId": "PLC-001",
      "etat": "VENDUE",
      "prix": {
        "montant": 50.00,
        "devise": "EUR"
      },
      "emplacement": {
        "rang": "J",
        "numero": 12
      }
    },
    {
      "placeId": "PLC-002",
      "etat": "VENDUE",
      "prix": {
        "montant": 35.00,
        "devise": "EUR"
      },
      "emplacement": {
        "rang": "J",
        "numero": 13
      }
    }
  ],
  "montantTotal": {
    "montant": 85.00,
    "devise": "EUR"
  },
  "_links": {
    "self": {
      "href": "/api/reservations/RES-ABC123"
    },
    "billets": {
      "href": "/api/reservations/RES-ABC123/billets"
    }
  }
}
```

**Cas d'erreur - Réservation introuvable (HTTP 404 Not Found)** :

```json
{
  "type": "RESERVATION_INTROUVABLE",
  "title": "Réservation non trouvée",
  "status": 404,
  "detail": "Aucune réservation n'existe avec l'identifiant RES-ABC123",
  "reservationId": "RES-ABC123",
  "timestamp": "2026-06-15T14:30:00Z"
}
```

**Cas d'erreur - Réservation expirée (HTTP 410 Gone)** :

```json
{
  "type": "RESERVATION_EXPIREE",
  "title": "Réservation expirée",
  "status": 410,
  "detail": "La réservation a expiré le 2026-06-15T14:33:45Z et les places ont été libérées",
  "reservationId": "RES-ABC123",
  "statut": "EXPIREE",
  "dateExpiration": "2026-06-15T14:33:45Z",
  "timestamp": "2026-06-15T14:35:00Z"
}
```