# Observer Pattern

---

## 1. Definition

The **Observer Pattern** establishes a one-to-many dependency between objects. When one object (the **Subject**) changes state, all its dependents (**Observers**) are notified and updated automatically.

---

## 2. The Idea

Instead of polling (repeatedly checking for changes), observers subscribe to the subject and receive **push** notifications when state changes.

```
Traditional: Observer ──► Check? ──► Check? ──► (wasteful polling)

Observer Pattern:
Subject Changes ──► Notify All Observers ──► Observer A Updates
                      ├──► Observer B Updates
                      └──► Observer C Updates
```

**Core Components:**
- **Subject** — Maintains state and observer list; notifies on change
- **Observer** — Interface with update method called by subject
- **Loose Coupling** — Subject knows only the observer interface, not concrete classes

---

## 3. Example

```csharp
// Subject
public class StockPrice
{
    private decimal _price;
    public event EventHandler<PriceChangedEventArgs> PriceChanged;

    public decimal Price
    {
        get => _price;
        set
        {
            if (_price == value) return;
            var oldPrice = _price;
            _price = value;
            PriceChanged?.Invoke(this, new PriceChangedEventArgs(oldPrice, value));
        }
    }
}

public class PriceChangedEventArgs : EventArgs
{
    public decimal OldPrice { get; } // read-only props
    public decimal NewPrice { get; }
    public PriceChangedEventArgs(decimal oldPrice, decimal newPrice)
    {
        OldPrice = oldPrice;
        NewPrice = newPrice;
    }
}

// Observer
public class StockTrader
{
    public void OnPriceChanged(object sender, PriceChangedEventArgs e)
    {
        var action = e.NewPrice > e.OldPrice ? "Selling" : "Buying";
        Console.WriteLine($"{action} at {e.NewPrice}");
    }
}

// Usage
var stock = new StockPrice { Price = 100 };
var trader = new StockTrader();

stock.PriceChanged += trader.OnPriceChanged;
stock.Price = 120;  // Automatically notifies trader
```

---

## 4. When to Use It

| Situation | Details |
|-----------|---------|
| **Event Handling** | UI elements reacting to user actions or data changes |
| **Distributed Systems** | Microservices reacting to state changes (pub/sub) |
| **MVC Architecture** | Model changes requiring multiple View updates |
| **Real-time Data** | Stock prices, sensors, chat messages, live feeds |
| **Caching** | Invalidating caches across services when source changes |
| **Audit/Logging** | Multiple loggers tracking state changes without modifying source |

---

## 5. Advantages and Disadvantages

### Advantages

- **Loose Coupling** — Subject knows only the interface, not concrete observers
- **Following OPC Principle** — Add new observers without modifying subject
- **Automatic Updates** — No polling; changes propagate instantly
- **Flexible** — Supports one-to-many and many-to-many relationships

### Disadvantages

- **Memory Leaks** — Observers must unsubscribe; dangling references prevent GC
- **Debugging** — Hard to trace implicit invocation chains
- **Thread Safety** — Adding/removing observers during notification requires synchronization

---

## Quick Summary

```
Observer Pattern
│
├── Subject (Observable)
│   ├── Maintains state and observer list
│   ├── Attach/Detach methods
│   └── Notifies observers on change
│
├── Observer
│   ├── Update interface
│   └── Reacts to subject notifications
│
├── Key Characteristics
│   ├── Push model (subject pushes updates)
│   ├── Loose coupling (interface-based)
│   └── One-to-many relationship
│
├── C# Implementation
│   ├── Events/Delegates (most common)
│   ├── IObserver<T>/IObservable<T> (built-in)
│   └── Custom interfaces
│
└── Trade-off: Flexibility vs. Memory management complexity
```