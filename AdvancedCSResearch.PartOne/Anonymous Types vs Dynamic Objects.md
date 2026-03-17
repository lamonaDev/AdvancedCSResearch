# Anonymous Types vs Dynamic Objects

---

## 1. Definition

In C#, **Anonymous Types** and **Dynamic Objects** (often colloquially called "Anonymous Objects") are two different approaches to create objects without explicitly defining a class, but they behave fundamentally differently.

**Anonymous Types** are compiler-generated, immutable reference types created using the `new { ... }` syntax. The compiler automatically creates a class with read-only properties at compile-time.

**Dynamic Objects** (typically `ExpandoObject` or `dynamic`) are runtime-generated objects where properties can be added, removed, or modified dynamically after creation, with type checking deferred until runtime.

---

## 2. The Idea

The core difference lies in **when** the object structure is defined and **how** type safety is enforced.

**Anonymous Types** are like having a temporary contract written at `compile-time`: the compiler secretly creates a real class behind the scenes (e.g., `<>f__AnonymousType0`), and the structure is locked forever. You get IntelliSense, compile-time checking, and type safety, but you cannot change the shape later.

**Dynamic Objects** are like a blank notebook at `runtime`: you start with an empty object and write properties into it as needed. There is no compiler-generated class, no IntelliSense, and errors only surface when the code runs. This offers flexibility similar to JavaScript objects.

```
Anonymous Types:    Compile-time structure → Immutable → Type-safe
Dynamic Objects:    Runtime structure     → Mutable   → Late-bound
```

---

## 3. Example

### Anonymous Type (Static, Compiler-Generated)
```csharp
// Compiler generates a real class with Name and Age properties
var person = new { Name = "Alice", Age = 30 };

Console.WriteLine(person.Name);
// person.Name = "Bob";
// Error: Read-only property

// Type is strongly typed
ProcessPerson(person);

void ProcessPerson<T>(T obj) where T : class { }
```

### Dynamic Object (Runtime, Expandable)
```csharp
using System.Dynamic;

// Empty object at start
dynamic person = new ExpandoObject();
person.Name = "Alice";    // Property created at runtime
person.Age = 30;          // Another property added

Console.WriteLine(person.Name);
person.Name = "Bob";      // mutable behavior

// Can even add methods as delegates
person.SayHello = (Action)(() => Console.WriteLine("Hello!"));
person.SayHello();        // Works at runtime

// Risk: typo only caught at runtime
Console.WriteLine(person.Nmae); // RuntimeBinderException so it gets un setted prop in person ExpandObject
```

### Side-by-Side Comparison
```csharp
// Scenario: LINQ projection
var query = users.Select(u => new { u.Name, u.Email }); // Anonymous Type

// Scenario: Building JSON response dynamically
dynamic response = new ExpandoObject();
response.Status = "Success";
response.Data = new ExpandoObject();
response.Data.Count = 42;
// Flexible, but no compile-time checking
```

---

## 4. When to Use It

| Situation | Anonymous Types | Dynamic Objects |
|-----------|----------------|-----------------|
| **LINQ Projections** | ✅ Ideal for `Select()` transformations | ❌ Not supported in LINQ-to-Entities |
| **Temporary Data Grouping** | ✅ Perfect for joining data in memory | ⚠️ Possible but loses type safety |
| **Interoperability** | ❌ Requires known structure | ✅ Ideal for consuming JSON/XML with unknown schema |
| **Rapid Prototyping** | ⚠️ Okay for small scopes | ✅ Better when structure changes frequently |
| **Return Types from Methods** | ❌ Cannot be used as return type (use `object` or `dynamic`) | ✅ Can be returned as `dynamic` |
| **Reflection-heavy Code** | ⚠️ Requires reflection to inspect | ✅ Easy property iteration via `IDictionary<string, object>` |
| **Performance Critical** | ✅ Fast as normal class access | ❌ Slower (DLR overhead, caching) |

> **Practical rule:** Use **Anonymous Types** when you need temporary, structured data within a method scope with compile-time safety. Use **Dynamic Objects** when integrating with dynamic languages, parsing unknown JSON schemas, or when the object structure must evolve at runtime.

---

## 5. Advantages and Disadvantages

### Anonymous Types

**Advantages**
- **Type Safety**: Compiler catches errors before runtime.
- **Performance**: Identical to regular classes (no overhead)
- **Immutability**: Thread-safe by default; prevents accidental modification
- **Scope Safety**: Cannot escape method boundaries accidentally (when using `var`)

**Disadvantages**
- **Limited Scope**: Cannot be passed between methods easily (must use `object` + reflection or `dynamic` + casting)
- **Immutability**: Cannot modify properties after creation (must create new instance)
- **No Methods**: Only properties allowed; cannot define behavior
- **Assembly Boundaries**: Two anonymous types with identical properties in different assemblies are considered different types

### Dynamic Objects (ExpandoObject)

**Advantages**
- **Flexibility**: Add/remove properties at runtime; shape changes as needed
- **Interoperability**: Easy integration with JavaScript, Python, or COM objects
- **Dictionary-like**: Implements `IDictionary<string, object>` for easy enumeration
- **Serialization**: Natural fit for JSON APIs with varying response schemas

**Disadvantages**
- **No IntelliSense**: Property names have no autocomplete; typo = runtime exception
- **Performance**: 10-100x slower than static access due to DLR (Dynamic Language Runtime) caching and binding
- **Runtime Errors**: `RuntimeBinderException` only appears when code runs, not during compilation
- **Debugging Difficulty**: Harder to inspect in debugger; property names not visible in static analysis

---

## Quick Summary

```
Anonymous Types vs Dynamic Objects
│
├── Anonymous Types (new { ... })
│   ├── Compile-time type generation
│   ├── Immutable properties
│   ├── Full IntelliSense and type safety
│   ├── Best for: LINQ projections, temporary grouping within methods
│   └── Limitation: Structure locked at compile-time, cannot escape scope easily
│
├── Dynamic Objects (ExpandoObject / dynamic)
│   ├── Runtime property creation
│   ├── Mutable and expandable
│   ├── Late-bound (errors at runtime)
│   ├── Best for: JSON interop, dynamic schemas, scripting scenarios
│   └── Limitation: Performance cost, no compile-time checking
│
└── Key Decision: Static safety vs Runtime flexibility
```