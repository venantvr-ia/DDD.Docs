# Actor Model et Domain Events

Comment concilier l'Actor Model et les Domain Events avec des outils comme RabbitMQ et Thespian pour construire des systèmes distribués robustes.

## Concepts fondamentaux

### Actor Model

L'Actor Model est une approche de la programmation concurrente qui permet de construire des systèmes distribués en utilisant des entités autonomes appelées acteurs qui communiquent par échange de messages asynchrones. Chaque acteur :

- Possède son propre état interne
- Traite les messages un par un
- Peut créer d'autres acteurs
- Peut envoyer des messages à d'autres acteurs

### Domain Events

Le Domain Events est une technique de conception de logiciels qui consiste à modéliser les événements du domaine de l'application sous forme d'objets et à les utiliser pour déclencher des actions dans le système. Un événement du domaine représente quelque chose qui s'est passé et qui est significatif pour le métier.

## RabbitMQ et Thespian

### RabbitMQ

RabbitMQ est un système de messagerie qui permet de transférer des messages entre différentes applications. Il supporte plusieurs patterns de communication :

- **Direct** : Message envoyé à une queue spécifique
- **Fanout** : Message envoyé à toutes les queues liées
- **Topic** : Message routé selon un pattern de routing key
- **Headers** : Routage basé sur les en-têtes du message

### Thespian

Thespian est une bibliothèque Python qui permet de créer des acteurs et de communiquer avec eux. Elle fournit :

- Création et gestion d'acteurs
- Communication asynchrone
- Supervision et tolérance aux pannes
- Distribution sur plusieurs nœuds

## Implémentation en mode Topic

Pour concilier l'Actor Model et le Domain Events avec RabbitMQ et Thespian en mode Topic :

### 1. Définir les événements comme sujets

Définissez les événements du domaine sous forme de sujets (topics) en utilisant un format de nommage qui reflète les domaines applicatifs pertinents :

```python
# Format: domaine.action
TOPICS = {
    "commandes.creee": "Nouvelle commande créée",
    "commandes.validee": "Commande validée",
    "paiements.recu": "Paiement reçu",
    "paiements.refuse": "Paiement refusé",
    "livraisons.expediee": "Livraison expédiée",
}
```

### 2. Publier les événements

Utilisez RabbitMQ pour publier les événements du domaine sous forme de messages avec le sujet correspondant :

```python
import pika
import json

class EventPublisher:
    def __init__(self, connection_params):
        self.connection = pika.BlockingConnection(connection_params)
        self.channel = self.connection.channel()
        self.channel.exchange_declare(exchange='domain_events', exchange_type='topic')

    def publish(self, topic: str, event: dict):
        """Publie un événement sur le topic spécifié"""
        message = json.dumps(event)
        self.channel.basic_publish(
            exchange='domain_events',
            routing_key=topic,
            body=message
        )

# Utilisation
publisher = EventPublisher(pika.ConnectionParameters('localhost'))
publisher.publish('commandes.creee', {
    'order_id': '12345',
    'customer_id': 'C001',
    'total': 150.00,
    'created_at': '2024-01-15T10:30:00Z'
})
```

### 3. Créer des acteurs pour écouter les événements

Utilisez Thespian pour créer des acteurs qui écoutent les messages publiés pour des sujets spécifiques :

```python
from thespian.actors import Actor, ActorSystem

class OrderProcessor(Actor):
    """Acteur qui traite les événements de commande"""

    def receiveMessage(self, message, sender):
        if message['type'] == 'commandes.creee':
            self.handle_order_created(message['data'])
        elif message['type'] == 'commandes.validee':
            self.handle_order_validated(message['data'])

    def handle_order_created(self, data):
        # Traitement de la création de commande
        order_id = data['order_id']
        # Vérifier le stock, calculer les taxes, etc.
        print(f"Traitement commande {order_id}")


class PaymentProcessor(Actor):
    """Acteur qui traite les événements de paiement"""

    def receiveMessage(self, message, sender):
        if message['type'] == 'paiements.recu':
            self.handle_payment_received(message['data'])

    def handle_payment_received(self, data):
        # Mise à jour du statut de la commande
        # Envoi d'un email de confirmation
        pass
```

### 4. Consumer RabbitMQ vers Acteurs

Bridge entre RabbitMQ et les acteurs Thespian :

```python
class RabbitMQActorBridge:
    def __init__(self, actor_system, actor_address):
        self.actor_system = actor_system
        self.actor_address = actor_address

    def start_consuming(self, topics: list):
        connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
        channel = connection.channel()

        channel.exchange_declare(exchange='domain_events', exchange_type='topic')
        result = channel.queue_declare(queue='', exclusive=True)
        queue_name = result.method.queue

        for topic in topics:
            channel.queue_bind(exchange='domain_events', queue=queue_name, routing_key=topic)

        def callback(ch, method, properties, body):
            event = json.loads(body)
            message = {
                'type': method.routing_key,
                'data': event
            }
            self.actor_system.tell(self.actor_address, message)

        channel.basic_consume(queue=queue_name, on_message_callback=callback, auto_ack=True)
        channel.start_consuming()
```

## Nommage des événements

### Convention : passé ou présent ?

Il n'y a pas de règle stricte pour nommer les événements du domaine au passé ou au présent. Cependant, la convention recommandée est de nommer les événements au **passé**, car cela reflète le fait que l'événement a déjà eu lieu et a eu un impact sur l'état actuel du système.

| Au lieu de | Préférer |
|------------|----------|
| création de commande | CommandeCréée |
| modification de profil | ProfilModifié |
| validation de paiement | PaiementValidé |
| expédition de colis | ColisExpédié |

L'important est de choisir un format de nommage cohérent et facile à comprendre pour tous les développeurs impliqués dans le projet.

## Event-Sourcing

L'Event-Sourcing est une approche de la gestion de l'état d'un système qui consiste à stocker l'historique complet des événements du domaine plutôt qu'à maintenir un état actuel.

### Avantages

- **Traçabilité complète** : On peut reconstruire l'état du système à tout moment
- **Auditabilité** : Historique complet de toutes les actions
- **Résilience** : Possibilité de rejouer les événements
- **Debug facilité** : On peut reproduire exactement une situation passée

### Implémentation

```python
class EventStore:
    def __init__(self):
        self._events = []

    def append(self, event: DomainEvent):
        """Ajoute un événement au store"""
        self._events.append({
            'type': type(event).__name__,
            'data': event.__dict__,
            'timestamp': datetime.utcnow()
        })

    def get_events(self, aggregate_id: str) -> list:
        """Récupère tous les événements d'un agrégat"""
        return [e for e in self._events if e['data'].get('aggregate_id') == aggregate_id]

    def replay(self, aggregate_id: str) -> Aggregate:
        """Reconstruit l'état d'un agrégat à partir de ses événements"""
        events = self.get_events(aggregate_id)
        aggregate = Aggregate(aggregate_id)
        for event in events:
            aggregate.apply(event)
        return aggregate
```

## Intégration entre systèmes

Les événements du domaine peuvent être utilisés pour faciliter l'intégration entre différents systèmes ou services, en agissant comme un mécanisme de communication basé sur les messages.

### Avantages

- **Découplage** : Les systèmes ne se connaissent pas directement
- **Évolutivité** : Ajout facile de nouveaux consommateurs
- **Résilience** : Les messages sont persistés en cas de panne
- **Asynchronicité** : Pas de blocage entre les systèmes

### Architecture

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Service    │     │  RabbitMQ   │     │  Service    │
│  Commandes  │────▶│  (Topics)   │◀────│  Stock      │
└─────────────┘     └──────┬──────┘     └─────────────┘
                           │
                    ┌──────┴──────┐
                    │             │
              ┌─────┴─────┐ ┌─────┴─────┐
              │  Service  │ │  Service  │
              │  Paiement │ │  Notif    │
              └───────────┘ └───────────┘
```

Chaque service publie ses événements et s'abonne aux événements qui l'intéressent, créant ainsi une architecture réactive et scalable.
