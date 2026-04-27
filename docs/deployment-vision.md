# Vision de déploiement

Le système serait déployé sous forme de 4 services conteneurisés, un par bounded context : Evenements, Réservation & Inventaire, Paiement, Billetterie & Accès.
Chaque service embarque sa logique métier propre et expose une API interne dédiée, afin de limiter le couplage entre contextes.
Le contexte Réservation & Inventaire joue le rôle de cœur transactionnel et serait dimensionné avec davantage de ressources CPU et mémoire.
Le contexte Paiement serait isolé dans un conteneur distinct avec des variables d’environnement dédiées aux secrets et aux URL prestataires.
Le contexte Billetterie & Accès consommerait les événements du bus pour générer les billets de manière asynchrone.
Le contexte Evenements fournirait les données de programmation (séances, capacités, ouverture de vente) au reste du système.
Un broker de messages (exemple : Kafka) serait ajouté comme composant d’infrastructure pour les flux publish/subscribe.
Une base de données relationnelle serait prévue pour Réservation & Inventaire, avec persistance séparée des données critiques.
Chaque service utiliserait un fichier de configuration externe (variables d’environnement) pour les ports, chaînes de connexion et niveaux de log.
Les conteneurs partageraient un réseau Docker privé pour les communications inter-services non exposées publiquement.
Un point d’entrée HTTP unique (API Gateway ou reverse proxy) pourrait router les appels externes vers le bon contexte.
Chaque conteneur embarquerait un healthcheck pour permettre le redémarrage automatique en cas de défaillance.
Les images Docker seraient versionnées par tag applicatif pour garantir des déploiements traçables entre environnements.

## Exemple annoté de Dockerfile (pseudo-code)

```dockerfile
# Image de base runtime (exemple, non imposé)

# Répertoire de travail dans le conteneur

# Copie des artefacts applicatifs du service

# Variables de configuration injectées au runtime

# Port exposé pour la communication interne

# Vérification de santé

# Démarrage du service
```