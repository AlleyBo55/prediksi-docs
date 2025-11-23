# Predixi Microservices Architecture

Predixi has been migrated to a microservices architecture to improve scalability, maintainability, and separation of concerns.

## Architecture Overview

The backend consists of the following services:

1.  **API Gateway (`api-backend`)**:
    *   **Role**: Entry point for all client requests (REST/WebSocket).
    *   **Responsibility**: Authentication, Routing, Aggregation (BFF), WebSocket handling.
    *   **Communication**: Calls other services via gRPC.
    *   **Data Access**: Read-only access to some collections for aggregation (e.g., Account Stats), but writes are delegated to services.

2.  **User Service (`user-service`)**:
    *   **Role**: Manages user identities and balances.
    *   **Responsibility**: User CRUD, Balance Updates, Profile Management.
    *   **Port**: 50052 (gRPC).

3.  **Market Service (`market-service`)**:
    *   **Role**: Manages prediction markets and bets.
    *   **Responsibility**: Market Creation, Betting Logic, Market Resolution, Comments.
    *   **Port**: 50053 (gRPC).

4.  **Payment Service (`payment-service`)**:
    *   **Role**: Handles external payment integrations.
    *   **Responsibility**: Deposits (Midtrans, Solana), Withdrawals.
    *   **Port**: 50051 (gRPC).

## Service Communication

*   **Protocol**: gRPC (Google Remote Procedure Call) is used for inter-service communication.
*   **Discovery**: Services are discovered via environment variables (e.g., `USER_SERVICE_ADDR`).

## Database

*   **MongoDB**: Primary database.
*   **Pattern**: Services own their collections, but for transition/performance, some collections are shared (e.g., `transactions` read by Gateway).
    *   `users` -> User Service
    *   `markets`, `bets` -> Market Service
    *   `transactions` -> Payment Service (and Gateway for history)

## Running the Services

### Prerequisites
*   Docker & Docker Compose
*   Go 1.24+ (for local dev)

### Using Docker Compose (Recommended)

To start all services including MongoDB:

```bash
./start-docker.sh
```

This will launch:
*   MongoDB (port 27017)
*   Payment Service
*   User Service
*   Market Service
*   API Gateway (port 8080)

### Running Locally

To run services individually (e.g., for development):

1.  **Start MongoDB**:
    ```bash
    docker-compose up -d mongo
    ```

2.  **Run Services**:
    You can use the helper script:
    ```bash
    ./start-all.sh
    ```
    Or run them manually in separate terminals:
    ```bash
    # Terminal 1
    cd payment-service && go run cmd/server/main.go

    # Terminal 2
    cd user-service && go run cmd/server/main.go

    # Terminal 3
    cd market-service && go run cmd/server/main.go

    # Terminal 4
    cd api-backend && go run cmd/api/main.go
    ```

## Environment Variables

Each service uses a `.env` file or environment variables. Key variables:

*   `MONGODB_URI`: Connection string for MongoDB.
*   `JWT_SECRET`: Secret for signing tokens.
*   `USER_SERVICE_ADDR`: Address of User Service (e.g., `localhost:50052`).
*   `MARKET_SERVICE_ADDR`: Address of Market Service.
*   `PAYMENT_SERVICE_ADDR`: Address of Payment Service.

## Directory Structure

*   `api-backend/`: API Gateway code.
*   `user-service/`: User microservice.
*   `market-service/`: Market microservice.
*   `payment-service/`: Payment microservice.
*   `proto/`: Shared Protocol Buffers definitions (in each service).

## Frontend

The frontend (`frontend/`) connects to the API Gateway at `http://localhost:8080`. No changes are required in the frontend code as the API Gateway maintains the same REST API contract.
