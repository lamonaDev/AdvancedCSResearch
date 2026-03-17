# Expression Trees

---

## 1. Definition

**Expression Trees** are data structures that represent code as a `tree` hierarchy of expressions.

In C#, when you write a lambda expression like `x => x.Price > 100`, the compiler normally compiles this into IL (Intermediate Language) **executable code**, when assigned to an `Expression<T>` instead of a `Func<T>`, the compiler instead emits instructions to build a tree structure representing that logic at runtime.

Each node in the tree represents an **operation**: method calls, property access.

---

## 2. The Idea

In traditional programming, code is **compiled** into instructions that the CPU executes. Once compiled, the original structure (variable names, operations, comparisons) is lost—it becomes opaque machine code.

Expression Trees flip this model: the code remains **data** that can be traversed programmatically.

```
Traditional Delegate (Func<T>):
────────────────────────────────
Code: x => x.Price > 100
↓
Compiled to IL instructions (opaque, executable only)
↓
Direct execution (CPU runs the logic)

Expression Tree (Expression<Func<T>>):
──────────────────────────────────────
Code: x => x.Price > 100
↓
Compiled to object graph (inspectable data structure)
↓
[Lambda]
│
[GreaterThan]
│           │
[Member]    [Constant]
(Price)       (100)
↑
Can inspect: "Oh, it's comparing a Member 'Price' against Constant 100"
↓
Can translate: Convert to SQL, modify tree, or compile to delegate
```

The core idea:
- **Code as Data** — Lambda logic becomes traversable objects (`BinaryExpression`, `MemberExpression`, etc.)
- **Deferred Translation** — LINQ providers walk the tree to translate C# syntax into SQL, REST APIs, or other target languages
- **Dynamic Compilation** — Trees can be compiled to delegates at runtime for execution

---

## 3. Example

```csharp
using System.Linq.Expressions;

// Traditional Delegate (Compiled Code)
Func<Product, bool> delegateFilter = p => p.Price > 100;

// Opaque: Cannot see inside, just execute
var result = products.Where(delegateFilter); // In-memory filtering only

// Expression Tree (Data Structure)
Expression<Func<Product, bool>> exprFilter = p => p.Price > 100;
// Transparent: Can inspect the tree structure

// Inspecting the tree
Console.WriteLine(exprFilter.Parameters[0]);           // Parameter: p
Console.WriteLine(exprFilter.Body.NodeType);           // GreaterThan
Console.WriteLine(((MemberExpression)((BinaryExpression)exprFilter.Body).Left).Member.Name); // Price
```

### Real-World: Dynamic Query Builder
```csharp
public static Expression<Func<T, bool>> BuildFilter<T>(
    string propertyName, object value)
{
    var param = Expression.Parameter(typeof(T), "x");
    var property = Expression.Property(param, propertyName);
    var constant = Expression.Constant(value);
    var equality = Expression.Equal(property, constant);

    return Expression.Lambda<Func<T, bool>>(equality, param);
}

// Usage: Build filter at runtime based on user input
var filter = BuildFilter<Product>("Category", "Electronics");
// Entity Framework translates this to SQL: WHERE Category = 'Electronics'
var products = dbContext.Products.Where(filter).ToList();
```

---

## 4. When to Use It

| Situation | Details |
|-----------|---------|
| **ORM Queries (Entity Framework)** | EF Core walks expression trees to translate `Where(p => p.Price > 100)` into SQL `WHERE Price > 100` |
| **Dynamic Filtering/Sorting** | When users choose filter criteria at runtime and you need to build queries programmatically |
| **Serialization of Logic** | Sending filtering/sorting logic over the wire to be executed remotely (e.g., OData, GraphQL) |
| **Compiler/Interpreter Writing** | Building scripting languages or domain-specific languages (DSLs) inside C# |

> **Practical rule:** Use Expression Trees when you need to **inspect, translate** code logic rather than just execute it.

---

**Advantages**
Expression Trees enable **translation** of C# code into other languages (like SQL for databases), allow **inspection** of code structure at runtime without executing it, support **dynamic composition** of logic (combining simple filters into complex queries), offer **deferred compilation** (pay the cost only when needed), and maintain **type safety** through compile-time checking of the lambda syntax.

**Disadvantages**
They have **limited syntax** (no statements, assignments, async/await, or ref/out parameters), carry a **performance penalty** (compiling trees is ~100x slower than creating delegates), feature a **verbose/complex API** for manual construction, consume **more memory** than raw delegates, and are **harder to debug** than regular code.
---

## Quick Summary

```
Expression Trees
│
├── Concept: Code represented as traversable data structure (tree of objects)
├── Namespace: System.Linq.Expressions (Expression<T>, ParameterExpression, etc.)
├── Key Difference: Func<T> = compiled code; Expression<Func<T>> = code blueprint
│
├── Use Cases
│   ├── ORM translation (LINQ to SQL/Entities)
│   ├── Dynamic query construction (runtime filters)
│   ├── Rule engines (business logic as data)
│   └── Reflection replacement (faster property access via compiled expressions)
│
├── Lifecycle
│   ├── Write: Lambda assigned to Expression<T> OR built manually via API
│   ├── Inspect: Walk tree nodes (BinaryExpression, MemberExpression, etc.)
│   ├── Modify: Combine with other expressions or change parameters
│   ├── Compile: Convert to executable Func<T> (costly operation)
│   └── Execute: Run the compiled delegate
│
└── Trade-off: Flexibility and introspection vs. Performance and complexity
```