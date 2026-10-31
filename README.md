# PrimeHaven - Spring Boot Microservices Real Estate Platform

**PrimeHaven** is a multi-module Spring Boot microservices platform built for real estate management. It includes user authentication with JWT (supporting CLIENT and AGENT roles), property listing management, visit scheduling, user inquiries with inter-service Feign calls, Eureka service registry, and Spring Cloud API Gateway.

---

## 🏗 Microservices Overview

| Microservice | Port | Database / Persistence | Responsibilities |
| :--- | :--- | :--- | :--- |
| **Eureka Server** | `8761` | N/A | Service discovery and registration |
| **API Gateway** | `8080` | N/A | Central entry point, load balancing (`lb://`), JWT filter |
| **Auth Service** | `8081` | H2 `authdb` (`users` table) | JWT authentication, user registration (CLIENT & AGENT roles) |
| **Property Service** | `8082` | H2 `propertydb` (`property` table) | Property listing management (location, price, status) |
| **Visit Service** | `8083` | H2 `visitdb` (`visit` table) | Visit scheduling (visit_id, property_id, user_id, visit_date) |
| **Inquiry Service** | `8084` | H2 `inquirydb` (`inquiry` table) | Inquiry processing; calls Property & Visit Services via OpenFeign |

---

## 🗄️ Database Schemas

### 1. Auth Service (`users` table)
- `id` (BIGINT, PRIMARY KEY, AUTO_INCREMENT)
- `username` (VARCHAR, UNIQUE)
- `password` (VARCHAR, BCrypt encoded)
- `email` (VARCHAR, UNIQUE)
- `role` (VARCHAR: `CLIENT` or `AGENT`)

### 2. Property Service (`property` table)
- `property_id` (BIGINT, PRIMARY KEY, AUTO_INCREMENT)
- `location` (VARCHAR)
- `price` (DECIMAL)
- `status` (VARCHAR: `AVAILABLE`, `PENDING`, `SOLD`)

### 3. Visit Service (`visit` table)
- `visit_id` (BIGINT, PRIMARY KEY, AUTO_INCREMENT)
- `property_id` (BIGINT)
- `user_id` (BIGINT)
- `visit_date` (TIMESTAMP)

### 4. Inquiry Service (`inquiry` table)
- `inquiry_id` (BIGINT, PRIMARY KEY, AUTO_INCREMENT)
- `user_id` (BIGINT)
- `property_id` (BIGINT)
- `message` (VARCHAR)

---

## 🛠️ How to Import into Spring Tool Suite (STS)

1. Open **Spring Tool Suite (STS)** or **Eclipse**.
2. Go to **File** -> **Import...**
3. Select **Maven** -> **Existing Maven Projects** and click **Next**.
4. Browse to the root directory `d:\PrimeHaven`.
5. STS will automatically discover the parent project `primehaven-parent` and all 6 submodules:
   - `eureka-server`
   - `api-gateway`
   - `auth-service`
   - `property-service`
   - `visit-service`
   - `inquiry-service`
6. Click **Finish**. STS will import all modules as a multi-module workspace structure.

---

## 🚀 Execution Order

To run the application locally:

1. **Start Eureka Server**:
   - Right-click `com.primehaven.eureka.EurekaServerApplication.java` -> **Run As** -> **Spring Boot App** (or `java -jar`).
   - Access dashboard at `http://localhost:8761`.

2. **Start API Gateway**:
   - Right-click `com.primehaven.gateway.ApiGatewayApplication.java` -> **Run As** -> **Spring Boot App**.

3. **Start Core Microservices**:
   - Run `com.primehaven.auth.AuthServiceApplication.java`
   - Run `com.primehaven.property.PropertyServiceApplication.java`
   - Run `com.primehaven.visit.VisitServiceApplication.java`
   - Run `com.primehaven.inquiry.InquiryServiceApplication.java`

---

## 🧪 Testing API Endpoints via Gateway (Port 8080)

### 1. Register Users

#### Register an AGENT user:
```bash
curl -X POST http://localhost:8080/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"username": "agent_sarah", "password": "password123", "email": "sarah@primehaven.com", "role": "AGENT"}'
```

#### Register a CLIENT user:
```bash
curl -X POST http://localhost:8080/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"username": "client_alex", "password": "password123", "email": "alex@gmail.com", "role": "CLIENT"}'
```

---

### 2. Authenticate & Obtain JWT Token

#### Login as AGENT:
```bash
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username": "agent_sarah", "password": "password123"}'
```
*Copy the returned `token` string.*

---

### 3. Property Management (AGENT Authorized)

#### Create a Property Listing (Requires AGENT JWT):
```bash
curl -X POST http://localhost:8080/api/properties \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <AGENT_JWT_TOKEN>" \
  -d '{"location": "Highland Park Villa #4", "price": 620000.00, "status": "AVAILABLE"}'
```

#### List Properties (CLIENT or AGENT JWT):
```bash
curl -X GET http://localhost:8080/api/properties \
  -H "Authorization: Bearer <CLIENT_OR_AGENT_JWT_TOKEN>"
```

---

### 4. Visit Scheduling

#### Schedule a Property Visit:
```bash
curl -X POST http://localhost:8080/api/visits \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <CLIENT_JWT_TOKEN>" \
  -d '{"propertyId": 1, "userId": 2, "visitDate": "2026-10-05T14:30:00"}'
```

---

### 5. Inquiries (Inter-Service Communication)

#### Create an Inquiry (Validates property with Property Service via Feign):
```bash
curl -X POST http://localhost:8080/api/inquiries \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <CLIENT_JWT_TOKEN>" \
  -d '{"userId": 2, "propertyId": 1, "message": "Is the price negotiable?"}'
```

#### Get Inquiry with Enriched Property & Visit Details:
```bash
curl -X GET http://localhost:8080/api/inquiries/1 \
  -H "Authorization: Bearer <CLIENT_OR_AGENT_JWT_TOKEN>"
```

---

## 🧪 Running Unit & Integration Tests

From the root project directory, run:
```bash
./mvnw clean test
```
Or right-click each project module in STS -> **Run As** -> **JUnit Test**.
