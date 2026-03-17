# Events in C#

---

## 1. Definition

An Event is a mechanism that allows an object to **notify other objects** when something happens.
it announces that something occurred in system,
---

## 2. The Idea 

Imagine a button in a UI. The button does not know what should happen when it is clicked
it does not know if it should save a file, open a window, or send a message.
All it knows is: *"I was clicked"*

core idea:
- **Publisher** — the object that raises the event
- **Subscriber** — the object that listens and reacts
- **Event** — the channel between them
- **Handler** — the method that runs when the event fires

---

## 3. Example

```csharp
// Define a delegate type for the event signature
public delegate void ProductAddedHandler(string productName);

// Publisher class that declares the event
public class ProductCatalog
{
    // Declare the event using the delegate
    public event ProductAddedHandler ProductAdded;

    public void AddProduct(string name)
    {
        Console.WriteLine($"Adding {name} to catalog...");

        // Raise the event
        ProductAdded?.Invoke(name);
    }
}
```

```csharp
// Subscriber that listens to the event
public class NotificationService
{
    public void OnProductAdded(string productName)
    {
        Console.WriteLine($"Notification sent: '{productName}' was added!");
    }
}

public class InventoryService
{
    public void OnProductAdded(string productName)
    {
        Console.WriteLine($"Inventory updated for: '{productName}'");
    }
}
```

```csharp
// Wire everything together
var catalog      = new ProductCatalog();
var notification = new NotificationService();
var inventory    = new InventoryService();

// Subscribe to the event
catalog.ProductAdded += notification.OnProductAdded;
catalog.ProductAdded += inventory.OnProductAdded;

// Trigger the event
catalog.AddProduct("Laptop");

// Output:
// Adding Laptop to catalog...
// Notification sent: 'Laptop' was added!
// Inventory updated for: 'Laptop'
```

```csharp
// built-in EventHandler no custom delegate needed
public class ProductCatalog
{
    public event EventHandler<string> ProductAdded;

    public void AddProduct(string name)
    {
        ProductAdded?.Invoke(this, name);
    }
}
```

```csharp
// Unsubscribing from an event
catalog.ProductAdded -= notification.OnProductAdded;

// Now only inventory gets notified
catalog.AddProduct("Phone");

// Output:
// Adding Phone to catalog...
// Inventory updated for: 'Phone'
```

---

## 4. When to Use It

| Situation | Details |
|-----------|---------|
| **UI interactions** | Button clicks, form submissions, keyboard input |
| **Decoupled communication** | When one object needs to notify others without knowing who they are |
| **Domain events** | When something important happens in business logic (order placed, payment received) |
| **Logging and monitoring** | When you want to observe actions without modifying the source class |
| **Plugin and extension systems** | When external components need to react to internal system changes |

> **Practical rule:** Use events when the publisher **(object raises event)** should have **zero knowledge** of its subscribers **(object receives events)**.
> If the publisher knows exactly who to call.
> If the publisher announces something and does not care who responds.

---

## 5. Advantages and Disadvantages

### Advantages

- **Loose coupling** — publisher and subscriber do not depend on each other directly
- **Multiple subscribers** — many objects can listen to the same event simultaneously
- **Clean separation of concerns** — the object that raises the event does not handle the response
- **Built into the language** — first-class support with `event`, `+=`, `-=` syntax
- **Safe by design** — subscribers cannot invoke or clear the event from outside the publisher

### Disadvantages

- **Memory leaks risk** — forgetting to unsubscribe (`-=`) keeps objects alive in memory longer than needed
- **Hard to trace** — when many subscribers are attached, it can be difficult to follow the execution flow
- **Overhead for simple cases** — for straightforward one-to-one communication, a direct method call is cleaner

---

## Quick Summary

```
Event
│
├── Built on top of Delegates
├── Publisher raises it — has no knowledge of subscribers
├── Subscribers register with += and unregister with -=
├── Multiple subscribers can listen simultaneously
├── Use EventHandler<T> for standard .NET conventions
└── Always unsubscribe when the listener is no longer needed
```