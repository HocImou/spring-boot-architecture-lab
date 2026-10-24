# Guide de démarrage technique

## 1. Objectif du document

Ce document décrit les étapes nécessaires pour préparer et démarrer l'environnement de développement local du projet `spring-boot-architecture-lab`.

Il présente les prérequis techniques, la configuration de PostgreSQL avec Docker Compose, la configuration de Spring Boot et les commandes permettant de vérifier le bon fonctionnement de l'application.

Ce guide est destiné aux développeurs souhaitant installer le projet et comprendre les principaux choix techniques de l'environnement local.

## 2. Prérequis

Les outils suivants sont nécessaires pour travailler sur le projet :

| Outil | Utilisation |
|---|---|
| JDK 25 | Compiler et exécuter l'application Java. |
| Maven Wrapper | Compiler le projet, exécuter les tests et vérifier le build sans installer Maven séparément. |
| Docker Desktop | Exécuter PostgreSQL dans un conteneur. |
| Docker Compose | Définir et démarrer les services nécessaires à l'environnement local. |
| Git | Versionner le code source et la documentation. |
| IntelliJ IDEA (recommandé) | Environnement de développement intégré. |

Le projet utilise le Maven Wrapper fourni dans le dépôt. Il est donc recommandé d'utiliser les scripts `mvnw` ou `mvnw.cmd` plutôt que de dépendre d'une installation locale de Maven.

## 3. Présentation de l'environnement technique

L'environnement local repose sur les composants suivants :

- **Java et Spring Boot** : développement et exécution de l'application backend.
- **Spring Data JPA et Hibernate** : accès aux données relationnelles.
- **PostgreSQL** : stockage des données métier.
- **Docker Compose** : démarrage et gestion du conteneur PostgreSQL.
- **Flyway** : gestion versionnée des migrations du schéma de base de données.
- **Maven** : gestion des dépendances, compilation, tests et vérification du projet.

PostgreSQL s'exécute dans Docker, tandis que l'application Spring Boot peut être lancée directement depuis l'environnement de développement.

Cette organisation permet de séparer le cycle de vie de la base de données de celui de l'application, tout en conservant un environnement local reproductible.

## 4. Configuration de PostgreSQL avec Docker Compose

### 4.1. Fichier de configuration

Le fichier `compose.yaml`, situé à la racine du dépôt, décrit le service PostgreSQL utilisé par l'application en environnement local.

La configuration actuelle repose sur l'image officielle `postgres:18`.

Les principaux paramètres sont les suivants :

| Paramètre | Description |
|---|---|
| `image` | Définit l'image Docker utilisée pour exécuter PostgreSQL. |
| `POSTGRES_DB` | Définit le nom de la base de données initialisée au premier démarrage. |
| `POSTGRES_USER` | Définit l'utilisateur créé lors de l'initialisation. |
| `POSTGRES_PASSWORD` | Définit son mot de passe local. |
| `ports` | Publie le port PostgreSQL du conteneur sur le port `5432` de la machine hôte. |
| `volumes` | Permet de conserver les données dans un volume Docker nommé. |
| `healthcheck` | Vérifie périodiquement que PostgreSQL est prêt à accepter les connexions. |

### 4.2. Persistance des données

Le volume nommé `postgres_data` permet de conserver les données de PostgreSQL indépendamment du cycle de vie du conteneur.

Ainsi, la suppression puis la recréation du conteneur ne suppriment pas automatiquement les données conservées dans ce volume.

**Attention :** la commande `docker compose down -v` supprime également les volumes gérés par le projet. Elle doit donc être utilisée avec précaution, car elle peut entraîner la perte des données locales.

### 4.3. Vérification de l'état de PostgreSQL

Le `healthcheck` utilise la commande `pg_isready` pour vérifier que PostgreSQL accepte les connexions.

L'état `healthy` indique que cette vérification réussit. Il ne garantit pas, à lui seul, que toutes les opérations applicatives fonctionnent correctement.

### 4.4. Démarrage du service

Depuis la racine du projet, démarrer PostgreSQL en arrière-plan :

```powershell
docker compose up -d
```

Vérifier l'état du conteneur :

```powershell
docker compose ps
```

Consulter les journaux du service :

```powershell
docker compose logs postgres
```

Arrêter les services sans supprimer les données persistantes :

```powershell
docker compose down
```

### 4.5. Accès à PostgreSQL depuis l'application

L'application Spring Boot s'exécute directement sur la machine de développement. Elle accède donc à PostgreSQL via le port publié sur la machine hôte :

```text
jdbc:postgresql://localhost:5432/architecture_lab
```

Les paramètres de connexion de l'application doivent correspondre à ceux définis dans `compose.yaml`.

## 5. Configuration de Spring Boot

### 5.1. Configuration de la datasource

La connexion à PostgreSQL est configurée dans le fichier :

`src/main/resources/application.properties`

Configuration actuelle :

```properties
spring.application.name=spring-boot-architecture-lab

# PostgreSQL
spring.datasource.url=jdbc:postgresql://localhost:5432/architecture_lab
spring.datasource.username=app
spring.datasource.password=local_dev_password
spring.datasource.driver-class-name=org.postgresql.Driver

# JPA
spring.jpa.hibernate.ddl-auto=none
```

Les propriétés principales sont les suivantes :

| Propriété | Rôle |
|---|---|
| `spring.datasource.url` | Définit l'URL JDBC de la base de données. |
| `spring.datasource.username` | Définit l'utilisateur utilisé pour la connexion. |
| `spring.datasource.password` | Définit le mot de passe de connexion. |
| `spring.datasource.driver-class-name` | Indique explicitement la classe du pilote JDBC PostgreSQL. |
| `spring.jpa.hibernate.ddl-auto` | Empêche Hibernate de modifier automatiquement le schéma de la base. |

Spring Boot peut généralement détecter automatiquement le pilote JDBC à partir des dépendances et de l'URL JDBC. La propriété `spring.datasource.driver-class-name` est donc explicite, mais n'est pas toujours indispensable.

### 5.2. Dépendance PostgreSQL

Le pilote JDBC PostgreSQL est déclaré dans le fichier `pom.xml` :

```xml
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
```

Cette dépendance permet à l'application Java de communiquer avec PostgreSQL via JDBC.

Le pilote est disponible à l'exécution grâce à la portée Maven `runtime`. Il ne faut pas confondre cette dépendance avec le serveur PostgreSQL, qui est exécuté séparément dans Docker.

### 5.3. Gestion du schéma avec Flyway

Le projet utilise Flyway pour versionner les évolutions du schéma de la base de données.

Le principe retenu est le suivant :

- les migrations SQL sont conservées dans le dépôt Git ;
- chaque migration représente une évolution identifiée du schéma ;
- Flyway suit les migrations exécutées dans la table `flyway_schema_history` ;
- Hibernate est configuré avec `spring.jpa.hibernate.ddl-auto=none` afin de ne pas créer ou modifier automatiquement les tables.

Cette séparation permet de rendre les évolutions du schéma explicites, contrôlables et reproductibles.

Le fonctionnement effectif des migrations sera vérifié lors de la création de la première migration SQL.

### 5.4. Gestion des identifiants

Le mot de passe présent dans la configuration actuelle est réservé à l'environnement local d'apprentissage.

Pour un environnement partagé ou de production, les identifiants ne doivent pas être stockés en clair dans les fichiers versionnés. Ils devront être externalisés, par exemple au moyen de variables d'environnement ou d'un gestionnaire de secrets adapté à l'environnement de déploiement.

## 6. Compilation et vérification du projet

### 6.1. Utilisation du Maven Wrapper

Le projet fournit le Maven Wrapper afin de lancer les commandes Maven sans nécessiter une installation locale distincte de Maven.

Sous Windows, les commandes suivantes sont exécutées depuis la racine du dépôt.

### 6.2. Vérification complète

Pour nettoyer les anciens fichiers de compilation, compiler le projet, exécuter les tests et vérifier le build :

```powershell
.\mvnw.cmd clean verify
```

La commande `verify` exécute les phases précédentes du cycle de vie Maven, notamment la compilation et les tests configurés pour le projet.

Un build réussi indique que les étapes configurées dans ce cycle de vie se sont terminées sans erreur. Il ne garantit pas, à lui seul, que tous les scénarios métier sont correctement couverts.

### 6.3. Exécution des tests

Les tests sont placés dans le répertoire :

`src/test/java`

Les tests générés initialement par Spring Boot comprennent notamment un test de chargement du contexte applicatif.

Ce test permet de vérifier que le contexte Spring démarre avec la configuration disponible, notamment la connexion à la base de données lorsqu'elle est requise.

Pour exécuter les tests sans lancer les autres phases de vérification :

```powershell
.\mvnw.cmd test
```

### 6.4. Vérifications avant l'exécution

Avant de lancer les tests, vérifier que PostgreSQL fonctionne :

```powershell
docker compose ps
```

Le conteneur PostgreSQL doit être en cours d'exécution et son contrôle de santé doit réussir.

Si nécessaire, démarrer le service :

```powershell
docker compose up -d
```

### 6.5. Résultats et diagnostic

En cas d'échec, examiner en priorité :

- les messages d'erreur de Maven ;
- l'état du conteneur PostgreSQL ;
- les journaux du service Docker Compose ;
- l'URL JDBC et les identifiants de connexion ;
- la résolution des dépendances Maven.

Les fichiers de compilation générés par Maven sont placés dans le répertoire `target/`, qui n'a pas vocation à être versionné dans Git.

## 7. Prochaines étapes

L'environnement de développement local constitue la base du projet. Les prochaines étapes prévues sont :

1. Créer et vérifier la première migration Flyway.
2. Définir le schéma relationnel des produits et des commandes.
3. Implémenter le modèle métier et les règles de gestion.
4. Développer les cas d'utilisation en respectant les responsabilités de l'architecture retenue.
5. Ajouter progressivement les tests unitaires, les tests d'intégration et les tests d'acceptation pertinents.
## 8. Gestion des fins de ligne avec Git

### 8.1. Problématique

Les systèmes d'exploitation n'utilisent pas tous la même convention pour représenter les fins de ligne dans les fichiers texte :

- `LF` (*Line Feed*, `\n`) : convention courante sous Linux et macOS.
- `CRLF` (*Carriage Return + Line Feed*, `\r\n`) : convention courante sous Windows.

Lorsque plusieurs développeurs travaillent sur le même dépôt avec des systèmes différents, ces conventions peuvent entraîner des avertissements Git ou des différences de formatage inutiles dans les fichiers.

### 8.2. Convention retenue pour le projet

Le fichier `.gitattributes`, situé à la racine du dépôt, définit explicitement les conventions de fins de ligne.

Le projet utilise principalement `LF` pour les fichiers Java, les fichiers de configuration et la documentation Markdown. Les scripts Windows avec l'extension `.cmd` conservent la convention `CRLF`.

Cette configuration permet de maintenir une convention cohérente dans le dépôt, quel que soit l'environnement de développement utilisé.

### 8.3. Comprendre les avertissements Git

Un message tel que :

```text
warning: in the working copy of 'docs/guide-demarrage-technique.md',
CRLF will be replaced by LF the next time Git touches it
```

indique que Git prévoit de normaliser les fins de ligne du fichier conformément aux règles définies dans `.gitattributes`.

Il s'agit d'un avertissement de normalisation, et non d'une erreur dans le contenu du fichier Markdown.

### 8.4. Vérifier les règles appliquées

Pour vérifier les attributs associés à un fichier :

```powershell
git check-attr text eol -- docs/guide-demarrage-technique.md
```

Pour réappliquer les règles de normalisation aux fichiers suivis :

```powershell
git add --renormalize .
```

Cette dernière commande peut modifier l'index Git pour plusieurs fichiers. Il convient donc de vérifier les changements avant de les commiter.

Pour contrôler les éventuels problèmes de formatage dans les modifications indexées :

```powershell
git diff --cached --check
```

Pour examiner les fichiers modifiés et leur état dans Git :

```powershell
git status
```

### 8.5. Bonnes pratiques

- Définir les conventions de fins de ligne dans `.gitattributes`.
- Éviter de modifier manuellement les fins de ligne sans raison.
- Vérifier les différences Git avant un commit, notamment après une renormalisation.
- Éviter d'inclure dans un commit des changements de formatage massifs sans rapport avec la fonctionnalité développée.

L'objectif est de limiter les différences inutiles dans les revues de code et de faciliter le travail collaboratif.