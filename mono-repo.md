# Principes d'Architecture

Règles et guidelines pour une architecture logicielle saine.

## Principes SOLID

### Single Responsibility Principle (SRP)

Une classe ne devrait avoir qu'une seule raison de changer. Chaque classe est responsable d'une tâche unique.

```pseudo
// SRP : une seule responsabilité par classe

// MAUVAIS : cette classe fait trop de choses
CLASSE CommandeService:
    créer_commande()          // Logique métier
    envoyer_email()           // Notification
    générer_pdf_facture()     // Génération de document
    sauvegarder_en_base()     // Persistance
    // 4 raisons de changer = 4 responsabilités = violation SRP

// BON : chaque classe a une seule raison de changer
CLASSE CommandeService:      créer_commande()
CLASSE NotificationService:  envoyer_email()
CLASSE FactureService:       générer_pdf_facture()
CLASSE CommandeRepository:   sauvegarder_en_base()
```

### Open/Closed Principle (OCP)

Une classe doit être ouverte à l'extension mais fermée à la modification. Ajout de fonctionnalités sans modifier le code existant.

```pseudo
// OCP : ouvert à l'extension, fermé à la modification

// MAUVAIS : on modifie la classe pour chaque nouveau type de paiement
CLASSE PaiementService:
    traiter(type):
        SI type == "carte": ...
        SI type == "virement": ...
        SI type == "crypto": ...     // Chaque ajout modifie la classe

// BON : on étend via une interface
INTERFACE MoyenPaiement:
    débiter(montant)

CLASSE PaiementCarte    IMPLÉMENTE MoyenPaiement: débiter(montant) → ...
CLASSE PaiementVirement IMPLÉMENTE MoyenPaiement: débiter(montant) → ...
CLASSE PaiementCrypto   IMPLÉMENTE MoyenPaiement: débiter(montant) → ...
// Ajouter un nouveau moyen = ajouter une classe, sans toucher au code existant
```

### Liskov Substitution Principle (LSP)

Une classe dérivée doit être substituable à sa classe de base sans altérer la validité du programme.

```pseudo
// LSP : une sous-classe doit pouvoir remplacer sa classe parent

// MAUVAIS : le Pingouin casse le contrat de Oiseau
CLASSE Oiseau:            voler() → ...
CLASSE Pingouin HÉRITE Oiseau: voler() → ERREUR "Je ne sais pas voler"
// Le code qui attend un Oiseau plante si on lui donne un Pingouin

// BON : on sépare les capacités
CLASSE Oiseau:              se_déplacer()
CLASSE OiseauVolant HÉRITE Oiseau: voler()
CLASSE Pingouin HÉRITE Oiseau:     nager()
// Un Pingouin est un Oiseau valide, il se déplace (en nageant)
```

### Interface Segregation Principle (ISP)

Une interface doit être spécifique à un seul client. Ne contenir que les méthodes nécessaires pour un client spécifique.

```pseudo
// ISP : des interfaces petites et spécifiques

// MAUVAIS : une interface énorme que personne n'utilise entièrement
INTERFACE Travailleur:
    coder()
    tester()
    designer()
    gérer_projet()
    // Un développeur n'a pas besoin de designer()

// BON : des interfaces ciblées
INTERFACE Développeur: coder()
INTERFACE Testeur:     tester()
INTERFACE Designer:    designer()
// Chaque rôle implémente SEULEMENT ce dont il a besoin
```

### Dependency Inversion Principle (DIP)

Les modules de haut niveau ne doivent pas dépendre des modules de bas niveau. Les deux devraient dépendre d'abstractions.

```pseudo
// DIP : dépendre des abstractions, pas des implémentations

// MAUVAIS : le service métier dépend directement de la base de données
CLASSE CommandeService:
    base = new PostgresDatabase()     // Couplage fort à Postgres
    // Impossible de tester sans une vraie base de données

// BON : le service dépend d'une abstraction
CLASSE CommandeService:
    DÉPEND DE repository: DépôtCommande   // Interface abstraite

// On peut injecter n'importe quelle implémentation :
CommandeService(repository = PostgresDépôt())   // En production
CommandeService(repository = MémoireDépôt())    // En test
```

## Loi de Demeter

Principe de "moindre connaissance" : un objet devrait avoir une connaissance limitée des autres objets avec lesquels il interagit.

```pseudo
// Loi de Demeter : "Ne parlez qu'à vos amis proches"

// MAUVAIS : chaîne d'appels = couplage profond
montant = commande.client().adresse().ville().taxe()
// Si la structure de Client ou Adresse change, ce code casse

// BON : demander directement à l'objet voisin
montant = commande.calculer_taxe_livraison()
// La Commande sait comment obtenir la taxe en interne
// Le code appelant ne connaît pas les détails internes
```

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

```pseudo
// KISS, DRY, YAGNI en pratique

// KISS - Garder les choses simples
// MAUVAIS : sur-ingénierie pour un simple calcul
CLASSE StratégieCalculTotalAvecFactoryEtBuilder: ...
// BON :
total = somme(ligne.prix * ligne.quantité POUR CHAQUE ligne DANS commande)

// DRY - Ne pas se répéter
// MAUVAIS : même validation copiée dans 5 fichiers
DANS fichier_1: SI email NE CONTIENT PAS "@" ALORS erreur
DANS fichier_2: SI email NE CONTIENT PAS "@" ALORS erreur  // Dupliqué !
// BON : une seule source de vérité
VALEUR Email(adresse): INVARIANT adresse CONTIENT "@"

// YAGNI - Ne pas anticiper des besoins inexistants
// MAUVAIS : "On aura peut-être besoin d'un export XML un jour"
CLASSE ExportXML: ...    // Jamais utilisé
// BON : ne coder que ce qui est demandé MAINTENANT
```

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

```pseudo
// Bounded Contexts dans un mono-repo
// Chaque contexte est autonome avec son propre langage ubiquitaire

CONTEXTE Trading:
    "Ordre" signifie instruction d'achat/vente sur un marché
    utilise : Price, Quantity, TradingPair

CONTEXTE Risk:
    "Ordre" signifie directive de conformité réglementaire
    utilise : RiskScore, Exposure, Limit

// Même mot "Ordre", sens DIFFÉRENT → contextes SÉPARÉS
// Le shared-kernel contient uniquement les concepts VRAIMENT partagés

SHARED_KERNEL:
    Money(montant, devise)     // Utilisé par Trading ET Risk ET Compliance
    UserId(identifiant)        // Identité commune entre contextes
```
