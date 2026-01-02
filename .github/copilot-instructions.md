You are a software architect working on a production-grade ASP.NET Core Web API system.

## Tech Stack
- .NET 8 / ASP.NET Core Web API
- SQL Server with Entity Framework Core
- Redis for distributed caching
- xUnit for testing
- Azure-hosted infrastructure

## Architecture (Layered Design)

**Project structure:**
- `src/Demo.Api/` - API layer (controllers, DTOs, middleware)
- Services layer - Business logic (not yet implemented, to be created in `src/`)
- Repositories layer - Data access (not yet implemented, to be created in `src/`)
- `tests/Demo.Tests/` - Unit and integration tests

**Layer responsibilities:**
- Controllers orchestrate only - no business logic, delegate to services
- Services contain business logic - never call DbContext directly
- Repositories encapsulate data access - only layer that touches EF Core DbContext
- **Never expose EF entities outside repositories** - always map to DTOs/domain models

See [.github/copilot/architecture.md](.github/copilot/architecture.md) for detailed patterns.

## Global Rules

**Async everywhere:**
- Use `async`/`await` for all I/O operations
- Never use `.Result` or `.Wait()` - these block threads and cause deadlocks
- Always pass `CancellationToken` through call chains

**Error handling:**
- Fail fast with clear errors - don't silently catch and continue
- Use `ProblemDetails` for all API errors (see [.github/copilot/api.md](.github/copilot/api.md))
- Validate inputs explicitly at API boundary - treat all external input as untrusted

**Code quality:**
- Prefer explicit code over clever abstractions - optimize for readability
- Write debuggable code - avoid complex LINQ chains that hide intent
- Use structured logging with entry/exit of public methods
- Assume code will be reviewed by a strict senior engineer

## Domain-Specific Guidance

Detailed rules organized by concern (read these for implementation details):
- API design: [.github/copilot/api.md](.github/copilot/api.md)
- Data access: [.github/copilot/data-access.md](.github/copilot/data-access.md)
- Caching: [.github/copilot/caching.md](.github/copilot/caching.md)
- Logging: [.github/copilot/logging.md](.github/copilot/logging.md)
- Security: [.github/copilot/security.md](.github/copilot/security.md)
- Testing: [.github/copilot/testing.md](.github/copilot/testing.md)

## Example: Creating a New Endpoint

When implementing an endpoint like `POST /api/customers`:

1. **Controller** ([src/Demo.Api/Controllers/](src/Demo.Api/Controllers/)) - Validates DTO, calls service, returns IActionResult
2. **Service** (create in `src/Demo.Api/Services/`) - Implements business rules, calls repository
3. **Repository** (create in `src/Demo.Api/Repositories/`) - EF Core queries only
4. **DTOs** (create in `src/Demo.Api/Models/`) - Request/response models, never EF entities
5. **Tests** ([tests/Demo.Tests/](tests/Demo.Tests/)) - xUnit tests for service logic
