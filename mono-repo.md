# Principes d'Architecture

Règles et guidelines pour une architecture logicielle saine.

## Principes SOLID

### Single Responsibility Principle (SRP)

Une classe ne devrait avoir qu'une seule raison de changer. Chaque classe est responsable d'une tâche unique.

### Open/Closed Principle (OCP)

Une classe doit être ouverte à l'extension mais fermée à la modification. Ajout de fonctionnalités sans modifier le code existant.

### Liskov Substitution Principle (LSP)

Une classe dérivée doit être substituable à sa classe de base sans altérer la validité du programme.

### Interface Segregation Principle (ISP)

Une interface doit être spécifique à un seul client. Ne contenir que les méthodes nécessaires pour un client spécifique.

### Dependency Inversion Principle (DIP)

Les modules de haut niveau ne doivent pas dépendre des modules de bas niveau. Les deux devraient dépendre d'abstractions.

## Loi de Demeter

Principe de "moindre connaissance" : un objet devrait avoir une connaissance limitée des autres objets avec lesquels il interagit.

### Application

- Interagir uniquement avec les voisins immédiats
- Éviter les chaînes d'appels (`a.b().c().d()`)
- Réduire le couplage entre composants

### Variantes

| Variante | Description |
|----------|-------------|
| Forte | Appels limités aux méthodes des voisins immédiats uniquement |
| Faible | Inclut les objets retournés par les voisins |
| Relâchée | Autorise si l'objet a été passé en paramètre |

## Autres Guidelines

### KISS (Keep It Simple, Stupid)

Garder les choses simples, minimiser la complexité. Facilite maintenance et évolutivité.

### DRY (Don't Repeat Yourself)

Ne pas répéter le même code. Centraliser la logique pour réduire la duplication.

### YAGNI (You Ain't Gonna Need It)

Ne pas ajouter de fonctionnalités tant que non strictement nécessaire. Réduit complexité et coût de maintenance.

## Organisation Mono-repo

### Structure type

```
mono-repo/
├── packages/
│   ├── domain/           # Objets métier
│   ├── infrastructure/   # Persistance, messaging
│   ├── application/      # Use cases
│   └── api/              # Interfaces externes
├── shared/
│   ├── utils/
│   └── types/
└── tools/
    ├── build/
    └── deploy/
```

### Avantages

- **Cohérence** : Versions synchronisées
- **Refactoring** : Changements transversaux facilités
- **Partage** : Code commun centralisé
- **CI/CD** : Pipeline unifié

### Bounded Contexts

Chaque package représente un Bounded Context DDD :

```
├── trading/          # Contexte Trading
├── risk/             # Contexte Risk Management
├── compliance/       # Contexte Compliance
└── shared-kernel/    # Concepts partagés
```
