# Event-Driven Architecture
 
---

## 1. Definition

Event-Driven Architecture (EDA) is a software design pattern where the flow of the program
is determined by **events**.

Instead of components calling each other **directly**, each component simply **announces that something happened**,
and any other component that cares will react to it independently.

An event can be anything meaningful: a user placed an order, a payment was received,
a file was uploaded, a sensor detected a change. The component that produces the event.
 
---

## 2. The Idea

In traditional architecture, components are **tightly coupled** — Service A calls Service B directly,
waits for a response, then continues. If Service B is slow or down, Service A is affected immediately (High coupling).

Event-Driven Architecture breaks that dependency entirely.

The station does not call each listener individually —
it just broadcasts, and whoever is tuned in responds.

the core idea:
- **Event Producer** — the component that detects something and announces it.
- **Event** — the message describing what happened (e.g. `OrderPlaced`, `PaymentReceived`)
- **Event Broker** — the channel that carries the event (e.g. Kafka, RabbitMQ, Azure Service Bus)
- **Event Consumer** — the component that listens and reacts to the event

---

## 3. Example

```
Traditional (Tightly Coupled) Flow:
─────────────────────────────────────
User places order
    → OrderService calls PaymentService directly
    → OrderService calls InventoryService directly
    → OrderService calls NotificationService directly
    → If any service is down — the whole flow breaks
```

```
Event-Driven Flow:
──────────────────
User places order
    → OrderService publishes event: "OrderPlaced"
         ↓
    [Event Broker — e.g. RabbitMQ / Kafka]
         ↓              ↓               ↓
  PaymentService  InventoryService  NotificationService
  reacts on its   reacts on its     reacts on its
  own time        own time          own time
 
→ OrderService does not know or care who reacted
→ Each service is independent — one failing does not break the others
```

```csharp
// The event message
public class OrderPlacedEvent
{
    public int      OrderId     { get; set; }
    public string   ProductName { get; set; }
    public double   TotalPrice  { get; set; }
    public DateTime PlacedAt    { get; set; }
}
 
// The event broker
public class EventBus
{
    private readonly Dictionary<Type, List<Action<object>>> _handlers = new();
 
    public void Subscribe<T>(Action<T> handler)
    {
        var type = typeof(T);
        if (!_handlers.ContainsKey(type))
            _handlers[type] = new List<Action<object>>();
 
        _handlers[type].Add(e => handler((T)e));
    }
 
    public void Publish<T>(T eventMessage)
    {
        var type = typeof(T);
        if (_handlers.ContainsKey(type))
            foreach (var handler in _handlers[type])
                handler(eventMessage);
    }
}
```

```csharp
// Event consumers — each reacts independently
public class PaymentService
{
    public void Handle(OrderPlacedEvent e)
        => Console.WriteLine($"Payment of ${e.TotalPrice} processed for Order #{e.OrderId}");
}
 
public class InventoryService
{
    public void Handle(OrderPlacedEvent e)
        => Console.WriteLine($"Stock updated for: {e.ProductName}");
}
 
public class NotificationService
{
    public void Handle(OrderPlacedEvent e)
        => Console.WriteLine($"Email sent: Your order #{e.OrderId} has been placed!");
}
```

```csharp
// Wire everything together
var bus          = new EventBus();
var payment      = new PaymentService();
var inventory    = new InventoryService();
var notification = new NotificationService();
 
// Consumers subscribe to the event
bus.Subscribe<OrderPlacedEvent>(payment.Handle);
bus.Subscribe<OrderPlacedEvent>(inventory.Handle);
bus.Subscribe<OrderPlacedEvent>(notification.Handle);
 
// Producer publishes — does not know who is listening
bus.Publish(new OrderPlacedEvent
{
    OrderId     = 101,
    ProductName = "Laptop",
    TotalPrice  = 1200,
    PlacedAt    = DateTime.Now
});
 
// Output:
// Payment of $1200 processed for Order #101
// Stock updated for: Laptop
// Email sent: Your order #101 has been placed!
```
 
---

## 4. When to Use It

| Situation | Details |
|-----------|---------|
| **Microservices** | When multiple independent services need to react to the same business action |
| **High scalability needs** | When components must scale independently without blocking each other |
| **Async workflows** | When tasks can run in parallel and do not need an immediate response |
| **Audit and logging** | When every action needs to be recorded without modifying the source service |
| **Notification systems** | When multiple channels (email, SMS, push) respond to the same trigger |
| **Real-time data processing** | When events stream continuously and need to be processed as they arrive |

> **Practical rule:** Use Event-Driven Architecture when services need to **react** to things
> rather than **be called** by things. If your system has grown to a point where one service
> is calling five others directly — that is the signal to consider EDA.
 
---

## 5. Advantages and Disadvantages

### Advantages

- **Loose coupling** — producers and consumers have zero direct dependency on each other
- **Independent scalability** — each consumer can scale on its own without affecting others
- **Fault isolation** — if one consumer fails, the others continue working unaffected
- **Extensibility** — adding a new consumer requires no change to the producer whatsoever
- **Natural async support** — events are processed asynchronously by default, improving throughput
- **Full audit trail** — every event can be stored and replayed, giving a complete history of what happened

### Disadvantages

- **Eventual consistency** — data across services may be temporarily out of sync after an event fires
- **Harder to debug** — tracing a flow across multiple consumers is more complex than a direct call chain
- **Infrastructure overhead** — requires a message broker (Kafka, RabbitMQ, etc.) which adds operational complexity
- **Event versioning** — when event structures change, all consumers must be updated carefully
- **No guaranteed order** — in distributed systems, events may arrive out of order if not handled explicitly

---

## Quick Summary

```
Event-Driven Architecture
│
├── Producer publishes an event — does not know who consumes it
├── Event Broker carries the event (Kafka, RabbitMQ, Azure Service Bus)
├── Consumers react independently and asynchronously
├── Services are fully decoupled — one failure does not break others
├── Best for microservices, async workflows, and scalable systems
└── Trade-off: simpler coupling vs. eventual consistency and debug complexity
```