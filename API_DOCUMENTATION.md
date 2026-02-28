# E-Commerce API Documentation & Examples

## Base URL
```
http://localhost:8080
```

All requests must include the Authorization header with a valid JWT token obtained from Keycloak.

---

## 🔐 Authentication Endpoints

### 1. Get Access Token
**Endpoint:** Keycloak OAuth2 Token Endpoint  
**URL:** `POST http://localhost:8080/auth/realms/ecommerce/protocol/openid-connect/token`

**Request:**
```bash
curl -X POST http://localhost:8080/auth/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "client_id=e-commerce-app" \
  -d "grant_type=password" \
  -d "username=customer@example.com" \
  -d "password=securePassword123" \
  -d "client_secret=your-client-secret"
```

**Response (200 OK):**
```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiJmYzYzZjA4Yi00OTEyLTQ3ZjUtOTE0Ni01YzQyZjc2MWMwYTMiLCJleHAiOjE3MDkwNDkwMDAsIm5iZiI6MCwiaWF0IjoxNzA5MDQ1NDAwLCJpc3MiOiJodHRwOi8vbG9jYWxob3N0OjgwODAvYXV0aC9yZWFsbXMvZWNvbW1lcmNlIiwiYXVkIjoiYWNjb3VudCIsInN1YiI6ImY4YjEzOTlkLWZmYzktNGZkZS05YTBjLTEzYjRkNDlhMWQxZiIsInR5cCI6IkJlYXJlciIsImF6cCI6ImUtY29tbWVyY2UtYXBwIiwibm9uY2UiOiI4YzRhNWI5Ny1iYzc1LTQ2NWQtOWFmZi1kNzQwMDQyZmI0MzMiLCJhdXRoX3RpbWUiOjE3MDkwNDU0MjAsInNlc3Npb25fc3RhdGUiOiI0YWQ4MzNjYy01ZDZiLTQyNmUtODg3MS1mZjdkYjJlZDMyNzAiLCJhY3IiOiIxIiwicmVhbG1fYWNjZXNzIjp7InJvbGVzIjpbImRlZmF1bHQtcm9sZXMtZWNvbW1lcmNlIiwib2ZmbGluZV9hY2Nlc3MiLCJ1bWFfYXV0aG9yaXphdGlvbiIsImN1c3RvbWVyIl19LCJyZXNvdXJjZV9hY2Nlc3MiOnsiYWNjb3VudCI6eyJyb2xlcyI6WyJtYW5hZ2UtYWNjb3VudCIsIm1hbmFnZS1hY2NvdW50LWxpbmtzIiwidmlldy1wcm9maWxlIl19fSwibmFtZSI6IkNvcmNvciBHaWVua28iLCJzdWIiOiJmOGIxMzk5ZC1mZmM5LTRmZGUtOWEwYy0xM2I0ZDQ5YTFkMWYiLCJwcmVmZXJyZWRfdXNlcm5hbWUiOiJjdXN0b21lckBleGFtcGxlLmNvbSIsImdpdmVuX25hbWUiOiJDb3Jjb3IiLCJmYW1pbHlfbmFtZSI6IkdpZW5rbyIsImVtYWlsIjoiY3VzdG9tZXJAZXhhbXBsZS5jb20ifQ.signature",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_expires_in": 1800,
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "scope": "openid profile email"
}
```

---

## 📦 Product Service API

### Service Details
- **Base URL:** `http://localhost:8080/api/product`
- **Port:** 8081 (standalone)
- **Database:** MongoDB
- **Authentication:** Required (Bearer Token)

### 1. Create Product
**Method:** `POST`  
**Endpoint:** `/api/product`  
**Status Code:** 201 Created

**Request Headers:**
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

**Request Body:**
```json
{
  "name": "iPhone 13 Pro",
  "price": 1099.99,
  "description": "Latest Apple smartphone with advanced camera system",
  "skuCode": "iphone-13-pro",
  "category": "Electronics",
  "stock_quantity": 100,
  "specifications": {
    "color": "Space Gray",
    "storage": "256GB",
    "processor": "A15 Bionic",
    "display": "6.1-inch Super Retina XDR",
    "battery": "3200mAh"
  }
}
```

**Response (201 Created):**
```json
{
  "id": "63f7e8b4d5c9a1b2e3f4g5h6",
  "name": "iPhone 13 Pro",
  "price": 1099.99,
  "description": "Latest Apple smartphone with advanced camera system",
  "skuCode": "iphone-13-pro",
  "category": "Electronics",
  "stock_quantity": 100,
  "specifications": {
    "color": "Space Gray",
    "storage": "256GB",
    "processor": "A15 Bionic",
    "display": "6.1-inch Super Retina XDR",
    "battery": "3200mAh"
  },
  "createdAt": "2024-02-28T10:00:00Z",
  "updatedAt": "2024-02-28T10:00:00Z"
}
```

---

### 2. Get All Products
**Method:** `GET`  
**Endpoint:** `/api/product`  
**Status Code:** 200 OK

**Request Headers:**
```
Authorization: Bearer {access_token}
```

**Response (200 OK):**
```json
[
  {
    "id": "63f7e8b4d5c9a1b2e3f4g5h6",
    "name": "iPhone 13 Pro",
    "price": 1099.99,
    "description": "Latest Apple smartphone with advanced camera system",
    "skuCode": "iphone-13-pro",
    "category": "Electronics",
    "stock_quantity": 100,
    "specifications": {
      "color": "Space Gray",
      "storage": "256GB"
    }
  },
  {
    "id": "63f7e8b4d5c9a1b2e3f4g5h7",
    "name": "Samsung Galaxy S21",
    "price": 899.99,
    "description": "Premium Android smartphone",
    "skuCode": "samsung-s21",
    "category": "Electronics",
    "stock_quantity": 75,
    "specifications": {
      "color": "Phantom Black",
      "storage": "128GB"
    }
  }
]
```

---

## 🛒 Order Service API

### Service Details
- **Base URL:** `http://localhost:8080/api/order`
- **Port:** 8083 (standalone)
- **Database:** PostgreSQL
- **Authentication:** Required (Bearer Token)
- **Resilience:** Circuit Breaker, Retry, Time Limiter

### 1. Place Order
**Method:** `POST`  
**Endpoint:** `/api/order`  
**Status Code:** 201 Created

**Request Headers:**
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

**Request Body:**
```json
{
  "skuCode": [
    "iphone-13-pro",
    "samsung-s21"
  ]
}
```

**Response (201 Created) - Success Case:**
```json
{
  "orderNumber": "ORD-20240228-001",
  "message": "Order Placed Successfully!",
  "status": "PENDING",
  "items": [
    {
      "skuCode": "iphone-13-pro",
      "quantity": 1,
      "price": 1099.99
    },
    {
      "skuCode": "samsung-s21",
      "quantity": 1,
      "price": 899.99
    }
  ],
  "totalPrice": 1999.98,
  "createdAt": "2024-02-28T10:30:00Z"
}
```

**Response (503 Service Unavailable) - Circuit Breaker Open:**
```json
{
  "message": "Oops! Something went wrong, please order after some time!",
  "status": "SERVICE_UNAVAILABLE",
  "timestamp": "2024-02-28T10:30:45Z"
}
```

**Resilience4j Behavior:**

| Scenario | Action | Result |
|----------|--------|--------|
| Normal | Process Order | ✅ 201 Created |
| Inventory Down | Retry 3x | If all fail → 503 |
| Timeout | Time Limiter | Fallback → 503 |
| Multiple Failures | Circuit Opens | Fast Fail → 503 |
| Service Recovery | Circuit Closes | Resume Normal Flow |

---

### 2. Get Order by ID
**Method:** `GET`  
**Endpoint:** `/api/order/{orderId}`  
**Status Code:** 200 OK

**Request Headers:**
```
Authorization: Bearer {access_token}
```

**Example URL:**
```
GET http://localhost:8080/api/order/550e8400-e29b-41d4-a716-446655440000
```

**Response (200 OK):**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "orderNumber": "ORD-20240228-001",
  "orderLineItems": [
    {
      "id": "660e8400-e29b-41d4-a716-446655440001",
      "skuCode": "iphone-13-pro",
      "price": 1099.99,
      "quantity": 1
    },
    {
      "id": "660e8400-e29b-41d4-a716-446655440002",
      "skuCode": "samsung-s21",
      "price": 899.99,
      "quantity": 1
    }
  ],
  "status": "CONFIRMED",
  "totalPrice": 1999.98,
  "createdDate": "2024-02-28T10:30:00Z",
  "lastModifiedDate": "2024-02-28T10:31:15Z"
}
```

---

### 3. Get All Orders
**Method:** `GET`  
**Endpoint:** `/api/order`  
**Status Code:** 200 OK

**Request Headers:**
```
Authorization: Bearer {access_token}
```

**Response (200 OK):**
```json
[
  {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "orderNumber": "ORD-20240228-001",
    "status": "CONFIRMED",
    "totalPrice": 1999.98,
    "createdDate": "2024-02-28T10:30:00Z"
  },
  {
    "id": "550e8400-e29b-41d4-a716-446655440001",
    "orderNumber": "ORD-20240228-002",
    "status": "PENDING",
    "totalPrice": 1099.99,
    "createdDate": "2024-02-28T11:00:00Z"
  }
]
```

---

## 📊 Inventory Service API

### Service Details
- **Base URL:** `http://localhost:8080/api/inventory`
- **Port:** 8082 (standalone)
- **Database:** PostgreSQL
- **Authentication:** Required (Bearer Token)
- **Purpose:** Real-time Stock Verification

### 1. Check Inventory
**Method:** `GET`  
**Endpoint:** `/api/inventory`  
**Status Code:** 200 OK  
**Query Params:** `skuCode` (multi-value supported)

**Request Headers:**
```
Authorization: Bearer {access_token}
```

**Example URLs:**
```
# Single SKU
GET http://localhost:8080/api/inventory?skuCode=iphone-13-pro

# Multiple SKUs
GET http://localhost:8080/api/inventory?skuCode=iphone-13-pro&skuCode=samsung-s21
```

**Response (200 OK) - All in Stock:**
```json
[
  {
    "skuCode": "iphone-13-pro",
    "isInStock": true,
    "quantity": 75,
    "lastUpdated": "2024-02-28T10:25:00Z"
  },
  {
    "skuCode": "samsung-s21",
    "isInStock": true,
    "quantity": 45,
    "lastUpdated": "2024-02-28T10:25:00Z"
  }
]
```

**Response (200 OK) - Partial Stock:**
```json
[
  {
    "skuCode": "iphone-13-pro",
    "isInStock": true,
    "quantity": 3,
    "lastUpdated": "2024-02-28T10:25:00Z"
  },
  {
    "skuCode": "samsung-s21",
    "isInStock": false,
    "quantity": 0,
    "lastUpdated": "2024-02-28T10:20:00Z"
  }
]
```

---

## 📧 Notification Service (Event-Driven)

### Service Details
- **Port:** 8084 (standalone)
- **Database:** None (Stateless)
- **Messaging:** Kafka Consumer
- **Purpose:** Async Email Notifications

### Event Model
**Kafka Topic:** `order-placed`

**Event Message Schema:**
```json
{
  "eventId": "evt-550e8400-e29b-41d4-a716-446655440000",
  "eventType": "OrderPlaced",
  "timestamp": "2024-02-28T10:30:00Z",
  "orderNumber": "ORD-20240228-001",
  "customerId": "cust-123",
  "customerEmail": "customer@example.com",
  "items": [
    {
      "skuCode": "iphone-13-pro",
      "quantity": 1,
      "price": 1099.99
    }
  ],
  "totalPrice": 1099.99,
  "shippingAddress": {
    "street": "123 Main St",
    "city": "New York",
    "zipCode": "10001",
    "country": "USA"
  }
}
```

### Email Notification Sent
**To:** customer@example.com  
**Subject:** Order Confirmation - ORD-20240228-001  

**Email Body:**
```
Dear Customer,

Your order has been successfully placed!

Order Number: ORD-20240228-001
Order Date: Feb 28, 2024 at 10:30 AM

Items Ordered:
- iPhone 13 Pro (1x) - $1,099.99

Total Amount: $1,099.99

Shipping Address:
123 Main St
New York, NY 10001
USA

Your order will be processed within 24 hours.
Tracking information will be sent once your package ships.

Thank you for shopping with us!

Best Regards,
E-Commerce Team
```

---

## Error Handling

### Common HTTP Status Codes

| Code | Meaning | Example |
|------|---------|---------|
| 200 | OK | Product retrieved successfully |
| 201 | Created | Order created |
| 400 | Bad Request | Invalid product data |
| 401 | Unauthorized | Missing/Invalid token |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Product/Order not found |
| 409 | Conflict | Duplicate SKU in product |
| 500 | Server Error | Unexpected error |
| 503 | Service Unavailable | Circuit breaker open |

### Error Response Format
```json
{
  "timestamp": "2024-02-28T10:30:45Z",
  "status": 400,
  "error": "Bad Request",
  "message": "Invalid product data: price must be greater than 0",
  "path": "/api/product"
}
```

---

## Circuit Breaker States

### State Diagram
```
┌─────────────┐
│   CLOSED    │ ← Normal operation
│   (Ready)   │
└──────┬──────┘
       │ Failure threshold exceeded
       ↓
┌─────────────┐
│    OPEN     │ ← Requests fail immediately
│   (Failing) │   Fallback is executed
└──────┬──────┘
       │ Wait for recovery time
       ↓
┌─────────────────┐
│ HALF_OPEN       │ ← Test if service recovered
│ (Testing)       │
└─────┬───────────┘
      │
      ├─ Success → CLOSED
      │
      └─ Failure → OPEN
```

### Configuration Example
```properties
# Resilience4j Circuit Breaker Config
resilience4j.circuitbreaker.instances.inventory.registerHealthIndicator=true
resilience4j.circuitbreaker.instances.inventory.slidingWindowSize=10
resilience4j.circuitbreaker.instances.inventory.failureRateThreshold=50
resilience4j.circuitbreaker.instances.inventory.waitDurationInOpenState=5000
resilience4j.circuitbreaker.instances.inventory.slowCallRateThreshold=50

# Retry Config
resilience4j.retry.instances.inventory.maxAttempts=3
resilience4j.retry.instances.inventory.waitDuration=1000

# Time Limiter Config
resilience4j.timelimiter.instances.inventory.timeoutDuration=3000
```

---

## Complete Example Flow

### Sample cURL Commands

```bash
#### 1. LOGIN
TOKEN=$(curl -s -X POST http://localhost:8080/auth/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "client_id=e-commerce-app" \
  -d "grant_type=password" \
  -d "username=customer@example.com" \
  -d "password=password" | jq -r '.access_token')

echo "Access Token: $TOKEN"

#### 2. CREATE PRODUCT
curl -X POST http://localhost:8080/api/product \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "iPhone 13 Pro",
    "price": 1099.99,
    "skuCode": "iphone-13-pro",
    "description": "Apple iPhone 13 Pro",
    "category": "Electronics"
  }'

#### 3. GET ALL PRODUCTS
curl -X GET http://localhost:8080/api/product \
  -H "Authorization: Bearer $TOKEN"

#### 4. CHECK INVENTORY
curl -X GET "http://localhost:8080/api/inventory?skuCode=iphone-13-pro" \
  -H "Authorization: Bearer $TOKEN"

#### 5. PLACE ORDER
curl -X POST http://localhost:8080/api/order \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "skuCode": ["iphone-13-pro"]
  }'

#### 6. GET ORDER
# Extract order ID from previous response
ORDER_ID="550e8400-e29b-41d4-a716-446655440000"
curl -X GET "http://localhost:8080/api/order/$ORDER_ID" \
  -H "Authorization: Bearer $TOKEN"
```

---

## WebSocket Events (Future Enhancement)

Real-time order status updates can be implemented via WebSocket:

```javascript
// JavaScript Client
const ws = new WebSocket('ws://localhost:8080/ws/orders');

ws.onmessage = (event) => {
  const orderUpdate = JSON.parse(event.data);
  console.log('Order Updated:', orderUpdate);
  // Update UI with real-time status
};

// Expected payload:
{
  "orderNumber": "ORD-20240228-001",
  "status": "SHIPPED",
  "trackingNumber": "TRACK123456",
  "estimatedDelivery": "2024-03-05",
  "timestamp": "2024-03-01T14:30:00Z"
}
```

---

## Performance Metrics

### Expected Response Times
| Operation | Expected Time | Database |
|-----------|--------------|----------|
| Create Product | 150-300ms | MongoDB |
| Get All Products | 100-200ms | MongoDB |
| Check Inventory | 50-100ms | PostgreSQL |
| Place Order | 200-500ms* | PostgreSQL + Kafka |
| Get Order | 75-150ms | PostgreSQL |

*Includes circuit breaker, retry, and timeout checks

---

## Rate Limiting (Future Enhancement)

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 99
X-RateLimit-Reset: 1709049000
```

Recommended per IP:
- 100 requests per minute for read operations
- 20 requests per minute for write operations

---

## Testing API Endpoints

### Using Postman
1. Import the collection from `postman/E-Commerce-API.json`
2. Set environment variables:
   - `base_url` = http://localhost:8080
   - `access_token` = (obtained from login)
3. Run tests in sequence

### Using cURL
All examples provided above use cURL for easy testing

### Using REST Client VSCode Extension
Create file `.http` with requests:
```
@baseUrl = http://localhost:8080
@token = eyJhbGciOiJIUzI1NiIs...

### Get All Products
GET {{baseUrl}}/api/product
Authorization: Bearer {{token}}
```

---

## Troubleshooting

### 401 Unauthorized
- **Cause:** Missing or expired token
- **Solution:** Get new token via login endpoint

### 503 Service Unavailable
- **Cause:** Circuit breaker open (Inventory Service down)
- **Solution:** Wait for service recovery or check service logs

### Connection Refused
- **Cause:** Services not running
- **Solution:** Run `docker-compose up -d`

### Validation Error on Order
- **Cause:** Invalid SKU code
- **Solution:** Check inventory for valid SKU codes

---

## API Documentation Summary

| Service | Total Endpoints | Key Operations |
|---------|-----------------|-----------------|
| Product | 3 | Create, Read All |
| Order | 3 | Create, Read One, Read All |
| Inventory | 1 | Stock Check |
| Notification | 0 | Event-driven |

**Total REST Endpoints: 7**

