# Trading - Application DDD

Application des principes du Domain-Driven Design au domaine du trading algorithmique.

## Grid Trading

Le grid trading est une stratégie de trading qui consiste à acheter et vendre à des niveaux de prix prédéterminés, créant ainsi une grille de positions. Cette approche est particulièrement adaptée aux marchés volatils avec des mouvements de prix latéraux.

### Principe de fonctionnement

1. Définir une plage de prix (upper_price, lower_price)
2. Créer une grille avec des niveaux régulièrement espacés
3. Placer des ordres d'achat en bas de la grille
4. Placer des ordres de vente en haut de la grille
5. Quand un ordre est exécuté, un nouvel ordre est placé au niveau opposé

### Paramètres clés

| Paramètre | Description |
|-----------|-------------|
| `grid_size` | Écart entre chaque niveau de la grille |
| `upper_price` | Prix plafond de la grille |
| `lower_price` | Prix plancher de la grille |
| `amount` | Quantité à trader par ordre |
| `max_positions` | Nombre maximum de positions ouvertes |

### Implémentation avec DDD

```python
import ccxt
import time
from dataclasses import dataclass
from typing import List, Tuple
from enum import Enum

# Value Objects
@dataclass(frozen=True)
class Price:
    value: float

    def __post_init__(self):
        if self.value <= 0:
            raise ValueError("Le prix doit être positif")

@dataclass(frozen=True)
class Quantity:
    value: float

    def __post_init__(self):
        if self.value <= 0:
            raise ValueError("La quantité doit être positive")

@dataclass(frozen=True)
class Symbol:
    base: str
    quote: str

    def __str__(self):
        return f"{self.base}/{self.quote}"


# Aggregate
class GridTrader:
    """Aggregate Root pour le grid trading"""

    def __init__(self, exchange, symbol: Symbol, grid_size: Price,
                 upper: Price, lower: Price, amount: Quantity, max_positions: int):
        self._exchange = exchange
        self._symbol = symbol
        self._grid_size = grid_size
        self._upper_price = upper
        self._lower_price = lower
        self._amount = amount
        self._max_positions = max_positions
        self._open_orders: List[Order] = []

    def calculate_grid(self) -> List[Tuple[Price, Price]]:
        """Calcule les niveaux de la grille"""
        grid = []
        for i in range(self._max_positions):
            buy_price = Price(self._lower_price.value + i * self._grid_size.value)
            sell_price = Price(self._upper_price.value - i * self._grid_size.value)
            grid.append((buy_price, sell_price))
        return grid

    def get_current_price(self) -> Price:
        """Récupère le prix actuel du marché"""
        ticker = self._exchange.fetch_ticker(str(self._symbol))
        return Price(ticker['last'])

    def execute_cycle(self):
        """Exécute un cycle de trading"""
        current_price = self.get_current_price()
        grid = self.calculate_grid()
        positions = self._exchange.fetch_open_orders(str(self._symbol))

        # Vérifier les ordres existants
        for position in positions:
            for buy_price, sell_price in grid:
                if position['side'] == 'buy' and position['price'] <= sell_price.value:
                    self._place_sell_order(sell_price)
                    break
                if position['side'] == 'sell' and position['price'] >= buy_price.value:
                    self._place_buy_order(buy_price)
                    break

        # Ouvrir de nouvelles positions si nécessaire
        if len(positions) < self._max_positions * 2:
            self._fill_grid(grid, positions)

    def _place_buy_order(self, price: Price):
        self._exchange.create_order(
            str(self._symbol), 'buy', 'limit',
            self._amount.value, price.value
        )

    def _place_sell_order(self, price: Price):
        self._exchange.create_order(
            str(self._symbol), 'sell', 'limit',
            self._amount.value, price.value
        )

    def _fill_grid(self, grid, existing_positions):
        """Remplit les niveaux vides de la grille"""
        for buy_price, sell_price in grid:
            # Vérifier si un ordre buy existe déjà à ce prix
            if not any(p['side'] == 'buy' and p['price'] == buy_price.value
                      for p in existing_positions):
                self._place_buy_order(buy_price)

            # Vérifier si un ordre sell existe déjà à ce prix
            if not any(p['side'] == 'sell' and p['price'] == sell_price.value
                      for p in existing_positions):
                self._place_sell_order(sell_price)
```

## Optimisation par Machine Learning

L'optimisation de la grille peut utiliser des données historiques et des techniques de machine learning pour affiner les paramètres.

### Étapes d'optimisation

1. **Collecter les données historiques** : Récupérer les données OHLCV (Open, High, Low, Close, Volume) pour une période donnée

2. **Préparer les données** : Nettoyage, normalisation et création des features

3. **Définir le modèle** : CNN, LSTM ou autre architecture adaptée aux séries temporelles

4. **Entraîner le modèle** : Sur les données historiques avec validation croisée

5. **Optimiser les paramètres** : Utiliser les prédictions pour ajuster la grille

### Modèle CNN pour prédiction de prix

```python
from keras.models import Sequential
from keras.layers import Conv1D, MaxPooling1D, Flatten, Dense, LSTM, Dropout
from sklearn.preprocessing import MinMaxScaler
import numpy as np

class PricePredictor:
    def __init__(self):
        self.model = None
        self.scaler = MinMaxScaler(feature_range=(0, 1))

    def build_cnn_model(self, input_shape):
        """Construit un modèle CNN pour la prédiction de prix"""
        model = Sequential([
            Conv1D(filters=64, kernel_size=2, activation='relu', input_shape=input_shape),
            MaxPooling1D(pool_size=2),
            Conv1D(filters=32, kernel_size=2, activation='relu'),
            Flatten(),
            Dense(50, activation='relu'),
            Dropout(0.2),
            Dense(1, activation='linear')
        ])
        model.compile(loss='mse', optimizer='adam', metrics=['mae'])
        self.model = model
        return model

    def build_lstm_model(self, input_shape):
        """Construit un modèle LSTM pour la prédiction de prix"""
        model = Sequential([
            LSTM(50, return_sequences=True, input_shape=input_shape),
            Dropout(0.2),
            LSTM(50, return_sequences=False),
            Dropout(0.2),
            Dense(25),
            Dense(1)
        ])
        model.compile(optimizer='adam', loss='mse')
        self.model = model
        return model

    def prepare_data(self, data, look_back=60):
        """Prépare les données pour l'entraînement"""
        scaled_data = self.scaler.fit_transform(data.reshape(-1, 1))

        X, y = [], []
        for i in range(look_back, len(scaled_data)):
            X.append(scaled_data[i-look_back:i, 0])
            y.append(scaled_data[i, 0])

        return np.array(X), np.array(y)

    def train(self, X, y, epochs=50, batch_size=32, validation_split=0.2):
        """Entraîne le modèle"""
        X = np.reshape(X, (X.shape[0], X.shape[1], 1))
        return self.model.fit(
            X, y,
            epochs=epochs,
            batch_size=batch_size,
            validation_split=validation_split,
            verbose=1
        )

    def predict(self, X):
        """Fait une prédiction"""
        X = np.reshape(X, (X.shape[0], X.shape[1], 1))
        scaled_prediction = self.model.predict(X)
        return self.scaler.inverse_transform(scaled_prediction)
```

### Optimiseur de grille

```python
import pandas as pd
import numpy as np

class GridOptimizer:
    def __init__(self, predictor: PricePredictor):
        self.predictor = predictor

    def optimize_grid_parameters(self, historical_data, predictions):
        """Optimise les paramètres de la grille basé sur les prédictions"""

        # Définir l'espace de recherche
        grid_sizes = np.linspace(10, 100, 10)
        max_prices = np.linspace(historical_data.max() * 0.9, historical_data.max() * 1.1, 10)
        min_prices = np.linspace(historical_data.min() * 0.9, historical_data.min() * 1.1, 10)

        best_params = None
        best_profit = float('-inf')

        for grid_size in grid_sizes:
            for max_price in max_prices:
                for min_price in min_prices:
                    if max_price <= min_price:
                        continue

                    profit = self._simulate_trading(
                        historical_data, predictions,
                        grid_size, max_price, min_price
                    )

                    if profit > best_profit:
                        best_profit = profit
                        best_params = {
                            'grid_size': grid_size,
                            'upper_price': max_price,
                            'lower_price': min_price
                        }

        return best_params, best_profit

    def _simulate_trading(self, data, predictions, grid_size, upper, lower):
        """Simule le trading pour évaluer les paramètres"""
        # Implémentation du backtest
        # ...
        pass
```

## Value Objects pour le Trading

Objets de valeur spécifiques au domaine du trading :

```python
from dataclasses import dataclass
from decimal import Decimal
from enum import Enum

class OrderSide(Enum):
    BUY = "buy"
    SELL = "sell"

class OrderType(Enum):
    MARKET = "market"
    LIMIT = "limit"
    STOP_LOSS = "stop_loss"
    TAKE_PROFIT = "take_profit"

@dataclass(frozen=True)
class Money:
    amount: Decimal
    currency: str

    def __add__(self, other: 'Money') -> 'Money':
        if self.currency != other.currency:
            raise ValueError("Impossible d'additionner des devises différentes")
        return Money(self.amount + other.amount, self.currency)

    def __mul__(self, factor: Decimal) -> 'Money':
        return Money(self.amount * factor, self.currency)

@dataclass(frozen=True)
class TradingPair:
    base_currency: str
    quote_currency: str

    def __str__(self):
        return f"{self.base_currency}/{self.quote_currency}"

@dataclass(frozen=True)
class PriceLevel:
    price: Decimal
    side: OrderSide
```

## Domain Events pour le Trading

Événements typiques du domaine trading :

```python
from dataclasses import dataclass
from datetime import datetime
from decimal import Decimal

@dataclass(frozen=True)
class OrderPlaced:
    """Événement émis quand un ordre est placé"""
    order_id: str
    symbol: str
    side: OrderSide
    price: Decimal
    quantity: Decimal
    placed_at: datetime

@dataclass(frozen=True)
class OrderFilled:
    """Événement émis quand un ordre est exécuté"""
    order_id: str
    fill_price: Decimal
    fill_quantity: Decimal
    commission: Decimal
    filled_at: datetime

@dataclass(frozen=True)
class PositionOpened:
    """Événement émis quand une position est ouverte"""
    position_id: str
    symbol: str
    side: OrderSide
    entry_price: Decimal
    quantity: Decimal
    opened_at: datetime

@dataclass(frozen=True)
class PositionClosed:
    """Événement émis quand une position est fermée"""
    position_id: str
    exit_price: Decimal
    realized_pnl: Decimal
    closed_at: datetime

@dataclass(frozen=True)
class StopLossTriggered:
    """Événement émis quand un stop loss est déclenché"""
    position_id: str
    trigger_price: Decimal
    triggered_at: datetime
```

## Architecture complète

```
trading/
├── domain/
│   ├── entities/
│   │   ├── order.py
│   │   ├── position.py
│   │   └── grid.py
│   ├── value_objects/
│   │   ├── money.py
│   │   ├── price.py
│   │   └── quantity.py
│   ├── events/
│   │   ├── order_events.py
│   │   └── position_events.py
│   └── services/
│       ├── grid_calculator.py
│       └── risk_manager.py
├── application/
│   ├── use_cases/
│   │   ├── place_grid_orders.py
│   │   └── adjust_grid.py
│   └── event_handlers/
│       └── order_filled_handler.py
├── infrastructure/
│   ├── exchange/
│   │   ├── binance_adapter.py
│   │   └── exchange_interface.py
│   └── persistence/
│       └── order_repository.py
└── ml/
    ├── models/
    │   ├── price_predictor.py
    │   └── grid_optimizer.py
    └── data/
        └── data_loader.py
```
