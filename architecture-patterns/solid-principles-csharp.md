# SOLID Principles in C#

Practical examples of the five SOLID principles for writing maintainable, scalable, and testable object-oriented code.

## Overview

SOLID is an acronym for five design principles that make software designs more understandable, flexible, and maintainable. These principles were introduced by Robert C. Martin (Uncle Bob).

## The Five Principles

### S - Single Responsibility Principle (SRP)

**A class should have one, and only one, reason to change.**

Each class should have a single, well-defined responsibility.

#### Bad Example

```csharp
public class User
{
    public string Name { get; set; }
    public string Email { get; set; }

    public void Save()
    {
        // Database logic
        var connection = new SqlConnection("...");
        // Save to database
    }

    public void SendEmail(string message)
    {
        // Email logic
        var client = new SmtpClient();
        // Send email
    }
}
```

#### Good Example

```csharp
// Represents user data only
public class User
{
    public string Name { get; set; }
    public string Email { get; set; }
}

// Handles database operations
public class UserRepository
{
    public void Save(User user)
    {
        var connection = new SqlConnection("...");
        // Save to database
    }
}

// Sends emails
public class EmailService
{
    public void SendEmail(string email, string message)
    {
        var client = new SmtpClient();
        // Send email
    }
}
```

**Benefits:**
- Easier to understand and maintain
- Changes to email logic don't affect database logic
- Each class can be tested independently

---

### O - Open/Closed Principle (OCP)

**Software entities should be open for extension, but closed for modification.**

You should be able to add new functionality without changing existing code.

#### Bad Example

```csharp
public class AreaCalculator
{
    public double CalculateArea(object shape)
    {
        if (shape is Circle circle)
            return Math.PI * circle.Radius * circle.Radius;
        else if (shape is Rectangle rectangle)
            return rectangle.Width * rectangle.Height;
        // Adding new shapes requires modifying this method

        throw new ArgumentException("Unknown shape");
    }
}
```

#### Good Example

```csharp
public interface IShape
{
    double Area();
}

public class Circle : IShape
{
    public double Radius { get; set; }

    public double Area() => Math.PI * Radius * Radius;
}

public class Rectangle : IShape
{
    public double Width { get; set; }
    public double Height { get; set; }

    public double Area() => Width * Height;
}

public class Triangle : IShape
{
    public double Base { get; set; }
    public double Height { get; set; }

    public double Area() => 0.5 * Base * Height;
}

public class AreaCalculator
{
    public double CalculateArea(IShape shape)
    {
        return shape.Area(); // No modification needed for new shapes
    }
}
```

**Benefits:**
- Add new shapes without modifying existing code
- Reduces risk of breaking existing functionality
- Follows polymorphism principles

---

### L - Liskov Substitution Principle (LSP)

**Objects of a superclass should be replaceable with objects of a subclass without breaking the application.**

Derived classes must be substitutable for their base classes.

#### Bad Example

```csharp
public class Bird
{
    public virtual void Fly()
    {
        Console.WriteLine("Flying");
    }
}

public class Penguin : Bird
{
    public override void Fly()
    {
        throw new NotImplementedException("Penguins can't fly!");
    }
}

// This breaks LSP - not all birds can fly
void MakeBirdFly(Bird bird)
{
    bird.Fly(); // Fails for Penguin
}
```

#### Good Example

```csharp
public class Bird
{
    public virtual void Move()
    {
        Console.WriteLine("Moving");
    }
}

public class Sparrow : Bird
{
    public override void Move()
    {
        Console.WriteLine("Flying");
    }
}

public class Penguin : Bird
{
    public override void Move()
    {
        Console.WriteLine("Swimming");
    }
}

void MakeBirdMove(Bird bird)
{
    bird.Move(); // Works for all birds
}
```

**Benefits:**
- Subclasses behave correctly when used in place of parent
- Prevents unexpected runtime errors
- Ensures proper inheritance hierarchy

---

### I - Interface Segregation Principle (ISP)

**Clients should not be forced to depend on interfaces they don't use.**

Create specific interfaces rather than one general-purpose interface.

#### Bad Example

```csharp
public interface IWorker
{
    void Work();
    void Eat();
    void Sleep();
}

public class Human : IWorker
{
    public void Work() => Console.WriteLine("Working");
    public void Eat() => Console.WriteLine("Eating");
    public void Sleep() => Console.WriteLine("Sleeping");
}

public class Robot : IWorker
{
    public void Work() => Console.WriteLine("Working");
    public void Eat() => throw new NotImplementedException(); // Robots don't eat!
    public void Sleep() => throw new NotImplementedException(); // Robots don't sleep!
}
```

#### Good Example

```csharp
public interface IWork
{
    void Work();
}

public interface IEat
{
    void Eat();
}

public interface ISleep
{
    void Sleep();
}

public class Human : IWork, IEat, ISleep
{
    public void Work() => Console.WriteLine("Working");
    public void Eat() => Console.WriteLine("Eating");
    public void Sleep() => Console.WriteLine("Sleeping");
}

public class Robot : IWork
{
    public void Work() => Console.WriteLine("Working");
}
```

**Benefits:**
- Classes only implement methods they actually need
- Smaller, more focused interfaces
- Easier to understand and maintain

---

### D - Dependency Inversion Principle (DIP)

**High-level modules should not depend on low-level modules. Both should depend on abstractions.**

Depend on interfaces or abstract classes, not concrete implementations.

#### Bad Example

```csharp
public class EmailService
{
    public void SendEmail(string message)
    {
        Console.WriteLine($"Email: {message}");
    }
}

public class Notification
{
    private EmailService _emailService = new EmailService();

    public void Send(string message)
    {
        _emailService.SendEmail(message);
    }
}
```

#### Good Example

```csharp
public interface IMessageService
{
    void Send(string message);
}

public class EmailService : IMessageService
{
    public void Send(string message)
    {
        Console.WriteLine($"Email: {message}");
    }
}

public class SmsService : IMessageService
{
    public void Send(string message)
    {
        Console.WriteLine($"SMS: {message}");
    }
}

public class Notification
{
    private readonly IMessageService _messageService;

    // Dependency injected through constructor
    public Notification(IMessageService messageService)
    {
        _messageService = messageService;
    }

    public void Send(string message)
    {
        _messageService.Send(message);
    }
}

// Usage with Dependency Injection
var emailNotification = new Notification(new EmailService());
emailNotification.Send("Hello via Email");

var smsNotification = new Notification(new SmsService());
smsNotification.Send("Hello via SMS");
```

**Benefits:**
- Easy to swap implementations
- Facilitates unit testing with mock objects
- Reduces coupling between classes

---

## Practical Application

### Using All SOLID Principles Together

```csharp
// DIP - Depend on abstractions
public interface ILogger
{
    void Log(string message);
}

// SRP - Single responsibility: console logging
public class ConsoleLogger : ILogger
{
    public void Log(string message)
    {
        Console.WriteLine($"[LOG] {message}");
    }
}

// SRP - Single responsibility: file logging
public class FileLogger : ILogger
{
    public void Log(string message)
    {
        File.AppendAllText("log.txt", $"{DateTime.Now}: {message}\n");
    }
}

// OCP - Open for extension (new payment methods), closed for modification
// DIP - Depends on ILogger abstraction
public interface IPaymentProcessor
{
    void ProcessPayment(decimal amount);
}

public class CreditCardProcessor : IPaymentProcessor
{
    private readonly ILogger _logger;

    public CreditCardProcessor(ILogger logger)
    {
        _logger = logger;
    }

    public void ProcessPayment(decimal amount)
    {
        _logger.Log($"Processing credit card payment: ${amount}");
        // Payment logic
    }
}

public class PayPalProcessor : IPaymentProcessor
{
    private readonly ILogger _logger;

    public PayPalProcessor(ILogger logger)
    {
        _logger = logger;
    }

    public void ProcessPayment(decimal amount)
    {
        _logger.Log($"Processing PayPal payment: ${amount}");
        // Payment logic
    }
}

// SRP - Single responsibility: order management
// DIP - Depends on abstractions
public class OrderService
{
    private readonly IPaymentProcessor _paymentProcessor;
    private readonly ILogger _logger;

    public OrderService(IPaymentProcessor paymentProcessor, ILogger logger)
    {
        _paymentProcessor = paymentProcessor;
        _logger = logger;
    }

    public void PlaceOrder(decimal amount)
    {
        _logger.Log("Placing order");
        _paymentProcessor.ProcessPayment(amount);
        _logger.Log("Order placed successfully");
    }
}
```

## Benefits of SOLID Principles

1. **Maintainability** - Code is easier to update and modify
2. **Testability** - Components can be tested in isolation
3. **Flexibility** - Easy to extend with new functionality
4. **Readability** - Code is clearer and more organized
5. **Reusability** - Components can be reused in different contexts

## Common Anti-Patterns to Avoid

- **God Objects** - Classes that do too much (violates SRP)
- **Tight Coupling** - Direct dependencies on concrete classes (violates DIP)
- **Breaking Contracts** - Subclasses that break parent class behavior (violates LSP)
- **Fat Interfaces** - Interfaces with too many methods (violates ISP)
- **Modification Over Extension** - Changing code instead of extending it (violates OCP)

## Related Articles

- [Design Patterns in C# - Overview](design-patterns-csharp-overview.md)
- [Dependency Injection in C#](dependency-injection-csharp.md)
- [Creational Design Patterns](creational-patterns-csharp.md)

## Resources

- [SOLID Principles - Uncle Bob](http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod)
- [Clean Code by Robert C. Martin](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)
- [Microsoft - SOLID Principles](https://docs.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/architectural-principles)

---

**Last Updated:** 2025
**Technology:** C# / .NET 8.0
