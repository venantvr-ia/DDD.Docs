# Ubiquitous Language (Langage Ubiquitaire)

L'Ubiquitous Language est un concept fondamental du Domain-Driven Design qui établit un vocabulaire partagé et cohérent entre tous les membres d'une équipe de développement logiciel.

## Définition

L'Ubiquitous Language est un vocabulaire partagé et cohérent, créé et utilisé par les membres d'une équipe de développement logiciel, comprenant développeurs, clients et parties prenantes, pour faciliter la compréhension mutuelle et améliorer la communication dans le contexte d'un projet spécifique.

## Rôle dans le Domain-Driven Design

L'Ubiquitous Language est un concept clé du DDD, une approche de conception logicielle axée sur la modélisation des domaines métier complexes et la collaboration entre les experts métier et les développeurs pour créer des solutions logicielles adaptées aux besoins spécifiques du domaine.

## Élimination des ambiguïtés

En définissant et en utilisant un langage commun, les membres de l'équipe réduisent les malentendus et les ambiguïtés qui peuvent découler de termes ou de concepts mal définis, améliorant ainsi la qualité du logiciel et évitant des erreurs potentiellement coûteuses.

Par exemple, si l'équipe métier parle de "client" et que les développeurs utilisent le terme "utilisateur" pour désigner la même entité, cela peut créer de la confusion. L'Ubiquitous Language établit un terme unique que tout le monde utilise de manière cohérente.

```pseudo
// Le problème SANS langage ubiquitaire
Expert_Métier dit : "Le client passe une commande"
Développeur code : créer Utilisateur.soumettre_requête()
Testeur écrit   : "Vérifier que le user fait un order"
// Résultat : 3 vocabulaires différents = confusion garantie

// La solution AVEC langage ubiquitaire
Expert_Métier dit : "Le client passe une commande"
Développeur code : créer Client.passer_commande()
Testeur écrit   : "Vérifier que le Client passe une Commande"
// Résultat : 1 seul vocabulaire = compréhension partagée
```

## Documentation et code

L'Ubiquitous Language doit être intégré à la fois dans la documentation du projet et dans le code source lui-même, afin de garantir une cohérence dans l'ensemble du cycle de vie du développement logiciel et de faciliter la maintenance et l'évolution du logiciel.

### Dans le code

```python
# Mauvais exemple - termes techniques
class User:
    def __init__(self, user_id, account_balance):
        self.user_id = user_id
        self.account_balance = account_balance

# Bon exemple - langage ubiquitaire du domaine bancaire
class Client:
    def __init__(self, numero_client, solde_compte):
        self.numero_client = numero_client
        self.solde_compte = solde_compte

    def effectuer_virement(self, montant, beneficiaire):
        # Le code reflète le vocabulaire métier
        pass
```

### Dans la documentation

- Les spécifications fonctionnelles utilisent les mêmes termes que le code
- Les user stories sont rédigées avec le vocabulaire métier
- Les tests sont nommés selon le langage ubiquitaire

## Évolution continue

L'Ubiquitous Language n'est pas figé et doit évoluer avec le projet. Les membres de l'équipe doivent collaborer pour affiner et étendre le vocabulaire au fur et à mesure de l'avancement du projet, en tenant compte des retours d'expérience et des nouvelles exigences métier.

### Processus d'évolution

1. **Découverte** : Lors des sessions de modélisation, de nouveaux termes émergent
2. **Discussion** : L'équipe débat du terme le plus approprié
3. **Validation** : Les experts métier valident le choix
4. **Propagation** : Le terme est utilisé partout (code, docs, conversations)
5. **Révision** : Périodiquement, le glossaire est revu et mis à jour

## Mise en pratique

### Ateliers de modélisation (Event Storming)

Réunir experts métier et développeurs pour identifier :
- Les événements du domaine
- Les commandes qui déclenchent ces événements
- Les agrégats concernés
- Le vocabulaire associé

```pseudo
// Déroulement d'un atelier Event Storming
ATELIER EventStorming(experts, développeurs):

    // Phase 1 : Brainstorm des événements
    événements = CHAQUE participant ÉCRIT les événements métier qu'il connaît
    // Ex: "Commande passée", "Paiement reçu", "Colis expédié"

    // Phase 2 : Chronologie
    timeline = Ordonner(événements) PAR ordre_chronologique

    // Phase 3 : Identifier les déclencheurs
    POUR CHAQUE événement DANS timeline:
        commande = Quelle_Action_Déclenche(événement)
        acteur   = Qui_Déclenche(commande)
        // Ex: Client → Passer_Commande → "Commande passée"

    // Phase 4 : Regrouper en agrégats
    agrégats = Regrouper(événements) PAR cohérence_métier
    // Ex: {Commande: ["passée", "validée", "annulée"]}

    // Phase 5 : Extraire le vocabulaire
    glossaire = Extraire_Termes_Communs(événements, commandes, agrégats)
    RETOURNER glossaire  // C'est le Langage Ubiquitaire
```

### Glossaire partagé

Maintenir un document vivant contenant :
- Chaque terme du domaine
- Sa définition précise
- Des exemples d'utilisation
- Les termes à éviter (synonymes non retenus)

### Revues de code orientées domaine

Lors des revues de code, vérifier que :
- Les noms de classes correspondent au vocabulaire métier
- Les méthodes sont nommées selon les actions du domaine
- Les commentaires utilisent le langage ubiquitaire

```pseudo
// Checklist de revue de code orientée domaine
REVUE_CODE(pull_request):

    POUR CHAQUE classe DANS pull_request:
        VÉRIFIER classe.nom EXISTE DANS glossaire
        // NON : "OrderMgr" → OUI : "GestionnaireCommande"

    POUR CHAQUE méthode DANS pull_request:
        VÉRIFIER méthode.nom REFLÈTE action_métier
        // NON : "process()" → OUI : "valider_commande()"

    POUR CHAQUE variable DANS pull_request:
        VÉRIFIER variable.nom EST compréhensible_par_expert_métier
        // NON : "tmp_val" → OUI : "montant_total"
```

## Bénéfices

1. **Communication améliorée** : Moins de malentendus entre équipes
2. **Code plus lisible** : Le code raconte l'histoire du domaine
3. **Onboarding facilité** : Les nouveaux arrivent comprennent plus vite
4. **Maintenance simplifiée** : Le code reflète les règles métier
5. **Évolution maîtrisée** : Les changements métier se traduisent directement dans le code

## Exemple concret : Domaine du trading

| Terme technique | Terme ubiquitaire | Définition |
|-----------------|-------------------|------------|
| `order` | `Ordre` | Instruction d'achat ou de vente |
| `fill` | `Exécution` | Réalisation effective d'un ordre |
| `position` | `Position` | Exposition sur un actif |
| `pnl` | `Gain/Perte` | Résultat financier |
| `stop_loss` | `Stop de protection` | Ordre de sortie automatique |
