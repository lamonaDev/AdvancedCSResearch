# Anonymous Types in C#

---

## 1. Definition

Anonymous Types are temporary data types created at runtime without needing to define a class before.
The compiler generates them automatically and assigns them an internal name that you never see.
They are used to group a set of properties together into a single object quickly and inline.

---

## 2. The Idea

In normal C# code, if you want to group multiple values together, you need to define a full class or struct.
Anonymous Types letting you create an object with specific properties where you need it.

The compiler handles the type behind the scenes. The properties it creates are **read-only** —
once the object is created, its values cannot be changed.

The most common place you will see anonymous types is inside **LINQ queries**, where you want to
project only a few fields out of a larger object without creating a dedicated class for it.

---

## 3. Example

```csharp
// inline anonoymous type -> 
var product = new { Name = "Laptop", Price = 1200, Category = "Electronics" };

Console.WriteLine(product.Name);      // Laptop
Console.WriteLine(product.Price);     // 1200
Console.WriteLine(product.Category);  // Electronics
```

```csharp
// LINQ code example, in this you need few fields only not the hole object of data
List<Product> catalog = ProductsCatalog.Catalog;

var summary = catalog
    .Where(p => p.Category == "Electronics")
    .Select(p => new { p.Name, p.Price })
    .ToList();

foreach (var item in summary)
{
    Console.WriteLine($"{item.Name} - ${item.Price}");
}

// Output:
// Laptop - $1200
// Phone - $800
// Headphones - $150
```

```csharp
// Without Anonymous Type you need a full class
public class ProductSummary
{
    public string Name { get; set; }
    public double Price { get; set; }
}

// With Anonymous Type no class needed
var summary = catalog.Select(p => new { p.Name, p.Price });
```

---

## 4. When to Use It

| Situation | Details |
|-----------|---------|
| **LINQ projections** | When you need only specific fields from a collection |
| **Temporary data** | When you need an object once and it does not deserve a full class |
| **Combining results** | When merging data from multiple sources into a temporary shape |
| **Quick prototyping** | When testing an idea or writing throwaway code |

> **Practical rule:** If you need the type in more than one place or want to pass it between methods use a real class.
> If you need it in one place only, just once Anonymous Type is fit for this situation.

---

## 5. Advantages and Disadvantages

### Advantages

- **Fast to write** — no need to declare a new class anywhere
- **Cleaner codebase** — temporary data does not pollute your project with unnecessary classes
- **Full LINQ support** — integrates naturally with `Select`, `GroupBy`, and other operators
- **Type-safe** — the compiler knows the type and catches errors at compile time
- **Read-only by design** — prevents accidental modification of temporary data

### Disadvantages

- **Cannot be passed between methods** — no type name exists to use as a parameter type
- **Read-only only** — properties cannot be modified after creation
- **No inheritance support** — cannot inherit from or be inherited by other types
- **Harder to debug** — the compiler-generated internal name is unreadable
- **Scoped to the method** — its lifetime is limited to the method it was created in

---

## Quick Summary

```
Anonymous Type
│
├── Created automatically by the compiler
├── Properties are read-only
├── Best used with LINQ projections
├── Temporary — cannot be passed between methods
└── A lightweight alternative to a class for one-time data shapes
```

