# Micro Commerce - Architecture Microservices

Helo

Voici un petit projet d'e-commerce basé sur une architecture microservices avec Spring Boot. Rien de trop compliqué, juste de quoi comprendre comment ça marche quand on sépare tout en petits services.

## Ce qu'on a dans le projet

### Les 3 microservices

- **product-service** (port 8080) : Gère les produits, leurs infos, prix, etc.
- **order-service** (port 8081) : S'occupe des commandes et fait le lien avec les autres services
- **user-service** (port 8082) : Gère les utilisateurs et l'auth (basique pour l'instant)

### La stack technique

- **Spring Boot** : Pour faire du Java propre et rapide
- **MongoDB** : Base de données NoSQL pour chaque service
- **WebClient** : Pour que les services se parlent entre eux
- **Docker** : Pour containeriser tout ça
- **Maven** : Pour build et gérer les dépendances

## Comment lancer tout ça ?

### Prérequis

- Java 17+
- Maven
- Docker (pour MongoDB)

### Étape 1 : MongoDB

```bash
docker run -d -p 27017:27017 --name mongodb-microservice mongo:latest
```

### Étape 2 : Lancer les services

Dans 3 terminaux différents :

```bash
# Terminal 1 - Product Service
cd product-service
mvn spring-boot:run

# Terminal 2 - User Service  
cd user-service
mvn spring-boot:run

# Terminal 3 - Order Service
cd order-service
mvn spring-boot:run
```

### Ou avec Docker Compose (plus simple)

```bash
docker-compose up
```

## Comment tester ?

### Récupérer les produits
```bash
curl http://localhost:8080/api/products
```

### Récupérer les utilisateurs
```bash
curl http://localhost:8082/api/users
```

### Créer une commande
```bash
curl -X POST http://localhost:8081/api/orders \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "ton-user-id",
    "items": [{
      "productId": "ton-product-id",
      "quantity": 2,
      "price": 999.99
    }]
  }'
```

## Architecture

Chaque service a sa propre base MongoDB et communique avec les autres via des appels HTTP. C'est du classique, mais ça marche bien !

### Communication inter-services

- Le **order-service** appelle le **product-service** pour récupérer les infos produits
- Il appelle aussi le **user-service** pour vérifier que l'utilisateur existe
- On utilise des DTOs optimisés pour éviter de transférer trop de données

### Endpoints internes

Chaque service expose des endpoints `/internal/` pour la communication entre services, avec des réponses allégées.

## Structure des dépôts Git

Chaque microservice a son propre dépôt Git :
- `order-service/.git`
- `product-service/.git` 
- `user-service/.git`

Plus un dépôt principal pour la config globale (docker-compose, etc.)

## Problèmes courants

**Les services ne se trouvent pas ?**
→ Vérifie que MongoDB tourne et que les ports sont libres

**Erreur 404 sur les endpoints ?**
→ Assure-toi que tous les services sont bien démarrés

---