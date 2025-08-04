# W2CLA - The way to a citadel logical architecture

`Way to Citadel Logical Architecture` est un projet expérimental et open source proposant un protocole pour la fortification de clés et de mots de passe partagés.

Il s'agit de ne jamais fournir directement les clés et secrets sensibles, et d'attribuer à chaque utilisateur une identité forte permettant d'accéder à des données et services (APIs, applications internes, etc.) hébergés au sein d'une Citadelle en passant par des proxys dédiés et spécialisés dans l'accès et l'échange de données sécurisées.

Ceci avec l'intention de faciliter la sécurité, la conformité et la gestion granulaire des accès.

## Avantages

- Un secret compromis ne met pas tout l'organisme en danger, rotation et révocation facilitées pour chaque utilisateur
- Traçabilité facilitée, chaque accès peut être loggé et identifié (qui, quoi, quand)
- Onboarding/offboarding automatisés, rotation de secrets centralisée : *les nouveaux utilisateurs reçoivent automatiquement leurs accès via leur identité, et lors des révocations, tous les accès sont supprimés instantanément depuis un point central*

L'idée idéale serait de tendre vers un système open source libre proposant une alternative aux architectures de type Zero Trust : *offrir une solution accessible aux entités qui souhaitent une approche ouverte et transparente de la sécurité.*

## Architecture technique

### Architecture locale
Un seul coffre-fort local hébergé à côté de la Citadelle chez l'entité, éliminant la complexité de synchronisation entre multiples instances.

### Les proxys
Deux types coexistent :
- **Proxys dédiés locaux** : ont accès direct au coffre-fort et gèrent l'authentification primaire. En cas de panne, l'architecture permet un basculement automatique vers un autre proxy
- **Proxys distants** : passent obligatoirement par les proxys dédiés pour accéder aux secrets, créant une chaîne de confiance hiérarchique

### Guardhouse
Module de validation situé à côté de la Citadelle, permettant à tous les proxys de vérifier la validité d'un utilisateur avant toute action sur la Citadelle ou le coffre-fort. Ce module centralise les statuts de révocation et les permissions en temps réel. En cas de panne, aucun accès n'est possible jusqu'à restauration. Une entité de secours peut être prévue pour la redondance.

### Guardhouse Auto-régénérante
En cas de corruption complète, la Citadelle conserve les métadonnées des utilisateurs (ID et logs d'accès). Un processus de reconstruction supervisé permet à un administrateur de relancer la Guardhouse : les utilisateurs sont progressivement réajoutés selon leurs interactions avec la Citadelle, l'administrateur validant ou refusant chaque réintégration via une interface de monitoring.

### Les Citadelles
Elles hébergent soit des accès, soit directement les services et données partagées au sein de l'organisme. Elles conservent les métadonnées utilisateurs nécessaires à la traçabilité et à la régénération du système.

### Utilisateurs
Ils possèdent leur trousseau ou wallet avec leurs secrets/clés basé sur les standards DID (Decentralized Identifiers), permettant une interopérabilité avec d'autres systèmes `W2CLA` et une gestion d'identité standardisée.

### APIs

#### API externe
Permet d'interfacer l'utilisateur aux proxys et gère l'identité décentralisée via les standards DID. Cette API permet également de faire passerelle pour la récupération et l'envoi des données entre un service de la Citadelle et l'utilisateur.

#### API interne
Communication unidirectionnelle entre Citadelle et proxy via authentification éphémère :
- La Citadelle reçoit un bloc d'instructions et les clés nécessaires avec une durée maximale limitée
- En sortie, elle initie une connexion éphémère vers le proxy
- Les connexions s'invalident automatiquement après usage, garantissant qu'aucune session persistante ne reste ouverte

### Le coffre-fort
C'est là où se trouvent toutes les clés et secrets gardés précieusement, accessible uniquement par les proxys dédiés locaux.

## Gestion des révocations et rotations

### Révocation
Détectée à la prochaine requête via la Guardhouse. Pour les transferts de données volumineuses, l'utilisateur révoqué recevra des chunks incomplets mais inutilisables sans validation finale.

### Rotation des clés
Déclenchée uniquement lors de changements sur les services/données de la Citadelle. La synchronisation entre coffre-fort et Citadelle s'effectue via cryptographie asymétrique (clés publiques/privées partagées).

### Continuité
Les proxys ne sont pas notifiés immédiatement mais découvrent les changements lors de leur prochaine interaction, permettant une architecture résiliente et découplée.

