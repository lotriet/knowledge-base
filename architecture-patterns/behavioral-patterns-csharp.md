# Behavioral Design Patterns in C#

Behavioral patterns focus on algorithms and the assignment of responsibilities between objects. They describe not just patterns of objects or classes but also the patterns of communication between them.

## Overview

Behavioral design patterns are concerned with the interaction and responsibility of objects. They help in defining how objects communicate and how responsibilities are distributed among them.

## Patterns Covered

1. Observer Pattern
2. Strategy Pattern
3. Command Pattern
4. Template Method Pattern
5. Chain of Responsibility Pattern
6. State Pattern

---

## Observer Pattern

**Defines a one-to-many dependency so that when one object changes state, all dependents are notified.**

Also known as the Publish-Subscribe pattern.

### Problem

You need to notify multiple objects when another object's state changes, without tight coupling.

### Solution

```csharp
// Observer interface
public interface ISubscriber
{
    void Update(string message);
}

// Concrete observers
public class EmailSubscriber : ISubscriber
{
    public string Email { get; set; }

    public EmailSubscriber(string email)
    {
        Email = email;
    }

    public void Update(string message)
    {
        Console.WriteLine($"Email sent to {Email}: {message}");
    }
}

public class SmsSubscriber : ISubscriber
{
    public string PhoneNumber { get; set; }

    public SmsSubscriber(string phoneNumber)
    {
        PhoneNumber = phoneNumber;
    }

    public void Update(string message)
    {
        Console.WriteLine($"SMS sent to {PhoneNumber}: {message}");
    }
}

// Subject
public class YouTubeChannel
{
    private List<ISubscriber> _subscribers = new();
    public string Name { get; set; }

    public YouTubeChannel(string name)
    {
        Name = name;
    }

    public void Subscribe(ISubscriber subscriber)
    {
        _subscribers.Add(subscriber);
        Console.WriteLine($"New subscriber added to {Name}");
    }

    public void Unsubscribe(ISubscriber subscriber)
    {
        _subscribers.Remove(subscriber);
    }

    public void UploadVideo(string title)
    {
        Console.WriteLine($"\n{Name} uploaded: {title}");
        NotifySubscribers($"New video uploaded: {title}");
    }

    private void NotifySubscribers(string message)
    {
        foreach (var subscriber in _subscribers)
        {
            subscriber.Update(message);
        }
    }
}

// Usage
var channel = new YouTubeChannel("Tech Tutorials");

var emailSub = new EmailSubscriber("user@example.com");
var smsSub = new SmsSubscriber("+1234567890");

channel.Subscribe(emailSub);
channel.Subscribe(smsSub);

channel.UploadVideo("Design Patterns in C#");
channel.UploadVideo("SOLID Principles Explained");
```

### Using C# Events

```csharp
// Modern C# approach using events
public class Stock
{
    private decimal _price;
    public string Symbol { get; set; }

    public decimal Price
    {
        get => _price;
        set
        {
            if (_price != value)
            {
                _price = value;
                OnPriceChanged(new PriceChangedEventArgs(Symbol, value));
            }
        }
    }

    // Event
    public event EventHandler<PriceChangedEventArgs> PriceChanged;

    protected virtual void OnPriceChanged(PriceChangedEventArgs e)
    {
        PriceChanged?.Invoke(this, e);
    }
}

public class PriceChangedEventArgs : EventArgs
{
    public string Symbol { get; }
    public decimal NewPrice { get; }

    public PriceChangedEventArgs(string symbol, decimal newPrice)
    {
        Symbol = symbol;
        NewPrice = newPrice;
    }
}

// Observers
public class StockTrader
{
    public string Name { get; set; }

    public StockTrader(string name)
    {
        Name = name;
    }

    public void OnPriceChanged(object sender, PriceChangedEventArgs e)
    {
        Console.WriteLine($"{Name} notified: {e.Symbol} is now ${e.NewPrice}");
    }
}

// Usage
var stock = new Stock { Symbol = "AAPL", Price = 150 };

var trader1 = new StockTrader("Alice");
var trader2 = new StockTrader("Bob");

stock.PriceChanged += trader1.OnPriceChanged;
stock.PriceChanged += trader2.OnPriceChanged;

stock.Price = 155; // Both traders get notified
stock.Price = 160; // Both traders get notified again
```

### Benefits

- Loose coupling between subject and observers
- Dynamic subscription at runtime
- Supports broadcast communication

### When to Use

- When changes to one object require changing others
- When an object should notify other objects without knowing who they are
- When you need a publish-subscribe mechanism

---

## Strategy Pattern

**Defines a family of algorithms, encapsulates each one, and makes them interchangeable.**

### Problem

You have multiple ways to perform an operation, and you want to switch between them at runtime.

### Solution

```csharp
// Strategy interface
public interface IPaymentStrategy
{
    void Pay(decimal amount);
}

// Concrete strategies
public class CreditCardPayment : IPaymentStrategy
{
    private string _cardNumber;
    private string _cvv;

    public CreditCardPayment(string cardNumber, string cvv)
    {
        _cardNumber = cardNumber;
        _cvv = cvv;
    }

    public void Pay(decimal amount)
    {
        Console.WriteLine($"Paid ${amount} with Credit Card ending in {_cardNumber.Substring(_cardNumber.Length - 4)}");
    }
}

public class PayPalPayment : IPaymentStrategy
{
    private string _email;

    public PayPalPayment(string email)
    {
        _email = email;
    }

    public void Pay(decimal amount)
    {
        Console.WriteLine($"Paid ${amount} via PayPal account {_email}");
    }
}

public class BitcoinPayment : IPaymentStrategy
{
    private string _walletAddress;

    public BitcoinPayment(string walletAddress)
    {
        _walletAddress = walletAddress;
    }

    public void Pay(decimal amount)
    {
        Console.WriteLine($"Paid ${amount} via Bitcoin to wallet {_walletAddress}");
    }
}

// Context
public class ShoppingCart
{
    private List<string> _items = new();
    private IPaymentStrategy _paymentStrategy;

    public void AddItem(string item)
    {
        _items.Add(item);
        Console.WriteLine($"Added {item} to cart");
    }

    public void SetPaymentMethod(IPaymentStrategy strategy)
    {
        _paymentStrategy = strategy;
    }

    public void Checkout(decimal amount)
    {
        if (_paymentStrategy == null)
        {
            throw new InvalidOperationException("Payment method not set");
        }

        Console.WriteLine($"\nChecking out {_items.Count} items...");
        _paymentStrategy.Pay(amount);
        Console.WriteLine("Order complete!\n");
    }
}

// Usage
var cart = new ShoppingCart();
cart.AddItem("Laptop");
cart.AddItem("Mouse");

// Pay with credit card
cart.SetPaymentMethod(new CreditCardPayment("1234567890123456", "123"));
cart.Checkout(1299.99m);

// Pay with PayPal
cart.SetPaymentMethod(new PayPalPayment("user@example.com"));
cart.Checkout(49.99m);
```

### Real-World Example: Sorting Strategies

```csharp
public interface ISortStrategy
{
    void Sort(List<int> list);
}

public class QuickSort : ISortStrategy
{
    public void Sort(List<int> list)
    {
        Console.WriteLine("Sorting using QuickSort");
        list.Sort(); // Using built-in sort
    }
}

public class BubbleSort : ISortStrategy
{
    public void Sort(List<int> list)
    {
        Console.WriteLine("Sorting using BubbleSort");
        // Bubble sort implementation
        for (int i = 0; i < list.Count - 1; i++)
        {
            for (int j = 0; j < list.Count - i - 1; j++)
            {
                if (list[j] > list[j + 1])
                {
                    (list[j], list[j + 1]) = (list[j + 1], list[j]);
                }
            }
        }
    }
}

public class SortContext
{
    private ISortStrategy _strategy;

    public void SetStrategy(ISortStrategy strategy)
    {
        _strategy = strategy;
    }

    public void Sort(List<int> list)
    {
        _strategy.Sort(list);
    }
}

// Usage
var numbers = new List<int> { 5, 2, 8, 1, 9 };
var context = new SortContext();

context.SetStrategy(new QuickSort());
context.Sort(numbers);
```

### Benefits

- Eliminates conditional statements
- Easy to add new strategies
- Strategies can be swapped at runtime

### When to Use

- When you have multiple algorithms for a specific task
- When you want to avoid conditional statements
- When algorithms should be interchangeable at runtime

---

## Command Pattern

**Encapsulates a request as an object, allowing parameterization and queuing of requests.**

### Problem

You want to issue requests without knowing the receiver or the operation being performed.

### Solution

```csharp
// Command interface
public interface ICommand
{
    void Execute();
    void Undo();
}

// Receiver
public class Light
{
    public void TurnOn()
    {
        Console.WriteLine("Light is ON");
    }

    public void TurnOff()
    {
        Console.WriteLine("Light is OFF");
    }
}

// Concrete commands
public class TurnOnCommand : ICommand
{
    private Light _light;

    public TurnOnCommand(Light light)
    {
        _light = light;
    }

    public void Execute()
    {
        _light.TurnOn();
    }

    public void Undo()
    {
        _light.TurnOff();
    }
}

public class TurnOffCommand : ICommand
{
    private Light _light;

    public TurnOffCommand(Light light)
    {
        _light = light;
    }

    public void Execute()
    {
        _light.TurnOff();
    }

    public void Undo()
    {
        _light.TurnOn();
    }
}

// Invoker
public class RemoteControl
{
    private Stack<ICommand> _commandHistory = new();

    public void ExecuteCommand(ICommand command)
    {
        command.Execute();
        _commandHistory.Push(command);
    }

    public void UndoLastCommand()
    {
        if (_commandHistory.Count > 0)
        {
            var command = _commandHistory.Pop();
            command.Undo();
        }
        else
        {
            Console.WriteLine("No commands to undo");
        }
    }
}

// Usage
var light = new Light();
var remote = new RemoteControl();

var turnOn = new TurnOnCommand(light);
var turnOff = new TurnOffCommand(light);

remote.ExecuteCommand(turnOn);    // Light is ON
remote.ExecuteCommand(turnOff);   // Light is OFF
remote.UndoLastCommand();         // Light is ON (undo off)
remote.UndoLastCommand();         // Light is OFF (undo on)
```

### Text Editor Example

```csharp
public class TextEditor
{
    public string Text { get; set; } = "";

    public void Append(string text)
    {
        Text += text;
    }

    public void Delete(int length)
    {
        if (length <= Text.Length)
            Text = Text.Substring(0, Text.Length - length);
    }
}

public class AppendCommand : ICommand
{
    private TextEditor _editor;
    private string _text;

    public AppendCommand(TextEditor editor, string text)
    {
        _editor = editor;
        _text = text;
    }

    public void Execute()
    {
        _editor.Append(_text);
    }

    public void Undo()
    {
        _editor.Delete(_text.Length);
    }
}

// Usage
var editor = new TextEditor();
var remote = new RemoteControl();

remote.ExecuteCommand(new AppendCommand(editor, "Hello "));
remote.ExecuteCommand(new AppendCommand(editor, "World!"));
Console.WriteLine(editor.Text); // "Hello World!"

remote.UndoLastCommand();
Console.WriteLine(editor.Text); // "Hello "
```

### Benefits

- Decouples sender and receiver
- Supports undo/redo operations
- Can queue and log commands

### When to Use

- When you need undo/redo functionality
- When you want to queue operations
- When you need to log or audit operations

---

## Template Method Pattern

**Defines the skeleton of an algorithm, deferring some steps to subclasses.**

### Problem

You have an algorithm with steps that vary, but the overall structure remains the same.

### Solution

```csharp
// Abstract class
public abstract class DataProcessor
{
    // Template method
    public void Process()
    {
        LoadData();
        ValidateData();
        ProcessData();
        SaveData();
    }

    protected abstract void LoadData();
    protected abstract void ValidateData();
    protected abstract void ProcessData();

    // Hook method (optional override)
    protected virtual void SaveData()
    {
        Console.WriteLine("Saving data to default location");
    }
}

// Concrete implementations
public class CsvDataProcessor : DataProcessor
{
    protected override void LoadData()
    {
        Console.WriteLine("Loading data from CSV file");
    }

    protected override void ValidateData()
    {
        Console.WriteLine("Validating CSV data format");
    }

    protected override void ProcessData()
    {
        Console.WriteLine("Processing CSV data");
    }
}

public class JsonDataProcessor : DataProcessor
{
    protected override void LoadData()
    {
        Console.WriteLine("Loading data from JSON file");
    }

    protected override void ValidateData()
    {
        Console.WriteLine("Validating JSON schema");
    }

    protected override void ProcessData()
    {
        Console.WriteLine("Processing JSON data");
    }

    protected override void SaveData()
    {
        Console.WriteLine("Saving data to JSON format");
    }
}

// Usage
DataProcessor csvProcessor = new CsvDataProcessor();
csvProcessor.Process();

Console.WriteLine();

DataProcessor jsonProcessor = new JsonDataProcessor();
jsonProcessor.Process();
```

### Benefits

- Reuses common code in the base class
- Prevents code duplication
- Enforces a specific algorithm structure

### When to Use

- When you have algorithms with similar steps
- When you want to let subclasses override specific steps
- When you want to prevent code duplication

---

## Chain of Responsibility Pattern

**Passes a request along a chain of handlers until one handles it.**

### Problem

You want to give multiple objects a chance to handle a request without coupling the sender to the receiver.

### Solution

```csharp
// Handler interface
public abstract class SupportHandler
{
    protected SupportHandler _nextHandler;

    public void SetNext(SupportHandler handler)
    {
        _nextHandler = handler;
    }

    public abstract void HandleRequest(SupportTicket ticket);
}

// Request
public class SupportTicket
{
    public int Priority { get; set; }
    public string Issue { get; set; }
}

// Concrete handlers
public class Level1Support : SupportHandler
{
    public override void HandleRequest(SupportTicket ticket)
    {
        if (ticket.Priority <= 1)
        {
            Console.WriteLine($"Level 1 Support handled: {ticket.Issue}");
        }
        else if (_nextHandler != null)
        {
            _nextHandler.HandleRequest(ticket);
        }
    }
}

public class Level2Support : SupportHandler
{
    public override void HandleRequest(SupportTicket ticket)
    {
        if (ticket.Priority <= 2)
        {
            Console.WriteLine($"Level 2 Support handled: {ticket.Issue}");
        }
        else if (_nextHandler != null)
        {
            _nextHandler.HandleRequest(ticket);
        }
    }
}

public class Level3Support : SupportHandler
{
    public override void HandleRequest(SupportTicket ticket)
    {
        Console.WriteLine($"Level 3 Support handled: {ticket.Issue}");
    }
}

// Usage
var level1 = new Level1Support();
var level2 = new Level2Support();
var level3 = new Level3Support();

level1.SetNext(level2);
level2.SetNext(level3);

var ticket1 = new SupportTicket { Priority = 1, Issue = "Password reset" };
var ticket2 = new SupportTicket { Priority = 2, Issue = "Software bug" };
var ticket3 = new SupportTicket { Priority = 3, Issue = "System outage" };

level1.HandleRequest(ticket1); // Handled by Level 1
level1.HandleRequest(ticket2); // Handled by Level 2
level1.HandleRequest(ticket3); // Handled by Level 3
```

### Benefits

- Reduces coupling
- Adds flexibility in assigning responsibilities
- Easy to add new handlers

### When to Use

- When more than one object can handle a request
- When you want to issue a request to one of several objects without specifying the receiver
- When the set of handlers should be specified dynamically

---

## State Pattern

**Allows an object to alter its behavior when its internal state changes.**

### Problem

An object needs to change its behavior based on its state, and you want to avoid large conditional statements.

### Solution

```csharp
// State interface
public interface IState
{
    void InsertCoin(VendingMachine machine);
    void PressButton(VendingMachine machine);
    void Dispense(VendingMachine machine);
}

// Context
public class VendingMachine
{
    public IState CurrentState { get; set; }
    public int ItemCount { get; set; }

    public VendingMachine(int itemCount)
    {
        ItemCount = itemCount;
        CurrentState = itemCount > 0 ? new NoCoinState() : new OutOfStockState();
    }

    public void InsertCoin() => CurrentState.InsertCoin(this);
    public void PressButton() => CurrentState.PressButton(this);
    public void Dispense() => CurrentState.Dispense(this);
}

// Concrete states
public class NoCoinState : IState
{
    public void InsertCoin(VendingMachine machine)
    {
        Console.WriteLine("Coin inserted");
        machine.CurrentState = new HasCoinState();
    }

    public void PressButton(VendingMachine machine)
    {
        Console.WriteLine("Insert coin first");
    }

    public void Dispense(VendingMachine machine)
    {
        Console.WriteLine("Insert coin first");
    }
}

public class HasCoinState : IState
{
    public void InsertCoin(VendingMachine machine)
    {
        Console.WriteLine("Coin already inserted");
    }

    public void PressButton(VendingMachine machine)
    {
        Console.WriteLine("Button pressed");
        machine.CurrentState = new DispensingState();
        machine.Dispense();
    }

    public void Dispense(VendingMachine machine)
    {
        Console.WriteLine("Press button first");
    }
}

public class DispensingState : IState
{
    public void InsertCoin(VendingMachine machine)
    {
        Console.WriteLine("Please wait, dispensing item");
    }

    public void PressButton(VendingMachine machine)
    {
        Console.WriteLine("Please wait, dispensing item");
    }

    public void Dispense(VendingMachine machine)
    {
        Console.WriteLine("Item dispensed");
        machine.ItemCount--;

        if (machine.ItemCount > 0)
            machine.CurrentState = new NoCoinState();
        else
            machine.CurrentState = new OutOfStockState();
    }
}

public class OutOfStockState : IState
{
    public void InsertCoin(VendingMachine machine)
    {
        Console.WriteLine("Out of stock, returning coin");
    }

    public void PressButton(VendingMachine machine)
    {
        Console.WriteLine("Out of stock");
    }

    public void Dispense(VendingMachine machine)
    {
        Console.WriteLine("Out of stock");
    }
}

// Usage
var machine = new VendingMachine(2);

machine.InsertCoin();    // Coin inserted
machine.PressButton();   // Button pressed, Item dispensed

machine.InsertCoin();    // Coin inserted
machine.PressButton();   // Button pressed, Item dispensed (last item)

machine.InsertCoin();    // Out of stock, returning coin
```

### Benefits

- Organizes state-specific code
- Makes state transitions explicit
- Eliminates large conditional statements

### When to Use

- When an object's behavior depends on its state
- When operations have large conditional statements based on state
- When state transitions are complex

---

## Comparison Table

| Pattern | Purpose | Use When |
|---------|---------|----------|
| **Observer** | One-to-many dependency notification | Multiple objects need updates when one changes |
| **Strategy** | Interchangeable algorithms | Multiple ways to perform an operation |
| **Command** | Encapsulate requests as objects | Need undo/redo or queue operations |
| **Template Method** | Define algorithm skeleton | Algorithm steps vary but structure is same |
| **Chain of Responsibility** | Pass request along chain | Multiple handlers for a request |
| **State** | Change behavior based on state | Behavior varies with state |

## Related Articles

- [Creational Design Patterns](creational-patterns-csharp.md)
- [Structural Design Patterns](structural-patterns-csharp.md)
- [SOLID Principles](solid-principles-csharp.md)
- [Design Patterns Overview](design-patterns-csharp-overview.md)

---

**Technology:** C# / .NET 8.0
**Category:** Behavioral Patterns
