# Spring Boot Architecture Lab

🇫🇷 Français | 🇬🇧 English

## 🇫🇷 Présentation

**Spring Boot Architecture Lab** est un projet pratique destiné à explorer et appliquer progressivement les principes d'architecture logicielle, de conception métier, de qualité du code et de tests automatisés avec Java et Spring Boot.

Le projet s'appuie sur une application de gestion de commandes qui évoluera au fil des apprentissages. Chaque évolution répondra à un besoin identifié et sera accompagnée d'explications, de tests et, lorsque cela est pertinent, d'une décision d'architecture documentée.

### Objectifs pédagogiques

- Comprendre et appliquer les principes de conception logicielle.
- Étudier la séparation des responsabilités, la cohésion et le couplage.
- Explorer l'inversion des dépendances et les frontières architecturales.
- Comparer différentes approches architecturales sans les appliquer mécaniquement.
- Développer une stratégie de tests adaptée aux règles métier et aux contraintes techniques.
- Étudier la gestion des transactions, de la concurrence et de la cohérence des données.
- Documenter les décisions et les compromis techniques.

### Application métier

L'application permettra de gérer un catalogue de produits et des commandes, de contrôler le stock, de conserver l'historique des prix et d'annuler les commandes selon des règles définies.

Consultez la [spécification fonctionnelle V1](docs/specification-fonctionnelle-v1.md) pour connaître le périmètre et les critères d'acceptation.

### Approche architecturale

Le projet commence comme un monolithe Spring Boot simple, avec PostgreSQL et des tests automatisés.

L'architecture évoluera progressivement à partir de besoins concrets. L'objectif n'est pas de multiplier les couches ou les abstractions, mais de comprendre quand une décision améliore réellement la maintenabilité, la testabilité ou la robustesse du système.

### Technologies prévues

- Java
- Spring Boot
- API REST
- PostgreSQL
- Docker Compose
- JUnit 5 et tests d'intégration

Les versions et les dépendances seront précisées lors de l'initialisation technique du projet.

### Documentation

- [Spécification fonctionnelle V1](docs/specification-fonctionnelle-v1.md)

Les principes généraux et les recommandations seront également documentés dans le dépôt [engineering-handbook](https://github.com/HocImou/engineering-handbook).

---

## 🇬🇧 Overview

**Spring Boot Architecture Lab** is a hands-on project designed to progressively explore and apply software architecture, domain design, code quality, and automated testing principles with Java and Spring Boot.

The project is based on an order management application that will evolve as new concepts are introduced. Each evolution will address an identified need and be supported by explanations, tests, and architectural decisions where appropriate.

### Learning objectives

- Understand and apply software design principles.
- Explore separation of concerns, cohesion, and coupling.
- Study dependency inversion and architectural boundaries.
- Compare architectural approaches without applying patterns mechanically.
- Develop a testing strategy suited to business rules and technical constraints.
- Explore transaction management, concurrency, and data consistency.
- Document technical decisions and trade-offs.

### Business application

The application will manage a product catalog and orders, control stock, preserve historical prices, and support order cancellation according to defined business rules.

See the [V1 functional specification](docs/specification-fonctionnelle-v1.md) for the scope and acceptance criteria.

### Architectural approach

The project starts as a simple Spring Boot monolith using PostgreSQL and automated tests.

Its architecture will evolve in response to concrete needs. The goal is not to introduce unnecessary layers or abstractions, but to understand when a design decision genuinely improves maintainability, testability, or system robustness.

### Planned technologies

- Java
- Spring Boot
- REST API
- PostgreSQL
- Docker Compose
- JUnit 5 and integration testing

Versions and dependencies will be specified when the technical project is initialized.

### Documentation

- [V1 functional specification](docs/specification-fonctionnelle-v1.md)

General principles and engineering recommendations will also be documented in the [engineering-handbook](https://github.com/HocImou/engineering-handbook) repository.
