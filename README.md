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
