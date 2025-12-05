# Practical_06 Report: Comprehensive Testing for Microservices

## Repository
### **Source Code**: The complete source code for this practical is available in the GitHub repository:  
#### **Repository Link**: https://github.com/DechenWangdraSherpa/web303-practical-six

A Student Cafe management system developed using microservices architecture with Go services communicating via gRPC. The system provides a unified RESTful API through an API Gateway and implements a multi-layered testing framework.

## System Architecture

The application comprises the following services:

- **API Gateway**: Entry point for external requests, translating HTTP calls to gRPC.
- **User Service**: Manages user accounts (CRUD operations via gRPC).
- **Menu Service**: Manages cafe menu items and pricing via gRPC.
- **Order Service**: Processes customer orders, communicating with User and Menu services via gRPC.
- **Databases**: Individual PostgreSQL instances (`user-db`, `menu-db`, `order-db`) for each service.
- **Protobufs**: Centralized Protocol Buffer definitions for gRPC service contracts.

### Service Communication

- **Client → API Gateway (HTTP REST)**: Clients send HTTP requests to the gateway.
- **API Gateway → Services (gRPC)**: Gateway translates HTTP to gRPC calls.
- **Service Interactions**: Order Service communicates with User and Menu services to validate data.

## Key Features

- **Microservice Architecture**: Independent, loosely-coupled services for scalability and maintainability.
- **gRPC Communication**: High-performance, type-safe inter-service messaging.
- **API Gateway Pattern**: Unified REST API abstracting internal complexity.
- **Docker Containerization**: Services containerized and orchestrated via Docker Compose.
- **Multi-Layered Testing**:
  - **Unit Tests**: Individual service functions using in-memory SQLite.
  - **Integration Tests**: Service interactions via buffered in-memory connections.
  - **E2E Tests**: Complete workflow validation via HTTP requests.
- **Makefile Automation**: Standardized commands for building, testing, and deployment.

## Prerequisites

- [Go](https://go.dev/doc/install) (1.22 or later)
- [Docker](https://docs.docker.com/get-docker/) and [Docker Compose](https://docs.docker.com/compose/install/)
- [Protocol Buffers Compiler (`protoc`)](https://grpc.io/docs/protoc-installation/)
- Go gRPC toolchain (installed via `make install-tools`)

## Setup and Deployment

**Using Makefile (Recommended):**

```bash
make dev-setup    # One-time setup
make docker-up    # Start all services
```

**Using Deployment Script:**

```bash
./deploy.sh
```

Access the API Gateway at `http://localhost:8080`.

## API Endpoints

All requests go through the API Gateway at `http://localhost:8080`.

**User Service**

- `POST /api/users`: Create user
- `GET /api/users`: List users
- `GET /api/users/{id}`: Get user by ID

**Menu Service**

- `POST /api/menu`: Create menu item
- `GET /api/menu`: List menu items
- `GET /api/menu/{id}`: Get menu item by ID

**Order Service**

- `POST /api/orders`: Create order
- `GET /api/orders`: List orders
- `GET /api/orders/{id}`: Get order by ID

### Example Requests

```bash
# Create a user
curl -X POST http://localhost:8080/api/users \
  -H 'Content-Type: application/json' \
  -d '{"name": "John Doe", "email": "john@example.com", "is_cafe_owner": false}'

# Create a menu item
curl -X POST http://localhost:8080/api/menu \
  -H 'Content-Type: application/json' \
  -d '{"name": "Espresso", "description": "Strong coffee", "price": 3.00}'

# Create an order
curl -X POST http://localhost:8080/api/orders \
  -H 'Content-Type: application/json' \
  -d '{"user_id": 1, "items": [{"menu_item_id": 1, "quantity": 2}]}'

# Get all orders
curl http://localhost:8080/api/orders
```

## Testing

Run tests using the Makefile:

```bash
make test-unit          # Unit tests with in-memory SQLite
make test-integration   # Integration tests with in-memory connections
make test-e2e-docker    # End-to-end tests with Docker
make test-all           # Run all test suites
make test-coverage      # Generate coverage reports (outputs to */coverage.html)
```

## Makefile Commands

| Command                 | Description                                            |
| ----------------------- | ------------------------------------------------------ |
| `make help`             | Display all available commands                         |
| `make dev-setup`        | Install dependencies, generate Protobufs, build images |
| `make proto-generate`   | Generate gRPC and Protobuf code from `.proto` files    |
| `make docker-build`     | Build Docker images for all services                   |
| `make docker-up`        | Start all services                                     |
| `make docker-down`      | Stop and remove containers                             |
| `make docker-logs`      | Tail logs from running services                        |
| `make test-unit`        | Run unit tests                                         |
| `make test-integration` | Run integration tests                                  |
| `make test-e2e-docker`  | Run end-to-end tests                                   |
| `make test-all`         | Run all test suites                                    |
| `make test-coverage`    | Generate HTML coverage reports                         |
| `make clean`            | Stop containers and remove volumes                     |

## Conclusion

This practical exercise demonstrates a comprehensive approach to microservices development and testing. The Student Cafe management system exemplifies modern architectural patterns including service decomposition, containerization, and multi-layered testing strategies. The implementation provides a foundation for understanding distributed system design and the importance of automated testing in maintaining code quality across interconnected services.
