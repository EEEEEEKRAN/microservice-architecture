# Sécurité JWT - Comment ça marche ?

Alors maintenant qu'on a nos 3 microservices qui tournent, on va pas les laisser ouverts à tous les vents ! On a mis en place Spring Security avec JWT pour sécuriser tout ça proprement.

## Ce qu'on a fait

Chaque service a maintenant sa petite couche de sécurité :
- **user-service** : C'est lui qui génère les tokens JWT et gère l'auth
- **order-service** : Il vérifie que ton token est valide avant de te laisser commander
- **product-service** : Pareil, il check ton token pour les actions sensibles

## Config JWT

Dans chaque `application.yml`, on a ajouté ça :
```yaml
spring:
  security:
    jwt:
      secret: mySecretKey123456789012345678901234567890
      expiration: 86400000 # 24 heures, largement suffisant
```

## Comment s'authentifier ?

### S'inscrire
```bash
curl -X POST http://localhost:8082/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John Doe",
    "email": "john@example.com",
    "password": "password123"
  }'
```

### Se connecter
```bash
curl -X POST http://localhost:8082/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "john@example.com",
    "password": "password123"
  }'
```

**Tu récupères ça :**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "userId": "64a1b2c3d4e5f6789012345",
  "email": "john@example.com",
  "name": "John Doe",
  "role": "USER",
  "message": "Connexion réussie",
  "expiresIn": 86400000
}
```

### Vérifier un token
```bash
curl -X POST http://localhost:8082/api/auth/validate \
  -H "Content-Type: application/json" \
  -d '{"token": "ton-super-token"}'
```

### Rafraîchir un token
```bash
curl -X POST http://localhost:8082/api/auth/refresh \
  -H "Authorization: Bearer ton-token"
```

## Comment utiliser les APIs maintenant ?

### Le header magique
Pour tous les endpoints protégés, tu dois ajouter ça dans tes requêtes :
```http
Authorization: Bearer ton-super-token-jwt
```

### Exemples concrets

#### User Service
```bash
# Voir tous les users (admin seulement)
curl -H "Authorization: Bearer ton-token" \
  http://localhost:8082/api/users

# Voir un user spécifique
curl -H "Authorization: Bearer ton-token" \
  http://localhost:8082/api/users/64a1b2c3d4e5f6789012345
```

#### Product Service
```bash
# Voir les produits (public, pas besoin de token)
curl http://localhost:8081/api/products

# Créer un produit (admin seulement)
curl -X POST http://localhost:8081/api/products \
  -H "Authorization: Bearer ton-token" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Super produit",
    "description": "Un truc génial",
    "price": 29.99,
    "stock": 100
  }'
```

#### Order Service
```bash
# Voir tes commandes
curl -H "Authorization: Bearer ton-token" \
  http://localhost:8083/api/orders

# Passer une commande
curl -X POST http://localhost:8083/api/orders \
  -H "Authorization: Bearer ton-token" \
  -H "Content-Type: application/json" \
  -d '{
    "productId": "64a1b2c3d4e5f6789012345",
    "quantity": 2
  }'
```

## Qui peut faire quoi ?

### Endpoints ouverts à tous
- `POST /api/auth/register` - S'inscrire
- `POST /api/auth/login` - Se connecter
- `GET /api/products` - Voir les produits
- `GET /actuator/health` - Check si ça tourne

### Endpoints pour utilisateurs connectés
- `GET /api/users/{id}` - Voir son propre profil
- `PUT /api/users/{id}` - Modifier son profil
- `GET /api/orders` - Voir ses commandes
- `POST /api/orders` - Passer une commande
- `POST /api/auth/validate` - Vérifier son token
- `POST /api/auth/refresh` - Rafraîchir son token

### Endpoints pour les admins
- `GET /api/users` - Voir tous les users
- `DELETE /api/users/{id}` - Supprimer un user
- `POST /api/products` - Créer des produits
- `PUT /api/products/{id}` - Modifier des produits
- `DELETE /api/products/{id}` - Supprimer des produits
- `GET /api/orders/all` - Voir toutes les commandes

## Quand ça foire

### 401 - T'es pas connecté
```json
{
  "error": "Unauthorized",
  "message": "Token JWT requis pour accéder à cette ressource",
  "timestamp": "2024-01-15T10:30:00Z",
  "path": "/api/users"
}
```

### 403 - T'as pas les droits
```json
{
  "error": "Forbidden",
  "message": "Accès refusé - privilèges insuffisants",
  "timestamp": "2024-01-15T10:30:00Z",
  "path": "/api/users"
}
```

## CORS et tout le bazar

On a configuré CORS pour que ça marche avec tes apps front :
- `http://localhost:3000` (React)
- `http://localhost:4200` (Angular)
- `http://localhost:8080` (Vue.js)

## Sécurité - Ce qu'on a mis en place

### Les trucs bien
1. **Mots de passe hachés** : BCrypt avec du sel, comme il faut
2. **JWT sécurisés** : Signature HMAC-SHA256
3. **Expiration** : 24h par défaut, pas trop long
4. **CORS propre** : Que les origines autorisées
5. **Pas de CSRF** : On en a pas besoin avec du REST pur
6. **Sessions stateless** : Tout dans le JWT

### À faire en prod
1. Change la clé secrète JWT (sérieusement !)
2. Passe en HTTPS
3. Mets en place la rotation des tokens
4. Ajoute des logs de sécurité
5. Configure des timeouts corrects

## Tester avec Postman

1. **Inscris-toi** avec `/api/auth/register`
2. **Connecte-toi** avec `/api/auth/login`
3. **Copie le token** de la réponse
4. **Colle-le** dans l'onglet Authorization de Postman
5. **Teste** les endpoints selon ton rôle

## Lancer tout ça

```bash
# Tout en une fois
docker-compose up -d

# Ou service par service
cd user-service && mvn spring-boot:run
cd product-service && mvn spring-boot:run
cd order-service && mvn spring-boot:run
```

Tes services tournent sur :
- User Service : http://localhost:8082
- Product Service : http://localhost:8081
- Order Service : http://localhost:8083

---

ET VOILA !