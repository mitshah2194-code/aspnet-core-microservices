# aspnet-core-microservices

I'll create a comprehensive ASP.NET Core Microservices guide as markdown that you can copy and convert to PDF. Let me generate all the topics with notes and C# examples.

# ASP.NET Core Microservices - Complete Guide

## Table of Contents
1. [Microservices – Foundations & Fundamentals](#microservices-foundations--fundamentals)
2. [Introduction to Microservices Architecture](#introduction-to-microservices-architecture)
3. [Microservices Design Principles](#microservices-design-principles)
4. [ASP.NET Core Web API Fundamentals](#aspnet-core-web-api-fundamentals)
5. [Clean Architecture in ASP.NET Core Web API](#clean-architecture-in-aspnet-core-web-api)
6. [Domain-Driven Design in ASP.NET Core Web API](#domain-driven-design-in-aspnet-core-web-api)
7. [Project Setup for Microservices in ASP.NET Core Web API](#project-setup-for-microservices-in-aspnet-core-web-api)
8. [Building User Microservice](#building-user-microservice)
9. [RabbitMQ in Microservices](#rabbitmq-in-microservices)
10. [Data Management Strategies & Saga Pattern](#data-management-strategies--saga-pattern)
11. [API Gateway in Microservices](#api-gateway-in-microservices)
12. [Advanced Patterns: gRPC, CQRS, GraphQL, Circuit Breaker, CORS, OData](#advanced-patterns)

---

### Microservices – Foundations & Fundamentals

Transitioning from a monolithic architecture to **Microservices** in **ASP.NET Core** involves more than just splitting code; it requires a fundamental shift in how you design, deploy, and manage data. In 2026, the ecosystem has matured significantly with tools like **.NET Aspire** making the "plumbing" of microservices much more manageable.

---

## 1. Core Foundations: What are Microservices?

Microservices are an architectural style where an application is composed of small, independent services that communicate over well-defined APIs.

* **Autonomous & Independent:** Each service can be developed, deployed, and scaled without affecting others.
* **Loosely Coupled:** Services interact through contracts (like REST or gRPC) rather than shared code libraries.
* **Technological Diversity:** Since services are isolated, you can use different versions of .NET (or even different languages) for different services.

---

## 2. Why ASP.NET Core?

ASP.NET Core is purpose-built for the cloud and microservices due to several key features:

* **High Performance:** Optimized for high-throughput and low-latency scenarios (essential for service-to-service calls).
* **Cross-Platform:** Runs natively on Linux containers, which is the industry standard for microservices deployment.
* **Dependency Injection (DI):** Built-in DI makes it easy to swap implementations and mock services for testing.
* **Middleware Pipeline:** Allows you to plug in cross-cutting concerns like logging, authentication, and health checks easily.

---

## 3. Foundational Communication Patterns

How services "talk" to each other is the biggest hurdle in microservices.

| Pattern | Technology | Best For |
| --- | --- | --- |
| **Synchronous** | REST, gRPC | Immediate response needed (e.g., checking a user's password). |
| **Asynchronous** | RabbitMQ, Azure Service Bus | Decoupling services and handling high traffic (e.g., "Order Placed" events). |
| **API Gateway** | YARP, Ocelot | Providing a single entry point for clients to route requests to internal services. |

---

## 4. Data Management: The Golden Rule

The most critical rule in microservices is **Database per Service**. Sharing a single database creates "distributed monoliths" and leads to tight coupling.

* **Eventual Consistency:** Instead of ACID transactions across services, you use events to sync data over time.
* **CQRS (Command Query Responsibility Segregation):** Splitting the "write" side from the "read" side of your data to optimize performance.

---

## 5. The Modern .NET Stack (2026)

In the current landscape, **.NET Aspire** has become the foundation for microservices development. It simplifies:

* **Orchestration:** Managing how multiple projects and containers start up during development.
* **Service Discovery:** Automatically letting Service A know where Service B is located.
* **Observability:** Built-in support for **OpenTelemetry**, providing dashboards for logs, traces, and metrics out of the box.

---

## 6. Resilience and DevOps

Microservices fail—it's a matter of when, not if.

* **Retries & Circuit Breakers:** Using libraries like **Polly** to handle transient faults (e.g., a service being temporarily down).
* **Containerization:** Wrapping your ASP.NET Core apps in **Docker** images to ensure they run the same in development as they do in **Kubernetes**.

What specific part of the microservices lifecycle are you most interested in—designing the boundaries of the services, or the technical setup of the communication bus?

[.NET Microservices – Full Course](https://www.youtube.com/watch?v=DgVjEo3OGBI)
This comprehensive guide walks you through building a real-world microservices application from scratch, covering everything from Docker containerization to API Gateways and messaging.

### Introduction to Microservices Architecture

**Notes:**
- Microservices architecture is an approach to developing a single application as a suite of small services
- Each service runs in its own process and communicates with lightweight mechanisms (HTTP/REST, gRPC, message queues)
- Services are organized around business capabilities
- Enables independent deployment, scalability, and technology diversity

**Key Benefits:**
- Independent scaling of services
- Faster deployment cycles
- Technology flexibility
- Better fault isolation
- Improved team autonomy

**Example - Basic Microservice Structure:**

```csharp
// UserService - Microservice 1
[ApiController]
[Route("api/[controller]")]
public class UserController : ControllerBase
{
    private readonly IUserService _userService;
    
    public UserController(IUserService userService)
    {
        _userService = userService;
    }
    
    [HttpGet("{id}")]
    public async Task<ActionResult<UserDto>> GetUserById(int id)
    {
        var user = await _userService.GetUserByIdAsync(id);
        if (user == null)
            return NotFound();
        return Ok(user);
    }
    
    [HttpPost]
    public async Task<ActionResult<UserDto>> CreateUser([FromBody] CreateUserRequest request)
    {
        var user = await _userService.CreateUserAsync(request);
        return CreatedAtAction(nameof(GetUserById), new { id = user.Id }, user);
    }
}

// OrderService - Microservice 2
[ApiController]
[Route("api/[controller]")]
public class OrderController : ControllerBase
{
    private readonly IOrderService _orderService;
    
    public OrderController(IOrderService orderService)
    {
        _orderService = orderService;
    }
    
    [HttpPost]
    public async Task<ActionResult<OrderDto>> CreateOrder([FromBody] CreateOrderRequest request)
    {
        var order = await _orderService.CreateOrderAsync(request);
        return CreatedAtAction(nameof(GetOrderById), new { id = order.Id }, order);
    }
}
```

---

### Microservices Design Principles

**Notes:**
- **Single Responsibility Principle (SRP):** Each microservice should have a single, well-defined responsibility
- **Loose Coupling:** Services should be independent and not tightly dependent on other services
- **High Cohesion:** Related functionality should be grouped together within a service
- **Bounded Contexts:** Use Domain-Driven Design to define clear boundaries
- **Decentralized Data:** Each service manages its own database
- **Resilience:** Design services to handle failures gracefully

**Example - Service Boundaries:**

```csharp
// UserService - User Management Bounded Context
namespace UserService.Domain
{
    public class User
    {
        public int Id { get; set; }
        public string Email { get; set; }
        public string FirstName { get; set; }
        public string LastName { get; set; }
        public DateTime CreatedAt { get; set; }
    }
}

// OrderService - Order Management Bounded Context
namespace OrderService.Domain
{
    public class Order
    {
        public int Id { get; set; }
        public int UserId { get; set; } // Reference to User, not object
        public List<OrderItem> Items { get; set; }
        public decimal TotalAmount { get; set; }
        public OrderStatus Status { get; set; }
        public DateTime CreatedAt { get; set; }
    }
    
    public enum OrderStatus
    {
        Pending,
        Processing,
        Completed,
        Cancelled
    }
}

// Clear separation - each service manages its own data
public interface IUserRepository
{
    Task<User> GetByIdAsync(int id);
    Task AddAsync(User user);
}

public interface IOrderRepository
{
    Task<Order> GetByIdAsync(int id);
    Task AddAsync(Order order);
}
```

---

## ASP.NET Core Web API Fundamentals

**Notes:**
- ASP.NET Core Web API is a framework for building HTTP-based APIs
- RESTful principles guide API design
- Built-in dependency injection and middleware support
- Cross-platform and lightweight
- Supports multiple serialization formats (JSON, XML)

**Example - Basic Web API Setup:**

```csharp
// Program.cs - ASP.NET Core 6+
var builder = WebApplication.CreateBuilder(args);

// Add services to the container
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Add custom services
builder.Services.AddScoped<IUserService, UserService>();
builder.Services.AddScoped<IUserRepository, UserRepository>();

var app = builder.Build();

// Configure the HTTP request pipeline
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();

// Controller Example
[ApiController]
[Route("api/[controller]")]
public class ProductController : ControllerBase
{
    private readonly IProductService _productService;
    
    public ProductController(IProductService productService)
    {
        _productService = productService;
    }
    
    [HttpGet("{id}")]
    public async Task<ActionResult<ProductDto>> GetProduct(int id)
    {
        var product = await _productService.GetProductAsync(id);
        if (product == null)
            return NotFound(new { message = "Product not found" });
        return Ok(product);
    }
    
    [HttpPost]
    public async Task<ActionResult<ProductDto>> CreateProduct([FromBody] CreateProductRequest request)
    {
        if (!ModelState.IsValid)
            return BadRequest(ModelState);
        
        var product = await _productService.CreateProductAsync(request);
        return CreatedAtAction(nameof(GetProduct), new { id = product.Id }, product);
    }
    
    [HttpPut("{id}")]
    public async Task<IActionResult> UpdateProduct(int id, [FromBody] UpdateProductRequest request)
    {
        var result = await _productService.UpdateProductAsync(id, request);
        if (!result)
            return NotFound();
        return NoContent();
    }
    
    [HttpDelete("{id}")]
    public async Task<IActionResult> DeleteProduct(int id)
    {
        var result = await _productService.DeleteProductAsync(id);
        if (!result)
            return NotFound();
        return NoContent();
    }
}
```

---

## Clean Architecture in ASP.NET Core Web API

**Notes:**
- Clean Architecture separates concerns into distinct layers
- Layers: Presentation, Application, Domain, Infrastructure
- Dependencies point inward (Dependency Inversion Principle)
- Easy to test, maintain, and extend
- Business logic is independent of frameworks

**Project Structure:**

```
Solution/
├── Core/
│   ├── Domain/              # Entities, Value Objects, Enums
│   └── Application/         # Use Cases, DTOs, Interfaces
├── Infrastructure/          # Database, External Services, Implementations
├── Presentation/            # Controllers, API Endpoints
└── Tests/                   # Unit Tests, Integration Tests
```

**Example - Clean Architecture Implementation:**

```csharp
// 1. Domain Layer (Core/Domain)
namespace UserService.Core.Domain
{
    public class User
    {
        public int Id { get; private set; }
        public string Email { get; private set; }
        public string FirstName { get; private set; }
        public string LastName { get; private set; }
        public DateTime CreatedAt { get; private set; }
        
        private User() { }
        
        public static User Create(string email, string firstName, string lastName)
        {
            return new User
            {
                Email = email,
                FirstName = firstName,
                LastName = lastName,
                CreatedAt = DateTime.UtcNow
            };
        }
        
        public void UpdateProfile(string firstName, string lastName)
        {
            FirstName = firstName;
            LastName = lastName;
        }
    }
}

// 2. Application Layer (Core/Application)
namespace UserService.Core.Application
{
    // DTO
    public class UserDto
    {
        public int Id { get; set; }
        public string Email { get; set; }
        public string FirstName { get; set; }
        public string LastName { get; set; }
    }
    
    // Interface
    public interface IUserRepository
    {
        Task<User> GetByIdAsync(int id);
        Task<User> GetByEmailAsync(string email);
        Task AddAsync(User user);
        Task UpdateAsync(User user);
        Task DeleteAsync(int id);
    }
    
    // Use Case
    public class CreateUserUseCase
    {
        private readonly IUserRepository _repository;
        
        public CreateUserUseCase(IUserRepository repository)
        {
            _repository = repository;
        }
        
        public async Task<UserDto> ExecuteAsync(string email, string firstName, string lastName)
        {
            var existingUser = await _repository.GetByEmailAsync(email);
            if (existingUser != null)
                throw new InvalidOperationException("User already exists");
            
            var user = User.Create(email, firstName, lastName);
            await _repository.AddAsync(user);
            
            return new UserDto
            {
                Id = user.Id,
                Email = user.Email,
                FirstName = user.FirstName,
                LastName = user.LastName
            };
        }
    }
}

// 3. Infrastructure Layer
namespace UserService.Infrastructure
{
    public class UserRepository : IUserRepository
    {
        private readonly ApplicationDbContext _dbContext;
        
        public UserRepository(ApplicationDbContext dbContext)
        {
            _dbContext = dbContext;
        }
        
        public async Task<User> GetByIdAsync(int id)
        {
            return await _dbContext.Users.FindAsync(id);
        }
        
        public async Task AddAsync(User user)
        {
            _dbContext.Users.Add(user);
            await _dbContext.SaveChangesAsync();
        }
    }
}

// 4. Presentation Layer (Controllers)
[ApiController]
[Route("api/[controller]")]
public class UserController : ControllerBase
{
    private readonly CreateUserUseCase _createUserUseCase;
    
    public UserController(CreateUserUseCase createUserUseCase)
    {
        _createUserUseCase = createUserUseCase;
    }
    
    [HttpPost]
    public async Task<ActionResult<UserDto>> CreateUser([FromBody] CreateUserRequest request)
    {
        var userDto = await _createUserUseCase.ExecuteAsync(
            request.Email, 
            request.FirstName, 
            request.LastName);
        
        return CreatedAtAction(nameof(GetUser), new { id = userDto.Id }, userDto);
    }
}
```

---

## Domain-Driven Design in ASP.NET Core Web API

**Notes:**
- DDD focuses on the core business logic and domain knowledge
- Key concepts: Entities, Value Objects, Aggregates, Bounded Contexts
- Ubiquitous Language: Common terminology between developers and domain experts
- Repository Pattern: Abstracts data access
- Domain Events: Record significant business events

**Example - DDD Implementation:**

```csharp
// Value Object
public class Email : ValueObject
{
    public string Value { get; private set; }
    
    private Email(string value)
    {
        if (string.IsNullOrWhiteSpace(value) || !value.Contains("@"))
            throw new ArgumentException("Invalid email");
        Value = value;
    }
    
    public static Email Create(string value) => new Email(value);
    
    protected override IEnumerable<object> GetAtomicValues()
    {
        yield return Value;
    }
}

// Entity with Domain Logic
public class Order : AggregateRoot
{
    public int Id { get; private set; }
    public int UserId { get; private set; }
    public List<OrderItem> Items { get; private set; } = new();
    public decimal TotalAmount { get; private set; }
    public OrderStatus Status { get; private set; }
    public DateTime CreatedAt { get; private set; }
    
    private Order() { }
    
    public static Order Create(int userId)
    {
        return new Order
        {
            UserId = userId,
            Status = OrderStatus.Pending,
            CreatedAt = DateTime.UtcNow
        };
    }
    
    public void AddItem(Product product, int quantity)
    {
        if (quantity <= 0)
            throw new InvalidOperationException("Quantity must be greater than 0");
        
        var item = new OrderItem 
        { 
            ProductId = product.Id, 
            Quantity = quantity, 
            UnitPrice = product.Price 
        };
        
        Items.Add(item);
        RecalculateTotal();
        
        // Raise Domain Event
        AddDomainEvent(new OrderItemAddedDomainEvent(this.Id, product.Id, quantity));
    }
    
    public void PlaceOrder()
    {
        if (Items.Count == 0)
            throw new InvalidOperationException("Order must contain items");
        
        Status = OrderStatus.Processing;
        AddDomainEvent(new OrderPlacedDomainEvent(this.Id, this.UserId, this.TotalAmount));
    }
    
    private void RecalculateTotal()
    {
        TotalAmount = Items.Sum(i => i.UnitPrice * i.Quantity);
    }
}

// Domain Event
public class OrderPlacedDomainEvent : IDomainEvent
{
    public int OrderId { get; }
    public int UserId { get; }
    public decimal TotalAmount { get; }
    public DateTime OccurredAt { get; }
    
    public OrderPlacedDomainEvent(int orderId, int userId, decimal totalAmount)
    {
        OrderId = orderId;
        UserId = userId;
        TotalAmount = totalAmount;
        OccurredAt = DateTime.UtcNow;
    }
}

// Repository Interface (abstraction)
public interface IOrderRepository
{
    Task<Order> GetByIdAsync(int id);
    Task AddAsync(Order order);
    Task UpdateAsync(Order order);
    Task<List<Order>> GetUserOrdersAsync(int userId);
}
```

---

## Project Setup for Microservices in ASP.NET Core Web API

**Notes:**
- Create separate projects for each service
- Use solution structure to organize microservices
- Configure dependency injection in each service
- Set up logging and configuration
- Use Docker for containerization

**Example - Project Structure and Setup:**

```csharp
// Directory Structure
Solution/
├── UserService/
│   ├── UserService.Api/
│   ├── UserService.Core/
│   ├── UserService.Infrastructure/
│   └── UserService.Tests/
├── OrderService/
│   ├── OrderService.Api/
│   ├── OrderService.Core/
│   ├── OrderService.Infrastructure/
│   └── OrderService.Tests/
├── ProductService/
│   └── ...
└── Shared/
    └── Shared.Messages/

// UserService/UserService.Api/Program.cs
var builder = WebApplication.CreateBuilder(args);

// Add services
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Add DbContext
builder.Services.AddDbContext<UserServiceDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

// Add repositories and services
builder.Services.AddScoped<IUserRepository, UserRepository>();
builder.Services.AddScoped<IUserService, UserService.Core.Application.UserService>();

// Add logging
builder.Services.AddLogging(config =>
{
    config.AddConsole();
    config.AddDebug();
});

// CORS configuration
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowAll", builder =>
    {
        builder.AllowAnyOrigin()
               .AllowAnyMethod()
               .AllowAnyHeader();
    });
});

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseCors("AllowAll");
app.UseAuthorization();
app.MapControllers();

// Health check endpoint
app.MapHealthChecks("/health");

app.Run();

// appsettings.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=UserServiceDb;Trusted_Connection=true;"
  },
  "ServiceUrls": {
    "UserService": "https://localhost:5001",
    "OrderService": "https://localhost:5002",
    "ProductService": "https://localhost:5003"
  },
  "RabbitMQ": {
    "HostName": "localhost",
    "Port": 5672,
    "UserName": "guest",
    "Password": "guest"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information"
    }
  }
}

// Dockerfile for containerization
FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src
COPY ["UserService/UserService.Api/UserService.Api.csproj", "UserService.Api/"]
RUN dotnet restore "UserService.Api/UserService.Api.csproj"
COPY . .
RUN dotnet build "UserService.Api/UserService.Api.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "UserService.Api/UserService.Api.csproj" -c Release -o /app/publish

FROM mcr.microsoft.com/dotnet/aspnet:6.0
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "UserService.Api.dll"]
```

---

## Building User Microservice

### User Microservice with ASP.NET Core Web API

**Notes:**
- User microservice handles user management operations
- Implements CRUD operations for users
- Manages user authentication and profile data
- Publishes events when users are created/updated
- Independently deployable and scalable

**Example - User Service Overview:**

```csharp
// UserService.Core/Domain/User.cs
namespace UserService.Core.Domain
{
    public class User : AggregateRoot
    {
        public int Id { get; private set; }
        public string Email { get; private set; }
        public string FirstName { get; private set; }
        public string LastName { get; private set; }
        public string PasswordHash { get; private set; }
        public bool IsActive { get; private set; }
        public DateTime CreatedAt { get; private set; }
        public DateTime? UpdatedAt { get; private set; }
        
        private User() { }
        
        public static User Create(string email, string firstName, string lastName, string password)
        {
            if (string.IsNullOrWhiteSpace(email))
                throw new ArgumentException("Email is required");
            
            return new User
            {
                Email = email,
                FirstName = firstName,
                LastName = lastName,
                PasswordHash = HashPassword(password),
                IsActive = true,
                CreatedAt = DateTime.UtcNow
            };
        }
        
        public void Update(string firstName, string lastName)
        {
            FirstName = firstName;
            LastName = lastName;
            UpdatedAt = DateTime.UtcNow;
        }
        
        public void Deactivate()
        {
            IsActive = false;
            UpdatedAt = DateTime.UtcNow;
        }
        
        private static string HashPassword(string password)
        {
            return BCrypt.Net.BCrypt.HashPassword(password);
        }
    }
}
```

---

### Implementing User Microservice Domain Layer

**Notes:**
- Domain layer contains business entities and logic
- No external dependencies (Framework agnostic)
- Value Objects represent domain concepts
- Aggregate Roots manage entity clusters
- Domain Events capture business events

**Example:**

```csharp
// UserService.Core/Domain/ValueObjects.cs
public class Email : ValueObject
{
    public string Value { get; }
    
    private Email(string value)
    {
        if (!IsValidEmail(value))
            throw new ArgumentException("Invalid email format");
        Value = value;
    }
    
    public static Email Create(string value) => new Email(value);
    
    private static bool IsValidEmail(string email)
    {
        try
        {
            var addr = new System.Net.Mail.MailAddress(email);
            return addr.Address == email;
        }
        catch
        {
            return false;
        }
    }
    
    protected override IEnumerable<object> GetAtomicValues()
    {
        yield return Value;
    }
}

// UserService.Core/Domain/Events/UserCreatedDomainEvent.cs
public class UserCreatedDomainEvent : IDomainEvent
{
    public int UserId { get; }
    public string Email { get; }
    public string FirstName { get; }
    public string LastName { get; }
    public DateTime OccurredAt { get; }
    
    public UserCreatedDomainEvent(int userId, string email, string firstName, string lastName)
    {
        UserId = userId;
        Email = email;
        FirstName = firstName;
        LastName = lastName;
        OccurredAt = DateTime.UtcNow;
    }
}
```

---

### Implementing User Microservice Infrastructure Layer

**Notes:**
- Infrastructure layer handles data persistence
- Implements repository interfaces
- Contains database context and migrations
- Handles external service integrations
- Manages dependency implementations

**Example:**

```csharp
// UserService.Infrastructure/Data/UserServiceDbContext.cs
public class UserServiceDbContext : DbContext
{
    public DbSet<User> Users { get; set; }
    
    public UserServiceDbContext(DbContextOptions<UserServiceDbContext> options) 
        : base(options)
    {
    }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        
        // Configure User entity
        modelBuilder.Entity<User>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Email).IsRequired().HasMaxLength(256);
            entity.Property(e => e.FirstName).IsRequired().HasMaxLength(100);
            entity.Property(e => e.LastName).IsRequired().HasMaxLength(100);
            entity.Property(e => e.PasswordHash).IsRequired();
            entity.Property(e => e.CreatedAt).IsRequired();
            
            entity.HasIndex(e => e.Email).IsUnique();
        });
    }
}

// UserService.Infrastructure/Repositories/UserRepository.cs
public class UserRepository : IUserRepository
{
    private readonly UserServiceDbContext _dbContext;
    
    public UserRepository(UserServiceDbContext dbContext)
    {
        _dbContext = dbContext;
    }
    
    public async Task<User> GetByIdAsync(int id)
    {
        return await _dbContext.Users
            .AsNoTracking()
            .FirstOrDefaultAsync(u => u.Id == id);
    }
    
    public async Task<User> GetByEmailAsync(string email)
    {
        return await _dbContext.Users
            .AsNoTracking()
            .FirstOrDefaultAsync(u => u.Email == email);
    }
    
    public async Task AddAsync(User user)
    {
        await _dbContext.Users.AddAsync(user);
        await _dbContext.SaveChangesAsync();
    }
    
    public async Task UpdateAsync(User user)
    {
        _dbContext.Users.Update(user);
        await _dbContext.SaveChangesAsync();
    }
    
    public async Task<List<User>> GetAllAsync()
    {
        return await _dbContext.Users
            .AsNoTracking()
            .Where(u => u.IsActive)
            .ToListAsync();
    }
}
```

---

### Implementing User Microservice Application Layer

**Notes:**
- Application layer contains use cases and business logic coordination
- Uses repositories to access data
- Publishes domain events
- Contains DTOs for API communication
- Implements application services

**Example:**

```csharp
// UserService.Core/Application/Dtos/UserDto.cs
public class UserDto
{
    public int Id { get; set; }
    public string Email { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public DateTime CreatedAt { get; set; }
}

// UserService.Core/Application/Services/IUserService.cs
public interface IUserService
{
    Task<UserDto> CreateUserAsync(CreateUserRequest request);
    Task<UserDto> GetUserByIdAsync(int id);
    Task<UserDto> UpdateUserAsync(int id, UpdateUserRequest request);
    Task DeleteUserAsync(int id);
    Task<List<UserDto>> GetAllUsersAsync();
}

// UserService.Core/Application/Services/UserService.cs
public class UserService : IUserService
{
    private readonly IUserRepository _repository;
    private readonly IEventBus _eventBus;
    private readonly IMapper _mapper;
    
    public UserService(IUserRepository repository, IEventBus eventBus, IMapper mapper)
    {
        _repository = repository;
        _eventBus = eventBus;
        _mapper = mapper;
    }
    
    public async Task<UserDto> CreateUserAsync(CreateUserRequest request)
    {
        // Check if user already exists
        var existingUser = await _repository.GetByEmailAsync(request.Email);
        if (existingUser != null)
            throw new InvalidOperationException("User with this email already exists");
        
        // Create user
        var user = User.Create(request.Email, request.FirstName, request.LastName, request.Password);
        
        // Add domain event
        var @event = new UserCreatedDomainEvent(user.Id, user.Email, user.FirstName, user.LastName);
        user.AddDomainEvent(@event);
        
        // Save user
        await _repository.AddAsync(user);
        
        // Publish event
        await _eventBus.PublishAsync(@event);
        
        return _mapper.Map<UserDto>(user);
    }
    
    public async Task<UserDto> GetUserByIdAsync(int id)
    {
        var user = await _repository.GetByIdAsync(id);
        if (user == null)
            throw new KeyNotFoundException($"User with id {id} not found");
        
        return _mapper.Map<UserDto>(user);
    }
    
    public async Task<UserDto> UpdateUserAsync(int id, UpdateUserRequest request)
    {
        var user = await _repository.GetByIdAsync(id);
        if (user == null)
            throw new KeyNotFoundException($"User with id {id} not found");
        
        user.Update(request.FirstName, request.LastName);
        await _repository.UpdateAsync(user);
        
        return _mapper.Map<UserDto>(user);
    }
    
    public async Task DeleteUserAsync(int id)
    {
        var user = await _repository.GetByIdAsync(id);
        if (user == null)
            throw new KeyNotFoundException($"User with id {id} not found");
        
        user.Deactivate();
        await _repository.UpdateAsync(user);
    }
    
    public async Task<List<UserDto>> GetAllUsersAsync()
    {
        var users = await _repository.GetAllAsync();
        return _mapper.Map<List<UserDto>>(users);
    }
}
```

---

### Implementing User Microservice API Layer

**Notes:**
- API layer exposes HTTP endpoints
- Handles request validation
- Returns appropriate HTTP status codes
- Maps DTOs for API contracts
- Includes documentation with Swagger

**Example:**

```csharp
// UserService.Api/Controllers/UserController.cs
[ApiController]
[Route("api/[controller]")]
[Produces("application/json")]
public class UserController : ControllerBase
{
    private readonly IUserService _userService;
    private readonly ILogger<UserController> _logger;
    
    public UserController(IUserService userService, ILogger<UserController> logger)
    {
        _userService = userService;
        _logger = logger;
    }
    
    /// <summary>
    /// Get user by ID
    /// </summary>
    [HttpGet("{id}")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<UserDto>> GetUserById(int id)
    {
        try
        {
            var user = await _userService.GetUserByIdAsync(id);
            return Ok(user);
        }
        catch (KeyNotFoundException ex)
        {
            _logger.LogWarning($"User not found: {ex.Message}");
            return NotFound(new { message = ex.Message });
        }
    }
    
    /// <summary>
    /// Get all users
    /// </summary>
    [HttpGet]
    [ProducesResponseType(StatusCodes.Status200OK)]
    public async Task<ActionResult<List<UserDto>>> GetAllUsers()
    {
        var users = await _userService.GetAllUsersAsync();
        return Ok(users);
    }
    
    /// <summary>
    /// Create new user
    /// </summary>
    [HttpPost]
    [ProducesResponseType(StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<ActionResult<UserDto>> CreateUser([FromBody] CreateUserRequest request)
    {
        if (!ModelState.IsValid)
            return BadRequest(ModelState);
        
        try
        {
            var user = await _userService.CreateUserAsync(request);
            _logger.LogInformation($"User created: {user.Email}");
            return CreatedAtAction(nameof(GetUserById), new { id = user.Id }, user);
        }
        catch (InvalidOperationException ex)
        {
            _logger.LogWarning($"User creation failed: {ex.Message}");
            return BadRequest(new { message = ex.Message });
        }
    }
    
    /// <summary>
    /// Update user
    /// </summary>
    [HttpPut("{id}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> UpdateUser(int id, [FromBody] UpdateUserRequest request)
    {
        if (!ModelState.IsValid)
            return BadRequest(ModelState);
        
        try
        {
            await _userService.UpdateUserAsync(id, request);
            _logger.LogInformation($"User updated: {id}");
            return NoContent();
        }
        catch (KeyNotFoundException ex)
        {
            return NotFound(new { message = ex.Message });
        }
    }
    
    /// <summary>
    /// Delete user
    /// </summary>
    [HttpDelete("{id}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> DeleteUser(int id)
    {
        try
        {
            await _userService.DeleteUserAsync(id);
            _logger.LogInformation($"User deleted: {id}");
            return NoContent();
        }
        catch (KeyNotFoundException ex)
        {
            return NotFound(new { message = ex.Message });
        }
    }
}

// UserService.Api/Requests/CreateUserRequest.cs
public class CreateUserRequest
{
    [Required(ErrorMessage = "Email is required")]
    [EmailAddress(ErrorMessage = "Invalid email format")]
    public string Email { get; set; }
    
    [Required(ErrorMessage = "FirstName is required")]
    [StringLength(100)]
    public string FirstName { get; set; }
    
    [Required(ErrorMessage = "LastName is required")]
    [StringLength(100)]
    public string LastName { get; set; }
    
    [Required(ErrorMessage = "Password is required")]
    [StringLength(255, MinimumLength = 6, ErrorMessage = "Password must be between 6 and 255 characters")]
    public string Password { get; set; }
}

public class UpdateUserRequest
{
    [Required]
    [StringLength(100)]
    public string FirstName { get; set; }
    
    [Required]
    [StringLength(100)]
    public string LastName { get; set; }
}
```

---

## RabbitMQ in Microservices

### Inter-Service Communication in Microservices

**Notes:**
- Microservices need to communicate efficiently
- Two main patterns: Synchronous (REST, gRPC) and Asynchronous (Message Queue)
- Message-based communication decouples services
- Enables eventual consistency
- Improves system resilience

**Comparison:**

| Pattern | Use Case | Benefits | Challenges |
|---------|----------|----------|------------|
| Synchronous (REST) | Real-time responses needed | Simple, immediate results | Tight coupling, cascading failures |
| Asynchronous (Messaging) | Fire-and-forget operations | Loose coupling, resilience | Complexity, eventual consistency |

---

### RabbitMQ for Asynchronous Messaging in Microservices

**Notes:**
- RabbitMQ is a message broker that implements AMQP protocol
- Supports publish-subscribe and queue patterns
- Ensures message delivery reliability
- Provides persistent message storage
- Enables service decoupling

**Key Concepts:**
- **Producer:** Service that sends messages
- **Consumer:** Service that receives and processes messages
- **Exchange:** Routes messages to queues
- **Queue:** Stores messages temporarily
- **Binding:** Connection between exchange and queue

---

### Download, Install, and Configure RabbitMQ Locally on Windows

**Steps:**

1. **Install Erlang Runtime**
   - Download from erlang.org
   - Install with default settings

2. **Install RabbitMQ Server**
   - Download from rabbitmq.com
   - Run installer
   - RabbitMQ runs as Windows Service

3. **Enable Management Plugin**
   ```bash
   cd "C:\Program Files\RabbitMQ Server\rabbitmq_server-3.x.x\sbin"
   rabbitmq-plugins enable rabbitmq_management
   ```

4. **Access Management UI**
   - URL: `http://localhost:15672`
   - Default credentials: guest/guest

---

### RabbitMQ End-to-End Test using Management UI

**Steps:**

1. Open Management UI at `http://localhost:15672`
2. Login with guest/guest
3. Go to "Queues" tab
4. Create new queue (e.g., "user.created")
5. Go to "Exchanges" tab
6. Create direct exchange (e.g., "user.events")
7. Bind queue to exchange with routing key
8. Use "Publish message" to test

---

### RabbitMQ Integration Steps

**Notes:**
- Install RabbitMQ NuGet package
- Configure connection settings
- Create publishers and consumers
- Handle message serialization
- Implement error handling

**Example - Basic Setup:**

```csharp
// Install NuGet: RabbitMQ.Client

// Program.cs - Producer Service
using RabbitMQ.Client;
using RabbitMQ.Client.Events;

var builder = WebApplication.CreateBuilder(args);

// Register RabbitMQ connection
builder.Services.AddSingleton<IConnection>(sp =>
{
    var factory = new ConnectionFactory()
    {
        HostName = "localhost",
        UserName = "guest",
        Password = "guest",
        Port = 5672,
        DispatchConsumersAsync = true
    };
    return factory.CreateConnection();
});

builder.Services.AddSingleton<IModel>(sp =>
{
    var connection = sp.GetRequiredService<IConnection>();
    return connection.CreateModel();
});

// Register publisher service
builder.Services.AddScoped<IMessagePublisher, RabbitMQPublisher>();

var app = builder.Build();
app.Run();

// Publisher Implementation
public interface IMessagePublisher
{
    Task PublishAsync<T>(T message, string exchangeName, string routingKey);
}

public class RabbitMQPublisher : IMessagePublisher
{
    private readonly IModel _channel;
    
    public RabbitMQPublisher(IModel channel)
    {
        _channel = channel;
    }
    
    public async Task PublishAsync<T>(T message, string exchangeName, string routingKey)
    {
        // Declare exchange
        _channel.ExchangeDeclare(exchangeName, ExchangeType.Direct, durable: true);
        
        // Serialize message
        var json = JsonSerializer.Serialize(message);
        var body = Encoding.UTF8.GetBytes(json);
        
        // Publish message
        var properties = _channel.CreateBasicProperties();
        properties.Persistent = true;
        
        _channel.BasicPublish(exchangeName, routingKey, properties, body);
        await Task.CompletedTask;
    }
}

// Consumer Implementation
public interface IMessageConsumer
{
    Task StartConsumingAsync<T>(string queueName, Func<T, Task> handler);
}

public class RabbitMQConsumer : IMessageConsumer
{
    private readonly IModel _channel;
    
    public RabbitMQConsumer(IModel channel)
    {
        _channel = channel;
    }
    
    public async Task StartConsumingAsync<T>(string queueName, Func<T, Task> handler)
    {
        _channel.QueueDeclare(queueName, durable: true, exclusive: false);
        
        var consumer = new AsyncEventingBasicConsumer(_channel);
        
        consumer.Received += async (model, ea) =>
        {
            try
            {
                var body = ea.Body.ToArray();
                var json = Encoding.UTF8.GetString(body);
                var message = JsonSerializer.Deserialize<T>(json);
                
                await handler(message);
                
                _channel.BasicAck(ea.DeliveryTag, false);
            }
            catch (Exception ex)
            {
                Console.WriteLine($"Error processing message: {ex.Message}");
                _channel.BasicNack(ea.DeliveryTag, false, true);
            }
        };
        
        _channel.BasicConsume(queueName, false, consumer);
        await Task.CompletedTask;
    }
}
```

---

### Integrating RabbitMQ in ASP.NET Core Web API

**Example - Complete Integration:**

```csharp
// Shared.Messages/Events/UserCreatedEvent.cs
namespace Shared.Messages.Events
{
    public class UserCreatedEvent
    {
        public int UserId { get; set; }
        public string Email { get; set; }
        public string FirstName { get; set; }
        public string LastName { get; set; }
        public DateTime CreatedAt { get; set; }
    }
}

// UserService - Publisher
[ApiController]
[Route("api/[controller]")]
public class UserController : ControllerBase
{
    private readonly IUserService _userService;
    private readonly IMessagePublisher _messagePublisher;
    
    public UserController(IUserService userService, IMessagePublisher messagePublisher)
    {
        _userService = userService;
        _messagePublisher = messagePublisher;
    }
    
    [HttpPost]
    public async Task<ActionResult<UserDto>> CreateUser([FromBody] CreateUserRequest request)
    {
        var user = await _userService.CreateUserAsync(request);
        
        // Publish event
        var @event = new UserCreatedEvent
        {
            UserId = user.Id,
            Email = user.Email,
            FirstName = user.FirstName,
            LastName = user.LastName,
            CreatedAt = DateTime.UtcNow
        };
        
        await _messagePublisher.PublishAsync(@event, "user.events", "user.created");
        
        return CreatedAtAction(nameof(GetUserById), new { id = user.Id }, user);
    }
}

// OrderService - Consumer
public class UserCreatedEventHandler
{
    private readonly IOrderService _orderService;
    
    public UserCreatedEventHandler(IOrderService orderService)
    {
        _orderService = orderService;
    }
    
    public async Task HandleAsync(UserCreatedEvent @event)
    {
        // Create user profile in OrderService
        await _orderService.CreateUserProfileAsync(@event.UserId, @event.Email);
    }
}

// Startup configuration
var builder = WebApplication.CreateBuilder(args);

// RabbitMQ Setup
builder.Services.AddSingleton<IConnection>(sp =>
{
    var factory = new ConnectionFactory()
    {
        HostName = builder.Configuration["RabbitMQ:HostName"],
        UserName = builder.Configuration["RabbitMQ:UserName"],
        Password = builder.Configuration["RabbitMQ:Password"]
    };
    return factory.CreateConnection();
});

builder.Services.AddScoped<IModel>(sp =>
{
    var connection = sp.GetRequiredService<IConnection>();
    return connection.CreateModel();
});

builder.Services.AddScoped<IMessagePublisher, RabbitMQPublisher>();
builder.Services.AddScoped<IMessageConsumer, RabbitMQConsumer>();
builder.Services.AddScoped<UserCreatedEventHandler>();

var app = builder.Build();

// Start consumer in background
var consumer = app.Services.GetRequiredService<IMessageConsumer>();
var handler = app.Services.GetRequiredService<UserCreatedEventHandler>();

_ = Task.Run(() => consumer.StartConsumingAsync<UserCreatedEvent>(
    "order.user-created-queue",
    handler.HandleAsync));

app.Run();
```

---

### End-to-end Order Placed Communication using RabbitMQ

**Example - Multi-Service Message Flow:**

```csharp
// Shared.Messages/Events/OrderPlacedEvent.cs
public class OrderPlacedEvent
{
    public int OrderId { get; set; }
    public int UserId { get; set; }
    public List<OrderItemDto> Items { get; set; }
    public decimal TotalAmount { get; set; }
    public DateTime PlacedAt { get; set; }
}

// OrderService - Publisher
[ApiController]
[Route("api/[controller]")]
public class OrderController : ControllerBase
{
    private readonly IOrderService _orderService;
    private readonly IMessagePublisher _messagePublisher;
    
    public OrderController(IOrderService orderService, IMessagePublisher messagePublisher)
    {
        _orderService = orderService;
        _messagePublisher = messagePublisher;
    }
    
    [HttpPost("{userId}/orders")]
    public async Task<ActionResult<OrderDto>> PlaceOrder(int userId, [FromBody] CreateOrderRequest request)
    {
        var order = await _orderService.CreateOrderAsync(userId, request);
        
        // Publish OrderPlaced event
        var @event = new OrderPlacedEvent
        {
            OrderId = order.Id,
            UserId = order.UserId,
            Items = order.Items.Select(i => new OrderItemDto { ProductId = i.ProductId, Quantity = i.Quantity }).ToList(),
            TotalAmount = order.TotalAmount,
            PlacedAt = DateTime.UtcNow
        };
        
        await _messagePublisher.PublishAsync(@event, "order.events", "order.placed");
        
        return CreatedAtAction(nameof(GetOrderById), new { id = order.Id }, order);
    }
}

// ProductService - Consumer (Update inventory)
public class OrderPlacedEventHandler
{
    private readonly IProductService _productService;
    private readonly IMessagePublisher _messagePublisher;
    
    public OrderPlacedEventHandler(IProductService productService, IMessagePublisher messagePublisher)
    {
        _productService = productService;
        _messagePublisher = messagePublisher;
    }
    
    public async Task HandleAsync(OrderPlacedEvent @event)
    {
        try
        {
            // Reserve inventory
            foreach (var item in @event.Items)
            {
                await _productService.ReserveInventoryAsync(item.ProductId, item.Quantity);
            }
            
            // Publish InventoryReservedEvent
            var inventoryReservedEvent = new InventoryReservedEvent
            {
                OrderId = @event.OrderId,
                UserId = @event.UserId,
                Items = @event.Items,
                ReservedAt = DateTime.UtcNow
            };
            
            await _messagePublisher.PublishAsync(inventoryReservedEvent, "product.events", "inventory.reserved");
        }
        catch (Exception ex)
        {
            // Publish OrderRejectedEvent
            var rejectedEvent = new OrderRejectedEvent
            {
                OrderId = @event.OrderId,
                Reason = ex.Message,
                RejectedAt = DateTime.UtcNow
            };
            
            await _messagePublisher.PublishAsync(rejectedEvent, "order.events", "order.rejected");
        }
    }
}

// NotificationService - Consumer (Send confirmation)
public class InventoryReservedEventHandler
{
    private readonly INotificationService _notificationService;
    
    public InventoryReservedEventHandler(INotificationService notificationService)
    {
        _notificationService = notificationService;
    }
    
    public async Task HandleAsync(InventoryReservedEvent @event)
    {
        // Send order confirmation email
        await _notificationService.SendOrderConfirmationAsync(@event.UserId, @event.OrderId);
    }
}
```

---

## Data Management Strategies & Saga Pattern

### Data Management Strategies in Microservices

**Notes:**
- Each microservice maintains its own database
- Prevents tight coupling through shared databases
- Challenges: Distributed transactions, data consistency
- Solutions: Saga Pattern, Event Sourcing, CQRS

**Strategies:**

1. **Database per Service**
   - Each service has isolated database
   - Full autonomy and scalability
   - Requires event-driven communication

2. **Shared Database** (Not recommended)
   - Services share single database
   - Easy consistency
   - Tight coupling risk

3. **Event Sourcing**
   - Store all state changes as events
   - Replay events to rebuild state
   - Complete audit trail

**Example - Database per Service:**

```csharp
// UserService uses SQL Server
public class UserServiceDbContext : DbContext
{
    public DbSet<User> Users { get; set; }
    
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer("Server=localhost;Database=UserServiceDb;");
    }
}

// OrderService uses separate SQL Server
public class OrderServiceDbContext : DbContext
{
    public DbSet<Order> Orders { get; set; }
    
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer("Server=localhost;Database=OrderServiceDb;");
    }
}

// Each service manages its own data
public class UserRepository : IUserRepository
{
    private readonly UserServiceDbContext _context;
    
    public async Task AddAsync(User user)
    {
        _context.Users.Add(user);
        await _context.SaveChangesAsync();
    }
}

public class OrderRepository : IOrderRepository
{
    private readonly OrderServiceDbContext _context;
    
    public async Task AddAsync(Order order)
    {
        _context.Orders.Add(order);
        await _context.SaveChangesAsync();
    }
}
```

---

### Saga Pattern in Microservices

**Notes:**
- Saga is a sequence of local transactions across services
- Maintains data consistency without distributed transactions
- Two types: Choreography and Orchestration
- Each step has a compensating transaction for rollback
- Handles long-running business processes

**Choreography vs Orchestration:**

| Aspect | Choreography | Orchestration |
|--------|--------------|---------------|
| Control | Distributed (event-driven) | Centralized (coordinator) |
| Complexity | Simple communication | Complex orchestrator logic |
| Monitoring | Hard to track | Easy to track |
| Testing | Difficult | Easier |

---

### Define Shared Messaging Infrastructure for Saga Pattern

**Example:**

```csharp
// Shared.Messages/Saga/SagaStep.cs
namespace Shared.Messages.Saga
{
    public abstract class SagaStep
    {
        public string SagaId { get; set; }
        public string CorrelationId { get; set; }
        public int StepNumber { get; set; }
        public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    }
}

// Shared.Messages/Events/OrderCreatedForSagaEvent.cs
public class OrderCreatedForSagaEvent : SagaStep
{
    public int OrderId { get; set; }
    public int UserId { get; set; }
    public List<OrderItem> Items { get; set; }
    public decimal TotalAmount { get; set; }
}

// Shared.Messages/Commands/ReserveInventoryCommand.cs
public class ReserveInventoryCommand : SagaStep
{
    public int OrderId { get; set; }
    public List<OrderItem> Items { get; set; }
}

// Shared.Messages/Events/InventoryReservedEvent.cs
public class InventoryReservedEvent : SagaStep
{
    public int OrderId { get; set; }
    public bool Success { get; set; }
    public string Message { get; set; }
}

// Shared.Messages/Commands/ProcessPaymentCommand.cs
public class ProcessPaymentCommand : SagaStep
{
    public int OrderId { get; set; }
    public int UserId { get; set; }
    public decimal Amount { get; set; }
}

// Shared.Messages/Events/PaymentProcessedEvent.cs
public class PaymentProcessedEvent : SagaStep
{
    public int OrderId { get; set; }
    public bool Success { get; set; }
    public string TransactionId { get; set; }
}
```

---

### Implementing Saga Pattern in OrderService

**Example - Order Creation Saga:**

```csharp
// OrderService/Sagas/CreateOrderSaga.cs
public class CreateOrderSaga
{
    private readonly IMessagePublisher _messagePublisher;
    private readonly IOrderRepository _orderRepository;
    private readonly ILogger<CreateOrderSaga> _logger;
    
    public CreateOrderSaga(IMessagePublisher messagePublisher, IOrderRepository orderRepository, ILogger<CreateOrderSaga> logger)
    {
        _messagePublisher = messagePublisher;
        _orderRepository = orderRepository;
        _logger = logger;
    }
    
    public async Task StartAsync(int userId, List<OrderItem> items, decimal totalAmount)
    {
        var orderId = Guid.NewGuid().ToString();
        var sagaId = Guid.NewGuid().ToString();
        var correlationId = Guid.NewGuid().ToString();
        
        _logger.LogInformation($"Starting CreateOrderSaga for OrderId: {orderId}");
        
        try
        {
            // Step 1: Create Order
            var order = new Order
            {
                Id = int.Parse(orderId),
                UserId = userId,
                Items = items,
                TotalAmount = totalAmount,
                Status = OrderStatus.Pending
            };
            
            await _orderRepository.AddAsync(order);
            
            // Step 2: Publish ReserveInventoryCommand
            var reserveCommand = new ReserveInventoryCommand
            {
                SagaId = sagaId,
                CorrelationId = correlationId,
                StepNumber = 1,
                OrderId = int.Parse(orderId),
                Items = items
            };
            
            await _messagePublisher.PublishAsync(reserveCommand, "product.commands", "reserve.inventory");
        }
        catch (Exception ex)
        {
            _logger.LogError($"Error in CreateOrderSaga: {ex.Message}");
            throw;
        }
    }
}

// OrderService/EventHandlers/InventoryReservedEventHandler.cs
public class InventoryReservedEventHandler
{
    private readonly IMessagePublisher _messagePublisher;
    private readonly IOrderRepository _orderRepository;
    private readonly ILogger<InventoryReservedEventHandler> _logger;
    
    public InventoryReservedEventHandler(IMessagePublisher messagePublisher, IOrderRepository orderRepository, ILogger<InventoryReservedEventHandler> logger)
    {
        _messagePublisher = messagePublisher;
        _orderRepository = orderRepository;
        _logger = logger;
    }
    
    public async Task HandleAsync(InventoryReservedEvent @event)
    {
        _logger.LogInformation($"Handling InventoryReservedEvent for OrderId: {@event.OrderId}");
        
        if (@event.Success)
        {
            // Inventory reserved successfully
            // Step 3: Publish ProcessPaymentCommand
            var paymentCommand = new ProcessPaymentCommand
            {
                SagaId = @event.SagaId,
                CorrelationId = @event.CorrelationId,
                StepNumber = 2,
                OrderId = @event.OrderId,
                UserId = 0, // Get from context
                Amount = 0 // Get from context
            };
            
            await _messagePublisher.PublishAsync(paymentCommand, "payment.commands", "process.payment");
        }
        else
        {
            // Inventory reservation failed - compensate
            var order = await _orderRepository.GetByIdAsync(@event.OrderId);
            order.Status = OrderStatus.Cancelled;
            await _orderRepository.UpdateAsync(order);
            
            _logger.LogWarning($"Order {@event.OrderId} cancelled due to inventory reservation failure");
        }
    }
}
```

---

### Implementing OrchestratorService for Saga Pattern

**Example - Central Orchestrator:**

```csharp
// OrchestratorService/Sagas/OrderSagaOrchestrator.cs
public class OrderSagaOrchestrator
{
    private readonly IMessagePublisher _messagePublisher;
    private readonly ISagaStateRepository _sagaStateRepository;
    private readonly ILogger<OrderSagaOrchestrator> _logger;
    
    public OrderSagaOrchestrator(IMessagePublisher messagePublisher, ISagaStateRepository sagaStateRepository, ILogger<OrderSagaOrchestrator> logger)
    {
        _messagePublisher = messagePublisher;
        _sagaStateRepository = sagaStateRepository;
        _logger = logger;
    }
    
    public async Task OrchestratePlaceOrderAsync(int userId, List<OrderItem> items, decimal totalAmount)
    {
        var sagaId = Guid.NewGuid().ToString();
        var correlationId = Guid.NewGuid().ToString();
        
        _logger.LogInformation($"Starting OrderSaga: {sagaId}");
        
        // Create saga state
        var sagaState = new SagaState
        {
            SagaId = sagaId,
            CorrelationId = correlationId,
            Status = SagaStatus.Started,
            CreatedAt = DateTime.UtcNow,
            Data = new Dictionary<string, object>
            {
                { "UserId", userId },
                { "Items", items },
                { "TotalAmount", totalAmount }
            }
        };
        
        await _sagaStateRepository.SaveAsync(sagaState);
        
        // Step 1: Create Order
        var createOrderCommand = new CreateOrderCommand
        {
            SagaId = sagaId,
            CorrelationId = correlationId,
            StepNumber = 1,
            UserId = userId,
            Items = items,
            TotalAmount = totalAmount
        };
        
        await _messagePublisher.PublishAsync(createOrderCommand, "order.commands", "create.order");
    }
    
    public async Task HandleOrderCreatedEventAsync(OrderCreatedEvent @event)
    {
        _logger.LogInformation($"OrderCreatedEvent received for SagaId: {@event.SagaId}");
        
        // Update saga state
        var sagaState = await _sagaStateRepository.GetAsync(@event.SagaId);
        sagaState.Data["OrderId"] = @event.OrderId;
        
        // Step 2: Reserve Inventory
        var reserveCommand = new ReserveInventoryCommand
        {
            SagaId = @event.SagaId,
            CorrelationId = @event.CorrelationId,
            StepNumber = 2,
            OrderId = @event.OrderId,
            Items = (List<OrderItem>)sagaState.Data["Items"]
        };
        
        await _messagePublisher.PublishAsync(reserveCommand, "product.commands", "reserve.inventory");
        await _sagaStateRepository.SaveAsync(sagaState);
    }
    
    public async Task HandleInventoryReservedEventAsync(InventoryReservedEvent @event)
    {
        _logger.LogInformation($"InventoryReservedEvent received for SagaId: {@event.SagaId}");
        
        var sagaState = await _sagaStateRepository.GetAsync(@event.SagaId);
        
        if (@event.Success)
        {
            // Step 3: Process Payment
            var paymentCommand = new ProcessPaymentCommand
            {
                SagaId = @event.SagaId,
                CorrelationId = @event.CorrelationId,
                StepNumber = 3,
                OrderId = @event.OrderId,
                UserId = (int)sagaState.Data["UserId"],
                Amount = (decimal)sagaState.Data["TotalAmount"]
            };
            
            await _messagePublisher.PublishAsync(paymentCommand, "payment.commands", "process.payment");
        }
        else
        {
            // Compensate: Cancel Order
            await CompensateAsync(sagaState);
        }
    }
    
    private async Task CompensateAsync(SagaState sagaState)
    {
        _logger.LogWarning($"Compensating SagaId: {sagaState.SagaId}");
        
        var cancelOrderCommand = new CancelOrderCommand
        {
            SagaId = sagaState.SagaId,
            CorrelationId = sagaState.CorrelationId,
            OrderId = (int)sagaState.Data["OrderId"]
        };
        
        await _messagePublisher.PublishAsync(cancelOrderCommand, "order.commands", "cancel.order");
        
        sagaState.Status = SagaStatus.Failed;
        await _sagaStateRepository.SaveAsync(sagaState);
    }
}

// OrchestratorService/Models/SagaState.cs
public class SagaState
{
    public string SagaId { get; set; }
    public string CorrelationId { get; set; }
    public SagaStatus Status { get; set; }
    public DateTime CreatedAt { get; set; }
    public Dictionary<string, object> Data { get; set; }
}

public enum SagaStatus
{
    Started,
    Completed,
    Failed,
    Compensating,
    Compensated
}
```

---

### Implementing Saga Pattern in Product Service

**Example - Product Service Consumer:**

```csharp
// ProductService/EventHandlers/ReserveInventoryCommandHandler.cs
public class ReserveInventoryCommandHandler
{
    private readonly IMessagePublisher _messagePublisher;
    private readonly IInventoryService _inventoryService;
    private readonly ILogger<ReserveInventoryCommandHandler> _logger;
    
    public ReserveInventoryCommandHandler(IMessagePublisher messagePublisher, IInventoryService inventoryService, ILogger<ReserveInventoryCommandHandler> logger)
    {
        _messagePublisher = messagePublisher;
        _inventoryService = inventoryService;
        _logger = logger;
    }
    
    public async Task HandleAsync(ReserveInventoryCommand command)
    {
        _logger.LogInformation($"Handling ReserveInventoryCommand for OrderId: {command.OrderId}");
        
        try
        {
            // Check inventory availability
            foreach (var item in command.Items)
            {
                var available = await _inventoryService.IsAvailableAsync(item.ProductId, item.Quantity);
                if (!available)
                    throw new InvalidOperationException($"Insufficient inventory for product {item.ProductId}");
            }
            
            // Reserve inventory
            foreach (var item in command.Items)
            {
                await _inventoryService.ReserveAsync(item.ProductId, item.Quantity, command.OrderId);
            }
            
            // Publish InventoryReservedEvent (Success)
            var @event = new InventoryReservedEvent
            {
                SagaId = command.SagaId,
                CorrelationId = command.CorrelationId,
                StepNumber = command.StepNumber + 1,
                OrderId = command.OrderId,
                Success = true,
                Message = "Inventory reserved successfully"
            };
            
            await _messagePublisher.PublishAsync(@event, "product.events", "inventory.reserved");
        }
        catch (Exception ex)
        {
            _logger.LogError($"Inventory reservation failed: {ex.Message}");
            
            // Publish InventoryReservedEvent (Failure)
            var @event = new InventoryReservedEvent
            {
                SagaId = command.SagaId,
                CorrelationId = command.CorrelationId,
                StepNumber = command.StepNumber + 1,
                OrderId = command.OrderId,
                Success = false,
                Message = ex.Message
            };
            
            await _messagePublisher.PublishAsync(@event, "product.events", "inventory.reserved");
        }
    }
}

// ProductService/Services/InventoryService.cs
public interface IInventoryService
{
    Task<bool> IsAvailableAsync(int productId, int quantity);
    Task ReserveAsync(int productId, int quantity, int orderId);
    Task ReleaseAsync(int productId, int orderId);
}

public class InventoryService : IInventoryService
{
    private readonly IInventoryRepository _repository;
    
    public InventoryService(IInventoryRepository repository)
    {
        _repository = repository;
    }
    
    public async Task<bool> IsAvailableAsync(int productId, int quantity)
    {
        var inventory = await _repository.GetByProductIdAsync(productId);
        return inventory != null && inventory.AvailableQuantity >= quantity;
    }
    
    public async Task ReserveAsync(int productId, int quantity, int orderId)
    {
        var inventory = await _repository.GetByProductIdAsync(productId);
        if (inventory.AvailableQuantity < quantity)
            throw new InvalidOperationException("Insufficient inventory");
        
        inventory.AvailableQuantity -= quantity;
        inventory.ReservedQuantity += quantity;
        
        var reservation = new Reservation
        {
            ProductId = productId,
            OrderId = orderId,
            Quantity = quantity,
            CreatedAt = DateTime.UtcNow
        };
        
        await _repository.SaveReservationAsync(reservation);
        await _repository.UpdateAsync(inventory);
    }
    
    public async Task ReleaseAsync(int productId, int orderId)
    {
        var reservation = await _repository.GetReservationAsync(productId, orderId);
        if (reservation != null)
        {
            var inventory = await _repository.GetByProductIdAsync(productId);
            inventory.AvailableQuantity += reservation.Quantity;
            inventory.ReservedQuantity -= reservation.Quantity;
            
            await _repository.UpdateAsync(inventory);
            await _repository.DeleteReservationAsync(reservation.Id);
        }
    }
}
```

---

### Implementing Saga Pattern in Notification Service

**Example - Notification Consumer:**

```csharp
// NotificationService/EventHandlers/OrderCompletedEventHandler.cs
public class OrderCompletedEventHandler
{
    private readonly IEmailService _emailService;
    private readonly IUserServiceClient _userServiceClient;
    private readonly ILogger<OrderCompletedEventHandler> _logger;
    
    public OrderCompletedEventHandler(IEmailService emailService, IUserServiceClient userServiceClient, ILogger<OrderCompletedEventHandler> logger)
    {
        _emailService = emailService;
        _userServiceClient = userServiceClient;
        _logger = logger;
    }
    
    public async Task HandleAsync(OrderCompletedEvent @event)
    {
        _logger.LogInformation($"Handling OrderCompletedEvent for OrderId: {@event.OrderId}");
        
        try
        {
            // Get user details
            var user = await _userServiceClient.GetUserAsync(@event.UserId);
            
            // Send confirmation email
            var emailBody = $@"
                Dear {user.FirstName},
                
                Your order #{@event.OrderId} has been completed successfully.
                Total Amount: ${@event.TotalAmount}
                
                Thank you for your purchase!
                
                Best regards,
                The Store Team
            ";
            
            await _emailService.SendAsync(user.Email, "Order Confirmation", emailBody);
            
            _logger.LogInformation($"Confirmation email sent to {user.Email}");
        }
        catch (Exception ex)
        {
            _logger.LogError($"Error sending notification: {ex.Message}");
            // Could implement retry logic here
        }
    }
}

// NotificationService/Services/IEmailService.cs
public interface IEmailService
{
    Task SendAsync(string to, string subject, string body);
}

public class SmtpEmailService : IEmailService
{
    private readonly IConfiguration _configuration;
    
    public SmtpEmailService(IConfiguration configuration)
    {
        _configuration = configuration;
    }
    
    public async Task SendAsync(string to, string subject, string body)
    {
        using (var client = new SmtpClient())
        {
            client.Host = _configuration["Smtp:Host"];
            client.Port = int.Parse(_configuration["Smtp:Port"]);
            client.EnableSsl = bool.Parse(_configuration["Smtp:EnableSsl"]);
            
            client.Credentials = new NetworkCredential(
                _configuration["Smtp:Username"],
                _configuration["Smtp:Password"]);
            
            var mailMessage = new MailMessage(
                _configuration["Smtp:FromAddress"],
                to,
                subject,
                body);
            
            await client.SendMailAsync(mailMessage);
        }
    }
}

// NotificationService/Clients/IUserServiceClient.cs
public interface IUserServiceClient
{
    Task<UserDto> GetUserAsync(int userId);
}

public class UserServiceClient : IUserServiceClient
{
    private readonly HttpClient _httpClient;
    
    public UserServiceClient(HttpClient httpClient)
    {
        _httpClient = httpClient;
    }
    
    public async Task<UserDto> GetUserAsync(int userId)
    {
        var response = await _httpClient.GetAsync($"https://localhost:5001/api/users/{userId}");
        response.EnsureSuccessStatusCode();
        
        var json = await response.Content.ReadAsStringAsync();
        return JsonSerializer.Deserialize<UserDto>(json);
    }
}
```

---

## API Gateway in Microservices

### API Gateway in ASP.NET Core Web API Microservices

**Notes:**
- API Gateway acts as single entry point for all client requests
- Routes requests to appropriate microservices
- Handles cross-cutting concerns (authentication, logging, rate limiting)
- Provides unified interface to clients
- Enables API versioning and aggregation

**Benefits:**
- Simplified client-side code
- Centralized security
- Service composition
- Protocol translation (REST to gRPC)
- Request/response transformation

---

### Ocelot API Gateway in ASP.NET Core Web API

**Notes:**
- Ocelot is lightweight, open-source API Gateway for .NET
- Supports routing, aggregation, service discovery
- Configuration through JSON files
- Built on ASP.NET Core
- Easy to customize

**Example - Ocelot Setup:**

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Add Ocelot
builder.Services.AddOcelot();

var app = builder.Build();

app.UseRouting();
app.UseEndpoints(endpoints =>
{
    endpoints.MapControllers();
});

await app.UseOcelot();

app.Run();

// ocelot.json - Configuration
{
  "Routes": [
    {
      "DownstreamPathTemplate": "/api/users/{id}",
      "DownstreamScheme": "https",
      "DownstreamHostAndPorts": [
        {
          "Host": "localhost",
          "Port": 5001
        }
      ],
      "UpstreamPathTemplate": "/gateway/users/{id}",
      "UpstreamHttpMethod": [ "Get" ]
    },
    {
      "DownstreamPathTemplate": "/api/orders",
      "DownstreamScheme": "https",
      "DownstreamHostAndPorts": [
        {
          "Host": "localhost",
          "Port": 5002
        }
      ],
      "UpstreamPathTemplate": "/gateway/orders",
      "UpstreamHttpMethod": [ "Get", "Post" ]
    },
    {
      "DownstreamPathTemplate": "/api/products/{id}",
      "DownstreamScheme": "https",
      "DownstreamHostAndPorts": [
        {
          "Host": "localhost",
          "Port": 5003
        }
      ],
      "UpstreamPathTemplate": "/gateway/products/{id}",
      "UpstreamHttpMethod": [ "Get" ]
    }
  ],
  "GlobalConfiguration": {
    "BaseUrl": "https://localhost:5000"
  }
}

// Usage
// Client calls: GET https://localhost:5000/gateway/users/1
// Gateway routes to: GET https://localhost:5001/api/users/1
```

---

### Logging with Ocelot API Gateway in ASP.NET Core Web API

**Example:**

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Add logging
builder.Services.AddLogging(config =>
{
    config.AddConsole();
    config.AddDebug();
});

builder.Services.AddOcelot();

// Custom Ocelot logger
builder.Services.AddSingleton<ITracer, HttpClientTrace>();

var app = builder.Build();

// Use Ocelot middleware with logging
app.UseOcelotLogging();

await app.UseOcelot();

app.Run();

// Custom middleware for logging
public class OcelotLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<OcelotLoggingMiddleware> _logger;
    
    public OcelotLoggingMiddleware(RequestDelegate next, ILogger<OcelotLoggingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        _logger.LogInformation($"Incoming request: {context.Request.Method} {context.Request.Path}");
        
        await _next(context);
        
        _logger.LogInformation($"Response status: {context.Response.StatusCode}");
    }
}

public static class OcelotLoggingExtensions
{
    public static IApplicationBuilder UseOcelotLogging(this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<OcelotLoggingMiddleware>();
    }
}

// Structured logging with Serilog
public class Program
{
    public static void Main(string[] args)
    {
        Log.Logger = new LoggerConfiguration()
            .MinimumLevel.Debug()
            .WriteTo.Console()
            .WriteTo.File("logs/ocelot-.txt", rollingInterval: RollingInterval.Day)
            .CreateLogger();
        
        try
        {
            CreateHostBuilder(args).Build().Run();
        }
        finally
        {
            Log.CloseAndFlush();
        }
    }
    
    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .UseSerilog()
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStartup<Startup>();
            });
}
```

---

### JWT Authentication in API Gateway

**Example:**

```csharp
// Program.cs - API Gateway with JWT
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.Authority = "https://your-auth-server.com";
        options.Audience = "api-gateway";
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true
        };
    });

builder.Services.AddOcelot();

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();

await app.UseOcelot();

app.Run();

// ocelot.json - With authentication
{
  "Routes": [
    {
      "DownstreamPathTemplate": "/api/users/{id}",
      "DownstreamScheme": "https",
      "DownstreamHostAndPorts": [
        {
          "Host": "localhost",
          "Port": 5001
        }
      ],
      "UpstreamPathTemplate": "/gateway/users/{id}",
      "UpstreamHttpMethod": [ "Get" ],
      "AuthenticationOptions": {
        "AuthenticationProviderKey": "Bearer",
        "AllowedScopes": []
      },
      "RouteClaimsRequirement": {
        "Role": "Admin"
      }
    }
  ]
}

// Auth Service - Token generation
[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    private readonly ITokenService _tokenService;
    
    public AuthController(ITokenService tokenService)
    {
        _tokenService = tokenService;
    }
    
    [HttpPost("login")]
    public async Task<ActionResult<LoginResponse>> Login([FromBody] LoginRequest request)
    {
        // Validate user credentials
        var user = await ValidateUserAsync(request.Email, request.Password);
        if (user == null)
            return Unauthorized();
        
        // Generate JWT token
        var token = _tokenService.GenerateToken(user);
        
        return Ok(new LoginResponse { Token = token });
    }
}

// Token Service
public interface ITokenService
{
    string GenerateToken(User user);
}

public class JwtTokenService : ITokenService
{
    private readonly IConfiguration _configuration;
    
    public JwtTokenService(IConfiguration configuration)
    {
        _configuration = configuration;
    }
    
    public string GenerateToken(User user)
    {
        var securityKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_configuration["Jwt:Key"]));
        var credentials = new SigningCredentials(securityKey, SecurityAlgorithms.HmacSha256);
        
        var claims = new[]
        {
            new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
            new Claim(ClaimTypes.Email, user.Email),
            new Claim(ClaimTypes.Name, user.FirstName),
            new Claim(ClaimTypes.Role, "User")
        };
        
        var token = new JwtSecurityToken(
            issuer: _configuration["Jwt:Issuer"],
            audience: _configuration["Jwt:Audience"],
            claims: claims,
            expires: DateTime.UtcNow.AddHours(1),
            signingCredentials: credentials);
        
        return new JwtSecurityTokenHandler().WriteToken(token);
    }
}
```

---

### Response Aggregation in API Gateway

**Example - Aggregating Multiple Services:**

```csharp
// ocelot.json - Route aggregation
{
  "Routes": [
    {
      "DownstreamPathTemplate": "/api/users/{id}",
      "DownstreamScheme": "https",
      "DownstreamHostAndPorts": [
        {
          "Host": "localhost",
          "Port": 5001
        }
      ],
      "UpstreamPathTemplate": "/gateway/dashboard/{id}/user",
      "UpstreamHttpMethod": [ "Get" ],
      "Key": "User"
    },
    {
      "DownstreamPathTemplate": "/api/orders/user/{id}",
      "DownstreamScheme": "https",
      "DownstreamHostAndPorts": [
        {
          "Host": "localhost",
          "Port": 5002
        }
      ],
      "UpstreamPathTemplate": "/gateway/dashboard/{id}/orders",
      "UpstreamHttpMethod": [ "Get" ],
      "Key": "Orders"
    },
    {
      "DownstreamPathTemplate": "/api/notifications/user/{id}",
      "DownstreamScheme": "https",
      "DownstreamHostAndPorts": [
        {
          "Host": "localhost",
          "Port": 5004
        }
      ],
      "UpstreamPathTemplate": "/gateway/dashboard/{id}/notifications",
      "UpstreamHttpMethod": [ "Get" ],
      "Key": "Notifications"
    }
  ],
  "Aggregates": [
    {
      "UpstreamPathTemplate": "/gateway/dashboard/{id}",
      "UpstreamHttpMethod": [ "Get" ],
      "RouteKeys": [ "User", "Orders", "Notifications"]
    }
  ]
}

// Custom aggregator for advanced scenarios
public interface IResponseAggregator
{
    Task<DownstreamResponse> Aggregate(List<DownstreamResponse> responses);
}

public class DashboardAggregator : IResponseAggregator
{
    public async Task<DownstreamResponse> Aggregate(List<DownstreamResponse> responses)
    {
        var aggregated = new
        {
            User = JsonSerializer.Deserialize(responses[0].Content),
            Orders = JsonSerializer.Deserialize(responses[1].Content),
            Notifications = JsonSerializer.Deserialize(responses[2].Content)
        };
        
        var content = JsonSerializer.Serialize(aggregated);
        
        return new DownstreamResponse(
            content,
            System.Net.HttpStatusCode.OK,
            responses.SelectMany(r => r.Headers).ToList(),
            "OK");
    }
}

// Usage - Single request returns aggregated response
// GET /gateway/dashboard/1
// Response includes user, orders, and notifications
```

---

### Response Compression in API Gateway

**Example:**

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Add compression
builder.Services.AddResponseCompression(options =>
{
    options.Providers.Add<BrotliCompressionProvider>();
    options.Providers.Add<GzipCompressionProvider>();
    options.MimeTypes = ResponseCompressionDefaults.MimeTypes.Concat(
        new[] { "application/json", "text/plain" });
});

builder.Services.AddOcelot();

var app = builder.Build();

app.UseResponseCompression();

await app.UseOcelot();

app.Run();

// Configure specific compression
var options = new BrotliCompressionProviderOptions
{
    Level = CompressionLevel.Optimal
};
```

---

### Rate Limiting and Throttling in API Gateway

**Example:**

```csharp
// ocelot.json - Rate limiting
{
  "Routes": [
    {
      "DownstreamPathTemplate": "/api/users/{id}",
      "DownstreamScheme": "https",
      "DownstreamHostAndPorts": [
        {
          "Host": "localhost",
          "Port": 5001
        }
      ],
      "UpstreamPathTemplate": "/gateway/users/{id}",
      "UpstreamHttpMethod": [ "Get" ],
      "RateLimitOptions": {
        "ClientWhitelist": [],
        "EnableRateLimiting": true,
        "Period": "1m",
        "PeriodTimespan": 60,
        "Limit": 100
      }
    }
  ],
  "GlobalConfiguration": {
    "RateLimitOptions": {
      "DisableRateLimitHeaders": false,
      "QuotaExceededMessage": "API limit exceeded",
      "HttpStatusCode": 429,
      "ClientIdHeader": "ClientId"
    }
  }
}

// Custom rate limiting middleware
public class RateLimitingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly IMemoryCache _cache;
    
    public RateLimitingMiddleware(RequestDelegate next, IMemoryCache cache)
    {
        _next = next;
        _cache = cache;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        var clientId = context.Request.Headers["X-Client-Id"].ToString();
        var key = $"rate_limit_{clientId}";
        
        if (_cache.TryGetValue(key, out int requestCount))
        {
            if (requestCount > 100)
            {
                context.Response.StatusCode = StatusCodes.Status429TooManyRequests;
                await context.Response.WriteAsync("Rate limit exceeded");
                return;
            }
        }
        
        _cache.Set(key, (requestCount + 1), TimeSpan.FromMinutes(1));
        
        await _next(context);
    }
}
```

---

### Response Caching in API Gateway

**Example:**

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddResponseCaching();
builder.Services.AddOcelot();

var app = builder.Build();

app.UseResponseCaching();

await app.UseOcelot();

app.Run();

// ocelot.json - Caching
{
  "Routes": [
    {
      "DownstreamPathTemplate": "/api/products/{id}",
      "DownstreamScheme": "https",
      "DownstreamHostAndPorts": [
        {
          "Host": "localhost",
          "Port": 5003
        }
      ],
      "UpstreamPathTemplate": "/gateway/products/{id}",
      "UpstreamHttpMethod": [ "Get" ],
      "HttpHandlerOptions": {
        "UseTracing": false,
        "CacheOptions": {
          "TtlSeconds": 600,
          "Region": "ProductCache"
        }
      }
    }
  ]
}

// Custom cache middleware
[ApiController]
[Route("api/[controller]")]
public class CachedController : ControllerBase
{
    private readonly IDistributedCache _cache;
    
    public CachedController(IDistributedCache cache)
    {
        _cache = cache;
    }
    
    [HttpGet("{id}")]
    public async Task<ActionResult<ProductDto>> GetProduct(int id)
    {
        var cacheKey = $"product_{id}";
        
        // Try to get from cache
        var cachedData = await _cache.GetStringAsync(cacheKey);
        if (!string.IsNullOrEmpty(cachedData))
        {
            return Ok(JsonSerializer.Deserialize<ProductDto>(cachedData));
        }
        
        // Get from service
        var product = await GetProductFromService(id);
        
        // Store in cache
        await _cache.SetStringAsync(cacheKey, JsonSerializer.Serialize(product), 
            new DistributedCacheEntryOptions 
            { 
                AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10) 
            });
        
        return Ok(product);
    }
    
    private async Task<ProductDto> GetProductFromService(int id)
    {
        // Implementation
        await Task.Delay(100); // Simulated call
        return new ProductDto { Id = id, Name = "Product" };
    }
}
```

---

### API Gateway using YARP in ASP.NET Core

**Notes:**
- YARP (Yet Another Reverse Proxy) is Microsoft's modern reverse proxy
- Built on ASP.NET Core
- High-performance alternative to Ocelot
- Supports dynamic configuration
- Extensible middleware pipeline

**Example - YARP Setup:**

```csharp
// Install NuGet: Yarp.ReverseProxy

// Program.cs
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddReverseProxy()
    .LoadFromConfig(builder.Configuration.GetSection("ReverseProxy"));

var app = builder.Build();

app.UseRouting();
app.UseEndpoints(endpoints =>
{
    endpoints.MapReverseProxy();
});

app.Run();

// appsettings.json - YARP Configuration
{
  "ReverseProxy": {
    "Routes": [
      {
        "RouteId": "users-route",
        "ClusterId": "users-cluster",
        "Match": {
          "Path": "/gateway/users/{**catch-all}"
        },
        "Transforms": [
          {
            "PathPattern": "/api/{**catch-all}"
          }
        ]
      },
      {
        "RouteId": "orders-route",
        "ClusterId": "orders-cluster",
        "Match": {
          "Path": "/gateway/orders/{**catch-all}"
        },
        "Transforms": [
          {
            "PathPattern": "/api/{**catch-all}"
          }
        ]
      }
    ],
    "Clusters": [
      {
        "ClusterId": "users-cluster",
        "Destinations": {
          "users-destination": {
            "Address": "https://localhost:5001"
          }
        }
      },
      {
        "ClusterId": "orders-cluster",
        "Destinations": {
          "orders-destination": {
            "Address": "https://localhost:5002"
          }
        }
      }
    ]
  }
}

// Custom YARP middleware for request/response handling
public class CustomTransformProvider : ITransformProvider
{
    public void ValidateRoute(TransformRouteValidationContext context)
    {
        // Validation logic
    }
    
    public void Apply(TransformBuilderContext transformBuildContext)
    {
        // Apply transformations
        transformBuildContext.AddRequestTransform(async transformContext =>
        {
            // Add authentication header
            transformContext.ProxyRequest.Headers.Add("X-Gateway-Id", "api-gateway-1");
            await Task.CompletedTask;
        });
    }
}

// Register custom provider
builder.Services.AddSingleton<ITransformProvider, CustomTransformProvider>();
```

---

## Advanced Patterns

### gRPC in ASP.NET Core Web API

**Notes:**
- gRPC uses HTTP/2 for efficient binary messaging
- Protocol Buffers for serialization
- Strongly typed service contracts
- Bidirectional streaming support
- Better performance than REST for inter-service communication

**Example - gRPC Setup:**

```csharp
// Install NuGet: Grpc.AspNetCore

// user.proto - Protocol Buffer definition
syntax = "proto3";

package UserService;

service UserService {
  rpc GetUser(GetUserRequest) returns (UserResponse);
  rpc CreateUser(CreateUserRequest) returns (UserResponse);
  rpc ListUsers(Empty) returns (stream UserResponse);
}

message GetUserRequest {
  int32 id = 1;
}

message CreateUserRequest {
  string email = 1;
  string firstName = 2;
  string lastName = 3;
}

message UserResponse {
  int32 id = 1;
  string email = 2;
  string firstName = 3;
  string lastName = 4;
}

message Empty {}

// Program.cs - gRPC Server
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddGrpc();

var app = builder.Build();

app.MapGrpcService<UserGrpcService>();

app.Run();

// UserGrpcService.cs - Implementation
public class UserGrpcService : UserService.UserServiceBase
{
    private readonly IUserService _userService;
    
    public UserGrpcService(IUserService userService)
    {
        _userService = userService;
    }
    
    public override async Task<UserResponse> GetUser(GetUserRequest request, ServerCallContext context)
    {
        var user = await _userService.GetUserByIdAsync(request.Id);
        if (user == null)
            throw new RpcException(new Status(StatusCode.NotFound, "User not found"));
        
        return new UserResponse
        {
            Id = user.Id,
            Email = user.Email,
            FirstName = user.FirstName,
            LastName = user.LastName
        };
    }
    
    public override async Task ListUsers(Empty request, IServerStreamWriter<UserResponse> responseStream, ServerCallContext context)
    {
        var users = await _userService.GetAllUsersAsync();
        
        foreach (var user in users)
        {
            await responseStream.WriteAsync(new UserResponse
            {
                Id = user.Id,
                Email = user.Email,
                FirstName = user.FirstName,
                LastName = user.LastName
            });
        }
    }
}

// Client implementation
public class UserGrpcClient
{
    private readonly UserService.UserServiceClient _client;
    
    public UserGrpcClient(UserService.UserServiceClient client)
    {
        _client = client;
    }
    
    public async Task<UserResponse> GetUserAsync(int userId)
    {
        var request = new GetUserRequest { Id = userId };
        return await _client.GetUserAsync(request);
    }
    
    public async Task ListUsersAsync()
    {
        var response = _client.ListUsers(new Empty());
        
        await foreach (var user in response.ResponseStream.ReadAllAsync())
        {
            Console.WriteLine($"User: {user.FirstName} {user.LastName}");
        }
    }
}

// Configure gRPC client in consuming service
builder.Services.AddGrpcClient<UserService.UserServiceClient>(options =>
{
    options.Address = new Uri("https://localhost:5001");
});
```

---

### Implementing gRPC in ASP.NET Core

**Example - Complete gRPC Service:**

```csharp
// OrderService consuming UserService via gRPC

[ApiController]
[Route("api/[controller]")]
public class OrderController : ControllerBase
{
    private readonly IOrderService _orderService;
    private readonly UserService.UserServiceClient _userClient;
    
    public OrderController(IOrderService orderService, UserService.UserServiceClient userClient)
    {
        _orderService = orderService;
        _userClient = userClient;
    }
    
    [HttpPost("user/{userId}")]
    public async Task<ActionResult<OrderDto>> CreateOrderForUser(int userId, [FromBody] CreateOrderRequest request)
    {
        try
        {
            // Verify user exists using gRPC
            var userResponse = await _userClient.GetUserAsync(new GetUserRequest { Id = userId });
            
            // Create order
            var order = await _orderService.CreateOrderAsync(userId, request);
            
            return CreatedAtAction(nameof(GetOrderById), new { id = order.Id }, order);
        }
        catch (RpcException ex) when (ex.StatusCode == StatusCode.NotFound)
        {
            return NotFound("User not found");
        }
    }
}
```

---

### CQRS in ASP.NET Core Web API

**Notes:**
- CQRS separates read (Query) and write (Command) models
- Optimizes read and write paths independently
- Enables event sourcing integration
- Improves scalability and performance
- Supports different data stores for reads and writes

**Example - CQRS Implementation:**

```csharp
// Commands
public abstract class CommandBase
{
    public string CorrelationId { get; set; } = Guid.NewGuid().ToString();
}

public class CreateUserCommand : CommandBase
{
    public string Email { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Password { get; set; }
}

public class UpdateUserCommand : CommandBase
{
    public int UserId { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
}

// Queries
public abstract class QueryBase<TResponse>
{
    public string CorrelationId { get; set; } = Guid.NewGuid().ToString();
}

public class GetUserByIdQuery : QueryBase<UserDto>
{
    public int UserId { get; set; }
}

public class GetAllUsersQuery : QueryBase<List<UserDto>>
{
}

// Query Handlers
public interface IQueryHandler<in TQuery, TResponse> where TQuery : QueryBase<TResponse>
{
    Task<TResponse> HandleAsync(TQuery query);
}

public class GetUserByIdQueryHandler : IQueryHandler<GetUserByIdQuery, UserDto>
{
    private readonly IUserRepository _repository;
    
    public GetUserByIdQueryHandler(IUserRepository repository)
    {
        _repository = repository;
    }
    
    public async Task<UserDto> HandleAsync(GetUserByIdQuery query)
    {
        var user = await _repository.GetByIdAsync(query.UserId);
        if (user == null)
            throw new KeyNotFoundException();
        
        return new UserDto
        {
            Id = user.Id,
            Email = user.Email,
            FirstName = user.FirstName,
            LastName = user.LastName
        };
    }
}

// Command Handlers
public interface ICommandHandler<in TCommand> where TCommand : CommandBase
{
    Task HandleAsync(TCommand command);
}

public class CreateUserCommandHandler : ICommandHandler<CreateUserCommand>
{
    private readonly IUserRepository _repository;
    private readonly IEventBus _eventBus;
    
    public CreateUserCommandHandler(IUserRepository repository, IEventBus eventBus)
    {
        _repository = repository;
        _eventBus = eventBus;
    }
    
    public async Task HandleAsync(CreateUserCommand command)
    {
        var user = User.Create(command.Email, command.FirstName, command.LastName, command.Password);
        
        await _repository.AddAsync(user);
        
        // Publish event
        var @event = new UserCreatedEvent
        {
            UserId = user.Id,
            Email = user.Email,
            FirstName = user.FirstName,
            LastName = user.LastName
        };
        
        await _eventBus.PublishAsync(@event);
    }
}

// Mediator
public interface IMediator
{
    Task<TResponse> SendAsync<TResponse>(QueryBase<TResponse> query);
    Task SendAsync(CommandBase command);
}

public class MediatorService : IMediator
{
    private readonly IServiceProvider _serviceProvider;
    
    public MediatorService(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }
    
    public async Task<TResponse> SendAsync<TResponse>(QueryBase<TResponse> query)
    {
        var handlerType = typeof(IQueryHandler<,>)
            .MakeGenericType(query.GetType(), typeof(TResponse));
        
        dynamic handler = _serviceProvider.GetService(handlerType);
        return await handler.HandleAsync((dynamic)query);
    }
    
    public async Task SendAsync(CommandBase command)
    {
        var handlerType = typeof(ICommandHandler<>)
            .MakeGenericType(command.GetType());
        
        dynamic handler = _serviceProvider.GetService(handlerType);
        await handler.HandleAsync((dynamic)command);
    }
}

// Register services
builder.Services.AddScoped<IMediator, MediatorService>();
builder.Services.AddScoped(typeof(IQueryHandler<,>), typeof(GetUserByIdQueryHandler));
builder.Services.AddScoped(typeof(ICommandHandler<>), typeof(CreateUserCommandHandler));

// Controller
[ApiController]
[Route("api/[controller]")]
public class UserController : ControllerBase
{
    private readonly IMediator _mediator;
    
    public UserController(IMediator mediator)
    {
        _mediator = mediator;
    }
    
    [HttpGet("{id}")]
    public async Task<ActionResult<UserDto>> GetUser(int id)
    {
        var query = new GetUserByIdQuery { UserId = id };
        var user = await _mediator.SendAsync(query);
        return Ok(user);
    }
    
    [HttpPost]
    public async Task<IActionResult> CreateUser([FromBody] CreateUserCommand command)
    {
        await _mediator.SendAsync(command);
        return Ok();
    }
}
```

---

### Implementing CQRS in ASP.NET Core Microservices

**Example - CQRS with Separate Read/Write Databases:**

```csharp
// Write Model (Command Database)
public class UserServiceWriteDbContext : DbContext
{
    public DbSet<User> Users { get; set; }
    
    public UserServiceWriteDbContext(DbContextOptions<UserServiceWriteDbContext> options) 
        : base(options)
    {
    }
}

// Read Model (Query Database)
public class UserServiceReadDbContext : DbContext
{
    public DbSet<UserReadModel> UserReadModels { get; set; }
    
    public UserServiceReadDbContext(DbContextOptions<UserServiceReadDbContext> options) 
        : base(options)
    {
    }
}

// Read Model Entity
public class UserReadModel
{
    public int Id { get; set; }
    public string Email { get; set; }
    public string FullName { get; set; }
    public int OrderCount { get; set; }
    public decimal TotalSpent { get; set; }
}

// Query Handler using Read Model
public class GetUserDetailsQueryHandler : IQueryHandler<GetUserDetailsQuery, UserDetailsDto>
{
    private readonly UserServiceReadDbContext _readContext;
    
    public GetUserDetailsQueryHandler(UserServiceReadDbContext readContext)
    {
        _readContext = readContext;
    }
    
    public async Task<UserDetailsDto> HandleAsync(GetUserDetailsQuery query)
    {
        var readModel = await _readContext.UserReadModels
            .AsNoTracking()
            .FirstOrDefaultAsync(u => u.Id == query.UserId);
        
        if (readModel == null)
            throw new KeyNotFoundException();
        
        return new UserDetailsDto
        {
            Id = readModel.Id,
            Email = readModel.Email,
            FullName = readModel.FullName,
            OrderCount = readModel.OrderCount,
            TotalSpent = readModel.TotalSpent
        };
    }
}

// Sync Read Model when commands execute
public class UserCreatedEventHandler
{
    private readonly UserServiceReadDbContext _readContext;
    
    public UserCreatedEventHandler(UserServiceReadDbContext readContext)
    {
        _readContext = readContext;
    }
    
    public async Task HandleAsync(UserCreatedEvent @event)
    {
        var readModel = new UserReadModel
        {
            Id = @event.UserId,
            Email = @event.Email,
            FullName = $"{@event.FirstName} {@event.LastName}",
            OrderCount = 0,
            TotalSpent = 0
        };
        
        _readContext.UserReadModels.Add(readModel);
        await _readContext.SaveChangesAsync();
    }
}

// Configuration
builder.Services.AddDbContext<UserServiceWriteDbContext>(options =>
    options.UseSqlServer("Server=localhost;Database=UserServiceWrite;"));

builder.Services.AddDbContext<UserServiceReadDbContext>(options =>
    options.UseSqlServer("Server=localhost;Database=UserServiceRead;"));
```

---

### GraphQL in ASP.NET Core Web API

**Notes:**
- GraphQL provides flexible querying of data
- Clients request exactly what they need
- Reduces over-fetching and under-fetching
- Strong typing and schema documentation
- Single endpoint for all queries

**Example - GraphQL with Hot Chocolate:**

```csharp
// Install NuGet: HotChocolate.AspNetCore

// GraphQL Query Type
public class Query
{
    public async Task<User> GetUser([Service] IUserService userService, int id)
    {
        return await userService.GetUserByIdAsync(id);
    }
    
    public async Task<List<User>> GetAllUsers([Service] IUserService userService)
    {
        return await userService.GetAllUsersAsync();
    }
}

// GraphQL Mutation Type
public class Mutation
{
    public async Task<User> CreateUser([Service] IUserService userService, 
        string email, string firstName, string lastName)
    {
        var request = new CreateUserRequest 
        { 
            Email = email, 
            FirstName = firstName, 
            LastName = lastName 
        };
        
        return await userService.CreateUserAsync(request);
    }
    
    public async Task<User> UpdateUser([Service] IUserService userService,
        int id, string firstName, string lastName)
    {
        var request = new UpdateUserRequest 
        { 
            FirstName = firstName, 
            LastName = lastName 
        };
        
        return await userService.UpdateUserAsync(id, request);
    }
}

// Program.cs - GraphQL Setup
var builder = WebApplication.CreateBuilder(args);

builder.Services
    .AddGraphQLServer()
    .AddQueryType<Query>()
    .AddMutationType<Mutation>();

var app = builder.Build();

app.MapGraphQL();

app.Run();

// GraphQL Queries
/*
Query:
{
  getAllUsers {
    id
    email
    firstName
    lastName
  }
}

Query with Filter:
{
  getUser(id: 1) {
    id
    email
    firstName
    lastName
  }
}

Mutation:
mutation {
  createUser(email: "user@example.com", firstName: "John", lastName: "Doe") {
    id
    email
    firstName
    lastName
  }
}
*/
```

---

### Implementing GraphQL in ASP.NET Core using Hot Chocolate

**Example - Advanced GraphQL:**

```csharp
// User Type
public class UserType : ObjectType<User>
{
    protected override void Configure(IObjectTypeDescriptor<User> descriptor)
    {
        descriptor
            .Field(u => u.Id)
            .Type<NonNullType<IntType>>();
        
        descriptor
            .Field(u => u.Email)
            .Type<NonNullType<StringType>>();
        
        descriptor
            .Field(u => u.FirstName)
            .Type<NonNullType<StringType>>();
        
        descriptor
            .Field(u => u.LastName)
            .Type<NonNullType<StringType>>();
        
        // Add field with resolver
        descriptor
            .Field("orders")
            .Resolve(async (context) =>
            {
                var orderService = context.Service<IOrderService>();
                return await orderService.GetUserOrdersAsync(context.Parent<User>().Id);
            });
    }
}

// Input Type
public class CreateUserInput
{
    public string Email { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Password { get; set; }
}

// Extended Query
public class ExtendedQuery
{
    [GraphQLType(typeof(NonNullType<ListType<NonNullType<UserType>>>))]
    public async Task<List<User>> GetAllUsers([Service] IUserService userService)
    {
        return await userService.GetAllUsersAsync();
    }
    
    public async Task<User> GetUser([Service] IUserService userService, int id)
    {
        return await userService.GetUserByIdAsync(id);
    }
    
    public async Task<List<User>> SearchUsers([Service] IUserService userService, string email)
    {
        return await userService.SearchUsersByEmailAsync(email);
    }
}

// Extended Mutation
public class ExtendedMutation
{
    public async Task<User> CreateUser(
        [Service] IUserService userService, 
        CreateUserInput input)
    {
        var request = new CreateUserRequest 
        { 
            Email = input.Email, 
            FirstName = input.FirstName, 
            LastName = input.LastName,
            Password = input.Password
        };
        
        return await userService.CreateUserAsync(request);
    }
}

// Setup
builder.Services
    .AddGraphQLServer()
    .AddQueryType<ExtendedQuery>()
    .AddMutationType<ExtendedMutation>()
    .AddType<UserType>()
    .AddQueryExecutor();
```

---

### Circuit Breaker in ASP.NET Core Web API

**Notes:**
- Circuit Breaker prevents cascading failures
- Three states: Closed (normal), Open (failing), Half-Open (recovering)
- Uses Polly library for implementation
- Helps maintain system resilience
- Reduces unnecessary load on failing services

**Example - Circuit Breaker with Polly:**

```csharp
// Install NuGet: Polly

// Program.cs - Configure Circuit Breaker
var builder = WebApplication.CreateBuilder(args);

var policy = Policy
    .Handle<HttpRequestException>()
    .OrResult<HttpResponseMessage>(r => (int)r.StatusCode >= 500)
    .CircuitBreaker(
        handledEventsAllowedBeforeBreaking: 3,
        durationOfBreak: TimeSpan.FromSeconds(30),
        onBreak: (outcome, timespan) =>
        {
            Console.WriteLine($"Circuit breaker opened for {timespan.TotalSeconds} seconds");
        },
        onReset: () =>
        {
            Console.WriteLine("Circuit breaker reset");
        });

builder.Services.AddHttpClient<IOrderServiceClient, OrderServiceClient>()
    .AddPolicyHandler(policy);

var app = builder.Build();

app.Run();

// Service Client with Circuit Breaker
public interface IOrderServiceClient
{
    Task<OrderDto> GetOrderAsync(int orderId);
}

public class OrderServiceClient : IOrderServiceClient
{
    private readonly HttpClient _httpClient;
    private readonly IAsyncPolicy<HttpResponseMessage> _circuitBreakerPolicy;
    
    public OrderServiceClient(HttpClient httpClient)
    {
        _httpClient = httpClient;
    }
    
    public async Task<OrderDto> GetOrderAsync(int orderId)
    {
        try
        {
            var response = await _httpClient.GetAsync($"https://localhost:5002/api/orders/{orderId}");
            response.EnsureSuccessStatusCode();
            
            var json = await response.Content.ReadAsStringAsync();
            return JsonSerializer.Deserialize<OrderDto>(json);
        }
        catch (HttpRequestException ex)
        {
            // Circuit breaker will catch this
            throw;
        }
    }
}

// Usage in service
public class OrderProcessor
{
    private readonly IOrderServiceClient _orderClient;
    
    public OrderProcessor(IOrderServiceClient orderClient)
    {
        _orderClient = orderClient;
    }
    
    public async Task ProcessOrderAsync(int orderId)
    {
        try
        {
            var order = await _orderClient.GetOrderAsync(orderId);
            // Process order
        }
        catch (BrokenCircuitException)
        {
            // Handle circuit open scenario
            Console.WriteLine("Order service is unavailable, please try again later");
        }
    }
}
```

---

### Implementing Circuit Breaker in ASP.NET Core Microservices

**Example - Advanced Circuit Breaker Patterns:**

```csharp
// Advanced Polly policies
var retryPolicy = Policy
    .Handle<HttpRequestException>()
    .Or<OperationCanceledException>()
    .WaitAndRetryAsync(
        retryCount: 3,
        sleepDurationProvider: attempt => 
            TimeSpan.FromMilliseconds(Math.Pow(2, attempt) * 100),
        onRetry: (outcome, timespan, retryCount, context) =>
        {
            Console.WriteLine($"Retry {retryCount} after {timespan.TotalMilliseconds}ms");
        });

var circuitBreakerPolicy = Policy
    .Handle<HttpRequestException>()
    .CircuitBreakerAsync(
        handledEventsAllowedBeforeBreaking: 3,
        durationOfBreak: TimeSpan.FromSeconds(30));

var timeoutPolicy = Policy.TimeoutAsync<HttpResponseMessage>(
    TimeSpan.FromSeconds(10));

var combinedPolicy = Policy.WrapAsync(
    retryPolicy, 
    circuitBreakerPolicy,
    timeoutPolicy);

builder.Services.AddHttpClient<IUserServiceClient, UserServiceClient>()
    .AddPolicyHandler(combinedPolicy);

// Alternative: Use Polly Registry
var policyRegistry = new PolicyRegistry()
{
    { "default", combinedPolicy },
    { "strict", Policy.TimeoutAsync<HttpResponseMessage>(TimeSpan.FromSeconds(5)) }
};

builder.Services.AddPolicyRegistry(policyRegistry);

// Implement with fallback
public class ResilientUserServiceClient : IUserServiceClient
{
    private readonly HttpClient _httpClient;
    private readonly IAsyncPolicy<HttpResponseMessage> _policy;
    
    public ResilientUserServiceClient(HttpClient httpClient, IAsyncPolicy<HttpResponseMessage> policy)
    {
        _httpClient = httpClient;
        _policy = policy;
    }
    
    public async Task<UserDto> GetUserAsync(int userId)
    {
        var fallbackPolicy = Policy<HttpResponseMessage>
            .Handle<HttpRequestException>()
            .OrResult(r => (int)r.StatusCode >= 500)
            .FallbackAsync(new HttpResponseMessage(HttpStatusCode.ServiceUnavailable)
            {
                Content = new StringContent(JsonSerializer.Serialize(new { error = "Service unavailable" }))
            });
        
        var response = await fallbackPolicy.WrapAsync(_policy)
            .ExecuteAsync(() => _httpClient.GetAsync($"https://localhost:5001/api/users/{userId}"));
        
        response.EnsureSuccessStatusCode();
        
        var json = await response.Content.ReadAsStringAsync();
        return JsonSerializer.Deserialize<UserDto>(json);
    }
}
```

---

### CORS in ASP.NET Core Web API

**Notes:**
- CORS (Cross-Origin Resource Sharing) enables cross-origin requests
- Prevents unauthorized cross-domain access
- Controlled via HTTP headers
- Critical for microservices APIs used by web clients

**Example - CORS Configuration:**

```csharp
// Program.cs - CORS Setup
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowSpecificOrigin", policy =>
    {
        policy.WithOrigins("https://app.example.com", "https://admin.example.com")
              .AllowAnyMethod()
              .AllowAnyHeader()
              .AllowCredentials();
    });
    
    options.AddPolicy("AllowAllOrigins", policy =>
    {
        policy.AllowAnyOrigin()
              .AllowAnyMethod()
              .AllowAnyHeader();
    });
    
    options.AddPolicy("StrictPolicy", policy =>
    {
        policy.WithOrigins("https://trusted.example.com")
              .WithMethods("GET", "POST")
              .WithHeaders("Content-Type", "Authorization")
              .WithExposedHeaders("X-Total-Count")
              .MaxAge(3600);
    });
});

var app = builder.Build();

app.UseCors("AllowSpecificOrigin");

app.Run();

// Controller-level CORS
[ApiController]
[Route("api/[controller]")]
[EnableCors("AllowAllOrigins")]
public class PublicController : ControllerBase
{
    [HttpGet]
    public IActionResult GetPublicData()
    {
        return Ok(new { data = "public" });
    }
}

// Action-level CORS
[ApiController]
[Route("api/[controller]")]
public class ProtectedController : ControllerBase
{
    [HttpGet]
    [EnableCors("StrictPolicy")]
    public IActionResult GetProtectedData()
    {
        return Ok(new { data = "protected" });
    }
    
    [HttpPost]
    [DisableCors]
    public IActionResult CreateData()
    {
        return Created("uri", new { data = "created" });
    }
}
```

---

### Implementing CORS in ASP.NET Core Microservices

**Example - CORS in API Gateway:**

```csharp
// API Gateway Program.cs
var builder = WebApplication.CreateBuilder(args);

// CORS for gateway
builder.Services.AddCors(options =>
{
    options.AddPolicy("GatewayPolicy", policy =>
    {
        policy.WithOrigins(
                builder.Configuration.GetSection("AllowedOrigins")
                    .Get<string[]>() ?? Array.Empty<string>())
              .AllowAnyMethod()
              .AllowAnyHeader()
              .AllowCredentials();
    });
});

builder.Services.AddOcelot();

var app = builder.Build();

app.UseCors("GatewayPolicy");

await app.UseOcelot();

app.Run();

// appsettings.json - Configuration
{
  "AllowedOrigins": [
    "https://app.example.com",
    "https://admin.example.com",
    "http://localhost:3000"
  ],
  "Routes": [
    {
      "DownstreamPathTemplate": "/api/users/{id}",
      "DownstreamScheme": "https",
      "DownstreamHostAndPorts": [
        {
          "Host": "localhost",
          "Port": 5001
        }
      ],
      "UpstreamPathTemplate": "/gateway/users/{id}",
      "UpstreamHttpMethod": ["Get", "Post", "Put", "Delete"]
    }
  ]
}

// Individual Microservice CORS
builder.Services.AddCors(options =>
{
    options.AddPolicy("MicroservicePolicy", policy =>
    {
        // Allow from API Gateway
        policy.WithOrigins("https://localhost:5000")
              .AllowAnyMethod()
              .AllowAnyHeader();
    });
});

var app = builder.Build();

app.UseCors("MicroservicePolicy");

app.Run();
```

---

### OData in ASP.NET Core Web API

**Notes:**
- OData (Open Data Protocol) enables advanced filtering, sorting, pagination
- Reduces client complexity
- Query parameters: $filter, $select, $expand, $orderby, $skip, $top
- Supports complex queries on server side

**Example - OData Implementation:**

```csharp
// Install NuGet: Microsoft.AspNetCore.OData

// Program.cs - OData Setup
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

builder.Services.AddControllers();
builder.Services.AddOData(opt => opt
    .AddRouteComponents("odata", GetEdmModel())
    .Count().Filter().OrderBy().Expand().Select().SetMaxTop(null));

var app = builder.Build();

app.UseRouting();
app.UseEndpoints(endpoints =>
{
    endpoints.MapControllers();
    endpoints.Select().Expand().Filter().OrderBy().Count().MaxTop(null);
    endpoints.MapODataRoute("odata", "odata", GetEdmModel());
});

app.Run();

// EDM Model builder
IEdmModel GetEdmModel()
{
    var builder = new ODataConventionModelBuilder();
    
    builder.EntitySet<User>("Users")
        .EntityType.HasKey(u => u.Id);
    
    builder.EntitySet<Order>("Orders")
        .EntityType.HasKey(o => o.Id);
    
    return builder.GetEdmModel();
}

// OData Controller
[ApiController]
[Route("odata/[controller]")]
public class UsersController : ODataController
{
    private readonly ApplicationDbContext _context;
    
    public UsersController(ApplicationDbContext context)
    {
        _context = context;
    }
    
    [HttpGet]
    [EnableQuery]
    public IActionResult GetUsers()
    {
        return Ok(_context.Users.AsQueryable());
    }
    
    [HttpGet("{id}")]
    [EnableQuery]
    public IActionResult GetUser(int id)
    {
        var user = _context.Users.Find(id);
        if (user == null)
            return NotFound();
        
        return Ok(SingleResult.Create(user.AsQueryable()));
    }
    
    [HttpPost]
    public async Task<IActionResult> CreateUser([FromBody] User user)
    {
        _context.Users.Add(user);
        await _context.SaveChangesAsync();
        
        return Created(user);
    }
    
    [HttpPut("{id}")]
    public async Task<IActionResult> UpdateUser(int id, [FromBody] User user)
    {
        var existingUser = _context.Users.Find(id);
        if (existingUser == null)
            return NotFound();
        
        existingUser.FirstName = user.FirstName;
        existingUser.LastName = user.LastName;
        
        await _context.SaveChangesAsync();
        
        return NoContent();
    }
    
    [HttpDelete("{id}")]
    public async Task<IActionResult> DeleteUser(int id)
    {
        var user = _context.Users.Find(id);
        if (user == null)
            return NotFound();
        
        _context.Users.Remove(user);
        await _context.SaveChangesAsync();
        
        return NoContent();
    }
}

// OData Query Examples
/*
GET /odata/users
- Returns all users

GET /odata/users?$select=id,firstName,lastName
- Returns only id, firstName, lastName

GET /odata/users?$filter=firstName eq 'John'
- Returns users with firstName = "John"

GET /odata/users?$filter=createdAt gt 2024-01-01
- Returns users created after 2024-01-01

GET /odata/users?$orderby=lastName desc
- Returns users sorted by lastName descending

GET /odata/users?$skip=10&$top=20
- Returns 20 users starting from the 11th

GET /odata/users?$expand=orders
- Returns users with their orders

GET /odata/users?$count=true
- Returns total user count
*/
```

---

### Implementing OData in ASP.NET Core Microservices

**Example - Advanced OData with Custom Operations:**

```csharp
// Custom OData operations
var builder = new ODataConventionModelBuilder();

var usersEntitySet = builder.EntitySet<User>("Users");
var getHighValueUsersAction = usersEntitySet.Collection
    .Action("GetHighValueUsers")
    .Returns<IEnumerable<User>>();
getHighValueUsersAction.Parameter<decimal>("minSpent");

builder.EntitySet<Order>("Orders");

var model = builder.GetEdmModel();

// OData Controller with custom actions
[ApiController]
[Route("odata/[controller]")]
public class UsersController : ODataController
{
    private readonly IUserService _userService;
    
    public UsersController(IUserService userService)
    {
        _userService = userService;
    }
    
    [HttpGet]
    [EnableQuery]
    public IActionResult GetUsers()
    {
        return Ok(_userService.GetAllUsers().AsQueryable());
    }
    
    [HttpPost("odata/Users/GetHighValueUsers")]
    public async Task<IActionResult> GetHighValueUsers([FromBody] ODataActionParameters parameters)
    {
        var minSpent = (decimal)parameters["minSpent"];
        var users = await _userService.GetHighValueUsersAsync(minSpent);
        
        return Ok(users);
    }
    
    [HttpGet("odata/Users({id})/Orders")]
    [EnableQuery]
    public IActionResult GetUserOrders(int id)
    {
        var orders = _userService.GetUserOrders(id).AsQueryable();
        return Ok(orders);
    }
}
```

---

## Conclusion

This comprehensive guide covers all major aspects of ASP.NET Core Microservices architecture including:

1. **Foundational Concepts:** Understanding microservices architecture and design principles
2. **Clean Architecture & DDD:** Building maintainable, domain-focused services
3. **Communication Patterns:** REST, gRPC, and message-based communication
4. **Data Management:** Saga pattern, event sourcing, and distributed data consistency
5. **API Gateway:** Request routing, aggregation, and cross-cutting concerns
6. **Advanced Patterns:** CQRS, GraphQL, Circuit Breaker, and more
7. **Cross-Cutting Concerns:** Authentication, CORS, OData, and advanced querying

---

**To convert this to PDF:**

1. **Using Microsoft Word:**
   - Copy all content
   - Paste into Word
   - File → Export as PDF

2. **Using Google Docs:**
   - Create new document
   - Paste content
   - File → Download → PDF Document

3. **Using Pandoc (CLI):**
   ```bash
   pandoc -f markdown -t pdf input.md -o output.pdf
   ```

4. **Using Online Converter:**
   - Visit markdown-to-pdf.com
   - Paste content
   - Download PDF

This comprehensive guide provides solid foundations for building scalable, maintainable microservices with ASP.NET Core!
