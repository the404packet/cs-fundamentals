## 1. What is Singleton?

**Singleton** is a **Creational Design Pattern** that ensures:

1. A class has **only one instance/object**.
2. That instance can be accessed from anywhere in the application.

```text
Normal Class

Class
 ├── Object 1
 ├── Object 2
 └── Object 3


Singleton Class

Class
 └── Only One Object
       ↑
   shared everywhere
```

---

# 2. What Problem Does Singleton Solve?

Sometimes we have a resource that should have **only one shared instance** throughout the application.

Examples:

* Application configuration
* Logger
* Cache
* Database connection manager
* Printer spooler
* Thread pool
* Device/resource manager

Suppose we have:

```java
class AppConfig {
    String databaseUrl;
}
```

Different parts of the application could create different objects:

```java
AppConfig config1 = new AppConfig();
AppConfig config2 = new AppConfig();
AppConfig config3 = new AppConfig();
```

Now we have:

```text
Service A → config1
Service B → config2
Service C → config3
```

This can lead to:

* unnecessary objects
* inconsistent state
* duplicated expensive resources
* different components using different configurations

What we actually want:

```text
             AppConfig
                 |
          Single Instance
          /      |      \
         /       |       \
Service A    Service B    Service C
```

Singleton provides this.

---

# 3. Basic Singleton Implementation

```java
public class AppConfig {

    private static AppConfig instance;

    private AppConfig() {
    }

    public static AppConfig getInstance() {

        if (instance == null) {
            instance = new AppConfig();
        }

        return instance;
    }
}
```

Usage:

```java
AppConfig a = AppConfig.getInstance();
AppConfig b = AppConfig.getInstance();

System.out.println(a == b);
```

Output:

```text
true
```

Both variables point to the **same object**.

---

# 4. How Does Singleton Work?

There are three important parts.

## Step 1: Private Constructor

```java
private AppConfig() {
}
```

Normally:

```java
AppConfig config = new AppConfig();
```

would create an object.

But because the constructor is `private`, outside classes cannot do this.

```java
new AppConfig(); // ❌ not allowed
```

This prevents arbitrary creation of objects.

---

## Step 2: Static Instance

```java
private static AppConfig instance;
```

The class itself stores its single object.

Because it is `static`, it belongs to the **class**, not to an individual object.

Initially:

```text
instance = null
```

---

## Step 3: getInstance()

```java
public static AppConfig getInstance() {

    if (instance == null) {
        instance = new AppConfig();
    }

    return instance;
}
```

First call:

```text
getInstance()

instance == null
       ↓
create object
       ↓
store object
       ↓
return object
```

Later calls:

```text
getInstance()

instance already exists
       ↓
return same object
```

---

# 5. Complete Example — Logger

Suppose the whole application should use one logger.

```java
public class Logger {

    private static Logger instance;

    private Logger() {
    }

    public static Logger getInstance() {

        if (instance == null) {
            instance = new Logger();
        }

        return instance;
    }

    public void log(String message) {
        System.out.println("[LOG] " + message);
    }
}
```

Usage:

```java
public class Main {

    public static void main(String[] args) {

        Logger logger1 = Logger.getInstance();
        Logger logger2 = Logger.getInstance();

        logger1.log("Application started");
        logger2.log("User logged in");

        System.out.println(logger1 == logger2);
    }
}
```

Output:

```text
[LOG] Application started
[LOG] User logged in
true
```

Only one `Logger` object exists.

---

# 6. Lazy Initialization

The previous implementation is called **Lazy Initialization**.

```java
private static Logger instance;

public static Logger getInstance() {

    if (instance == null) {
        instance = new Logger();
    }

    return instance;
}
```

The object is created **only when it is first needed**.

```text
Application starts
      ↓
No Singleton object

getInstance()
      ↓
Object created
```

### Advantage

If creating the object is expensive, we don't create it unless it is actually needed.

### Problem

This implementation is **not thread-safe**.

---

# 7. Thread-Safety Problem

Imagine two threads call:

```java
Logger.getInstance();
```

at exactly the same time.

```text
Thread 1                     Thread 2

instance == null             instance == null

new Logger()                 new Logger()
     ↓                            ↓
 Object A                     Object B
```

Now we accidentally created **two objects**.

That breaks the Singleton guarantee.

---

# 8. Thread-Safe Singleton Using synchronized

One solution:

```java
public class Logger {

    private static Logger instance;

    private Logger() {
    }

    public static synchronized Logger getInstance() {

        if (instance == null) {
            instance = new Logger();
        }

        return instance;
    }
}
```

`synchronized` means only one thread can execute `getInstance()` at a time.

```text
Thread 1 → getInstance()
              |
           locked
              |
Thread 2 → waiting
```

### Problem

Every call requires synchronization.

Even after the object has already been created:

```java
Logger.getInstance();
```

still goes through synchronization.

That can add unnecessary overhead.

---

# 9. Double-Checked Locking

A more optimized version:

```java
public class Logger {

    private static volatile Logger instance;

    private Logger() {
    }

    public static Logger getInstance() {

        if (instance == null) {

            synchronized (Logger.class) {

                if (instance == null) {
                    instance = new Logger();
                }

            }
        }

        return instance;
    }
}
```

Notice there are **two checks**:

```java
if (instance == null)
```

and inside synchronization:

```java
if (instance == null)
```

Hence:

**Double-Checked Locking**

### Why second check?

Suppose:

```text
Thread 1 → passes first check
Thread 2 → passes first check

Thread 1 → gets lock
           creates object

Thread 2 → gets lock afterward
```

Without the second check, Thread 2 would create another object.

The second check sees:

```text
instance != null
```

and doesn't create another object.

---

# 10. Why `volatile`?

```java
private static volatile Logger instance;
```

`volatile` helps make sure threads see the latest value of `instance` and prevents problematic instruction reordering around object initialization.

For interviews:

> `volatile` is required with double-checked locking so threads safely observe a fully initialized Singleton instance.

---

# 11. Eager Initialization

Instead of creating the object when requested:

```java
public class Logger {

    private static final Logger INSTANCE = new Logger();

    private Logger() {
    }

    public static Logger getInstance() {
        return INSTANCE;
    }
}
```

The instance is created when the class is initialized.

```text
Class loaded
     ↓
Logger object created
     ↓
getInstance()
     ↓
return object
```

### Advantages

* Very simple
* Thread-safe because Java class initialization is thread-safe
* No synchronization code

### Disadvantage

The object is created even if the application never uses it.

---

# 12. Initialization-on-Demand Holder

A clean lazy Singleton implementation in Java:

```java
public class Logger {

    private Logger() {
    }

    private static class Holder {
        private static final Logger INSTANCE = new Logger();
    }

    public static Logger getInstance() {
        return Holder.INSTANCE;
    }
}
```

The nested `Holder` class isn't initialized until:

```java
getInstance()
```

is called.

Therefore we get:

* Lazy initialization
* Thread safety
* No explicit synchronization
* Simple code

This is often a good implementation when you specifically need a Singleton class.

---

# 13. Enum Singleton

Another strong Java implementation:

```java
public enum Logger {

    INSTANCE;

    public void log(String message) {
        System.out.println(message);
    }
}
```

Usage:

```java
Logger.INSTANCE.log("Hello");
```

Advantages:

* Very simple
* Thread-safe
* Serialization-safe
* Resistant to reflection-based duplicate construction

For many Java cases, an enum Singleton is considered one of the safest implementations.

---

# 14. Singleton Structure

Conceptually:

```text
+----------------------------+
|         Singleton          |
+----------------------------+
| - instance : Singleton     |
+----------------------------+
| - Singleton()              |
| + getInstance(): Singleton |
+----------------------------+
```

Key ideas:

```text
private constructor
        +
static instance
        +
public getInstance()
        =
Singleton
```

---

# 15. Real-World Example — Configuration

```java
public class Configuration {

    private static final Configuration INSTANCE =
            new Configuration();

    private String databaseUrl;

    private Configuration() {
        databaseUrl = "jdbc:mysql://localhost:3306/app";
    }

    public static Configuration getInstance() {
        return INSTANCE;
    }

    public String getDatabaseUrl() {
        return databaseUrl;
    }
}
```

Now:

```java
class UserService {

    public void connect() {

        Configuration config =
                Configuration.getInstance();

        System.out.println(config.getDatabaseUrl());
    }
}
```

And another service:

```java
class PaymentService {

    public void connect() {

        Configuration config =
                Configuration.getInstance();

        System.out.println(config.getDatabaseUrl());
    }
}
```

Both access the same configuration object.

---

# 16. Why Can Singleton Become "Brittle"?

Singleton solves a real problem, but it is easy to misuse.

The biggest problem is that Singleton can become **global state**.

Example:

```java
class OrderService {

    public void order() {

        Logger logger = Logger.getInstance();

        logger.log("Order created");
    }
}
```

At first this looks convenient.

But now `OrderService` secretly depends on `Logger`.

Looking at:

```java
new OrderService();
```

you cannot tell that it needs a logger.

The dependency is hidden inside:

```java
Logger.getInstance();
```

This makes the code more tightly coupled.

---

# 17. Testing Problem

Suppose:

```java
class PaymentService {

    public void pay() {

        Database db = Database.getInstance();

        db.save();
    }
}
```

During testing, we may want:

```text
PaymentService
      ↓
FakeDatabase
```

instead of:

```text
PaymentService
      ↓
RealDatabase
```

But `PaymentService` directly calls:

```java
Database.getInstance();
```

So replacing it with a fake implementation becomes harder.

This is one of the major reasons excessive Singleton usage makes applications brittle.

---

# 18. How to Make Singleton Less Brittle

## Use Dependency Injection

Instead of:

```java
class PaymentService {

    public void pay() {

        Database db = Database.getInstance();

        db.save();
    }
}
```

Do:

```java
class PaymentService {

    private final Database db;

    public PaymentService(Database db) {
        this.db = db;
    }

    public void pay() {
        db.save();
    }
}
```

Now:

```java
Database db = Database.getInstance();

PaymentService service =
        new PaymentService(db);
```

The application decides which instance to provide.

`PaymentService` doesn't care whether it is Singleton.

---

# 19. Even Better — Depend on an Interface

```java
interface Database {

    void save();
}
```

Real implementation:

```java
class MySQLDatabase implements Database {

    public void save() {
        System.out.println("Saving to MySQL");
    }
}
```

Service:

```java
class PaymentService {

    private final Database db;

    public PaymentService(Database db) {
        this.db = db;
    }

    public void pay() {
        db.save();
    }
}
```

Production:

```java
Database db = new MySQLDatabase();

PaymentService service =
        new PaymentService(db);
```

Testing:

```java
class FakeDatabase implements Database {

    public void save() {
        System.out.println("Fake save");
    }
}
```

Then:

```java
PaymentService service =
        new PaymentService(new FakeDatabase());
```

Now the service is much easier to test.

---

# 20. Singleton + Dependency Injection

A useful principle is:

> The fact that an object has one instance should ideally be controlled by the application's object-management layer rather than every class calling `getInstance()`.

Instead of:

```text
Service
   ↓
Singleton.getInstance()
```

prefer:

```text
Application / DI Container
          ↓
    Single Instance
       /      \
Service A    Service B
```

Both services still receive the **same object**, but they don't know or care that it is a Singleton.

This reduces coupling.

---

# 21. Singleton in Spring Boot

This is especially important for Spring interviews.

By default, Spring beans use **singleton scope**.

```java
@Service
public class PaymentService {
}
```

Spring normally creates one `PaymentService` bean per application context.

Similarly:

```java
@Repository
public class UserRepository {
}
```

or:

```java
@Component
public class Logger {
}
```

Spring manages the lifecycle.

You usually **do not need to manually implement**:

```java
private static instance;
getInstance();
```

Instead:

```java
@Service
class OrderService {

    private final Logger logger;

    public OrderService(Logger logger) {
        this.logger = logger;
    }
}
```

Spring injects the shared `Logger` bean.

```text
             Spring Container
                   |
               Logger Bean
               /         \
              ↓           ↓
       OrderService   UserService
```

This gives the benefits of one shared object without hard-coding `Logger.getInstance()` everywhere.

---

# 22. When Should You Use Singleton?

Singleton can make sense when:

* Exactly one instance logically represents a resource.
* The object is expensive to create.
* Multiple copies could cause inconsistent behavior.
* Shared coordination is required.

Possible examples:

```text
Configuration manager
Logger
Application-wide cache
Resource manager
Thread pool manager
```

But don't automatically make every service a Singleton.

---

# 23. When Should You Avoid Singleton?

Avoid or reconsider it when:

* You only want Singleton for convenience.
* It stores lots of mutable global state.
* Unit tests need different implementations.
* Multiple instances might be required later.
* Classes directly call `getInstance()` everywhere.
* A dependency-injection framework already manages object lifetimes.

Bad:

```java
Singleton.getInstance().doSomething();
```

scattered throughout the entire codebase.

Better:

```java
class Service {

    private final SomeDependency dependency;

    Service(SomeDependency dependency) {
        this.dependency = dependency;
    }
}
```

Let the application decide whether `SomeDependency` has one instance or many.

---

# 24. Singleton vs Static Class

These are related but not identical.

### Static

```java
class Utils {

    static void log() {
    }
}
```

Called using:

```java
Utils.log();
```

There is no object.

### Singleton

```java
Logger logger = Logger.getInstance();
```

There **is an object**, but only one instance is intended.

Because it is an object, Singleton can:

* implement interfaces
* be passed as a dependency
* potentially be substituted/mocked
* maintain object state
* participate in object-oriented design

---

# 25. Quick Comparison

| Implementation         | Lazy                    | Thread Safe | Complexity |
| ---------------------- | ----------------------- | ----------- | ---------- |
| Basic Singleton        | ✅                       | ❌           | Low        |
| `synchronized`         | ✅                       | ✅           | Low        |
| Double-checked locking | ✅                       | ✅           | Medium     |
| Eager initialization   | ❌                       | ✅           | Very Low   |
| Holder pattern         | ✅                       | ✅           | Low        |
| Enum Singleton         | effectively JVM-managed | ✅           | Very Low   |

---

# 26. Interview Answer

### What is Singleton?

> Singleton is a creational design pattern that ensures a class has only one instance and provides a globally accessible way to obtain that instance.

### How is it implemented?

Usually using:

```text
1. Private constructor
2. Static instance
3. Public static getInstance()
```

Example:

```java
class Singleton {

    private static Singleton instance;

    private Singleton() {
    }

    public static Singleton getInstance() {

        if (instance == null) {
            instance = new Singleton();
        }

        return instance;
    }
}
```

---

# 27. Important Interview Follow-Up: Is Singleton Thread-Safe?

Basic implementation:

```java
if (instance == null) {
    instance = new Singleton();
}
```

❌ Not thread-safe.

Possible solutions:

```text
synchronized method
double-checked locking + volatile
eager initialization
holder pattern
enum Singleton
```

---

# 28. Important Interview Follow-Up: Why Is Singleton Sometimes Considered an Anti-Pattern?

Because badly used Singletons create:

```text
Global state
     ↓
Hidden dependencies
     ↓
Tight coupling
     ↓
Harder testing
     ↓
Brittle code
```

Singleton itself isn't automatically bad.

The bigger problem is **using global access everywhere**.

---

# 29. Making Singleton Less Brittle — Checklist

Prefer:

```text
✅ Keep Singleton state minimal
✅ Avoid mutable global state
✅ Use dependency injection
✅ Depend on interfaces
✅ Don't scatter getInstance() everywhere
✅ Let DI containers manage object lifecycle when possible
✅ Make thread safety explicit
```

Avoid:

```text
❌ Singleton.getInstance() everywhere
❌ Huge global Singleton objects
❌ Storing unrelated application state
❌ Using Singleton just because it is convenient
❌ Manual Singleton when Spring already manages the object
```

---

# 30. Mental Model

Remember Singleton as:

```text
Problem
   ↓
"I need exactly one shared instance"

Solution
   ↓
Private Constructor
+
Static Instance
+
getInstance()

But...

Global getInstance() everywhere
   ↓
Tight Coupling
   ↓
Hard Testing

Better Architecture
   ↓
Create/Manage One Instance
   ↓
Inject It Where Needed
```

---

# 31. 30-Second Revision

```text
SINGLETON = One instance of a class

Purpose:
→ Shared resource
→ Prevent multiple instances

Implementation:
→ private constructor
→ static instance
→ static getInstance()

Basic lazy implementation:
→ NOT thread-safe

Thread-safe options:
→ synchronized
→ double-check + volatile
→ eager initialization
→ holder pattern
→ enum

Main drawback:
→ global state
→ hidden dependencies
→ tight coupling
→ difficult testing

Make it less brittle:
→ Dependency Injection
→ Interfaces
→ Avoid direct getInstance() everywhere
→ Keep mutable state minimal

Spring Boot:
→ Beans are singleton-scoped by default
→ Prefer constructor injection
→ Usually don't manually implement Singleton
```

### One-line interview answer

> **Singleton ensures that only one instance of a class exists and provides access to it; however, direct global access can create tight coupling, so in modern applications it is often better to manage the single instance through dependency injection.**
