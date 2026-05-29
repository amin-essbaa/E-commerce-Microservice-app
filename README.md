# 🛒 E-commerce Microservices App

> Plateforme e-commerce construite selon une **architecture microservices** avec Spring Boot & Spring Cloud.
> Chaque domaine métier (clients, produits, commandes) est isolé dans son propre service, découvert dynamiquement et exposé derrière une passerelle unique.

<p align="center">
  <img src="https://img.shields.io/badge/Java-21-orange?logo=openjdk&logoColor=white" alt="Java 21"/>
  <img src="https://img.shields.io/badge/Spring%20Boot-3.4.3-6DB33F?logo=springboot&logoColor=white" alt="Spring Boot"/>
  <img src="https://img.shields.io/badge/Spring%20Cloud-2024.0.0-6DB33F?logo=spring&logoColor=white" alt="Spring Cloud"/>
  <img src="https://img.shields.io/badge/MySQL-8.0-4479A1?logo=mysql&logoColor=white" alt="MySQL"/>
  <img src="https://img.shields.io/badge/Docker-ready-2496ED?logo=docker&logoColor=white" alt="Docker"/>
  <img src="https://img.shields.io/badge/Maven-build-C71A36?logo=apachemaven&logoColor=white" alt="Maven"/>
</p>

---

## 📑 Sommaire

- [Aperçu](#-aperçu)
- [Architecture](#-architecture)
- [Les microservices](#-les-microservices)
- [Stack technique](#-stack-technique)
- [Prérequis](#-prérequis)
- [Installation & démarrage](#-installation--démarrage)
- [Endpoints de l'API](#-endpoints-de-lapi)
- [Documentation Swagger](#-documentation-swagger)
- [Structure du projet](#-structure-du-projet)
- [Auteur](#-auteur)

---

## 🎯 Aperçu

Cette application décompose un système e-commerce classique en services indépendants qui communiquent entre eux via des appels REST (**OpenFeign**) et sont protégés par un **circuit breaker** (Resilience4j).

L'ensemble repose sur les piliers Spring Cloud :

- **Service Discovery** — chaque service s'enregistre auprès d'un annuaire central (Eureka).
- **API Gateway** — un point d'entrée unique route les requêtes vers le bon service.
- **Config Server** — la configuration est centralisée et externalisée.
- **Communication inter-services** — les clients HTTP déclaratifs (Feign) relient les services entre eux.

---

## 🏗 Architecture

```
                                ┌────────────────────────┐
                                │     Client / Front      │
                                └───────────┬────────────┘
                                            │
                                            ▼
                                ┌────────────────────────┐
                                │   Gateway Service       │   :8888
                                │ (Spring Cloud Gateway)  │
                                └───────────┬────────────┘
                                            │  routage dynamique (lb://)
              ┌─────────────────────────────┼─────────────────────────────┐
              ▼                              ▼                             ▼
   ┌────────────────────┐       ┌────────────────────┐        ┌────────────────────┐
   │  Customer Service  │       │  Command Service   │◄──────►│ Inventory Service  │
   │       :8077        │       │       :8081        │  Feign │     (produits)     │
   └─────────┬──────────┘       └─────────┬──────────┘        └─────────┬──────────┘
             │                            │                             │
             └────────────┬───────────────┴──────────────┬─────────────┘
                          ▼                               ▼
                ┌────────────────────┐         ┌────────────────────┐
                │  Discovery (Eureka)│         │   Config Service    │
                │       :8761        │         │       :801          │
                └────────────────────┘         └────────────────────┘
                          │                               │
                          └───────────► MySQL  ◄──────────┘
                                       (app_db)
```

> 🔄 Les services se découvrent mutuellement via **Eureka** et s'appellent par leur nom logique (`lb://customer-service`, `lb://inventory-service`, …) plutôt que par leur adresse IP.

---

## 🧩 Les microservices

| Service              | Port   | Rôle                                                                            |
|----------------------|--------|---------------------------------------------------------------------------------|
| **discovery**        | `8761` | Serveur **Eureka** — annuaire de tous les services.                             |
| **config-service**   | `801`  | **Config Server** — configuration centralisée (depuis un dépôt Git).            |
| **gateway-service**  | `8888` | **API Gateway** — point d'entrée unique, routage dynamique vers les services.   |
| **customer-service** | `8077` | Gestion des **clients** (CRUD), récupération de leurs commandes via Feign.      |
| **command-service**  | `8081` | Gestion des **commandes** + lignes de produits, avec circuit breaker.           |
| **inventory-service**| `8080` | Catalogue **produits** (consultation du stock).                                 |

---

## 🛠 Stack technique

| Catégorie               | Technologies                                                             |
|-------------------------|--------------------------------------------------------------------------|
| **Langage**             | Java 21                                                                  |
| **Framework**           | Spring Boot 3.4.3                                                         |
| **Microservices**       | Spring Cloud 2024.0.0 (Eureka, Gateway, Config, OpenFeign)               |
| **Résilience**          | Resilience4j (Circuit Breaker)                                           |
| **Persistance**         | Spring Data JPA / Hibernate, MySQL                                       |
| **Documentation API**   | SpringDoc OpenAPI (Swagger UI)                                           |
| **Boilerplate**         | Lombok                                                                    |
| **Observabilité**       | Spring Boot Actuator                                                      |
| **Build & Conteneurs**  | Maven, Docker                                                            |

---

## ✅ Prérequis

Avant de démarrer, assurez-vous d'avoir installé :

- ☕ **JDK 21**
- 📦 **Maven 3.9+** (ou utilisez le wrapper `./mvnw` fourni)
- 🐬 **MySQL 8** avec une base nommée `app_db`
- 🐳 **Docker** *(optionnel, pour le déploiement conteneurisé)*

> ⚙️ Par défaut, les services se connectent à `jdbc:mysql://localhost:3306/app_db` avec l'utilisateur `root` et un mot de passe vide. Adaptez ces valeurs dans les fichiers `application.properties` si nécessaire.

---

## 🚀 Installation & démarrage

### 1. Cloner le projet

```bash
git clone <url-du-depot>
cd E-commerce-Microservice-app
```

### 2. Préparer la base de données

```sql
CREATE DATABASE app_db;
```

> Les tables sont générées automatiquement par Hibernate (`spring.jpa.hibernate.ddl-auto=update`).

### 3. Lancer les services dans l'ordre

L'ordre de démarrage est important : l'annuaire et la configuration d'abord, les services métier ensuite.

```bash
# 1) Service Discovery (Eureka)
cd discovery && ./mvnw spring-boot:run

# 2) Config Service
cd config-service && ./mvnw spring-boot:run

# 3) Gateway
cd gateway-service && ./mvnw spring-boot:run

# 4) Services métier
cd customer-service  && ./mvnw spring-boot:run
cd command-service   && ./mvnw spring-boot:run
cd inventory-service && ./mvnw spring-boot:run
```

> Sous Windows, utilisez `mvnw.cmd` à la place de `./mvnw`.

### 4. Vérifier

- **Eureka Dashboard** 👉 http://localhost:8761 — tous les services doivent apparaître en `UP`.
- **Gateway** 👉 http://localhost:8888

### 🐳 Avec Docker (optionnel)

Chaque service possède son propre `Dockerfile`. Construisez d'abord le jar, puis l'image :

```bash
cd command-service
./mvnw clean package -DskipTests
docker build -t command-service .
docker run -p 8081:8081 command-service
```

---

## 🔌 Endpoints de l'API

> Les services sont accessibles directement ou via la **Gateway** (`http://localhost:8888/<nom-service>/...`).

### 👤 Customer Service

| Méthode  | Endpoint                          | Description                  |
|----------|-----------------------------------|------------------------------|
| `GET`    | `/getAllCustomers`                | Liste tous les clients       |
| `GET`    | `/getCustomerById?id={id}`        | Récupère un client par son ID|
| `POST`   | `/createCustomer`                 | Crée un nouveau client       |
| `PUT`    | `/updateCustomer?id={id}`         | Met à jour un client         |
| `DELETE` | `/deleteCustomer?id={id}`         | Supprime un client           |

### 🧾 Command Service

| Méthode  | Endpoint                                  | Description                          |
|----------|-------------------------------------------|--------------------------------------|
| `GET`    | `/getAllCommands`                         | Liste toutes les commandes           |
| `GET`    | `/getCommandById?id={id}`                 | Récupère une commande par son ID     |
| `GET`    | `/getCommandsByCustomerId?customerId={id}`| Commandes d'un client donné          |
| `POST`   | `/createCommand`                          | Crée une commande                    |
| `PUT`    | `/updateCommand?id={id}`                  | Met à jour une commande              |
| `DELETE` | `/deleteCommand?id={id}`                  | Supprime une commande                |

### 📦 Inventory Service

| Méthode  | Endpoint                | Description                  |
|----------|-------------------------|------------------------------|
| `GET`    | `/getProducts`          | Liste tous les produits      |
| `GET`    | `/getProduct?id={id}`   | Récupère un produit par ID   |

---

## 📖 Documentation Swagger

Chaque service métier expose sa documentation OpenAPI via **Swagger UI** :

| Service              | URL Swagger                                   |
|----------------------|-----------------------------------------------|
| Command Service      | http://localhost:8081/                        |
| Customer Service     | http://localhost:8077/swagger-ui.html         |
| Inventory Service    | http://localhost:8080/swagger-ui.html         |

---

## 📂 Structure du projet

```
E-commerce-Microservice-app/
├── discovery/            # Serveur Eureka (Service Discovery)
├── config-service/       # Serveur de configuration centralisée
├── gateway-service/      # API Gateway (point d'entrée unique)
├── customer-service/     # Microservice Clients
├── command-service/      # Microservice Commandes
└── inventory-service/    # Microservice Produits / Stock
```

Chaque microservice suit une organisation en couches cohérente :

```
src/main/java/com/example/<service>/
├── web/          # Contrôleurs REST
├── service/      # Logique métier
├── repository/   # Accès aux données (Spring Data JPA)
├── model/
│   ├── entity/   # Entités persistées
│   ├── dto/      # Objets de transfert
│   └── mapping/  # Mappers Entity ↔ DTO
└── proxy/        # Clients Feign (communication inter-services)
```

---

## 👤 Auteur

Projet développé dans un cadre académique (UPEC).

---

<p align="center">
  ⭐ N'hésitez pas à laisser une étoile si ce projet vous a été utile !
</p>
