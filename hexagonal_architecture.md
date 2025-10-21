# 🧩 [Hexagonal Architecture (Ports and Adapters)]([url](https://dev.to/bytehide/hexagonal-architectural-pattern-in-c-full-guide-2024-3fhp))

**Hexagonal Architecture**, also known as **Ports and Adapters Architecture**, is a software design pattern introduced by **Alistair Cockburn**.  
It focuses on making the application **independent of frameworks, databases, and external systems**, allowing for **flexibility, testability, and maintainability**.

---

## 🎯 Core Idea

In traditional layered architecture, dependencies usually point inward (UI → Service → Repository → Database).  
In Hexagonal Architecture, the **core application** is **isolated from the outside world** — meaning the business logic doesn’t depend on external systems; rather, **external systems depend on the core**.

---

## 🏗️ Structure Overview

The system is divided into **three main parts**:

### 1. Domain (Core)

- Contains the **business logic** and **domain entities**.
- Completely **independent of any frameworks or databases**.
- Has **no knowledge** of how input/output operations are performed.

**Example:**
```csharp
// Domain Layer
public class Order
{
    public int Id { get; set; }
    public decimal Amount { get; set; }
    public bool IsPaid { get; private set; }

    public void Pay()
    {
        if (IsPaid) throw new InvalidOperationException("Already paid");
        IsPaid = true;
    }
}
```

---

### 2. Ports

Ports define **interfaces** that describe how the core interacts with the outside world.  
They are **contracts**, not implementations.

There are two types of ports:

- **Inbound Ports:** Define how external actors (like APIs, UIs) can interact with the application core.
- **Outbound Ports:** Define how the core interacts with external systems (like databases, message brokers, external APIs).

**Example:**
```csharp
// Inbound Port
public interface IOrderService
{
    void PayOrder(int orderId);
}

// Outbound Port
public interface IOrderRepository
{
    Order GetById(int id);
    void Save(Order order);
}
```

---

### 3. Adapters

Adapters are **implementations** of the ports.  
They are responsible for translating communication between the core and the external world.

- **Inbound Adapters:** Implement the entry points (e.g., REST Controllers, CLI commands).
- **Outbound Adapters:** Implement external communication (e.g., database repositories, third-party API calls).

**Example:**
```csharp
// Outbound Adapter (Repository Implementation)
public class SqlOrderRepository : IOrderRepository
{
    private readonly AppDbContext _db;

    public SqlOrderRepository(AppDbContext db)
    {
        _db = db;
    }

    public Order GetById(int id) => _db.Orders.Find(id);

    public void Save(Order order)
    {
        _db.Orders.Update(order);
        _db.SaveChanges();
    }
}
```

```csharp
// Inbound Adapter (Service Implementation)
public class OrderService : IOrderService
{
    private readonly IOrderRepository _repository;

    public OrderService(IOrderRepository repository)
    {
        _repository = repository;
    }

    public void PayOrder(int orderId)
    {
        var order = _repository.GetById(orderId);
        order.Pay();
        _repository.Save(order);
    }
}
```

---

## ⚙️ Flow of Control

1. A **user** interacts with the system via an interface (e.g., REST API, CLI).
2. The **inbound adapter** handles the request and invokes the **core logic** through the **inbound port**.
3. The **core** executes business logic and may call **outbound ports** to interact with external systems.
4. **Outbound adapters** perform those external operations (e.g., database writes, sending emails).

---

## 🧱 Example Folder Structure

```
/src
 ├── ApplicationCore
 │    ├── Entities
 │    ├── Interfaces (Ports)
 │    └── Services (Business Logic)
 │
 ├── Infrastructure
 │    ├── Persistence (EF Core, MongoDB)
 │    └── ExternalServices
 │
 ├── WebApi
 │    ├── Controllers
 │    └── DTOs
```

---

Hexagonal architecture maps nicely in .NET Core Web API:

Hexagonal Concept	.NET Example
Domain	Entities, Services, Interfaces
Inbound Port	Application Service Interface
Outbound Port	Repository Interface
Inbound Adapter	Controller
Outbound Adapter	Repository Implementation, External API Service

## 🧠 Key Benefits

- ✅ **Independent of frameworks:** Core logic is not tied to any specific technology.
- ✅ **Easily testable:** Business logic can be tested without real databases or APIs.
- ✅ **Maintainable:** Changes in I/O layers don’t affect the domain layer.
- ✅ **Flexible:** You can replace adapters (like database or API clients) without modifying core logic.
- ✅ **Scalable:** Works well for complex, evolving systems and microservices.

---

## 🧩 Summary

Hexagonal Architecture ensures that the **application core is central and stable**, while **adapters around it can change** freely.  
By defining clear boundaries through **ports (interfaces)** and **adapters (implementations)**, it creates a clean separation between **business logic** and **technical details**, leading to a more modular and maintainable system.
