# E-Commerce Microservices - Quick Start & Deployment Guide

## 🚀 Quick Start Guide

### Prerequisites
```bash
✓ Docker & Docker Compose (v1.29+)
✓ Java 17 JDK
✓ Maven 3.8+
✓ Git
```

### 5-Minute Setup

#### 1️⃣ Clone Repository
```bash
cd c:\Users\Admin\OneDrive\Documents\E-commerce-main\E-commerce-main
```

#### 2️⃣ Start All Services
```bash
docker-compose up -d

# Verify all containers are running
docker-compose ps
```

**Expected Output:**
```
NAME                    STATUS
postgres-order          Up (healthy)
postgres-inventory      Up (healthy)
mongo                   Up (healthy)
keycloak-mysql          Up (healthy)
keycloak                Up (healthy)
zookeeper               Up (healthy)
broker                  Up (healthy)
```

#### 3️⃣ Build & Run Microservices

**Terminal 1: Product Service**
```bash
cd product-service
mvn spring-boot:run
# Runs on http://localhost:8081
```

**Terminal 2: Order Service**
```bash
cd order-service
mvn spring-boot:run
# Runs on http://localhost:8083
```

**Terminal 3: Inventory Service**
```bash
cd inventory-service
mvn spring-boot:run
# Runs on http://localhost:8082
```

**Terminal 4: Notification Service**
```bash
cd notification-service
mvn spring-boot:run
# Runs on http://localhost:8084
```

**Terminal 5: API Gateway**
```bash
cd api-gateway
mvn spring-boot:run
# Runs on http://localhost:8080
```

#### 4️⃣ Access Services
```
🚪 API Gateway:       http://localhost:8080
🔐 Keycloak:          http://localhost:8080/auth
📊 MongoDB:           mongodb://localhost:27017
🐘 PostgreSQL Order:  jdbc:postgresql://localhost:5431/order-service
🐘 PostgreSQL Inv:    jdbc:postgresql://localhost:5432/inventory-service
🔄 Kafka:             localhost:9092
⚙️  Zookeeper:         localhost:2181
```

---

## 📊 Service Ports & Access

```
┌────────────────────────────────────────────────────┐
│ Service Mapping                                    │
├────────────────────────────────────────────────────┤
│ 8080  │ API Gateway & Keycloak                    │
│ 8081  │ Product Service                           │
│ 8082  │ Inventory Service                         │
│ 8083  │ Order Service                             │
│ 8084  │ Notification Service                      │
│ 8761  │ Eureka Server (Optional)                  │
├────────────────────────────────────────────────────┤
│ 27017 │ MongoDB                                    │
│ 5432  │ PostgreSQL Inventory                      │
│ 5431  │ PostgreSQL Order                          │
│ 9092  │ Kafka Broker                              │
│ 2181  │ Zookeeper                                 │
│ 3306  │ MySQL (Keycloak DB)                       │
└────────────────────────────────────────────────────┘
```

---

## 🔑 Default Credentials

### Keycloak Admin Access
```
URL:      http://localhost:8080/auth
Username: admin
Password: admin
Realm:    ecommerce
```

### Database Credentials

**PostgreSQL (both services)**
```
Host:     localhost
Port:     5431 (Orders), 5432 (Inventory)
User:     ptechie
Password: password
```

**MongoDB**
```
Host:     localhost
Port:     27017
Database: product-service
```

**MySQL (Keycloak)**
```
Host:     localhost
User:     keycloak
Password: password
Database: keycloak
```

---

## 📋 Initial Data Setup

### 1. Load Sample Products
```bash
# Login to Keycloak first, then:
curl -X POST http://localhost:8080/api/product \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "iPhone 13",
    "price": 999.99,
    "skuCode": "iphone-13",
    "description": "Apple iPhone 13",
    "category": "Electronics"
  }'
```

### 2. Load Sample Inventory
Insert into PostgreSQL Inventory:
```sql
INSERT INTO inventory (id, quantity, sku_code) VALUES
(gen_random_uuid(), 50, 'iphone-13'),
(gen_random_uuid(), 30, 'iphone-13-red'),
(gen_random_uuid(), 25, 'samsung-s21');
```

---

## 🔄 Service Communication Flow

### Synchronous (REST)
```
Client → API Gateway → Order Service → Inventory Service
         (authentication)    (validation)
         ↓
      Returns Response
```

### Asynchronous (Kafka)
```
Order Service → Publish Event to Kafka Topic "order-placed"
                          ↓
                  Notification Service (Consumer)
                          ↓
                   Send Email Notification
```

---

## 📈 Monitoring & Logging

### View Logs
```bash
# Product Service
docker-compose logs -f postgres-inventory

# Order Service  
docker-compose logs -f postgres-order

# See all logs
docker-compose logs -f
```

### Access Kafka UI (Optional - Install Kafka UI)
```bash
# Add to docker-compose.yml and run:
docker-compose up -d kafka-ui
# Visit: http://localhost:8081
```

### Monitor Eureka (if running)
```
http://localhost:8761
```

---

## 🧪 Testing

### Run Unit Tests
```bash
# All tests
mvn clean test

# Specific service
cd product-service && mvn clean test
```

### Integration Tests
```bash
mvn clean verify
```

### Load Testing
```bash
# Using Apache JMeter
# Create test plan for order placement
# Target: 100 concurrent users
# Ramp-up: 10 seconds
```

---

## 📦 Docker Compose Useful Commands

```bash
# Start services
docker-compose up -d

# Stop services
docker-compose down

# View running services
docker-compose ps

# View service logs
docker-compose logs -f service-name

# Rebuild images
docker-compose build

# Remove volumes (⚠️ data loss)
docker-compose down -v

# Execute command in container
docker-compose exec postgres-order psql -U ptechie -d order-service

# Scale a service
docker-compose up -d --scale product-service=3
```

---

## 🛠️ Troubleshooting

### Service won't start
```bash
# Check logs
docker-compose logs -f service-name

# Verify ports are not in use
netstat -ano | findstr :8080

# Kill process on port
Get-Process | Where-Object {$_.ProcessName -eq "java"} | Stop-Process -Force
```

### Database connection errors
```bash
# Verify containers running
docker-compose ps

# Check network
docker network ls
docker network inspect ecommerce-main_default

# Test connection
docker-compose exec postgres-order psql -U ptechie -d order-service -c "SELECT 1"
```

### Kafka issues
```bash
# Check broker
docker-compose exec broker kafka-broker-api-versions.sh --bootstrap-server localhost:9092

# List topics
docker-compose exec broker kafka-topics.sh --list --bootstrap-server localhost:9092

# Create topic if missing
docker-compose exec broker kafka-topics.sh --bootstrap-server localhost:9092 \
  --create --topic order-placed --partitions 3 --replication-factor 1
```

### Keycloak not accessible
```bash
# Check if Keycloak is running
docker-compose logs keycloak

# Verify MySQL is running
docker-compose ps keycloak-mysql

# Restart Keycloak
docker-compose restart keycloak
```

---

## 🔐 Security Checklist

- [ ] Change default Keycloak admin password
- [ ] Change database passwords
- [ ] Enable HTTPS in production
- [ ] Use environment variables for secrets (not hardcoded)
- [ ] Enable Spring Security globally
- [ ] Implement rate limiting
- [ ] Regular security updates
- [ ] Enable audit logging
- [ ] Use network policies for service isolation
- [ ] Implement JWT token rotation

---

## 📊 Performance Tuning

### Database Optimization
```sql
-- Create indexes
CREATE INDEX idx_order_number ON orders(order_number);
CREATE INDEX idx_sku_code ON inventory(sku_code);
```

### Kafka Performance
```yaml
# Increase consumer partitions
partitions: 10
replication-factor: 3

# Batch configuration
batch.size: 16384
linger.ms: 100
```

### Spring Boot Optimization
```properties
# Async processing
spring.task.execution.pool.core-size=10
spring.task.execution.pool.max-size=20

# Connection pooling
spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.minimum-idle=5
```

---

## 🚀 Production Deployment

### Kubernetes Deployment (Example)
```bash
# Build Docker images
docker build -t myregistry/product-service:1.0 ./product-service
docker build -t myregistry/order-service:1.0 ./order-service
docker build -t myregistry/inventory-service:1.0 ./inventory-service
docker build -t myregistry/api-gateway:1.0 ./api-gateway
docker build -t myregistry/notification-service:1.0 ./notification-service

# Push to registry
docker push myregistry/product-service:1.0
docker push myregistry/order-service:1.0
# ... etc

# Deploy to Kubernetes
kubectl apply -f k8s/namespaces.yaml
kubectl apply -f k8s/configmaps.yaml
kubectl apply -f k8s/services.yaml
kubectl apply -f k8s/deployments.yaml
```

### Environment Configuration
```
Development:  docker-compose.yml (local)
Staging:      docker-compose.staging.yml
Production:   Kubernetes manifests
```

---

## 📈 Scaling Strategy

### Horizontal Scaling
```bash
# Scale Order Service to 3 instances
docker-compose up -d --scale order-service=3

# Behind load balancer (Nginx/HAProxy)
upstream order-service {
  server localhost:8083;
  server localhost:8084;
  server localhost:8085;
}
```

### Vertical Scaling
```properties
# Increase JVM heap
JAVA_OPTS="-Xmx2g -Xms1g"

# Thread pools
spring.task.execution.pool.max-size=50
```

### Database Scaling
- Read replicas for read-heavy operations
- Database sharding by product/order ID
- MongoDB horizontal scaling via replica sets

---

## 🔍 Health Checks

### Service Health Endpoints
```bash
# Product Service
curl http://localhost:8081/actuator/health

# Response:
{
  "status": "UP",
  "components": {
    "db": {"status": "UP"},
    "mongo": {"status": "UP"},
    "kafka": {"status": "UP"}
  }
}
```

### Monitoring Stack (Optional)
```yaml
services:
  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
  
  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
```

---

## 📚 Useful Resources

### Documentation Files in Project
- `ARCHITECTURE_DESIGN.md` - Complete architecture overview
- `API_DOCUMENTATION.md` - API endpoints & examples
- `README.md` - Project description

### External Resources
- Spring Boot: https://spring.io/projects/spring-boot
- Spring Cloud: https://spring.io/projects/spring-cloud
- Kafka: https://kafka.apache.org/
- Keycloak: https://www.keycloak.org/
- Docker Compose: https://docs.docker.com/compose/

---

## 🎯 Next Steps

1. **Setup Development Environment**
   - Clone and start docker-compose
   - Run all 5 microservices
   - Verify API Gateway access

2. **Load Sample Data**
   - Create products via API
   - Insert inventory data
   - Create test user in Keycloak

3. **Test Complete Flow**
   - Login → Browse → Order → Check Notification
   - Verify E2E functionality

4. **Performance Testing**
   - Load test with multiple concurrent users
   - Monitor service response times
   - Check database performance

5. **Production Deployment**
   - Set up CI/CD pipeline
   - Configure Kubernetes
   - Enable monitoring & logging

---

## 📞 Support & Debugging

### Enable Debug Logging
```yaml
logging:
  level:
    root: INFO
    com.programmingtechie: DEBUG
    org.springframework.security: DEBUG
```

### Common Issues & Solutions

| Issue | Solution |
|-------|----------|
| Port already in use | Kill process or change port in application.properties |
| Kafka topic not created | Create via kafka-topics.sh script |
| JWT token expired | Re-login to get new token |
| Circuit breaker open | Wait for recovery time or check inventory service |
| Database locked | Restart containers with `docker-compose down && up` |

---

## 🎓 Learning Path

1. **Understand Microservices Concepts**
   - Service boundaries
   - Communication patterns
   - Data consistency

2. **Study Each Service**
   - Product Service (MongoDB)
   - Inventory Service (PostgreSQL)
   - Order Service (Kafka, Circuit Breaker)
   - Notification Service (Event Consumer)

3. **Deep Dive into Patterns**
   - API Gateway pattern
   - Event-driven architecture
   - Circuit breaker pattern
   - SAGA pattern (future enhancement)

4. **Learn DevOps**
   - Docker containerization
   - Kubernetes orchestration
   - CI/CD pipelines
   - Monitoring & logging

---

## Summary

This E-Commerce microservices project provides:

✅ **Production-Ready Architecture** - Industry best practices  
✅ **Scalable Services** - Independent scaling  
✅ **Resilient Communication** - Circuit breakers & retries  
✅ **Secure Access** - OAuth2 with Keycloak  
✅ **Real-time Notifications** - Kafka event streaming  
✅ **Flexible Data Storage** - Right DB for each service  

**Ready to Scale! 🚀**
