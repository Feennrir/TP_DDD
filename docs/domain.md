# Scenario choisi

Billeterie et reservations

# Contexte metier

Le secteur de la billetterie événementielle (concerts, festivals, théâtres) a radicalement évolué avec la numérisation, passant de guichets physiques à des plateformes web capables de gérer des milliers de transactions simultanées. Pour une organisation comme une salle de spectacle nationale, l'enjeu est de garantir une expérience utilisateur fluide tout en assurant une gestion rigoureuse des stocks de places, souvent limités et soumis à une forte demande.

L'organisation doit jongler entre des contraintes de temps réel (disponibilité immédiate des sièges), des politiques de prix dynamiques (promotions, tarifs réduits) et une sécurité renforcée contre la fraude ou la revente illégale. La réussite repose sur la capacité du système à maintenir une "source de vérité" unique pour chaque événement, afin d'éviter toute erreur de réservation qui nuirait à l'image de marque de l'institution.

# Rôles utilisateurs

| Rôle | Type | Description |
| :--- | :--- | :--- |
| Responsable de Programmation   | Direction | Il définit la stratégie tarifaire et supervise le remplissage global des salles. Son objectif est d'optimiser les revenus et l'occupation des lieux sur l'ensemble de la saison. |
| Gestionnaire de Billetterie | Opérationnel | Il gère la configuration technique des événements, l'ouverture des ventes et le support en cas de litige. Il est garant de l'intégrité des données de réservation au quotidien |
| Spectateur | Client | Il recherche des événements, sélectionne ses places sur un plan de salle et procède au paiement sécurisé. Il attend une confirmation immédiate et un accès simple à ses billets numériques. |

# Problématiques métier

Le système doit répondre aux défis majeurs suivants:
- Gestion de la concurrence (Double réservation) : Empêcher de manière absolue que deux clients puissent réserver et payer simultanément le même siège.
- Gestion des pics de charge : Maintenir la stabilité du système lors de l'ouverture des ventes pour des artistes de renommée mondiale où le trafic peut être multiplié par 100.
- Lutte contre le "Scalping" : Mettre en place des mécanismes pour limiter l'achat massif par des robots destinés à la revente illicite.
- Flexibilité des tarifs : Appliquer des règles complexes de réductions (étudiants, groupes, abonnés) tout en conservant une trace d'audit claire.

# Scénario fil rouge

Le scénario type débute lorsque le Responsable de Programmation valide l'ouverture des ventes pour un nouveau festival de musique. Le Gestionnaire de Billetterie configure alors l'événement dans le système : il définit le plan de salle, le nombre de places disponibles en fosse et les différents paliers de prix.

Un Spectateur se connecte à la plateforme, consulte la fiche du festival et sélectionne deux places en catégorie "Carré Or". Le système place ces sièges dans un état "Réservé temporairement" pendant 10 minutes pour permettre au client de saisir ses informations de paiement. Durant ce laps de temps, aucun autre utilisateur ne peut sélectionner ces mêmes places.

Une fois le paiement validé par le service bancaire, le système confirme définitivement la transaction et met à jour l'inventaire en temps réel. Le spectateur reçoit alors un mail de confirmation contenant ses billets sous forme de QR codes infalsifiables. Enfin, le Responsable de Programmation consulte son tableau de bord et observe que 80% des places de cette catégorie ont été vendues en moins d'une heure, validant ainsi la stratégie tarifaire initiale.
