# W2CLA - Un chemin vers l’architecture logique de la Citadelle

`Way to Citadel Logical Architecture` est un projet expérimental et open source proposant un protocole de fortification des clés et mots de passe partagés.

L’objectif est de ne jamais fournir directement de clés ou secrets sensibles, et d’attribuer à chaque utilisateur une identité forte lui permettant d’accéder aux données et services (API, applications internes, etc.) hébergés au sein d’une Citadelle en passant par des proxys dédiés spécialisés dans l’accès et l’échange sécurisés de données.

Cette approche vise à faciliter la sécurité, la conformité et la gestion granulaire des accès.

## Avantages

- Un secret compromis ne met pas en danger l’ensemble de l’organisation, la rotation et la révocation sont facilitées pour chaque utilisateur
- Traçabilité renforcée, chaque accès peut être enregistré et identifié (qui, quoi, quand)
- Onboarding/offboarding automatisés, rotation centralisée des secrets : *les nouveaux utilisateurs reçoivent automatiquement leurs accès via leur identité, et lors d’une révocation, tous les accès sont instantanément supprimés depuis un point central*

L’objectif idéal serait de tendre vers un système open source adoptant la philosophie Zero Trust tout en offrant une alternative ouverte aux solutions de sécurité propriétaires : fournir une solution accessible aux entités souhaitant une approche ouverte et transparente de la sécurité.

## Architecture technique

### Architecture locale
Un seul coffre-fort local hébergé aux côtés de la Citadelle au sein de l’entité, éliminant la complexité de synchronisation entre plusieurs instances.

### Proxys
Deux types coexistent :
- **Proxys dédiés locaux** : ont un accès direct au coffre-fort et gèrent l’authentification primaire. En cas d’échec, l’architecture permet un basculement automatique vers un autre proxy
- **Proxys distants** : doivent passer par les proxys dédiés pour accéder aux secrets, créant une chaîne de confiance hiérarchique

### Guardhouse
Module de validation situé aux côtés de la Citadelle, permettant à tous les proxys de vérifier la validité des utilisateurs avant toute action sur la Citadelle ou le coffre-fort. Ce module centralise les statuts de révocation et les permissions en temps réel. En cas de défaillance, aucun accès n’est possible jusqu’à la restauration. Une entité de secours peut être prévue pour la redondance.

### Guardhouse auto-régénérante
En cas de corruption complète, la Citadelle conserve les métadonnées des utilisateurs (ID et journaux d’accès). Un processus de reconstruction supervisée permet à un administrateur de relancer le Guardhouse : les utilisateurs sont progressivement réintégrés selon leurs interactions avec la Citadelle, avec validation ou refus manuel de l’administrateur via une interface de supervision.

### Citadelles
Elles hébergent soit les accès, soit directement les services et données partagés au sein de l’organisation. Elles conservent les métadonnées utilisateurs nécessaires à la traçabilité et à la régénération du système.

### Utilisateurs
Ils possèdent leur trousseau ou portefeuille contenant leurs secrets/clés basés sur les standards DID (Decentralized Identifiers), assurant l’interopérabilité avec d’autres systèmes W2CLA et une gestion d’identité standardisée.

### APIs

#### API externe
Interface les utilisateurs avec les proxys et gère l’identité décentralisée via les standards DID. Cette API sert également de passerelle pour la récupération et la transmission de données entre un service de Citadelle et l’utilisateur.

#### API interne
Communication unidirectionnelle entre Citadelle et proxy via authentification éphémère :
- La Citadelle reçoit un bloc d’instructions et les clés nécessaires avec une durée maximale limitée
- En sortie, elle initie une connexion éphémère vers le proxy
- Les connexions sont automatiquement invalidées après utilisation, garantissant qu’aucune session persistante ne reste ouverte

### Le coffre-fort
C’est l’endroit où toutes les clés et secrets sont soigneusement stockés, accessibles uniquement par les proxys dédiés locaux.

## Gestion de la révocation et de la rotation

### Révocation
Détectée lors de la prochaine requête via le Guardhouse. Pour les transferts de données volumineux, l’utilisateur révoqué recevra des fragments incomplets, inutilisables sans validation finale.

### Rotation des clés
Déclenchée uniquement lors de modifications des services ou données de la Citadelle. La synchronisation entre le coffre-fort et la Citadelle s’effectue via cryptographie asymétrique (clés publiques/privées partagées).

### Continuité
Les proxys ne sont pas immédiatement notifiés mais découvrent les changements lors de leur prochaine interaction, permettant une architecture résiliente et découplée.

## Composants principaux

- [w2cla-vault](https://github.com/Th6uD1nk/w2cla-vault)  
  Module coffre-fort responsable du stockage sécurisé des clés et secrets.

- [w2cla-proxy-dedicated](https://github.com/Th6uD1nk/w2cla-proxy-dedicated)  
  Proxys dédiés gérant l’authentification primaire et l’accès direct au coffre-fort.

- [w2cla-proxy-remote](https://github.com/Th6uD1nk/w2cla-proxy-remote)  
  Proxys distants accédant aux secrets via les proxys dédiés, établissant une hiérarchie de confiance.

- [w2cla-guardhouse](https://github.com/Th6uD1nk/w2cla-guardhouse)  
  Module de validation centralisant les statuts utilisateurs, les permissions et la révocation.

- [w2cla-citadel](https://github.com/Th6uD1nk/w2cla-citadel)  
  Héberge les services et données partagés au sein de l’organisation, tout en conservant les métadonnées utilisateurs pour la traçabilité.

- [w2cla-monitoring](https://github.com/Th6uD1nk/w2cla-monitoring)  
  Supervision centralisée et observabilité de l’ensemble des modules W2CLA.
