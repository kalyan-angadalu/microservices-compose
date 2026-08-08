# Spring Boot Microservices Order Management System

A production-style microservices application built using Spring Boot and Spring Cloud.

The project demonstrates service discovery, centralized configuration, API Gateway routing, REST-based communication, Kafka event-driven messaging, email notifications, MySQL persistence, and Docker Compose container orchestration.

## Architecture Diagram

![Spring Boot Microservices Architecture](microservices-architecture.png)
The system consists of the following microservices:

| Service | Responsibility |
|---|---|
| Service Registry | Eureka Server used for service discovery |
| Config Server | Provides centralized configuration for microservices |
| API Gateway | Single entry point and routes requests to backend services |
| Customer Service | Manages customer information |
| Order Service | Handles order operations and publishes order events to Kafka |
| Notification Service | Consumes Kafka order events and sends email notifications |

## High-Level Flow

Client / Postman  
↓  
API Gateway  
↓  
Customer Service / Order Service  
↓  
Kafka  
↓  
Notification Service  
↓  
Email Notification

All microservices register with the Eureka Service Registry and obtain centralized configuration through Spring Cloud Config Server.

## Tech Stack

- Java 17
- Spring Boot
- Spring Cloud
- Netflix Eureka
- Spring Cloud Gateway
- Spring Cloud Config
- Spring Data JPA
- MySQL
- Apache Kafka
- Spring Kafka
- Java Mail
- Docker
- Docker Compose
- Maven
- Git & GitHub
- Postman

## Microservice Repositories

The application is divided into independently maintained microservices:

- [Service Registry](https://github.com/kalyan-angadalu/service-registry)
- [Config Server](https://github.com/kalyan-angadalu/config-server)
- [API Gateway](https://github.com/kalyan-angadalu/api-gateway)
- [Customer Service](https://github.com/kalyan-angadalu/customer-service)
- [Order Service](https://github.com/kalyan-angadalu/order-service)
- [Notification Service](https://github.com/kalyan-angadalu/notification-service)
- [Microservices Config](https://github.com/kalyan-angadalu/microservices-config)

## Containerized Deployment

The complete microservices environment can be started using Docker Compose.

The Docker Compose configuration manages the application services and supporting infrastructure required by the system.

Detailed setup and execution instructions will be added below.
## Running the Application with Docker Compose

The complete microservices environment is containerized using Docker and orchestrated using Docker Compose.

### Prerequisites

Make sure the following tools are installed:

- Java 17
- Maven
- Docker Desktop
- Git

### 1. Clone the Repository

```bash
git clone https://github.com/kalyan-angadalu/microservices-compose.git
cd microservices-compose
```

### 2. Configure Environment Variables

The Notification Service uses environment variables for email authentication.

Create a `.env` file in the root directory based on `.env.example`.

```env
MAIL_USERNAME: ${MAIL_USERNAME}
MAIL_APP_PASSWORD: ${MAIL_APP_PASSWORD}
```

Do not commit the `.env` file to GitHub.

### 3. Build the Microservice Docker Images

The Docker Compose configuration uses locally built Docker images.

Build each microservice before starting Docker Compose:

```bash
docker build -t service-registry:latest ../service-registry
docker build -t config-server:latest ../config-server
docker build -t api-gateway:latest ../api-gateway
docker build -t customer-service:latest ../customer-service
docker build -t order-service:latest ../order-service
docker build -t notification-service:latest ../notification-service
```

### 4. Start the Complete System

Run:

```bash
docker compose up -d
```

Docker Compose starts the infrastructure and application services on the shared `microservices-network`.

### 5. Verify Running Containers

```bash
docker compose ps
```

You can also check:

```bash
docker ps
```

### Service Ports

| Component | Host Port | Purpose |
|---|---:|---|
| API Gateway | 8080 | Main entry point for client requests |
| Customer Service | 8081 | Customer management |
| Order Service | 8082 | Order management |
| Notification Service | 8083 | Kafka consumer and email notification |
| Config Server | 8888 | Centralized configuration |
| Eureka Server | 8761 | Service discovery dashboard |
| Kafka | 9092 | Event broker |
| MySQL | 3307 | Database access from host |

## API Endpoints

All client requests can be routed through the API Gateway running on port `8080`.

### Customer Service

The Customer Service provides customer information used by the Order Service.

| Method | Endpoint | Description |
|---|---|---|
| GET | `/customers/{id}` | Retrieves customer information by customer ID |

#### Get Customer by ID

```http
GET /customers/1
```

Example response:

```json
{
  "id": 1,
  "name": "kalyan",
  "email": "Kalyan@gmail.com",
  "phone": "879000"
}
```

> The current Customer Service uses sample customer data for demonstrating inter-service communication.

---

### Order Service

The Order Service manages order operations and communicates with Customer Service to retrieve customer information.

| Method | Endpoint | Description |
|---|---|---|
| POST | `/orders/place` | Places a new order |
| GET | `/orders/getAll` | Retrieves all orders |
| GET | `/orders/get/{id}` | Retrieves an order by ID |
| PUT | `/orders/update/{id}` | Updates an existing order |
| DELETE | `/orders/delete/{id}` | Deletes an order |
| GET | `/orders/customer/{customerId}` | Retrieves customer information from Customer Service |
| GET | `/orders/{id}/details` | Retrieves combined order and customer details |

### Place an Order

```http
POST /orders/place
```

Example request:

```json
{
  "customerId": 1,
  "productName": "Laptop",
  "quantity": 1,
  "price": 65000
}
```

When an order is placed:

1. Order Service receives the request.
2. Customer information is retrieved from Customer Service.
3. The order is persisted in MySQL.
4. The order status is set to `PLACED`.
5. An order event is published to Kafka.
6. Notification Service consumes the event.
7. An email notification is sent.

### Get All Orders

```http
GET /orders/getAll
```

### Get Order by ID

```http
GET /orders/get/1
```

### Update Order

```http
PUT /orders/update/1
```

### Delete Order

```http
DELETE /orders/delete/1
```

Successful response:

```text
Order deleted successfully.
```

### Get Customer through Order Service

```http
GET /orders/customer/1
```

This endpoint demonstrates synchronous communication between Order Service and Customer Service.

### Get Order with Customer Details

```http
GET /orders/1/details
```

Returns combined order and customer information.

## Kafka Event-Driven Communication

The project uses Apache Kafka for asynchronous communication between the Order Service and Notification Service.

When a new order is successfully placed, the Order Service acts as a Kafka producer and publishes an order event. The Notification Service acts as a Kafka consumer and processes the event independently.

### Kafka Flow

```text
Client
  |
  v
API Gateway
  |
  v
Order Service
  |
  | Publish Order Event
  v
Apache Kafka
  |
  | Consume Order Event
  v
Notification Service
  |
  v
Email Notification
```

### Order Service - Kafka Producer

After an order is successfully created, the Order Service publishes an order event to Kafka.

The event contains the information required by the Notification Service to process the notification.

This provides asynchronous communication between the services and avoids tightly coupling the Order Service with the notification logic.

### Notification Service - Kafka Consumer

The Notification Service listens for order events published to Kafka.

When an event is received, the service:

1. Consumes the order event from Kafka.
2. Extracts the required order/customer information.
3. Prepares the email notification.
4. Sends the email using the configured mail server.

## Email Notification Flow

Email credentials are supplied through environment variables rather than being hard-coded into the application.

```env
MAIL_USERNAME=your-email@example.com
MAIL_APP_PASSWORD=your-app-password
```

The real credentials are stored in the local `.env` file, which should not be committed to GitHub.

The `.env.example` file documents the required environment variables without exposing sensitive credentials.

### Complete Order Flow

```text
POST /orders/place
        |
        v
   API Gateway
        |
        v
   Order Service
        |
        +----> Customer Service
        |       Retrieve Customer
        |
        v
      MySQL
   Persist Order
        |
        v
      Kafka
   Publish Event
        |
        v
Notification Service
        |
        v
 Email Notification
```

This architecture demonstrates both:

- **Synchronous communication** — Order Service communicates with Customer Service.
- **Asynchronous communication** — Order Service communicates with Notification Service through Kafka.

### Eureka Dashboard

After the containers are running, open:

```text
http://localhost:8761
```

The registered microservices should appear on the Eureka dashboard.

### API Gateway

Client requests should normally enter the system through:

```text
http://localhost:8080
```

The API Gateway routes requests to the appropriate registered microservice.

### Stop the Application

```bash
docker compose down
```

To stop the application and also remove the persistent Docker volume:

```bash
docker compose down -v
```

> **Warning:** `docker compose down -v` removes the MySQL Docker volume and therefore deletes the database data stored in that volume.

## Project Status

The core microservices architecture, Kafka communication, email notification flow, centralized configuration, service discovery, API Gateway, database integration, and Docker Compose setup are implemented.

Further improvements include cloud deployment, monitoring, centralized logging, security, and Kubernetes orchestration.
