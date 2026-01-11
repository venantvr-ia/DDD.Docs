# Architecture DDD - Patterns et Structures

Les patterns fondamentaux du Domain-Driven Design pour structurer et organiser le modèle de domaine.

## Patterns principaux

### Entity (Entité)

Un objet qui a une identité unique et qui persiste dans le temps. Les entités représentent généralement des éléments du domaine métier ayant une identité et une durée de vie.

```python
class Client:
    def __init__(self, client_id: str, nom: str, email: str):
        self._id = client_id  # Identité unique
        self.nom = nom
        self.email = email

    @property
    def id(self) -> str:
        return self._id

    def __eq__(self, other):
        if isinstance(other, Client):
            return self._id == other._id
        return False

    def __hash__(self):
        return hash(self._id)
```

### Value Object (Objet de valeur)

Un objet qui n'a pas d'identité unique et qui est immuable. Les objets de valeur sont utilisés pour représenter des concepts ou des propriétés du domaine métier sans identité propre.

```python
class Email:
    def __init__(self, email: str):
        if not self._is_valid_email(email):
            raise ValueError(f"L'e-mail '{email}' n'est pas valide")
        self._email = email

    def __str__(self):
        return self._email

    def __repr__(self):
        return f"Email('{self._email}')"

    def __eq__(self, other):
        if isinstance(other, Email):
            return self._email == other._email
        return False

    def __hash__(self):
        return hash(self._email)

    @staticmethod
    def _is_valid_email(email: str) -> bool:
        import re
        email_regex = r"[^@]+@[^@]+\.[^@]+"
        return bool(re.match(email_regex, email))
```

### Aggregate (Agrégat)

Un ensemble d'entités et d'objets de valeur qui forment une unité cohérente. Les agrégats sont traités comme une seule unité pour les opérations de persistance et de modification, et ils garantissent la consistance du domaine métier.

#### Caractéristiques des agrégats

1. **Racine d'agrégat** : Chaque agrégat a une racine d'agrégat, qui est une entité unique responsable de l'intégrité et de la cohérence de l'ensemble de l'agrégat. Les autres objets de l'agrégat ne sont accessibles qu'à travers la racine, ce qui permet d'encapsuler la logique métier et de garantir la validité des opérations.

2. **Limites de cohérence** : Les agrégats définissent des limites de cohérence, qui déterminent les règles et les contraintes d'accès et de modification des objets à l'intérieur de l'agrégat. Ces limites garantissent que les opérations sur l'agrégat sont atomiques et que l'état de l'agrégat est toujours cohérent après chaque opération.

3. **Invariants métier** : Les agrégats sont conçus pour protéger et maintenir les invariants métier, c'est-à-dire les règles ou les contraintes qui doivent toujours être respectées par le domaine. En regroupant les objets du domaine liés par des invariants métier, les agrégats permettent de gérer efficacement la complexité et d'assurer l'intégrité des données.

4. **Optimisation des performances** : En délimitant les limites de cohérence et en regroupant les objets du domaine liés, les agrégats permettent de réduire les coûts de communication et de synchronisation entre les objets, d'optimiser les performances du système et de faciliter la scalabilité.

```python
class Order:
    """Aggregate Root - Racine d'agrégat pour une commande"""

    def __init__(self, order_id: str, customer_id: str):
        self._id = order_id
        self._customer_id = customer_id
        self._shipping_address: Address = None
        self._order_lines: List[OrderLine] = []
        self._status: OrderStatus = OrderStatus.DRAFT

    @property
    def id(self) -> str:
        return self._id

    def add_order_line(self, product: Product, quantity: int):
        """Ajoute une ligne de commande en respectant les invariants"""
        if self._status != OrderStatus.DRAFT:
            raise InvalidOperationError("Impossible d'ajouter à une commande validée")

        order_line = OrderLine(product, quantity)
        self._order_lines.append(order_line)

    def remove_order_line(self, order_line_id: str):
        """Supprime une ligne de commande"""
        if self._status != OrderStatus.DRAFT:
            raise InvalidOperationError("Impossible de modifier une commande validée")

        self._order_lines = [
            line for line in self._order_lines
            if line.id != order_line_id
        ]

    def validate(self):
        """Valide la commande en vérifiant les invariants"""
        if not self._order_lines:
            raise InvalidOperationError("Commande vide")
        if not self._shipping_address:
            raise InvalidOperationError("Adresse de livraison requise")

        self._status = OrderStatus.VALIDATED

    def get_total(self) -> Money:
        """Calcule le total de la commande"""
        return sum(line.get_subtotal() for line in self._order_lines)


class OrderLine:
    """Entité appartenant à l'agrégat Order"""

    def __init__(self, product: Product, quantity: int):
        self._id = str(uuid.uuid4())
        self._product = product
        self._quantity = quantity

    def get_subtotal(self) -> Money:
        return self._product.price * self._quantity
```

### Domain Event (Événement du domaine)

Un événement qui représente un changement d'état significatif dans le domaine métier. Les événements du domaine sont utilisés pour déclencher des actions ou des comportements en réponse à ces changements.

Les événements sont **immuables** : une fois créés, ils ne peuvent pas être modifiés. Cela garantit la fiabilité et la traçabilité.

```python
from dataclasses import dataclass
from datetime import datetime

@dataclass(frozen=True)
class OrderValidated:
    """Événement émis quand une commande est validée"""
    order_id: str
    customer_id: str
    total: Money
    validated_at: datetime
```

### Repository (Dépôt)

Un pattern qui fournit des méthodes pour stocker, récupérer et gérer les agrégats et les entités. Les dépôts abstraient la logique de persistance et permettent d'interagir avec les objets du domaine sans se soucier de la manière dont ils sont stockés.

```python
from abc import ABC, abstractmethod

class OrderRepository(ABC):
    @abstractmethod
    def save(self, order: Order) -> None:
        pass

    @abstractmethod
    def find_by_id(self, order_id: str) -> Optional[Order]:
        pass

    @abstractmethod
    def find_by_customer(self, customer_id: str) -> List[Order]:
        pass


class SqlOrderRepository(OrderRepository):
    def __init__(self, session):
        self._session = session

    def save(self, order: Order) -> None:
        self._session.add(order)
        self._session.commit()

    def find_by_id(self, order_id: str) -> Optional[Order]:
        return self._session.query(Order).filter_by(id=order_id).first()
```

### Factory (Fabrique)

Un pattern utilisé pour encapsuler la logique de création et d'initialisation des objets du domaine. Les fabriques permettent de centraliser et de simplifier la création d'objets complexes.

```python
class OrderFactory:
    @staticmethod
    def create_from_cart(cart: ShoppingCart, customer: Customer) -> Order:
        """Crée une commande à partir d'un panier"""
        order = Order(
            order_id=str(uuid.uuid4()),
            customer_id=customer.id
        )
        order.set_shipping_address(customer.default_address)

        for item in cart.items:
            order.add_order_line(item.product, item.quantity)

        return order
```

### Service (Service métier)

Un pattern utilisé pour encapsuler des opérations métier qui ne sont pas naturellement associées à un objet spécifique du domaine. Les services métier sont des opérations indépendantes des objets sur lesquels ils opèrent et sont souvent sans état.

```python
class PaymentService:
    def __init__(self, payment_gateway: PaymentGateway, order_repository: OrderRepository):
        self._gateway = payment_gateway
        self._repository = order_repository

    def process_payment(self, order_id: str, payment_method: PaymentMethod) -> PaymentResult:
        """Traite le paiement d'une commande"""
        order = self._repository.find_by_id(order_id)
        if not order:
            raise OrderNotFoundError(order_id)

        result = self._gateway.charge(order.get_total(), payment_method)

        if result.success:
            order.mark_as_paid(result.transaction_id)
            self._repository.save(order)

        return result
```

## Résumé des patterns

| Pattern | Description | Caractéristique clé |
|---------|-------------|---------------------|
| Entity | Objet avec identité unique | Identité persistante |
| Value Object | Objet sans identité, immuable | Égalité par valeur |
| Aggregate | Groupe d'objets cohérents | Racine d'agrégat |
| Domain Event | Changement d'état significatif | Immuable |
| Repository | Abstraction de persistance | Interface de stockage |
| Factory | Création d'objets complexes | Encapsulation de la création |
| Service | Opération sans état | Logique transverse |
