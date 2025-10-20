# Creational Design Patterns in C#

Creational patterns provide various object creation mechanisms, which increase flexibility and reuse of existing code.

## Overview

Creational design patterns abstract the instantiation process. They help make a system independent of how its objects are created, composed, and represented.

## Patterns Covered

1. Factory Pattern
2. Builder Pattern
3. Singleton Pattern
4. Abstract Factory Pattern
5. Prototype Pattern

---

## Factory Pattern

**Creates objects without specifying exact classes to create.**

The Factory pattern defines an interface for creating objects but lets subclasses decide which class to instantiate.

### Problem

You need to create different types of objects based on certain conditions, but you don't want the client code to know about all the concrete classes.

### Solution

```csharp
// Product interface
public interface INotification
{
    void Send(string message);
}

// Concrete products
public class EmailNotification : INotification
{
    public void Send(string message)
    {
        Console.WriteLine($"Email: {message}");
    }
}

public class SmsNotification : INotification
{
    public void Send(string message)
    {
        Console.WriteLine($"SMS: {message}");
    }
}

public class PushNotification : INotification
{
    public void Send(string message)
    {
        Console.WriteLine($"Push: {message}");
    }
}

// Factory
public class NotificationFactory
{
    public static INotification Create(string type)
    {
        return type.ToLower() switch
        {
            "email" => new EmailNotification(),
            "sms" => new SmsNotification(),
            "push" => new PushNotification(),
            _ => throw new ArgumentException($"Unknown notification type: {type}")
        };
    }
}

// Usage
var notification = NotificationFactory.Create("email");
notification.Send("Hello World");

var sms = NotificationFactory.Create("sms");
sms.Send("Alert!");
```

### Benefits

- Decouples object creation from usage
- Easy to add new types without modifying client code
- Centralizes object creation logic

### When to Use

- When you don't know the exact types of objects beforehand
- When you want to provide a library of products hiding implementation details
- When you need to centralize resource management

---

## Builder Pattern

**Constructs complex objects step-by-step.**

The Builder pattern separates the construction of a complex object from its representation, allowing the same construction process to create different representations.

### Problem

You need to create complex objects with many optional parameters, and constructor with many parameters becomes unwieldy.

### Solution

```csharp
// Product
public class Pizza
{
    public string Size { get; set; }
    public string Crust { get; set; }
    public bool Cheese { get; set; }
    public bool Pepperoni { get; set; }
    public bool Mushrooms { get; set; }
    public bool Olives { get; set; }

    public override string ToString()
    {
        return $"{Size} pizza with {Crust} crust, " +
               $"Cheese: {Cheese}, Pepperoni: {Pepperoni}, " +
               $"Mushrooms: {Mushrooms}, Olives: {Olives}";
    }
}

// Builder
public class PizzaBuilder
{
    private Pizza _pizza = new Pizza();

    public PizzaBuilder SetSize(string size)
    {
        _pizza.Size = size;
        return this;
    }

    public PizzaBuilder SetCrust(string crust)
    {
        _pizza.Crust = crust;
        return this;
    }

    public PizzaBuilder AddCheese()
    {
        _pizza.Cheese = true;
        return this;
    }

    public PizzaBuilder AddPepperoni()
    {
        _pizza.Pepperoni = true;
        return this;
    }

    public PizzaBuilder AddMushrooms()
    {
        _pizza.Mushrooms = true;
        return this;
    }

    public PizzaBuilder AddOlives()
    {
        _pizza.Olives = true;
        return this;
    }

    public Pizza Build()
    {
        return _pizza;
    }
}

// Usage
var pizza = new PizzaBuilder()
    .SetSize("Large")
    .SetCrust("Thin")
    .AddCheese()
    .AddPepperoni()
    .AddMushrooms()
    .Build();

Console.WriteLine(pizza);

// Create different variation
var veggiePizza = new PizzaBuilder()
    .SetSize("Medium")
    .SetCrust("Thick")
    .AddCheese()
    .AddMushrooms()
    .AddOlives()
    .Build();
```

### Fluent Builder with Validation

```csharp
public class HttpRequestBuilder
{
    private string _url;
    private string _method = "GET";
    private Dictionary<string, string> _headers = new();
    private string _body;

    public HttpRequestBuilder SetUrl(string url)
    {
        if (string.IsNullOrWhiteSpace(url))
            throw new ArgumentException("URL cannot be empty");
        _url = url;
        return this;
    }

    public HttpRequestBuilder SetMethod(string method)
    {
        _method = method;
        return this;
    }

    public HttpRequestBuilder AddHeader(string key, string value)
    {
        _headers[key] = value;
        return this;
    }

    public HttpRequestBuilder SetBody(string body)
    {
        _body = body;
        return this;
    }

    public HttpRequestMessage Build()
    {
        if (string.IsNullOrWhiteSpace(_url))
            throw new InvalidOperationException("URL must be set");

        var request = new HttpRequestMessage(
            new HttpMethod(_method),
            _url
        );

        foreach (var header in _headers)
        {
            request.Headers.Add(header.Key, header.Value);
        }

        if (!string.IsNullOrWhiteSpace(_body))
        {
            request.Content = new StringContent(_body);
        }

        return request;
    }
}

// Usage
var request = new HttpRequestBuilder()
    .SetUrl("https://api.example.com/users")
    .SetMethod("POST")
    .AddHeader("Authorization", "Bearer token123")
    .AddHeader("Content-Type", "application/json")
    .SetBody("{\"name\": \"John\"}")
    .Build();
```

### Benefits

- Makes code more readable
- Allows step-by-step construction
- Can produce different representations using the same construction code
- Enforces immutability if designed properly

### When to Use

- When creating complex objects with many optional parameters
- When you want to avoid telescoping constructors
- When object creation involves multiple steps

---

## Singleton Pattern

**Ensures a class has only one instance and provides global access to it.**

### Problem

You need exactly one instance of a class (e.g., database connection, configuration manager) and want to ensure no other instance can be created.

### Thread-Safe Singleton

```csharp
public class DatabaseConnection
{
    private static DatabaseConnection _instance;
    private static readonly object _lock = new object();

    private DatabaseConnection()
    {
        Console.WriteLine("Database connection established!");
        // Initialize connection
    }

    public static DatabaseConnection Instance
    {
        get
        {
            if (_instance == null)
            {
                lock (_lock)
                {
                    if (_instance == null)
                    {
                        _instance = new DatabaseConnection();
                    }
                }
            }
            return _instance;
        }
    }

    public void Query(string sql)
    {
        Console.WriteLine($"Executing query: {sql}");
    }
}

// Usage
var db1 = DatabaseConnection.Instance;
var db2 = DatabaseConnection.Instance;

Console.WriteLine(db1 == db2); // True - same instance

db1.Query("SELECT * FROM Users");
```

### Lazy Initialization (Modern C#)

```csharp
public sealed class ConfigurationManager
{
    private static readonly Lazy<ConfigurationManager> _lazy =
        new Lazy<ConfigurationManager>(() => new ConfigurationManager());

    public static ConfigurationManager Instance => _lazy.Value;

    private Dictionary<string, string> _settings;

    private ConfigurationManager()
    {
        Console.WriteLine("Loading configuration...");
        _settings = new Dictionary<string, string>
        {
            { "AppName", "MyApp" },
            { "Version", "1.0" }
        };
    }

    public string GetSetting(string key)
    {
        return _settings.TryGetValue(key, out var value) ? value : null;
    }
}

// Usage
var config = ConfigurationManager.Instance;
Console.WriteLine(config.GetSetting("AppName")); // MyApp
```

### Singleton with Dependency Injection

```csharp
public class Logger
{
    public void Log(string message)
    {
        Console.WriteLine($"[{DateTime.Now}] {message}");
    }
}

// In Program.cs or Startup.cs
services.AddSingleton<Logger>();

// Usage through DI
public class OrderService
{
    private readonly Logger _logger;

    public OrderService(Logger logger)
    {
        _logger = logger;
    }

    public void ProcessOrder()
    {
        _logger.Log("Processing order...");
    }
}
```

### Benefits

- Controlled access to a single instance
- Reduces global namespace pollution
- Permits lazy initialization
- Can be easily extended to control the number of instances

### Drawbacks

- Difficult to unit test (tight coupling)
- Violates Single Responsibility Principle (controls its own creation and lifecycle)
- Can be problematic in multi-threaded environments if not implemented carefully

### When to Use

- When exactly one instance is needed (logging, caching, thread pools)
- When the instance needs to be accessible from multiple places
- When you need lazy initialization with thread safety

---

## Abstract Factory Pattern

**Creates families of related objects without specifying their concrete classes.**

### Problem

You need to create multiple related objects that must be used together.

### Solution

```csharp
// Abstract products
public interface IButton
{
    void Render();
}

public interface ICheckbox
{
    void Render();
}

// Concrete products - Windows
public class WindowsButton : IButton
{
    public void Render() => Console.WriteLine("Rendering Windows button");
}

public class WindowsCheckbox : ICheckbox
{
    public void Render() => Console.WriteLine("Rendering Windows checkbox");
}

// Concrete products - Mac
public class MacButton : IButton
{
    public void Render() => Console.WriteLine("Rendering Mac button");
}

public class MacCheckbox : ICheckbox
{
    public void Render() => Console.WriteLine("Rendering Mac checkbox");
}

// Abstract factory
public interface IUIFactory
{
    IButton CreateButton();
    ICheckbox CreateCheckbox();
}

// Concrete factories
public class WindowsUIFactory : IUIFactory
{
    public IButton CreateButton() => new WindowsButton();
    public ICheckbox CreateCheckbox() => new WindowsCheckbox();
}

public class MacUIFactory : IUIFactory
{
    public IButton CreateButton() => new MacButton();
    public ICheckbox CreateCheckbox() => new MacCheckbox();
}

// Client code
public class Application
{
    private readonly IButton _button;
    private readonly ICheckbox _checkbox;

    public Application(IUIFactory factory)
    {
        _button = factory.CreateButton();
        _checkbox = factory.CreateCheckbox();
    }

    public void Render()
    {
        _button.Render();
        _checkbox.Render();
    }
}

// Usage
var os = "Windows"; // Could come from runtime detection

IUIFactory factory = os switch
{
    "Windows" => new WindowsUIFactory(),
    "Mac" => new MacUIFactory(),
    _ => throw new NotSupportedException()
};

var app = new Application(factory);
app.Render();
```

### Benefits

- Ensures created products are compatible
- Separates product creation from usage
- Easy to introduce new product variants

### When to Use

- When your system needs to work with multiple families of related products
- When you want to provide a library of products without exposing implementation
- When you need to enforce constraints about which products can be used together

---

## Prototype Pattern

**Creates new objects by cloning existing ones.**

### Problem

You need to create copies of objects without coupling to their specific classes.

### Solution

```csharp
// Prototype interface
public interface IPrototype<T>
{
    T Clone();
}

// Concrete prototype
public class Document : IPrototype<Document>
{
    public string Title { get; set; }
    public string Content { get; set; }
    public List<string> Tags { get; set; }

    public Document(string title, string content)
    {
        Title = title;
        Content = content;
        Tags = new List<string>();
    }

    // Deep clone
    public Document Clone()
    {
        var clone = new Document(Title, Content);
        clone.Tags = new List<string>(Tags); // Clone the list
        return clone;
    }

    public override string ToString()
    {
        return $"{Title}: {Content} [Tags: {string.Join(", ", Tags)}]";
    }
}

// Usage
var original = new Document("Design Patterns", "Content here");
original.Tags.Add("programming");
original.Tags.Add("architecture");

var copy = original.Clone();
copy.Title = "Copy of Design Patterns";
copy.Tags.Add("copy");

Console.WriteLine(original); // Design Patterns: Content here [Tags: programming, architecture]
Console.WriteLine(copy);     // Copy of Design Patterns: Content here [Tags: programming, architecture, copy]
```

### Using ICloneable

```csharp
public class Employee : ICloneable
{
    public string Name { get; set; }
    public int Age { get; set; }
    public Address Address { get; set; }

    public object Clone()
    {
        // Shallow clone
        return this.MemberwiseClone();
    }

    public Employee DeepClone()
    {
        var clone = (Employee)this.MemberwiseClone();
        clone.Address = new Address
        {
            Street = this.Address.Street,
            City = this.Address.City
        };
        return clone;
    }
}

public class Address
{
    public string Street { get; set; }
    public string City { get; set; }
}
```

### Benefits

- Reduces the need for subclassing
- Hides complexity of creating new instances
- Allows adding/removing objects at runtime

### When to Use

- When object creation is expensive (database queries, network calls)
- When you need independent copies of objects
- When you want to avoid building class hierarchies

---

## Comparison Table

| Pattern | Purpose | Use When |
|---------|---------|----------|
| **Factory** | Create objects without specifying exact class | Multiple types of related objects |
| **Abstract Factory** | Create families of related objects | Need compatible product families |
| **Builder** | Construct complex objects step-by-step | Many optional parameters |
| **Singleton** | Ensure only one instance exists | Single instance needed globally |
| **Prototype** | Clone existing objects | Object creation is expensive |

## Related Articles

- [SOLID Principles in C#](solid-principles-csharp.md)
- [Behavioral Design Patterns](behavioral-patterns-csharp.md)
- [Structural Design Patterns](structural-patterns-csharp.md)
- [Design Patterns Overview](design-patterns-csharp-overview.md)

---

**Technology:** C# / .NET 8.0
**Category:** Creational Patterns
