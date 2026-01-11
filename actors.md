# Actor Model et Domain Events

Conciliation de l'Actor Model et des Domain Events avec des outils comme RabbitMQ et Thespian.

## Concepts

### Actor Model

Approche de programmation concurrente pour construire des systèmes distribués avec des entités autonomes (acteurs) communiquant par échange de messages asynchrones.

### Domain Events

Technique de conception consistant à modéliser les événements du domaine sous forme d'objets pour déclencher des actions dans le système.

## Caractéristiques des Domain Events

- **Immutabilité** : Une fois créés, ils ne peuvent pas être modifiés
- **Notification** : Informent les autres parties du système des changements d'état
- **Traçabilité** : Enregistrement précis de ce qui s'est passé

## Implémentation avec RabbitMQ

### Mode Topic

1. **Définir les sujets** : Format reflétant les domaines (`commandes.creee`, `paiements.recu`)
2. **Publier les événements** : Messages avec le sujet correspondant
3. **Écouter les messages** : Acteurs abonnés aux sujets pertinents
4. **Déclencher les actions** : Réactions aux événements reçus

### Nommage des événements

Convention recommandée : nommer au **passé** pour refléter que l'événement a déjà eu lieu.

| Au lieu de | Préférer |
|------------|----------|
| création de commande | commande créée |
| modification de profil | profil modifié |

## Event-Sourcing

Approche de gestion de l'état stockant l'historique complet des événements plutôt qu'un état actuel.

**Avantages :**
- Reconstruction de l'état à tout moment
- Traçabilité et auditabilité
- Résilience accrue

## Intégration inter-systèmes

Les Domain Events facilitent l'intégration entre systèmes en agissant comme mécanisme de communication basé sur les messages, permettant une intégration souple et décentralisée.
