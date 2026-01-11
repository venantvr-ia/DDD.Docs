# Trading - Application DDD

Application des principes DDD au domaine du trading algorithmique.

## Grid Trading

Le grid trading est une stratégie consistant à acheter et vendre à des niveaux de prix prédéterminés, créant une grille de positions.

### Paramètres clés

| Paramètre | Description |
|-----------|-------------|
| `grid_size` | Écart entre chaque niveau |
| `upper_price` | Prix plafond |
| `lower_price` | Prix plancher |
| `amount` | Quantité par ordre |
| `max_positions` | Nombre maximum de positions |

### Exemple d'implémentation

```python
import ccxt

class GridTrader:
    def __init__(self, exchange, symbol, grid_size, upper, lower, amount):
        self.exchange = exchange
        self.symbol = symbol
        self.grid_size = grid_size
        self.upper_price = upper
        self.lower_price = lower
        self.amount = amount

    def calculate_grid(self, max_positions):
        grid = []
        for i in range(max_positions):
            buy_price = self.lower_price + i * self.grid_size
            sell_price = self.upper_price - i * self.grid_size
            grid.append((buy_price, sell_price))
        return grid

    def execute(self):
        ticker = self.exchange.fetch_ticker(self.symbol)
        price = ticker['last']
        # Logique d'exécution...
```

## Optimisation par Machine Learning

L'optimisation de la grille peut utiliser des données historiques pour affiner les paramètres.

### Approche

1. **Collecte** : Données historiques (OHLCV)
2. **Préparation** : Nettoyage et normalisation
3. **Modélisation** : Réseau de neurones (CNN/LSTM)
4. **Entraînement** : Sur données historiques
5. **Optimisation** : Ajustement des paramètres de grille

### Modèle CNN pour prédiction

```python
from keras.models import Sequential
from keras.layers import Conv1D, MaxPooling1D, Flatten, Dense

model = Sequential([
    Conv1D(filters=64, kernel_size=2, activation='relu', input_shape=(3, 1)),
    MaxPooling1D(pool_size=2),
    Flatten(),
    Dense(50, activation='relu'),
    Dense(1, activation='linear')
])
model.compile(loss='mse', optimizer='adam')
```

## Value Objects Trading

Objets de valeur spécifiques au domaine trading :

- `Price` : Prix avec validation
- `Quantity` : Quantité avec contraintes
- `Symbol` : Paire de trading
- `OrderSide` : BUY/SELL

## Domain Events Trading

Événements typiques :
- `OrderPlaced` : Ordre passé
- `OrderFilled` : Ordre exécuté
- `PositionOpened` : Position ouverte
- `PositionClosed` : Position fermée
