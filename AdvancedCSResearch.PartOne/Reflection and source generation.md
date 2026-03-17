# Reflection and Source Generation

---

## 1. Definition

**Reflection** is runtime type inspection—examining and manipulating assemblies, types, and members while the program runs.

**Source Generation** is compile-time code generation—analyzing source code during compilation and injecting new code files into the build.

---

## 2. The Idea

**`Reflection`** operates at runtime by loading metadata and invoking members dynamically.

**`Source Generation`** shifts this work to compile-time, generating static code that replaces dynamic lookups.

```
Reflection (Runtime):
─────────────────────
Program Runs ──► Inspect Type ──► Discover Member ──► Invoke (slow, boxing)
     ↑
     └── Metadata loaded on-demand; flexible but expensive

Source Generation (Compile-time):
────────────────────────────────
Code Compiles ──► Analyzer Runs ──► Generate Static Code ──► Compile Result (fast, direct)
     ↑
     └── Logic executed ahead-of-time; zero runtime cost
```

**Core Idea Of Both:**
-
- **Reflection** — Discovers structure dynamically; works with unknown types but incurs performance penalties and AOT/Trimming limitations


- **Source Generation** — Discovers structure statically; produces typed code compatible with Native AOT with no runtime overhead

---

## 3. Example

### Reflection Approach
```csharp
// Runtime inspection and invocation
public static string GetPropertyValue(object obj, string propertyName)
{
    var type = obj.GetType();
    var property = type.GetProperty(propertyName);        // Slow lookup
    return property?.GetValue(obj)?.ToString();          // Boxing overhead
}

// Usage
var person = new Person { Name = "Alice" };
var name = GetPropertyValue(person, "Name");              // Runtime discovery
```

### Source Generator Approach
```csharp
// Compile-time generator
[Generator]
public class PropertyAccessorGenerator : ISourceGenerator
{
    public void Execute(GeneratorExecutionContext context)
    {
        // Generates at compile time:
        // public static string GetName(this Person p) => p.Name;

        var code = @"
        public static class PersonExtensions {
            public static string GetName(this Person p) => p.Name;
        }";

        context.AddSource("PersonExtensions.g.cs", code);
    }

    public void Initialize(GeneratorInitializationContext context) { }
}

// Usage - generated code
var person = new Person { Name = "Alice" };
var name = person.GetName();                              // Direct call, zero overhead
```

---

## 4. When to Use It

| Situation | Reflection | Source Generation |
|-----------|-----------|-------------------|
| **Plugin Systems** | ✅ Loading unknown assemblies at runtime | ❌ Requires compile-time knowledge |
| **Serialization** | ⚠️ Works but slow (Newtonsoft.Json) | ✅ High-performance (System.Text.Json source gen) |
| **Dependency Injection** | ⚠️ Runtime assembly scanning | ✅ Compile-time service registration |
| **AOT/Trimming** | ❌ Requires dynamic code access | ✅ Fully compatible |
| **ORM Mapping** | ⚠️ Runtime entity configuration | ✅ Generated SQL mappers |
| **Property Change Notification** | ⚠️ Reflection-based INotifyPropertyChanged | ✅ Generated observable properties |

> **Practical rule:** Use **Reflection** when structure is unknown until runtime (plugins, user scripts). Use **Source Generation** when structure is known at compile time but you want to eliminate reflection overhead.

---

## 5. Advantages and Disadvantages

### Reflection

**Advantages**
- **Flexibility** — Works with any assembly, including ones not referenced at compile time
- **Late Binding** — Resolve types and methods by string names at runtime
- **Introspection** — Inspect attributes, generics, and private members dynamically

**Disadvantages**
- **Performance** — 10-100x slower than direct calls; boxing and method lookup overhead
- **Fragility** — Runtime errors for renamed members; no compile-time validation
- **Security** — Requires elevated permissions in some sandboxed environments

### Source Generation

**Advantages**
- **Zero Runtime Cost** — Generated code runs as fast as hand-written code
- **AOT Compatible** — Works with Native AOT and trimming (no runtime metadata needed)
- **Compile-time Safety** — Errors caught during build, not at runtime
- **IntelliSense** — Generated code appears in IDE with full autocomplete support

**Disadvantages**
- **Complexity** — Requires learning Roslyn APIs and incremental generator patterns
- **Debugging** — Harder to debug generated code (though `.g.cs` files are viewable)
- **Limited Context** — Only knows compile-time information (cannot see runtime values)
- **Build Time** — Increases compilation duration

---

## Quick Summary

```
Reflection vs Source Generation
│
├── Reflection (System.Reflection)
│   ├── Timing: Runtime
│   ├── Mechanism: Inspect metadata, invoke dynamically
│   ├── Performance: Slow (boxing, lookup overhead)
│   ├── AOT: Incompatible (requires preserved metadata)
│   └── Best for: Plugins, dynamic loading, unknown types at compile time
│
├── Source Generation (ISourceGenerator, IIncrementalGenerator)
│   ├── Timing: Compile-time
│   ├── Mechanism: Analyze syntax, emit C# code
│   ├── Performance: Native speed (static code)
│   ├── AOT: Fully compatible
│   └── Best for: High-performance scenarios, serialization, DI registration
│
└── Modern Trend: Source Generation replaces Reflection wherever possible
    for performance and AOT compatibility (e.g., System.Text.Json, Regex)
```