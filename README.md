# OrderService

A production-ready order management microservice built with **.NET 10.0** using Clean Architecture, CQRS, JWT authentication, Redis caching, and RabbitMQ event-driven messaging.

---

## Architecture

The solution follows **Clean Architecture** with four layers:

```
OrderService/
├── OrderService.API/              # REST API controllers, middleware, DI setup
├── OrderService.Application/      # CQRS commands, queries, and handlers (MediatR)
├── OrderService.Domain/           # Core entities, repository interfaces
├── OrderService.Infrastructure/   # EF Core, repositories, JWT, messaging
├── docker-compose.yml
└── Dockerfile
```

### Key patterns

- **CQRS** via MediatR — commands and queries are fully separated
- **Repository Pattern** — data access abstracted behind domain interfaces
- **Event-Driven Messaging** — RabbitMQ + MassTransit for async order status updates
- **Distributed Caching** — Redis caches order queries with a 5-minute TTL

---

## Tech Stack

| Concern | Technology |
|---|---|
| Runtime | .NET 10.0 |
| Database | SQL Server (EF Core 10.0.7) |
| Caching | Redis (StackExchange.Redis) |
| Messaging | RabbitMQ + MassTransit 8.1.3 |
| Auth | JWT Bearer (HS256) |
| CQRS | MediatR 14.1.0 |
| Password hashing | BCrypt.Net-Next 4.1.0 |
| API docs | Swagger / Swashbuckle |

---

## Getting Started

### Prerequisites

- [.NET 10 SDK](https://dotnet.microsoft.com/download)
- SQL Server on `localhost:1433`
- Redis on `localhost:6379`
- RabbitMQ on `localhost:5672`

### Run locally

```bash
cd OrderService.API
dotnet run
```

Swagger UI: [http://localhost:5019/swagger](http://localhost:5019/swagger)

### Run with Docker Compose

```bash
docker-compose up
```

This starts SQL Server, RabbitMQ, and the API together.

| Service | URL |
|---|---|
| API | http://localhost:8080 |
| RabbitMQ Management | http://localhost:15672 (guest / guest) |

---

## API Endpoints

### Auth

| Method | Route | Description |
|---|---|---|
| POST | `/api/auth/register` | Register a new user |
| POST | `/api/auth/login` | Login and receive a JWT token |

### Orders (requires JWT)

| Method | Route | Description |
|---|---|---|
| GET | `/api/orders` | Get all orders (Redis cached) |
| GET | `/api/orders/{id}` | Get order by ID |
| POST | `/api/orders` | Create a new order |
| PUT | `/api/orders/{id}` | Update an order |
| DELETE | `/api/orders/{id}` | Delete an order |

---

## Event Flow

1. A `POST /api/orders` request triggers `CreateOrderCommandHandler`.
2. The handler saves the order with status `Pending` and publishes an `OrderCreatedEvent` to RabbitMQ.
3. The Redis cache for all orders is invalidated.
4. `OrderStatusUpdatedConsumer` receives the event and updates the order status to `Processing`.

---

## Domain Entities

**Order**
- `Id`, `CustomerName`, `TotalAmount`, `Status` (`Pending` / `Processing`), `CreatedAt`

**User**
- `Id`, `FirstName`, `LastName`, `Username`, `PasswordHash`, `Role`, `CreatedAt`

---

## Configuration

Key settings in `appsettings.json`:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost,1433;Database=OrderService;User Id=sa;Password=..."
  },
  "Jwt": {
    "Key": "...",
    "Issuer": "...",
    "Audience": "..."
  },
  "Redis": {
    "ConnectionString": "localhost:6379"
  }
}
```

---

## CI/CD

GitHub Actions workflow (`.github/workflows/ci.yml`) runs on every push or pull request to `main`:

1. Restore NuGet packages
2. Build in Release configuration
3. Run tests

---

## Database Migrations

Migrations are managed with EF Core. To apply them locally:

```bash
cd OrderService.API
dotnet ef database update
```