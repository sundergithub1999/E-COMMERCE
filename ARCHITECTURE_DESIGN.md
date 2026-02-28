# E-Commerce Microservices Architecture Design

## 📋 Table of Contents
1. [System Overview](#system-overview)
2. [Tech Stack](#tech-stack)
3. [Microservices Architecture](#microservices-architecture)
4. [Database ER Diagrams](#database-er-diagrams)
5. [API Flow & Examples](#api-flow--examples)
6. [Communication Patterns](#communication-patterns)
7. [Security & Authentication](#security--authentication)
8. [Deployment Architecture](#deployment-architecture)

---

## System Overview

### Project Description
This is a **distributed E-Commerce platform** built using **Spring Boot microservices** architecture. It handles product management, inventory tracking, order processing, and customer notifications across independent, scalable services.

### Key Features
✅ Product Catalog Management  
✅ Real-time Inventory Tracking  
✅ Order Processing with Validation  
✅ Event-Driven Notifications  
✅ API Gateway for Routing  
✅ Distributed Authentication  
✅ Circuit Breaker Pattern for Resilience  
✅ Message Queue for Async Communication  

---

## Tech Stack

### Backend Framework
| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| **Language** | Java | 17+ | Core development |
| **Framework** | Spring Boot | 3.x | Application framework |
| **Cloud** | Spring Cloud | Spring Cloud 2022.x | Microservices patterns |

### Databases
| Service | Database | Type | Purpose |
|---------|----------|------|---------|
| **Product Service** | MongoDB | NoSQL | Flexible product schema |
| **Order Service** | PostgreSQL | RDBMS | ACID compliance for orders |
| **Inventory Service** | PostgreSQL | RDBMS | Stock management |

### Message Queue & Event Streaming
| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Message Broker** | Apache Kafka | Async inter-service communication |
| **Zookeeper** | Coordination | Kafka cluster coordination |

### API Gateway & Service Discovery
| Component | Technology | Purpose |
|-----------|-----------|---------|
| **API Gateway** | Spring Cloud Gateway | Request routing & filtering |
| **Service Registry** | Netflix Eureka | Service discovery |

### Authentication & Authorization
| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Auth Server** | Keycloak v18 | OAuth2 / OpenID Connect |
| **Auth DB** | MySQL 5.7 | Keycloak data storage |
| **Security** | Spring Security | Application-level security |

### Resilience & Monitoring
| Component | Library | Purpose |
|-----------|---------|---------|
| **Circuit Breaker** | Resilience4j | Fault tolerance |
| **Retry Logic** | Resilience4j | Automatic retry on failure |
| **Timeout Management** | Resilience4j | Request timeout control |
| **Distributed Tracing** | Micrometer + Brave + Zipkin | Request tracing |
| **Metrics** | Micrometer | Performance monitoring |

### Containerization & Orchestration
| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Containerization** | Docker | Service containerization |
| **Orchestration** | Docker Compose | Local development & testing |

---

## Microservices Architecture

### 1. **API GATEWAY** (Port: 8080)
**Responsibilities:**
- Route incoming requests to appropriate microservices
- OAuth2 resource server authentication
- Request/Response filtering
- Security validation

**Dependencies:**
- Spring Cloud Gateway
- Spring Security
- Eureka Client
- Micrometer for tracing

**Key Features:**
```
Usage: Acts as single entry point for all client requests
Routes:
  /api/product/* → Product Service (8081)
  /api/order/*   → Order Service (8083)
  /api/inventory/* → Inventory Service (8082)
```

---

### 2. **PRODUCT SERVICE** (Port: 8081)
**Responsibilities:**
- Management of product catalog
- CRUD operations for products
- Product listing and search

**Database:** MongoDB (NoSQL)
- Flexible schema for product data
- Fast read operations for catalog

**Key Endpoints:**
```
POST   /api/product           → Create new product
GET    /api/product           → Get all products
GET    /api/product/{id}      → Get product by ID
PUT    /api/product/{id}      → Update product
DELETE /api/product/{id}      → Delete product
```

**Key Classes:**
- `ProductController` - REST endpoints
- `ProductService` - Business logic
- `Product` - JPA entity
- `ProductRepository` - Data access

---

### 3. **ORDER SERVICE** (Port: 8083)
**Responsibilities:**
- Order creation and management
- Inventory validation before order placement
- Event publishing for order notifications
- Resilience with circuit breaker pattern

**Database:** PostgreSQL (RDBMS)
- ACID properties for transactional safety
- Complex joins for order details

**Key Endpoints:**
```
POST /api/order              → Place new order
GET  /api/order/{id}         → Get order details
GET  /api/order              → List all orders
```

**Key Features:**
- **Circuit Breaker Pattern**: Prevents cascading failures
- **Retry Logic**: Automatic retries on transient failures
- **Time Limiter**: Prevents hanging requests
- **Event Publishing**: Publishes `OrderPlacedEvent` to Kafka

**Key Classes:**
- `OrderController` - REST endpoints with resilience annotations
- `OrderService` - Order placement & validation logic
- `OrderPlacedEventListener` - Publishes events to Kafka
- `Order` - JPA entity
- `OrderRepository` - Data access

---

### 4. **INVENTORY SERVICE** (Port: 8082)
**Responsibilities:**
- Track product stock levels
- Check inventory availability
- Update stock on order placement

**Database:** PostgreSQL (RDBMS)
- Relationship management for SKU codes
- Consistent inventory tracking

**Key Endpoints:**
```
GET /api/inventory?skuCode=code1&skuCode=code2   → Check stock availability
PUT /api/inventory/{id}                          → Update inventory
```

**Example Request:**
```bash
GET /api/inventory?skuCode=iphone-13&skuCode=iphone13-red
```

**Key Classes:**
- `InventoryController` - REST endpoints
- `InventoryService` - Stock management logic
- `Inventory` - JPA entity
- `InventoryRepository` - Data access

---

### 5. **NOTIFICATION SERVICE** (Port: 8084)
**Responsibilities:**
- Listen to `order-placed` events from Kafka
- Send email/SMS notifications to customers
- Async event processing

**Database:** None (Stateless service)

**Key Features:**
- Kafka consumer for `order-placed` topic
- Email notification sending
- Event-driven architecture

**Key Classes:**
- `OrderPlacedEventListener` - Kafka consumer
- `NotificationService` - Email sending logic
- `OrderPlacedEvent` - Event model

---

## Database ER Diagrams

### PostgreSQL - ORDER SERVICE
```
┌─────────────────────────────────────┐
│           ORDERS TABLE              │
├─────────────────────────────────────┤
│ ◆ id (PK, UUID)                    │
│ • order_number (UNIQUE, VARCHAR)   │
│ • order_line_items_id (FK)         │
│ • created_date (TIMESTAMP)         │
│ • last_modified_date (TIMESTAMP)   │
└─────────────────────────────────────┘
           │
           │ 1:M
           │
┌─────────────────────────────────────┐
│       ORDER_LINE_ITEMS TABLE        │
├─────────────────────────────────────┤
│ ◆ id (PK, UUID)                    │
│ • price (DECIMAL)                  │
│ • quantity (INTEGER)               │
│ • sku_code (VARCHAR)               │
│ • orders_id (FK)                   │
└─────────────────────────────────────┘
```

### PostgreSQL - INVENTORY SERVICE
```
┌─────────────────────────────────────┐
│        INVENTORY TABLE              │
├─────────────────────────────────────┤
│ ◆ id (PK, UUID)                    │
│ • quantity (INTEGER)               │
│ • sku_code (VARCHAR, UNIQUE)       │
├─────────────────────────────────────┤
│ Indexes: idx_sku_code              │
└─────────────────────────────────────┘

Example Data:
┌────────┬──────────────┬──────────┐
│  id    │  sku_code    │ quantity │
├────────┼──────────────┼──────────┤
│ UUID1  │ iphone-13    │   50     │
│ UUID2  │ iphone13-red │   30     │
│ UUID3  │ samsung-s21  │   25     │
└────────┴──────────────┴──────────┘
```

### MongoDB - PRODUCT SERVICE
```
{
  "_id": ObjectId,
  "name": "iPhone 13",
  "description": "Latest iPhone model",
  "price": 999.99,
  "category": "Electronics",
  "stock_quantity": 100,
  "sku_code": "iphone-13",
  "specifications": {
    "color": "Black",
    "storage": "128GB",
    "processor": "A15 Bionic"
  },
  "created_at": ISODate,
  "updated_at": ISODate
}
```

---

## API Flow & Examples

### Complete User Journey: From Browse to Order

#### **Step 1: User Registers & Authenticates**
```bash
# Login via Keycloak (OAuth2)
POST http://localhost:8080/auth/token
Content-Type: application/json

{
  "username": "customer@example.com",
  "password": "securePassword123",
  "client_id": "e-commerce-app",
  "grant_type": "password"
}

Response:
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

---

#### **Step 2: Browse Products**
```bash
# Get all products from Product Service
GET http://localhost:8080/api/product
Authorization: Bearer {access_token}

Response (Status: 200):
[
  {
    "id": "prod-001",
    "name": "iPhone 13",
    "price": 999.99,
    "category": "Electronics",
    "description": "Latest iPhone model",
    "specifications": {
      "color": "Black",
      "storage": "128GB"
    }
  },
  {
    "id": "prod-002",
    "name": "iPhone 13 Red",
    "price": 1099.99,
    "category": "Electronics",
    "description": "Premium red variant",
    "specifications": {
      "color": "Red",
      "storage": "256GB"
    }
  }
]
```

---

#### **Step 3: Check Product Availability**
```bash
# Check inventory before adding to cart
GET http://localhost:8080/api/inventory?skuCode=iphone-13&skuCode=iphone13-red
Authorization: Bearer {access_token}

Response (Status: 200):
[
  {
    "skuCode": "iphone-13",
    "isInStock": true,
    "quantity": 50
  },
  {
    "skuCode": "iphone13-red",
    "isInStock": true,
    "quantity": 30
  }
]
```

---

#### **Step 4: Place Order (With Resilience)**
```bash
# Create order with items
POST http://localhost:8080/api/order
Authorization: Bearer {access_token}
Content-Type: application/json

Request:
{
  "skuCode": [
    "iphone-13",
    "iphone13-red"
  ]
}

# Backend Flow with Resilience4j:
1. OrderController receives request
2. @CircuitBreaker decorator checks circuit state
3. @Retry attempts up to 3 times if inventory fails
4. @TimeLimiter enforces 3-second timeout
5. OrderService.placeOrder() is called

# If all checks pass:
Response (Status: 201):
{
  "message": "Order Placed Successfully!",
  "orderNumber": "ORD-20240228-001",
  "status": "PENDING",
  "items": [
    {
      "skuCode": "iphone-13",
      "quantity": 1,
      "price": 999.99
    },
    {
      "skuCode": "iphone13-red",
      "quantity": 1,
      "price": 1099.99
    }
  ],
  "totalPrice": 2099.98,
  "createdAt": "2024-02-28T10:30:00Z"
}

# If Circuit Breaker is OPEN (fallback):
Response (Status: 503):
{
  "message": "Oops! Something went wrong, please order after some time!"
}
```

---

#### **Step 5: Order Placed Event Publishing**
```
Order Service:
┌────────────────────────────────────────┐
│ Order Placed Successfully              │
├────────────────────────────────────────┤
│ 1. publishOrderPlacedEvent()           │
│ 2. OrderPlacedEventListener triggered  │
│ 3. Kafka Message Published:            │
└────────────────────────────────────────┘
    ↓
Kafka Topic: "order-placed"
Message Payload:
{
  "orderNumber": "ORD-20240228-001",
  "timestamp": "2024-02-28T10:30:00Z",
  "items": [{ SKU, quantity }],
  "totalPrice": 2099.98
}
    ↓
┌────────────────────────────────────────┐
│ Notification Service (Consumer)        │
├────────────────────────────────────────┤
│ Receives event and sends email:        │
│ "Your order ORD-20240228-001 placed!"  │
└────────────────────────────────────────┘
```

---

#### **Step 6: Get Order Details**
```bash
# Retrieve placed order
GET http://localhost:8080/api/order/order-uuid-123
Authorization: Bearer {access_token}

Response (Status: 200):
{
  "id": "order-uuid-123",
  "orderNumber": "ORD-20240228-001",
  "orderLineItems": [
    {
      "id": "item-uuid-1",
      "skuCode": "iphone-13",
      "price": 999.99,
      "quantity": 1
    },
    {
      "id": "item-uuid-2",
      "skuCode": "iphone13-red",
      "price": 1099.99,
      "quantity": 1
    }
  ],
  "status": "CONFIRMED",
  "createdDate": "2024-02-28T10:30:00Z",
  "lastModifiedDate": "2024-02-28T10:30:45Z"
}
```

---

## Communication Patterns

### 1. **Synchronous Communication (REST)**
Used for immediate request-response operations:
```
Client
  ↓
API Gateway
  ↓
Service A → (Direct HTTP Call) → Service B
  ↓
Response flows back
```

**Example:** Order Service → Inventory Service (Check Stock)

**Implementation:**
```java
@CircuitBreaker(name = "inventory")
@TimeLimiter(name = "inventory")
@Retry(name = "inventory")
public void checkInventory(List<String> skuCodes) {
    // Direct REST call to Inventory Service
    inventoryClient.isInStock(skuCodes);
}
```

---

### 2. **Asynchronous Communication (Event-Driven)**
Used for decoupled, non-blocking operations:
```
Order Service
  ↓
Publishes Event to Kafka Topic
  ↓
Notification Service (Consumer)
  ↓
Processes independently
```

**Example:** Order Placed → Send Notification Email

**Implementation:**
```java
@EventListener
public void handleOrderPlacedEvent(OrderPlacedEvent event) {
    kafkaTemplate.send("order-placed", event);
    // Event topic subscribers process asynchronously
}
```

---

## Security & Authentication

### OAuth2 Flow with Keycloak
```
┌──────────┐
│  Client  │
└────┬─────┘
     │ 1. Login Request
     ↓
┌─────────────────────────┐
│   API Gateway           │
│  (OAuth2 Resource       │
│   Server)               │
└────┬────────────────────┘
     │ 2. Validate Token with Keycloak
     ↓
┌─────────────────────────┐
│   Keycloak Server       │
│   (Port: 8080)          │
│   - Auth/Token endpoint │
│   - User Database       │
└────┬────────────────────┘
     │ 3. Issue JWT Token
     ↓
┌──────────┐              ┌──────────────────┐
│  Client  │ Access Token │   Microservices  │
│          │──────────→   │  - Validate JWT  │
└──────────┘              │  - Check Scopes  │
                          └──────────────────┘
```

### JWT Token Structure
```
Header.Payload.Signature

Payload:
{
  "sub": "customer@example.com",
  "email": "customer@example.com",
  "iat": 1709045400,
  "exp": 1709049000,
  "scope": "openid profile email"
}
```

---

## Deployment Architecture

### Docker Compose Services

```yaml
Services Running on Docker:
├── API Gateway          (Port 8080)
├── Product Service      (Port 8081)
│   └── MongoDB          (Port 27017)
├── Inventory Service    (Port 8082)
│   └── PostgreSQL       (Port 5432)
├── Order Service        (Port 8083)
│   └── PostgreSQL       (Port 5431)
├── Notification Service (Port 8084)
├── Kafka Broker         (Port 9092)
├── Zookeeper            (Port 2181)
├── Keycloak             (Port 8080)
│   └── MySQL 5.7        (Custom Port)
└── Eureka Server        (Port 8761) [Optional]
```

### Network Communication
```
┌────────────────────────────────────────────────────────┐
│                 Docker Network                         │
├────────────────────────────────────────────────────────┤
│                                                        │
│  ┌──────────────┐                                      │
│  │  API Gateway │                                      │
│  └──────┬───────┘                                      │
│         │                                              │
│  ┌──────┴──────┬──────────┬──────────┐                 │
│  ↓             ↓          ↓          ↓                 │
│ [Product]   [Order]   [Inventory] [Notification]      │
│   Service   Service    Service      Service           │
│  │   ↓      │  ↓       │  ↓        │  ↓              │
│  │[MongoDB] │[PG]      │[PG]       │[Kafka]          │
│  │          │          │           │                 │
│  └──────────┴──────────┴───────────┘                  │
│              ↓                                         │
│        ┌──────────────────┐                           │
│        │  Kafka Cluster   │                           │
│        │  + Zookeeper     │                           │
│        └──────────────────┘                           │
│                                                        │
└────────────────────────────────────────────────────────┘
```

---

## Key Design Patterns Used

### 1. **API Gateway Pattern**
- Single entry point for all clients
- Centralizes cross-cutting concerns (auth, logging, rate limiting)

### 2. **Database per Service**
- Each service owns its data
- Loose coupling between services
- Technology choice per service needs

### 3. **Event-Driven Architecture**
- Kafka for async communication
- Services publish/subscribe to events
- Eventual consistency model

### 4. **Circuit Breaker Pattern**
- Prevents cascading failures
- Fallback mechanisms in Order Service
- Implemented with Resilience4j

### 5. **Service Discovery**
- Eureka for dynamic service registration
- Services discover each other automatically

### 6. **Distributed Tracing**
- Micrometer + Brave + Zipkin
- Track requests across services
- Performance monitoring

---

## Running the Project

### Prerequisites
```bash
- Docker & Docker Compose
- Java 17+
- Maven 3.8+
```

### Start All Services
```bash
cd /path/to/project
docker-compose up -d

# Services will be available:
- API Gateway: http://localhost:8080
- Keycloak: http://localhost:8080/auth
- Kafka: localhost:9092
- MongoDB: localhost:27017
- PostgreSQL: localhost:5432 & 5431
```

### Environment Variables
```bash
# Keycloak Admin
KEYCLOAK_ADMIN=admin
KEYCLOAK_ADMIN_PASSWORD=admin

# Database Credentials
DB_USER=ptechie
DB_PASSWORD=password

# Kafka
KAFKA_BROKER=broker:29092
```

---

## Example End-to-End Request

### Timeline of a Complete Order
```
T0: User logs in
   └─→ Keycloak validates credentials
       └─→ Issues JWT token

T1: User browses products
   └─→ GET /api/product (via API Gateway)
       └─→ Product Service retrieves from MongoDB
           └─→ Returns product catalog

T2: User checks inventory
   └─→ GET /api/inventory?skuCode=iphone-13
       └─→ Inventory Service checks PostgreSQL
           └─→ Returns stock status

T3: User places order
   └─→ POST /api/order
       └─→ API Gateway routes to Order Service
           └─→ (CircuitBreaker CHECK) → Open/Closed?
               ├─ If Closed:
               │  └─→ OrderService.placeOrder()
               │      └─→ Check Inventory → Inventory Service
               │          └─→ If Stock: Create Order
               │              └─→ Publish OrderPlacedEvent
               │                  └─→ Kafka Topic: "order-placed"
               │                      └─→ Notification Service receives
               │                          └─→ Sends email to customer
               │                              └─→ Order Status: CONFIRMED
               │
               └─ If Open:
                  └─→ Fallback: Return error message

T4: User views order details
   └─→ GET /api/order/order-uuid
       └─→ Order Service retrieves from PostgreSQL
           └─→ Returns order with line items
```

---

## Conclusion

This E-Commerce microservices architecture provides:

✅ **Scalability** - Each service scales independently  
✅ **Resilience** - Circuit breakers and retry logic  
✅ **Flexibility** - Different databases per service  
✅ **Maintainability** - Clear service boundaries  
✅ **Real-time Communication** - Kafka for event streaming  
✅ **Security** - OAuth2 with Keycloak  
✅ **Observability** - Distributed tracing and metrics  

**Perfect for:** Growing e-commerce platforms requiring high availability and independent service scaling!
