# 12-16. RabbitMQ and Inter-Service Communication

## Notes

### What is RabbitMQ?

RabbitMQ is a message broker that enables asynchronous communication between services. It uses the AMQP (Advanced Message Queuing Protocol) standard.

### Key Concepts

- **Producer**: Service that sends messages
- **Consumer**: Service that receives messages
- **Queue**: Buffer for messages
- **Exchange**: Routes messages to queues
- **Binding**: Connects exchange to queue

### Exchange Types

- **Direct**: Routes based on exact routing key match
- **Fanout**: Routes to all bound queues
- **Topic**: Routes based on pattern matching
- **Headers**: Routes based on message headers

---

## C# Code Examples

### RabbitMQ Connection Setup

```csharp
namespace Shared.MessageQueue
{
    using RabbitMQ.Client;
    using System;

    public interface IMessageQueueConnection
    {
        IModel CreateChannel();
        void Dispose();
    }

    public class RabbitMQConnection : IMessageQueueConnection
    {
        private readonly IConnection _connection;
        private readonly IModel _channel;

        public RabbitMQConnection(string hostName, string userName, string password)
        {
            var factory = new ConnectionFactory
            {
                HostName = hostName,
                UserName = userName,
                Password = password,
                DispatchConsumersAsync = true
            };

            _connection = factory.CreateConnection();
            _channel = _connection.CreateModel();
        }

        public IModel CreateChannel() => _connection.CreateModel();

        public void Dispose()
        {
            _channel?.Close();
            _connection?.Close();
            _channel?.Dispose();
            _connection?.Dispose();
        }
    }
}
```

### Event Publisher

```csharp
namespace Shared.MessageQueue
{
    using RabbitMQ.Client;
    using System;
    using System.Text;
    using System.Text.Json;
    using System.Threading.Tasks;

    public interface IEventPublisher
    {
        Task PublishAsync<T>(T @event, string exchangeName) where T : class;
        Task PublishAsync<T>(T @event, string exchangeName, string routingKey) where T : class;
    }

    public class RabbitMQEventPublisher : IEventPublisher
    {
        private readonly IMessageQueueConnection _connection;
        private readonly ILogger<RabbitMQEventPublisher> _logger;

        public RabbitMQEventPublisher(IMessageQueueConnection connection, ILogger<RabbitMQEventPublisher> logger)
        {
            _connection = connection;
            _logger = logger;
        }

        public async Task PublishAsync<T>(T @event, string exchangeName) where T : class
        {
            await PublishAsync(@event, exchangeName, string.Empty);
        }

        public async Task PublishAsync<T>(T @event, string exchangeName, string routingKey) where T : class
        {
            try
            {
                using (var channel = _connection.CreateChannel())
                {
                    channel.ExchangeDeclare(
                        exchange: exchangeName,
                        type: ExchangeType.Topic,
                        durable: true,
                        autoDelete: false);

                    var eventJson = JsonSerializer.Serialize(@event);
                    var body = Encoding.UTF8.GetBytes(eventJson);

                    var properties = channel.CreateBasicProperties();
                    properties.Persistent = true;
                    properties.ContentType = "application/json";
                    properties.Timestamp = new AmqpTimestamp(DateTimeOffset.UtcNow.ToUnixTimeSeconds());

                    channel.BasicPublish(
                        exchange: exchangeName,
                        routingKey: routingKey,
                        basicProperties: properties,
                        body: body);

                    _logger.LogInformation($"Event published: {typeof(T).Name} with routing key: {routingKey}");
                }

                await Task.CompletedTask;
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, $"Error publishing event: {typeof(T).Name}");
                throw;
            }
        }
    }
}
```

### Event Subscriber

```csharp
namespace Shared.MessageQueue
{
    using RabbitMQ.Client;
    using RabbitMQ.Client.Events;
    using System;
    using System.Text;
    using System.Text.Json;
    using System.Threading.Tasks;

    public interface IEventSubscriber
    {
        Task SubscribeAsync<T>(Func<T, Task> handler, string exchangeName, string queueName, string routingKey) where T : class;
    }

    public class RabbitMQEventSubscriber : IEventSubscriber
    {
        private readonly IMessageQueueConnection _connection;
        private readonly ILogger<RabbitMQEventSubscriber> _logger;

        public RabbitMQEventSubscriber(IMessageQueueConnection connection, ILogger<RabbitMQEventSubscriber> logger)
        {
            _connection = connection;
            _logger = logger;
        }

        public async Task SubscribeAsync<T>(Func<T, Task> handler, string exchangeName, string queueName, string routingKey) where T : class
        {
            try
            {
                using (var channel = _connection.CreateChannel())
                {
                    channel.ExchangeDeclare(
                        exchange: exchangeName,
                        type: ExchangeType.Topic,
                        durable: true,
                        autoDelete: false);

                    channel.QueueDeclare(
                        queue: queueName,
                        durable: true,
                        exclusive: false,
                        autoDelete: false);

                    channel.QueueBind(
                        queue: queueName,
                        exchange: exchangeName,
                        routingKey: routingKey);

                    var consumer = new AsyncEventingBasicConsumer(channel);

                    consumer.Received += async (model, ea) =>
                    {
                        try
                        {
                            var body = ea.Body.ToArray();
                            var json = Encoding.UTF8.GetString(body);
                            var @event = JsonSerializer.Deserialize<T>(json);

                            await handler(@event);

                            channel.BasicAck(ea.DeliveryTag, false);
                            _logger.LogInformation($"Event processed: {typeof(T).Name}");
                        }
                        catch (Exception ex)
                        {
                            _logger.LogError(ex, $"Error processing event: {typeof(T).Name}");
                            channel.BasicNack(ea.DeliveryTag, false, true);
                        }
                    };

                    channel.BasicConsume(queue: queueName, autoAck: false, consumer: consumer);

                    _logger.LogInformation($"Subscribed to queue: {queueName}");
                    await Task.Delay(Timeout.Infinite);
                }
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, $"Error subscribing to queue: {queueName}");
                throw;
            }
        }
    }
}
```

### Shared Domain Events

```csharp
namespace Shared.Events
{
    using System;

    public abstract class DomainEvent
    {
        public Guid EventId { get; set; } = Guid.NewGuid();
        public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
        public string EventType { get; set; }
    }

    // User Service Events
    public class UserCreatedEvent : DomainEvent
    {
        public int UserId { get; set; }
        public string Email { get; set; }
        public string FirstName { get; set; }
        public string LastName { get; set; }
    }

    public class UserUpdatedEvent : DomainEvent
    {
        public int UserId { get; set; }
        public string Email { get; set; }
    }

    // Order Service Events
    public class OrderCreatedEvent : DomainEvent
    {
        public int OrderId { get; set; }
        public int CustomerId { get; set; }
        public decimal TotalAmount { get; set; }
    }

    public class OrderPlacedEvent : DomainEvent
    {
        public int OrderId { get; set; }
        public int CustomerId { get; set; }
        public List<OrderItemDto> Items { get; set; }
    }

    // Product Service Events
    public class ProductStockUpdatedEvent : DomainEvent
    {
        public int ProductId { get; set; }
        public int NewStockQuantity { get; set; }
    }

    public class OrderItemDto
    {
        public int ProductId { get; set; }
        public int Quantity { get; set; }
        public decimal Price { get; set; }
    }
}
```

### Using RabbitMQ in a Microservice

```csharp
// UserService Integration
namespace UserService.API
{
    using Microsoft.AspNetCore.Builder;
    using Microsoft.Extensions.DependencyInjection;
    using Shared.MessageQueue;
    using UserService.Infrastructure.EventHandlers;

    public static class RabbitMQExtensions
    {
        public static IServiceCollection AddRabbitMQ(this IServiceCollection services, IConfiguration configuration)
        {
            var rabbitMQConfig = configuration.GetSection("RabbitMQ");
            var hostName = rabbitMQConfig["HostName"];
            var userName = rabbitMQConfig["UserName"];
            var password = rabbitMQConfig["Password"];

            services.AddSingleton<IMessageQueueConnection>(
                new RabbitMQConnection(hostName, userName, password));

            services.AddScoped<IEventPublisher, RabbitMQEventPublisher>();
            services.AddScoped<IEventSubscriber, RabbitMQEventSubscriber>();

            return services;
        }

        public static WebApplicationBuilder AddRabbitMQSubscribers(this WebApplicationBuilder builder)
        {
            var subscriber = builder.Services.BuildServiceProvider().GetRequiredService<IEventSubscriber>();

            // Subscribe to events
            // Example: Product stock updated event
            subscriber.SubscribeAsync<ProductStockUpdatedEvent>(
                handler: async @event => await HandleProductStockUpdated(@event),
                exchangeName: "product.events",
                queueName: "user-service.product.stock.updated",
                routingKey: "product.stock.updated"
            ).GetAwaiter().GetResult();

            return builder;
        }

        private static async Task HandleProductStockUpdated(ProductStockUpdatedEvent @event)
        {
            // Handle the event
            Console.WriteLine($"Product {event.ProductId} stock updated to {event.NewStockQuantity}");
            await Task.CompletedTask;
        }
    }
}
```

### Publishing Events from Service

```csharp
namespace UserService.Application.Services
{
    using Shared.MessageQueue;
    using Shared.Events;

    public class UserApplicationService
    {
        private readonly IUnitOfWork _unitOfWork;
        private readonly IEventPublisher _eventPublisher;
        private readonly ILogger<UserApplicationService> _logger;

        public UserApplicationService(
            IUnitOfWork unitOfWork,
            IEventPublisher eventPublisher,
            ILogger<UserApplicationService> logger)
        {
            _unitOfWork = unitOfWork;
            _eventPublisher = eventPublisher;
            _logger = logger;
        }

        public async Task<UserDto> CreateUserAsync(CreateUserDto dto)
        {
            _logger.LogInformation($"Creating user: {dto.Email}");

            var user = new User
            {
                FirstName = dto.FirstName,
                LastName = dto.LastName,
                Email = dto.Email,
                PhoneNumber = dto.PhoneNumber,
                Role = Enum.Parse<UserRole>(dto.Role),
                IsActive = true
            };

            var createdUser = await _unitOfWork.Users.AddAsync(user);
            await _unitOfWork.SaveChangesAsync();

            // Publish event
            await _eventPublisher.PublishAsync(
                new UserCreatedEvent
                {
                    UserId = createdUser.Id,
                    Email = createdUser.Email,
                    FirstName = createdUser.FirstName,
                    LastName = createdUser.LastName
                },
                exchangeName: "user.events",
                routingKey: "user.created"
            );

            _logger.LogInformation($"User {createdUser.Id} created and event published");
            return MapToDto(createdUser);
        }
    }
}
```

### Program.cs Configuration

```csharp
var builder = WebApplication.CreateBuilder(args);

// Add RabbitMQ
builder.Services.AddRabbitMQ(builder.Configuration);

builder.Services.AddControllers();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Configure RabbitMQ subscribers
app.AddRabbitMQSubscribers();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

### appsettings.json

```json
{
  "RabbitMQ": {
    "HostName": "localhost",
    "Port": 5672,
    "UserName": "guest",
    "Password": "guest",
    "VirtualHost": "/"
  },
  "MessageQueue": {
    "UserService": {
      "ExchangeName": "user.events",
      "QueueName": "user-service.queue"
    },
    "OrderService": {
      "ExchangeName": "order.events",
      "QueueName": "order-service.queue"
    },
    "ProductService": {
      "ExchangeName": "product.events",
      "QueueName": "product-service.queue"
    }
  }
}
```

---

## RabbitMQ Installation (Windows)

```powershell
# Using Chocolatey
choco install rabbitmq

# Or download from: https://www.rabbitmq.com/install-windows.html

# Start RabbitMQ service
Start-Service RabbitMQ

# Enable management plugin
rabbitmq-plugins enable rabbitmq_management

# Access management UI: http://localhost:15672
# Default credentials: guest/guest
```

### Docker Setup

```yaml
services:
  rabbitmq:
    image: rabbitmq:3.9-management
    environment:
      RABBITMQ_DEFAULT_USER: guest
      RABBITMQ_DEFAULT_PASS: guest
    ports:
      - "5672:5672"     # AMQP port
      - "15672:15672"   # Management UI
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq

volumes:
  rabbitmq_data:
```

---

## Best Practices

✓ Use persistent queues
✓ Implement retry logic
✓ Handle dead letter queues
✓ Monitor message throughput
✓ Use transactional messaging
✓ Implement idempotent handlers
✓ Add distributed tracing
✓ Document event contracts

---

## NuGet Packages Required

```bash
dotnet add package RabbitMQ.Client
dotnet add package MassTransit (Alternative higher-level abstraction)
dotnet add package NServiceBus (Another alternative)
```
