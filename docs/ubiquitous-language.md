# Glossaire (Ubiquitous Language v1)

| Terme | Définition métier | Exemple concret |
| :--- | :--- | :--- |
| Evenement | Représente la manifestation culturelle ou sportive globale organisée par l'institution. Il sert de conteneur pour l'ensemble des représentations et définit les informations générales comme le titre ou l'affiche. | Le festival "Rock en Seine" est un Evenement qui regroupe plusieurs jours de concerts. |
| Seance | Instance temporelle unique d'un Evenement, caractérisée par une date et une heure de début précises. C'est à ce niveau que l'on gère l'inventaire des places disponibles pour un créneau spécifique. | Le concert de 20h00 le vendredi soir constitue une Seance spécifique du festival. |
| Lieu | Infrastructure physique ou virtuelle qui accueille la Seance. Il possède des propriétés techniques comme une adresse et une configuration de salle fixe ou modulable. | Le "Stade de France" est le Lieu identifié pour les concerts de grande envergure. |
| PlanDeSalle | Représentation schématique du Lieu permettant de visualiser la disposition des sièges et des zones de spectateurs. Il est indispensable pour permettre au client de choisir sa place numérotée. | Le PlanDeSalle de l'Opéra Garnier permet de distinguer l'orchestre des balcons. |
| Zone | Subdivision d'un PlanDeSalle regroupant un ensemble de places ayant des caractéristiques similaires. Une zone peut être configurée en "placement libre" ou en "sièges numérotés". | La "Fosse" est une Zone en placement libre située au plus proche de la scène. |
| CategorieDePlace | Classification des places déterminant leur niveau de confort ou de visibilité, et influençant directement le prix. On distingue généralement plusieurs niveaux de prestige au sein d'une même Seance. | La "Catégorie 1" offre une visibilité optimale comparée à la "Catégorie 3". |
| Jauge | Capacité maximale d'accueil autorisée pour une Zone ou pour l'ensemble du Lieu. Elle est fixée pour des raisons de sécurité ou de confort et ne peut être dépassée lors de la vente. | La Jauge de la salle est limitée à 500 personnes pour respecter les normes de sécurité incendie. |
| Place | Unité minimale d'inventaire représentant le droit pour une personne d'assister à une Seance. Elle peut être associée à un numéro de siège précis ou être anonyme dans le cas d'un placement libre. | La Place "Rang J, Siège 12" est réservée pour le spectateur dans le secteur Balcon. |
| Tarif | Règle de prix appliquée à une Place en fonction du profil du spectateur ou de conditions commerciales. Plusieurs tarifs peuvent coexister pour une même place (Plein tarif, Enfant, Senior) | Le "Tarif Étudiant" permet de bénéficier d'une réduction de 30% sur le prix de base. |
| Justificatif | Document officiel requis pour valider l'éligibilité à un Tarif spécifique lors de l'achat ou du contrôle d'accès. Son absence peut entraîner l'invalidité du Billet le jour de l'événement. | Une carte d'identité est demandée comme Justificatif pour le tarif "Moins de 25 ans". |
| Panier | Espace de stockage temporaire où le Spectateur regroupe les places sélectionnées avant de finaliser sa transaction. Le panier possède une durée de vie limitée après laquelle les places sont libérées. | Le Panier contient trois places pour le spectacle de danse avant que l'utilisateur ne passe au paiement. |
| BlocageTemporaire | Mécanisme de verrouillage d'une Place au profit d'un Spectateur pendant qu'il finalise son achat dans le Panier. Cela garantit que personne d'autre ne peut acheter la même place durant ce court laps de temps. | Dès que le client clique sur un siège, un BlocageTemporaire de 10 minutes est activé sur le serveur. |
| Reservation | État intermédiaire d'une commande où les places sont réservées mais le paiement n'a pas encore été confirmé par l'organisme bancaire. Elle devient caduque si le paiement échoue ou n'est pas effectué à temps. | La Reservation est enregistrée avec le statut "En attente de paiement". |
| Billet | Titre de propriété dématérialisé ou physique permettant au porteur d'accéder à la Seance. Il comporte des informations de sécurité comme un code unique pour empêcher la falsification. | Le spectateur télécharge son Billet sur son smartphone pour le présenter à l'entrée. |
| CommandeClient | Document comptable finalisant la transaction financière entre l'acheteur et l'organisateur. Elle regroupe un ou plusieurs Billets et sert de preuve d'achat pour l'édition de factures. | La CommandeClient n°8492 confirme l'achat de quatre places pour un montant total de 120€. |
| Acheteur | Personne physique ou morale qui effectue le paiement de la CommandeClient. Il n'est pas forcément le bénéficiaire final du Billet (cas d'un cadeau ou d'un comité d'entreprise). | L'Acheteur utilise sa carte bleue pour régler les places de toute sa famille. |
| Beneficiaire | Personne qui utilisera effectivement le Billet pour entrer dans le Lieu. Son nom peut être inscrit sur le billet pour les événements nominatifs afin de lutter contre la revente. | Bien que le père soit l'acheteur, son fils est le Beneficiaire du billet de concert. |
| QRCode | Symbole graphique imprimé sur le Billet qui contient les données chiffrées de la réservation. Il est scanné à l'entrée du Lieu pour valider le droit d'accès en temps réel. | L'agent de sécurité scanne le QRCode du billet pour vérifier sa validité à l'entrée. |
| StatutVente | État actuel de la commercialisation d'une Seance (ex: Ouverte, Bientôt disponible, Épuisée). Ce statut guide l'affichage sur le site web et les possibilités d'achat pour le client. | Le StatutVente est passé à "Complet" dès que la dernière place de la Jauge a été vendue. |
| Guichet | Point de vente physique ou interface de vente assistée utilisée par un opérationnel pour délivrer des billets. Contrairement au site web, il permet souvent des modes de règlement spécifiques comme les espèces. | Le Guichet du théâtre ouvre une heure avant le début de la représentation pour les retardataires. |
| PublierEvenement | Action de rendre un Evenement visible et accessible à la vente sur les canaux de distribution. Cette étape est cruciale pour déclencher la commercialisation et nécessite une validation préalable du Responsable de Programmation. | Le Gestionnaire de Billetterie publie l'Evenement sur le site officiel une fois que tous les détails sont configurés. |
| ConfigurerPlanSalle | Processus de création ou de mise à jour du PlanDeSalle pour un Lieu donné. Cela inclut la définition des Zones, des CatégoriesDePlace et des Jauges associées. | Avant chaque saison, le Gestionnaire de Billetterie configure le PlanDeSalle pour s'adapter aux nouveaux spectacles. |
| SelectionnerPlace | Interaction utilisateur permettant de choisir une Place spécifique sur le PlanDeSalle. Cette action déclenche la création d'une ReservationTemporaire pour garantir la disponibilité de la place pendant le processus d'achat. | Le spectateur clique sur une place dans la catégorie "Balcon" et voit immédiatement son panier se mettre à jour avec cette sélection. |
| ValiderTransaction | Processus de confirmation finale d'une CommandeClient après que le paiement a été autorisé par l'organisme bancaire. Cette étape met à jour le statut de la réservation, génère les billets correspondants et envoie les confirmations au client. | Après que le paiement a été validé, le système génère automatiquement les QR codes pour les billets et les envoie par email au spectateur. |
| InitierPaiement | Action déclenchée par le système pour commencer le processus de paiement après que le client a confirmé son panier. Cela implique la création d'une session de paiement avec le prestataire choisi et la préparation des données nécessaires pour la transaction. | Dès que le client valide son panier, le système initie un paiement via Stripe en créant une session de paiement sécurisée. |
| GenererBillet | Processus de création d'un billet numérique ou physique à partir d'une réservation confirmée. Cela inclut l'attribution d'un code unique, la personnalisation avec les informations du client et la préparation pour l'impression ou l'envoi électronique. | Une fois que la transaction est confirmée, le système génère un billet numérique avec un QR code unique pour chaque place achetée. |
| ContexteBilleterieAcces | Bounded Context dédié à la gestion des billets et de l'accès aux événements. Il englobe les processus de génération des billets, de contrôle d'accès et de lutte contre la fraude. Ce contexte est responsable de transformer les réservations confirmées en titres d'accès sécurisés pour les spectateurs. | Le ContexteBilleterieAcces reçoit les informations de réservation validées et génère les billets numériques avec les QR codes correspondants. |
| ContexteReservationInventaire | Bounded Context central du système, responsable de la gestion des réservations et de l'inventaire en temps réel. Il gère le cycle de vie des paniers, les mécanismes de blocage temporaire des places et la mise à jour immédiate de l'inventaire pour éviter toute double réservation. Ce contexte applique également les règles de tarification complexes et les quotas de sécurité. | Le ContexteReservationInventaire reçoit les demandes de sélection de places, applique les règles de tarification et met à jour l'inventaire en temps réel pour garantir une expérience utilisateur fluide. |
| ContexteEvenements | Bounded Context dédié à la gestion de la configuration des événements, des lieux et des plans de salle. Il fournit le référentiel de données nécessaire à l'ouverture des ventes par le gestionnaire et assure la cohérence des informations utilisées par le moteur de réservation. | Le ContexteEvenements permet au gestionnaire de configurer les détails d'un événement, comme les dates, les lieux et les plans de salle, qui seront ensuite utilisés par le moteur de réservation pour valider les sélections des clients. |
| ContextePaiement | Bounded Context responsable de l'interface avec les prestataires de services de paiement externes. Il gère l'initiation des sessions de paiement, le suivi des transactions financières et la notification du succès ou de l'échec du règlement au reste du système. Ce contexte ne contient aucune logique métier spécifique à la billetterie et se concentre sur la gestion sécurisée des paiements. | Le ContextePaiement reçoit les demandes d'initiation de paiement du moteur de réservation, communique avec le prestataire externe pour valider la transaction et informe le système du résultat pour finaliser la commande. |

| Terme |	Contexte principal
| :--- | :---
| Evenement | ContexteEvenements
| Seance | ContexteEvenements
| Lieu | ContexteEvenements | 
| PlanDeSalle | ContexteEvenements | 
| Zone | ContexteEvenements | 
| CategorieDePlace | ContexteEvenements
| StatutVente | ContexteEvenements | 
| Guichet | ContexteEvenements | 
| PublierEvenement | ContexteEvenements
| ConfigurerPlanSalle | ContexteEvenements
| Place | ContexteReservationInventaire
| Jauge | ContexteReservationInventaire
| Tarif | ContexteReservationInventaire
| Panier | ContexteReservationInventaire
| BlocageTemporaire | ContexteReservationInventaire
| Reservation | ContexteReservationInventaire
| SelectionnerPlace | ContexteReservationInventaire
| CommandeClient | ContextePaiement
| Acheteur | ContextePaiement
| InitierPaiement | ContextePaiement
| ValiderTransaction | ContextePaiement
| Billet | ContexteBilleterieAcces
| Beneficiaire | ContexteBilleterieAcces
| QRCode | ContexteBilleterieAcces
| GenererBillet | ContexteBilleterieAcces
| Justificatif | ContexteBilleterieAcces