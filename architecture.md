# Architecture DDD

Patterns et structures fondamentaux du Domain-Driven Design.

## Patterns principaux

### Entity (Entité)

Un objet avec une identité unique qui persiste dans le temps. Les entités représentent des éléments du domaine métier ayant une identité et une durée de vie.

### Value Object (Objet de valeur)

Un objet sans identité unique, immuable. Utilisé pour représenter des concepts ou propriétés du domaine sans identité propre.

```python
class Email:
    def __init__(self, email: str):
        if not self._is_valid_email(email):
            raise ValueError(f"L'e-mail '{email}' n'est pas valide")
        self._email = email

    def __eq__(self, other):
        if isinstance(other, Email):
            return self._email == other._email
        return False

    def __hash__(self):
        return hash(self._email)
```

### Aggregate (Agrégat)

Un ensemble d'entités et d'objets de valeur formant une unité cohérente. Les agrégats sont traités comme une seule unité pour les opérations de persistance.

**Caractéristiques :**
- **Racine d'agrégat** : Entité unique responsable de l'intégrité de l'ensemble
- **Limites de cohérence** : Règles d'accès et de modification
- **Invariants métier** : Règles devant toujours être respectées

```python
class Order:
    id: int
    customer_id: int
    shipping_address: Address
    order_lines: List[OrderLine]
    status: OrderStatus

    def add_order_line(self, product: Product, quantity: int):
        order_line = OrderLine(product, quantity)
        self.order_lines.append(order_line)

    def update_status(self, new_status: OrderStatus):
        self.status = new_status
```

### Repository (Dépôt)

Pattern fournissant des méthodes pour stocker, récupérer et gérer les agrégats. Abstrait la logique de persistance.

### Factory (Fabrique)

Encapsule la logique de création et d'initialisation des objets du domaine.

### Service (Service métier)

Encapsule des opérations métier non naturellement associées à un objet spécifique. Opérations indépendantes et souvent sans état.

## Réduction de la dette technique

Le DDD réduit la dette technique par :

1. **Code clair et simple** : Modèles reflétant les processus métier
2. **Tests rigoureux** : Détection rapide des erreurs
3. **Évolutivité** : Logiciels faciles à modifier dans le temps
