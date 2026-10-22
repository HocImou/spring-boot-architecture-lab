# Spécification fonctionnelle — Spring Boot Architecture Lab

**Version :** 1.0  
**Statut :** Spécification initiale

## 1. Objectif

Développer une API REST de gestion de commandes afin d'étudier progressivement les principes d'architecture logicielle, de conception métier, de qualité et de test avec Java et Spring Boot.

## 2. Périmètre fonctionnel

### Produits
- Créer, consulter, modifier et désactiver un produit.
- Garantir l'unicité de sa référence.
- Gérer son nom, son prix et son stock.
- Interdire la commande d'un produit désactivé.

### Commandes
- Créer une commande associée à un identifiant client.
- Inclure une ou plusieurs lignes avec des quantités strictement positives.
- Vérifier l'existence, l'état et le stock des produits.
- Conserver le nom et le prix unitaire appliqués à chaque ligne.
- Calculer le total à partir des lignes enregistrées.
- Consulter une commande et son historique.

### Annulation
- Autoriser l'annulation uniquement au statut `PENDING`.
- Passer la commande au statut `CANCELLED`.
- Restituer les quantités de stock une seule fois.
- Conserver la commande et ses lignes après annulation.

## 3. Règles métier

1. Une référence produit est unique.
2. Le prix est strictement positif et exprimé en euros avec deux décimales.
3. Le stock est un entier positif ou nul.
4. Une commande possède un identifiant client et au moins une ligne.
5. Chaque quantité commandée est strictement positive.
6. Le stock ne peut jamais devenir négatif.
7. Le stock est décrémenté lors de la création de commande.
8. Le prix et le nom historiques d'un produit ne changent pas lorsqu'il est modifié dans le catalogue.
9. Le statut initial d'une commande est `PENDING`.
10. Une commande annulée ne peut pas être annulée une seconde fois.
11. La création d'une commande et la modification du stock sont atomiques.
12. L'annulation et la restitution du stock sont atomiques.

## 4. Modèle métier initial

- **Product** : identifiant, référence, nom, prix, stock, état actif.
- **Order** : identifiant, identifiant client, statut, date de création, total.
- **OrderLine** : produit référencé, nom historique, prix unitaire historique, quantité, sous-total.

Les montants seront représentés avec `BigDecimal` en Java et `NUMERIC(12,2)` dans PostgreSQL. Les montants ayant plus de deux décimales seront refusés à l'entrée.

## 5. Critères d'acceptation

### Catalogue
- Une création valide enregistre le produit.
- Une référence déjà utilisée est refusée.
- Un prix négatif ou nul est refusé.
- Un stock négatif est refusé.
- Une modification du prix n'altère pas les anciennes commandes.
- Un produit désactivé ne peut plus être commandé.

### Commandes
- Une commande valide est créée en `PENDING` et décrémente le stock.
- Une commande multi-produits enregistre toutes ses lignes et un total exact.
- Une quantité supérieure au stock entraîne un rejet sans modification partielle.
- Un produit inexistant ou désactivé entraîne un rejet sans modification partielle.
- Une quantité nulle ou négative et une commande sans ligne sont refusées.
- Des commandes concurrentes ne peuvent pas faire passer le stock sous zéro.
- Une commande existante est consultable ; une commande inconnue renvoie une erreur adaptée.

### Annulation
- Une commande en attente peut être annulée et son stock restitué.
- Une seconde annulation ne restitue pas de stock supplémentaire.
- Toutes les lignes sont restituées de façon cohérente.
- Une erreur ne laisse pas le statut et le stock dans des états contradictoires.
- Une commande annulée reste consultable.

## 6. Hors périmètre

- Paiement et remboursement.
- Expédition et suivi de livraison.
- Notifications.
- Gestion complète des clients.
- Authentification et autorisations avancées.
- Interface web Angular.
- Microservices et messagerie asynchrone.

## 7. Exigences techniques

- Java et Spring Boot.
- API REST.
- PostgreSQL.
- Docker Compose pour l'environnement local.
- Tests automatisés avec JUnit 5 et les outils Spring Boot adaptés.
- Tests d'intégration sur PostgreSQL.
- Protection des mises à jour de stock contre les accès concurrents.
- Gestion cohérente des transactions.
- Documentation des décisions d'architecture dans des ADR.

## 8. Principes d'évolution

Le projet commence comme un monolithe simple. Chaque évolution architecturale doit répondre à un besoin identifié, être accompagnée de tests et expliciter ses avantages, ses coûts et ses compromis.

Le code sert d'illustration pratique ; le dépôt `engineering-handbook` conserve les principes généraux, les recommandations et les critères de revue associés.
