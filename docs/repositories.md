# Repositories métier

Les repositories sont les interfaces (ports) du domaine qui abstraient la persistance. Ils sont définis dans la couche Domain et implémentés dans la couche Adapters. Ils ne manipulent que des agrégats complets et n'exposent aucune notion technique de base de données.

---

## 1. ReservationRepository

Ce repository gère le cycle de vie complet de l'agrégat `Reservation`. Il constitue le point d'accès exclusif à la persistance des réservations et garantit l'intégrité des transactions lors des pics de charge.

| Opération métier | Description | Contraintes / Règles métier |
|-----------------|-------------|----------------------------|
| **Créer une réservation** | Persiste une nouvelle réservation initialisée avec le statut `EnCours`, les coordonnées de l'acheteur, et les places bloquées. L'agrégat est sauvegardé dans son intégralité à partir de l'état produit par le domaine. Une expiration à 10 minutes est enregistrée simultanément. | La création doit être atomique : soit toutes les places sont bloquées, soit aucune ne l'est. Si une seule place est déjà verrouillée par une autre réservation active, l'ensemble de l'opération est rejeté pour éviter tout surbooking. L'unicité temporelle des places est vérifiée au niveau du domaine avant la persistance. |
| **Récupérer une réservation par identifiant** | Charge l'agrégat `Reservation` complet depuis la persistance à partir de son `ReservationId`. Reconstitue l'ensemble des lignes de réservation, les coordonnées et le statut courant. | L'agrégat doit être retourné dans son état cohérent. Si l'identifiant est inconnu, une erreur métier `ReservationIntrouvable` est levée. La récupération d'une réservation expirée est autorisée mais son statut doit refléter `Expirée`. |
| **Mettre à jour le statut** | Persiste le changement d'état d'une réservation existante suite à un événement domaine : passage de `EnCours` à `Confirmée` après paiement, ou à `Annulée` après expiration ou annulation volontaire. | Seules les transitions de statut valides définies par le domaine sont autorisées. Il est interdit de repasser une réservation `Confirmée` à `EnCours`. La mise à jour d'une réservation `Annulée` est également impossible. Toute tentative de transition invalide lève une exception métier. |
| **Libérer les réservations expirées** | Identifie et fait transiter vers le statut `Expirée` l'ensemble des réservations au statut `EnCours` dont l'horodatage d'expiration est dépassé. Les places associées sont automatiquement libérées et repassent à l'état `Disponible`. | Cette opération est déclenchée par un scheduler et doit être exécutée de manière idempotente : relancer l'opération sur des réservations déjà expirées ne doit produire aucun effet de bord. Elle ne doit jamais affecter une réservation dont le paiement a été validé entre-temps. |

---

## 2. SeanceRepository

Ce repository gère la consultation et la mise à jour de l'agrégat `Seance`, qui encapsule le plan de salle, les zones et l'inventaire des places disponibles. Il est sollicité en lecture intensive lors des pics d'ouverture des ventes.

| Opération métier | Description | Contraintes / Règles métier |
|-----------------|-------------|----------------------------|
| **Récupérer une séance par identifiant** | Charge l'agrégat `Seance` complet depuis la persistance, incluant l'ensemble des zones et des places avec leur état courant (`Disponible`, `Bloquée`, `Vendue`). Fournit la vue d'inventaire nécessaire avant tout blocage de place. | La séance doit être retournée avec un état cohérent et à jour. En cas de forte concurrence, la lecture doit utiliser un mécanisme garantissant que l'état consulté est bien le dernier état persisté, pour éviter des décisions basées sur un inventaire obsolète. |
| **Mettre à jour l'état d'une place** | Modifie l'état d'une `Place` unitaire au sein de la séance : de `Disponible` à `Bloquée` lors d'une mise au panier, ou de `Bloquée` à `Vendue` après confirmation du paiement. | La mise à jour doit être protégée par un verrou optimiste ou pessimiste pour éviter les conditions de concurrence. Deux requêtes simultanées ne peuvent pas toutes les deux réussir à bloquer la même place : la seconde doit recevoir un rejet métier `PlaceNonDisponible`. Le respect de la jauge de sécurité par zone doit être vérifié avant toute confirmation de l'état `Vendue`. |
| **Consulter les places disponibles** | Retourne la liste des places dont le statut est `Disponible` pour une séance et une zone données. Utilisée pour alimenter l'affichage du plan de salle interactif en temps réel. | Les places au statut `Bloquée` ne doivent pas apparaître comme disponibles, même si leur blocage est temporaire. La liste retournée reflète un état instantané et ne constitue pas une réservation. Une place peut changer de statut entre la consultation et la tentative de blocage, ce qui est géré par la règle d'unicité temporelle du domaine. |
| **Vérifier la disponibilité d'une place** | Vérifie ponctuellement si une place spécifique est encore au statut `Disponible` avant d'initier un blocage. Retourne un booléen métier sans lever d'exception. | Cette opération ne garantit pas la disponibilité au moment du blocage effectif : elle est indicative. Seule l'opération de création dans le `ReservationRepository` est transactionnellement sûre. Cette vérification préalable sert uniquement à améliorer l'expérience utilisateur en rejetant rapidement les requêtes manifestement impossibles. |
