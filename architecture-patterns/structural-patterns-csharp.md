# Structural Design Patterns in C#

Structural patterns explain how to assemble objects and classes into larger structures while keeping these structures flexible and efficient.

## Overview

Structural design patterns deal with object composition, creating relationships between objects to form larger structures. They help ensure that if one part of a system changes, the entire system doesn't need to change.

## Patterns Covered

1. Adapter Pattern
2. Facade Pattern
3. Decorator Pattern
4. Proxy Pattern
5. Composite Pattern
6. Bridge Pattern
7. Repository Pattern

---

## Adapter Pattern

**Converts the interface of a class into another interface clients expect.**

Also known as the Wrapper pattern.

### Problem

You have incompatible interfaces and need to make them work together.

### Solution

```csharp
// Target interface (what client expects)
public interface IPayment
{
    void ProcessPayment(decimal amount);
}

// Adaptee (existing incompatible class)
public class LegacyPaymentSystem
{
    public void MakePayment(double value, string currency)
    {
        Console.WriteLine($"Legacy system: Processing {currency} {value}");
    }
}

// Adapter
public class PaymentAdapter : IPayment
{
    private readonly LegacyPaymentSystem _legacySystem;

    public PaymentAdapter(LegacyPaymentSystem legacySystem)
    {
        _legacySystem = legacySystem;
    }

    public void ProcessPayment(decimal amount)
    {
        // Convert decimal to double and add currency
        _legacySystem.MakePayment((double)amount, "USD");
    }
}

// Usage
IPayment modernPayment = new PaymentAdapter(new LegacyPaymentSystem());
modernPayment.ProcessPayment(99.99m);
```

### Real-World Example: XML to JSON Adapter

```csharp
// Modern interface
public interface IDataProvider
{
    string GetJsonData();
}

// Legacy XML provider
public class XmlDataProvider
{
    public string GetXmlData()
    {
        return "<user><name>John</name><age>30</age></user>";
    }
}

// Adapter
public class XmlToJsonAdapter : IDataProvider
{
    private readonly XmlDataProvider _xmlProvider;

    public XmlToJsonAdapter(XmlDataProvider xmlProvider)
    {
        _xmlProvider = xmlProvider;
    }

    public string GetJsonData()
    {
        var xmlData = _xmlProvider.GetXmlData();
        // Simplified conversion (in reality, use proper XML/JSON libraries)
        return "{\"name\":\"John\",\"age\":30}";
    }
}

// Client code
public class Application
{
    private readonly IDataProvider _dataProvider;

    public Application(IDataProvider dataProvider)
    {
        _dataProvider = dataProvider;
    }

    public void DisplayData()
    {
        var data = _dataProvider.GetJsonData();
        Console.WriteLine($"Data: {data}");
    }
}

// Usage
var xmlProvider = new XmlDataProvider();
var adapter = new XmlToJsonAdapter(xmlProvider);
var app = new Application(adapter);
app.DisplayData();
```

### Benefits

- Allows incompatible interfaces to work together
- Promotes reusability of existing code
- Single Responsibility Principle (separation of interface conversion)

### When to Use

- When you want to use an existing class with an incompatible interface
- When you need to create a reusable class that cooperates with unrelated classes
- When integrating with legacy systems

---

## Facade Pattern

**Provides a simplified interface to a complex subsystem.**

### Problem

A system has many interdependent classes with complex interactions, making it difficult to use.

### Solution

```csharp
// Complex subsystem
public class TV
{
    public void TurnOn() => Console.WriteLine("TV is ON");
    public void TurnOff() => Console.WriteLine("TV is OFF");
    public void SetInput(string input) => Console.WriteLine($"TV input set to {input}");
}

public class SoundSystem
{
    public void TurnOn() => Console.WriteLine("Sound system ON");
    public void TurnOff() => Console.WriteLine("Sound system OFF");
    public void SetVolume(int level) => Console.WriteLine($"Volume set to {level}");
    public void SetSurround() => Console.WriteLine("Surround sound enabled");
}

public class DVDPlayer
{
    public void TurnOn() => Console.WriteLine("DVD player ON");
    public void TurnOff() => Console.WriteLine("DVD player OFF");
    public void Play(string movie) => Console.WriteLine($"Playing: {movie}");
    public void Stop() => Console.WriteLine("DVD stopped");
}

public class Lights
{
    public void Dim(int level) => Console.WriteLine($"Lights dimmed to {level}%");
    public void Brighten() => Console.WriteLine("Lights brightened");
}

// Facade
public class HomeTheaterFacade
{
    private readonly TV _tv;
    private readonly SoundSystem _soundSystem;
    private readonly DVDPlayer _dvdPlayer;
    private readonly Lights _lights;

    public HomeTheaterFacade(TV tv, SoundSystem soundSystem, DVDPlayer dvdPlayer, Lights lights)
    {
        _tv = tv;
        _soundSystem = soundSystem;
        _dvdPlayer = dvdPlayer;
        _lights = lights;
    }

    public void WatchMovie(string movie)
    {
        Console.WriteLine("\n=== Starting movie ===");
        _lights.Dim(10);
        _tv.TurnOn();
        _tv.SetInput("DVD");
        _soundSystem.TurnOn();
        _soundSystem.SetVolume(50);
        _soundSystem.SetSurround();
        _dvdPlayer.TurnOn();
        _dvdPlayer.Play(movie);
    }

    public void EndMovie()
    {
        Console.WriteLine("\n=== Ending movie ===");
        _dvdPlayer.Stop();
        _dvdPlayer.TurnOff();
        _soundSystem.TurnOff();
        _tv.TurnOff();
        _lights.Brighten();
    }
}

// Usage
var homeTheater = new HomeTheaterFacade(
    new TV(),
    new SoundSystem(),
    new DVDPlayer(),
    new Lights()
);

homeTheater.WatchMovie("Inception");
// ... watch movie ...
homeTheater.EndMovie();
```

### E-commerce Example

```csharp
// Subsystems
public class InventoryService
{
    public bool CheckStock(int productId) => true;
    public void UpdateStock(int productId, int quantity)
        => Console.WriteLine($"Stock updated for product {productId}");
}

public class PaymentService
{
    public bool ProcessPayment(decimal amount)
    {
        Console.WriteLine($"Payment of ${amount} processed");
        return true;
    }
}

public class ShippingService
{
    public void ShipOrder(int orderId)
        => Console.WriteLine($"Order {orderId} shipped");
}

public class NotificationService
{
    public void SendConfirmation(string email)
        => Console.WriteLine($"Confirmation sent to {email}");
}

// Facade
public class OrderFacade
{
    private readonly InventoryService _inventory;
    private readonly PaymentService _payment;
    private readonly ShippingService _shipping;
    private readonly NotificationService _notification;

    public OrderFacade()
    {
        _inventory = new InventoryService();
        _payment = new PaymentService();
        _shipping = new ShippingService();
        _notification = new NotificationService();
    }

    public bool PlaceOrder(int productId, decimal amount, string email)
    {
        Console.WriteLine("\n=== Processing order ===");

        if (!_inventory.CheckStock(productId))
        {
            Console.WriteLine("Product out of stock");
            return false;
        }

        if (!_payment.ProcessPayment(amount))
        {
            Console.WriteLine("Payment failed");
            return false;
        }

        _inventory.UpdateStock(productId, -1);
        _shipping.ShipOrder(productId);
        _notification.SendConfirmation(email);

        Console.WriteLine("Order placed successfully!");
        return true;
    }
}

// Usage
var orderSystem = new OrderFacade();
orderSystem.PlaceOrder(123, 99.99m, "customer@example.com");
```

### Benefits

- Simplifies complex subsystems
- Reduces dependencies on subsystem classes
- Promotes loose coupling

### When to Use

- When you need a simple interface to a complex system
- When you want to layer your subsystems
- When there are many dependencies between clients and implementation classes

---

## Decorator Pattern

**Adds new functionality to objects dynamically without altering their structure.**

### Problem

You want to add responsibilities to individual objects, not entire classes, and you want to do it dynamically.

### Solution

```csharp
// Component interface
public interface ICoffee
{
    string GetDescription();
    decimal GetCost();
}

// Concrete component
public class SimpleCoffee : ICoffee
{
    public string GetDescription() => "Simple coffee";
    public decimal GetCost() => 2.00m;
}

// Decorator base class
public abstract class CoffeeDecorator : ICoffee
{
    protected ICoffee _coffee;

    protected CoffeeDecorator(ICoffee coffee)
    {
        _coffee = coffee;
    }

    public virtual string GetDescription() => _coffee.GetDescription();
    public virtual decimal GetCost() => _coffee.GetCost();
}

// Concrete decorators
public class MilkDecorator : CoffeeDecorator
{
    public MilkDecorator(ICoffee coffee) : base(coffee) { }

    public override string GetDescription() => $"{_coffee.GetDescription()}, Milk";
    public override decimal GetCost() => _coffee.GetCost() + 0.50m;
}

public class SugarDecorator : CoffeeDecorator
{
    public SugarDecorator(ICoffee coffee) : base(coffee) { }

    public override string GetDescription() => $"{_coffee.GetDescription()}, Sugar";
    public override decimal GetCost() => _coffee.GetCost() + 0.25m;
}

public class WhipDecorator : CoffeeDecorator
{
    public WhipDecorator(ICoffee coffee) : base(coffee) { }

    public override string GetDescription() => $"{_coffee.GetDescription()}, Whipped Cream";
    public override decimal GetCost() => _coffee.GetCost() + 0.75m;
}

// Usage
ICoffee coffee = new SimpleCoffee();
Console.WriteLine($"{coffee.GetDescription()} = ${coffee.GetCost()}");

coffee = new MilkDecorator(coffee);
Console.WriteLine($"{coffee.GetDescription()} = ${coffee.GetCost()}");

coffee = new SugarDecorator(coffee);
Console.WriteLine($"{coffee.GetDescription()} = ${coffee.GetCost()}");

coffee = new WhipDecorator(coffee);
Console.WriteLine($"{coffee.GetDescription()} = ${coffee.GetCost()}");

// Output:
// Simple coffee = $2.00
// Simple coffee, Milk = $2.50
// Simple coffee, Milk, Sugar = $2.75
// Simple coffee, Milk, Sugar, Whipped Cream = $3.50
```

### Stream Example (Real .NET Usage)

```csharp
// The .NET Stream classes use the Decorator pattern
// FileStream is the base component
// BufferedStream, GZipStream, CryptoStream are decorators

var fileStream = new FileStream("data.txt", FileMode.Open);
var bufferedStream = new BufferedStream(fileStream);
var gzipStream = new GZipStream(bufferedStream, CompressionMode.Decompress);
```

### Benefits

- More flexible than inheritance
- Add/remove responsibilities at runtime
- Avoid feature-laden classes high in the hierarchy

### When to Use

- When you need to add responsibilities to objects dynamically
- When extension by subclassing is impractical
- When you want to add features without affecting other objects

---

## Proxy Pattern

**Provides a surrogate or placeholder to control access to an object.**

### Problem

You need to control access to an object, add lazy initialization, or implement caching.

### Solution

```csharp
// Subject interface
public interface IImage
{
    void Display();
}

// Real subject
public class RealImage : IImage
{
    private string _fileName;

    public RealImage(string fileName)
    {
        _fileName = fileName;
        LoadFromDisk();
    }

    private void LoadFromDisk()
    {
        Console.WriteLine($"Loading image from disk: {_fileName}");
        // Simulate expensive operation
        Thread.Sleep(1000);
    }

    public void Display()
    {
        Console.WriteLine($"Displaying {_fileName}");
    }
}

// Proxy
public class ImageProxy : IImage
{
    private string _fileName;
    private RealImage _realImage;

    public ImageProxy(string fileName)
    {
        _fileName = fileName;
    }

    public void Display()
    {
        // Lazy loading - only create real image when needed
        if (_realImage == null)
        {
            _realImage = new RealImage(_fileName);
        }
        _realImage.Display();
    }
}

// Usage
IImage image1 = new ImageProxy("photo1.jpg");
IImage image2 = new ImageProxy("photo2.jpg");

// Images not loaded yet
Console.WriteLine("Images created\n");

// Image loaded on first access
image1.Display();
// Image already loaded, just display
image1.Display();

// Different image loaded
image2.Display();
```

### Protection Proxy Example

```csharp
public interface IDocument
{
    void View();
    void Edit();
    void Delete();
}

public class Document : IDocument
{
    public string Name { get; set; }

    public Document(string name)
    {
        Name = name;
    }

    public void View() => Console.WriteLine($"Viewing {Name}");
    public void Edit() => Console.WriteLine($"Editing {Name}");
    public void Delete() => Console.WriteLine($"Deleting {Name}");
}

public class DocumentProxy : IDocument
{
    private Document _document;
    private string _userRole;

    public DocumentProxy(Document document, string userRole)
    {
        _document = document;
        _userRole = userRole;
    }

    public void View()
    {
        _document.View();
    }

    public void Edit()
    {
        if (_userRole == "Admin" || _userRole == "Editor")
        {
            _document.Edit();
        }
        else
        {
            Console.WriteLine("Access denied: Insufficient permissions to edit");
        }
    }

    public void Delete()
    {
        if (_userRole == "Admin")
        {
            _document.Delete();
        }
        else
        {
            Console.WriteLine("Access denied: Only admins can delete");
        }
    }
}

// Usage
var document = new Document("Confidential.docx");

var adminProxy = new DocumentProxy(document, "Admin");
adminProxy.View();   // OK
adminProxy.Edit();   // OK
adminProxy.Delete(); // OK

var viewerProxy = new DocumentProxy(document, "Viewer");
viewerProxy.View();   // OK
viewerProxy.Edit();   // Access denied
viewerProxy.Delete(); // Access denied
```

### Benefits

- Controls access to the real object
- Can add additional functionality (caching, logging, lazy initialization)
- Reduces memory usage (lazy loading)

### When to Use

- When you need lazy initialization (virtual proxy)
- When you need access control (protection proxy)
- When you need local execution of a remote service (remote proxy)
- When you need logging/caching (smart reference)

---

## Composite Pattern

**Composes objects into tree structures to represent part-whole hierarchies.**

### Problem

You need to work with tree structures and want to treat individual objects and compositions uniformly.

### Solution

```csharp
// Component interface
public interface IFileSystemItem
{
    void Display(int indent = 0);
    int GetSize();
}

// Leaf
public class File : IFileSystemItem
{
    public string Name { get; set; }
    public int Size { get; set; }

    public File(string name, int size)
    {
        Name = name;
        Size = size;
    }

    public void Display(int indent = 0)
    {
        Console.WriteLine($"{new string(' ', indent)}File: {Name} ({Size} KB)");
    }

    public int GetSize() => Size;
}

// Composite
public class Folder : IFileSystemItem
{
    public string Name { get; set; }
    private List<IFileSystemItem> _items = new();

    public Folder(string name)
    {
        Name = name;
    }

    public void Add(IFileSystemItem item)
    {
        _items.Add(item);
    }

    public void Remove(IFileSystemItem item)
    {
        _items.Remove(item);
    }

    public void Display(int indent = 0)
    {
        Console.WriteLine($"{new string(' ', indent)}Folder: {Name}");
        foreach (var item in _items)
        {
            item.Display(indent + 2);
        }
    }

    public int GetSize()
    {
        return _items.Sum(item => item.GetSize());
    }
}

// Usage
var root = new Folder("Root");

var documents = new Folder("Documents");
documents.Add(new File("Resume.docx", 50));
documents.Add(new File("CoverLetter.docx", 30));

var pictures = new Folder("Pictures");
pictures.Add(new File("Photo1.jpg", 200));
pictures.Add(new File("Photo2.jpg", 250));

var vacation = new Folder("Vacation");
vacation.Add(new File("Beach.jpg", 300));
pictures.Add(vacation);

root.Add(documents);
root.Add(pictures);
root.Add(new File("readme.txt", 5));

root.Display();
Console.WriteLine($"\nTotal size: {root.GetSize()} KB");
```

### Benefits

- Simplifies client code
- Makes it easy to add new kinds of components
- Provides flexibility in structure

### When to Use

- When you need to represent part-whole hierarchies
- When you want clients to ignore differences between compositions and individual objects
- When working with tree structures (file systems, UI components, organizational charts)

---

## Bridge Pattern

**Decouples abstraction from implementation so both can vary independently.**

### Problem

You have multiple dimensions of variation and want to avoid a proliferation of classes.

### Solution

```csharp
// Implementation interface
public interface IDevice
{
    void TurnOn();
    void TurnOff();
    void SetChannel(int channel);
}

// Concrete implementations
public class TV : IDevice
{
    public void TurnOn() => Console.WriteLine("TV is ON");
    public void TurnOff() => Console.WriteLine("TV is OFF");
    public void SetChannel(int channel) => Console.WriteLine($"TV channel set to {channel}");
}

public class Radio : IDevice
{
    public void TurnOn() => Console.WriteLine("Radio is ON");
    public void TurnOff() => Console.WriteLine("Radio is OFF");
    public void SetChannel(int channel) => Console.WriteLine($"Radio station set to {channel}");
}

// Abstraction
public abstract class RemoteControl
{
    protected IDevice _device;

    protected RemoteControl(IDevice device)
    {
        _device = device;
    }

    public virtual void TurnOn() => _device.TurnOn();
    public virtual void TurnOff() => _device.TurnOff();
}

// Refined abstractions
public class BasicRemote : RemoteControl
{
    public BasicRemote(IDevice device) : base(device) { }

    public void SetChannel(int channel)
    {
        _device.SetChannel(channel);
    }
}

public class AdvancedRemote : RemoteControl
{
    public AdvancedRemote(IDevice device) : base(device) { }

    public void SetChannel(int channel)
    {
        _device.SetChannel(channel);
    }

    public void Mute()
    {
        Console.WriteLine("Device muted");
    }
}

// Usage
var tv = new TV();
var radio = new Radio();

var basicRemoteForTV = new BasicRemote(tv);
basicRemoteForTV.TurnOn();
basicRemoteForTV.SetChannel(5);

var advancedRemoteForRadio = new AdvancedRemote(radio);
advancedRemoteForRadio.TurnOn();
advancedRemoteForRadio.SetChannel(101);
advancedRemoteForRadio.Mute();
```

### Benefits

- Separates abstraction from implementation
- Improves extensibility
- Hides implementation details from clients

### When to Use

- When you want to avoid permanent binding between abstraction and implementation
- When both abstractions and implementations should be extensible by subclassing
- When you have a proliferation of classes from a coupled interface and implementation

---

## Repository Pattern

**Mediates between the domain and data mapping layers using a collection-like interface.**

### Problem

You need to abstract data access logic and make it easier to test business logic.

### Solution

```csharp
// Entity
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}

// Repository interface
public interface IRepository<T> where T : class
{
    void Add(T entity);
    void Update(T entity);
    void Delete(int id);
    T GetById(int id);
    IEnumerable<T> GetAll();
}

// Concrete repository
public class ProductRepository : IRepository<Product>
{
    private List<Product> _products = new();
    private int _nextId = 1;

    public void Add(Product entity)
    {
        entity.Id = _nextId++;
        _products.Add(entity);
        Console.WriteLine($"Product added: {entity.Name}");
    }

    public void Update(Product entity)
    {
        var existing = GetById(entity.Id);
        if (existing != null)
        {
            existing.Name = entity.Name;
            existing.Price = entity.Price;
            Console.WriteLine($"Product updated: {entity.Name}");
        }
    }

    public void Delete(int id)
    {
        var product = GetById(id);
        if (product != null)
        {
            _products.Remove(product);
            Console.WriteLine($"Product deleted: {product.Name}");
        }
    }

    public Product GetById(int id)
    {
        return _products.FirstOrDefault(p => p.Id == id);
    }

    public IEnumerable<Product> GetAll()
    {
        return _products.AsReadOnly();
    }
}

// Service layer using repository
public class ProductService
{
    private readonly IRepository<Product> _repository;

    public ProductService(IRepository<Product> repository)
    {
        _repository = repository;
    }

    public void CreateProduct(string name, decimal price)
    {
        var product = new Product { Name = name, Price = price };
        _repository.Add(product);
    }

    public void ListProducts()
    {
        var products = _repository.GetAll();
        foreach (var product in products)
        {
            Console.WriteLine($"ID: {product.Id}, Name: {product.Name}, Price: ${product.Price}");
        }
    }
}

// Usage
IRepository<Product> repository = new ProductRepository();
var service = new ProductService(repository);

service.CreateProduct("Laptop", 999.99m);
service.CreateProduct("Mouse", 29.99m);
service.ListProducts();
```

### Benefits

- Separates business logic from data access
- Easier to test (can mock repository)
- Centralized data access logic

### When to Use

- When you want to decouple business logic from data access
- When you need to make your code more testable
- When you want to centralize data access logic

---

## Comparison Table

| Pattern | Purpose | Use When |
|---------|---------|----------|
| **Adapter** | Convert interface to another | Incompatible interfaces |
| **Facade** | Simplified interface to subsystem | Complex subsystem |
| **Decorator** | Add functionality dynamically | Runtime feature addition |
| **Proxy** | Control access to object | Lazy loading, access control |
| **Composite** | Tree structures | Part-whole hierarchies |
| **Bridge** | Separate abstraction from implementation | Multiple dimensions of change |
| **Repository** | Abstract data access | Separate business from data layer |

## Related Articles

- [Creational Design Patterns](creational-patterns-csharp.md)
- [Behavioral Design Patterns](behavioral-patterns-csharp.md)
- [SOLID Principles](solid-principles-csharp.md)
- [Design Patterns Overview](design-patterns-csharp-overview.md)

---

**Technology:** C# / .NET 8.0
**Category:** Structural Patterns
