# Babbly - Enterprise Twitter Clone

A production-ready, microservices-based social media platform built as a demonstration of modern software architecture and cloud-native development practices.

## Overview

Babbly is a Twitter-like social media application developed as a school project for **Fontys Hogescholen** focused on enterprise application architecture. The project showcases a complete microservices ecosystem with event-driven communication, multiple database technologies, and Kubernetes orchestration.

### Current Status

⚠️ **This application is currently non-active.** The semester ended and maintaining a full application stack in Azure Kubernetes Service proved too costly for continued operation. Additionally, resolving outstanding issues would require time and motivation that are currently unavailable.

However, **the application was fully functional and running** in an Azure Kubernetes cluster during development. The deployment used Neon PostgreSQL (serverless PostgreSQL) instead of Azure Database for PostgreSQL due to cost.

**Key Highlights:**
- 7 independent microservices with clearly defined boundaries
- Event-driven architecture using Apache Kafka
- Polyglot persistence (PostgreSQL, Cassandra)
- API Gateway pattern with YARP
- Auth0 integration for enterprise authentication
- Container orchestration with Kubernetes
- Full CI/CD pipeline integration ready

## Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────────┐
│                         Frontend                             │
│                    (Next.js/React)                           │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                      API Gateway                             │
│                   (YARP/ASP.NET Core)                        │
└──────┬────────┬────────┬────────┬────────┬─────────────────┘
       │        │        │        │        │
       ▼        ▼        ▼        ▼        ▼
┌──────────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐
│   Auth   │ │ User │ │ Post │ │Comment│ │ Like │
│ Service  │ │Service│ │Service│ │Service│ │Service│
└────┬─────┘ └──┬───┘ └──┬───┘ └──┬───┘ └──┬───┘
     │          │        │        │        │
     └──────────┴────────┴────────┴────────┘
                        │
                        ▼
             ┌───────────────────┐
             │   Kafka Cluster   │
             │  (Event Streaming) │
             └───────────────────┘
```

### Technology Stack

**Frontend:**
- Next.js 14 (App Router)
- React
- Tailwind CSS
- SWR for data fetching
- Auth0 SDK

**Backend Microservices:**
- ASP.NET Core 9.0
- Entity Framework Core
- YARP (Reverse Proxy)

**Databases:**
- PostgreSQL - User data (relational)
- Apache Cassandra - Posts, Comments, Likes (NoSQL, distributed)

**Infrastructure:**
- Apache Kafka + Zookeeper - Event streaming
- Auth0 - Authentication & authorization
- Docker + Docker Compose - Containerization
- Kubernetes (Minikube) - Orchestration
- Prometheus + Grafana - Monitoring
- InfluxDB + K6 - Load testing

## Services

### [Frontend Service](./babbly-frontend)
**Technology:** Next.js, React  
**Port:** 3000  
**Purpose:** Web interface providing user authentication, post creation, feed viewing, and profile management.

### [API Gateway](./babbly-api-gateway)
**Technology:** ASP.NET Core, YARP  
**Port:** 5010  
**Purpose:** Single entry point for all client requests. Handles routing, authentication, rate limiting, and request aggregation.

### [Auth Service](./babbly-auth-service)
**Technology:** ASP.NET Core, Auth0  
**Port:** 5001  
**Purpose:** Manages authentication via Auth0, validates JWT tokens, and publishes authentication events to Kafka.

### [User Service](./babbly-user-service)
**Technology:** ASP.NET Core, PostgreSQL  
**Port:** 8081  
**Purpose:** Manages user profiles and data. Consumes authentication events from Kafka to maintain user records.

### [Post Service](./babbly-post-service)
**Technology:** ASP.NET Core, Cassandra  
**Port:** 8080  
**Purpose:** Handles post creation, retrieval, updates, and deletion. Publishes post events to Kafka.

### [Comment Service](./babbly-comment-service)
**Technology:** ASP.NET Core, Cassandra  
**Port:** 8082  
**Purpose:** Manages comments on posts. Publishes comment events to Kafka.

### [Like Service](./babbly-like-service)
**Technology:** ASP.NET Core, Cassandra  
**Port:** 8083  
**Purpose:** Handles likes/reactions on posts. Publishes like events to Kafka.

## Local Development

### Prerequisites

- Docker Desktop
- Docker Compose (have a lot of ram available docker likes to eat all of it)
- .NET SDK 9.0+ (for individual service development)
- Node.js 18+ (for frontend development)
- Auth0 account (free tier available)

### Quick Start

1. **Clone the organization repositories:**
   ```bash
   # Each service is its own repository
   git clone https://github.com/yourusername/babbly-frontend.git
   git clone https://github.com/yourusername/babbly-api-gateway.git
   git clone https://github.com/yourusername/babbly-auth-service.git
   git clone https://github.com/yourusername/babbly-user-service.git
   git clone https://github.com/yourusername/babbly-post-service.git
   git clone https://github.com/yourusername/babbly-comment-service.git
   git clone https://github.com/yourusername/babbly-like-service.git
   ```

2. **Configure Auth0:**
   - Create an Auth0 application (Regular Web Application)
   - Create an Auth0 API with identifier `https://api.babbly.com`
   - Enable RBAC and "Add Permissions in the Access Token"
   - Note your domain, client ID, client secret, and audience

3. **Set up environment variables:**
   
   Create a `.env` file in the root directory:
   ```env
   # Auth0 Configuration
   AUTH0_DOMAIN=your-tenant.auth0.com
   AUTH0_CLIENT_ID=your_client_id
   AUTH0_CLIENT_SECRET=your_client_secret
   AUTH0_AUDIENCE=https://api.babbly.com
   AUTH0_SECRET=your_long_random_secret_min_32_chars
   AUTH0_BASE_URL=http://localhost:3000
   ```

4. **Start all services:**
   ```bash
   docker-compose up -d
   ```

5. **Access the application:**
   - Frontend: http://localhost:3000
   - API Gateway: http://localhost:5010
   - Swagger UI: http://localhost:5010/swagger

### Service Startup Order

The docker-compose configuration handles dependencies automatically:
1. Infrastructure (Zookeeper, Kafka, Cassandra, PostgreSQL)
2. Backend services (Auth, User, Post, Comment, Like)
3. API Gateway
4. Frontend

### Stopping Services

```bash
docker-compose down
```

To remove volumes and clean up completely:
```bash
docker-compose down -v
```

## Key Features Demonstrated

### 1. Microservices Architecture
- **Service Independence**: Each service can be deployed, scaled, and updated independently
- **Domain-Driven Design**: Clear service boundaries based on business domains
- **API Gateway Pattern**: Centralized entry point for client requests
- **Service Discovery**: Services communicate via Docker networking

### 2. Event-Driven Communication
- **Kafka Integration**: Asynchronous communication between services
- **Event Sourcing**: User events flow from Auth Service to User Service
- **Loose Coupling**: Services react to events without direct dependencies
- **Topics**: Separate topics for users, posts, comments, and likes

### 3. Polyglot Persistence
- **PostgreSQL**: Relational data for users (ACID compliance needed)
- **Cassandra**: NoSQL for posts, comments, likes (high write throughput, horizontal scaling)
- **Right Tool for the Job**: Database selection based on service requirements

### 4. Authentication & Authorization
- **Auth0 Integration**: Enterprise-grade authentication
- **JWT Tokens**: Stateless authentication across services
- **Claims Forwarding**: API Gateway extracts and forwards user context
- **Role-Based Access Control**: Admin and user roles

### 5. API Design
- **RESTful APIs**: Standard HTTP methods and status codes
- **Swagger/OpenAPI**: Comprehensive API documentation
- **Consistent Error Handling**: Standardized error responses
- **Health Checks**: Monitoring endpoints for all services

### 6. Containerization & Orchestration
- **Docker**: All services containerized for consistency
- **Docker Compose**: Local multi-container orchestration
- **Kubernetes**: Production-ready K8s manifests included
- **Infrastructure as Code**: Declarative configuration

### 7. Observability
- **Health Checks**: Each service exposes health endpoints
- **Prometheus Metrics**: Performance and resource monitoring
- **Structured Logging**: Consistent log formats across services
- **Distributed Tracing Ready**: Architecture supports tracing integration

### 8. Scalability Patterns
- **Horizontal Scaling**: Services designed to scale horizontally
- **Database Replication**: Cassandra's built-in replication
- **Rate Limiting**: API Gateway enforces rate limits
- **Caching Ready**: Architecture supports Redis integration

## Project Highlights

### Technical Achievements

✅ **Microservices from Scratch**: Designed and implemented 7 independent services with clear boundaries

✅ **Polyglot Persistence**: Successfully integrated both SQL and NoSQL databases based on service requirements

✅ **Event-Driven Architecture**: Implemented Kafka-based event streaming for asynchronous communication

✅ **API Gateway Pattern**: Built a fully functional gateway with routing, auth, and aggregation

✅ **Kubernetes Deployment**: Created K8s manifests for production deployment on Minikube

✅ **Enterprise Authentication**: Integrated Auth0 with custom claims forwarding

✅ **Database Migrations**: Implemented EF Core migrations for PostgreSQL schema management

✅ **Container Orchestration**: Multi-container application with proper dependency management

### Challenges Solved

🔧 **Service Discovery**: Configured Docker networking for service-to-service communication

🔧 **Distributed Transactions**: Handled eventual consistency in event-driven architecture

🔧 **Cassandra Schema Design**: Designed efficient schemas for high-write scenarios

🔧 **JWT Validation**: Implemented token validation across multiple services

🔧 **CORS Configuration**: Properly configured cross-origin requests for frontend

🔧 **Database Initialization**: Automated Cassandra keyspace and PostgreSQL schema creation

## Testing

Each service includes:
- **Unit Tests**: Core business logic validation
- **Integration Tests**: Database and Kafka integration
- **API Tests**: HTTP endpoint testing with `.http` files
- **Load Testing**: K6 scripts for performance testing

## Deployment

### Docker Compose (Local/Development)
```bash
docker-compose up -d
```

### Kubernetes (Production)
```bash
# Deploy to Minikube
./deploy-to-minikube.ps1

# Or manually
kubectl apply -f infrastructure/k8s/
kubectl apply -f babbly-*/k8s/
```

### CI/CD Integration
Each service includes:
- GitHub Actions workflows (ready to configure)
- Docker image building and pushing
- SonarCloud integration for code quality
- Automated testing on pull requests

## Monitoring & Observability

- **Prometheus**: `http://localhost:9090` - Metrics collection
- **Grafana**: `http://localhost:3001` - Visualization dashboards
- **InfluxDB**: Time-series data for load testing metrics
- **Health Endpoints**: Each service exposes `/api/health`

## Documentation

- Each service has its own detailed README
- API documentation available via Swagger UI
- Architecture diagrams in `/docs`
- Database schemas documented in service READMEs

## Future Enhancements

Potential improvements and learning opportunities:
- [ ] Redis caching layer
- [ ] Distributed tracing (Jaeger/Zipkin)
- [ ] API versioning
- [ ] GraphQL gateway alternative
- [ ] WebSocket support for real-time updates
- [ ] Message queuing for notifications
- [ ] Advanced monitoring with ELK stack
- [ ] Service mesh (Istio/Linkerd)
- [ ] Automated backup strategies
- [ ] Multi-region deployment

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Contact

This project was created as a demonstration of enterprise software architecture and microservices design patterns. Feel free to explore the code and reach out with any questions!

---

**Note**: This is a educational project demonstrating modern software architecture patterns. While production-ready in architecture, additional hardening would be needed for a true production deployment (secrets management, advanced security, high availability, etc.).
