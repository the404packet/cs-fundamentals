## 1. What is Prototype?

**Prototype** is a **Creational Design Pattern** that creates new objects by **copying/cloning an existing object** instead of creating them from scratch.

```text
Normal Creation

new Object()
     ↓
Initialize everything
     ↓
New Object


Prototype

Existing Object
     ↓
   clone()
     ↓
Copied Object
```

Simple idea:

> **Have an object you like? Clone it instead of rebuilding it.**

---

# 2. What Problem Does Prototype Solve?

Suppose creating an object is:

* Expensive
* Complicated
* Requires many configuration values
* Requires database/API operations
* Has lots of fields
* Needs almost the same configuration as an existing object

Example:

```java
class GameCharacter {

    String name;
    String weapon;
    int health;
    int attack;
    int defense;
    String armor;
    String ability;
}
```

Creating similar characters repeatedly:

```java
GameCharacter c1 = new GameCharacter();

c1.weapon = "Sword";
c1.health = 100;
c1.attack = 80;
c1.defense = 60;
c1.armor = "Iron";
c1.ability = "Fire";
```

Now we want another character with almost the same configuration.

Without Prototype:

```java
GameCharacter c2 = new GameCharacter();

c2.weapon = "Sword";
c2.health = 100;
c2.attack = 80;
c2.defense = 60;
c2.armor = "Iron";
c2.ability = "Fire";

c2.name = "Player 2";
```

We duplicated all the initialization.

Instead:

```java
GameCharacter c2 = c1.clone();

c2.name = "Player 2";
```

Much easier.

---

# 3. Core Idea

```text
             Prototype
                 |
             Existing
              Object
                 |
              clone()
             /       \
            ↓         ↓
         Copy 1     Copy 2
```

Instead of:

```java
new SomeComplexObject(...);
```

we do:

```java
existingObject.clone();
```

---

# 4. Basic Example

Let's implement our own `copy()` method.

```java
class Car {

    String brand;
    String color;
    int speed;

    public Car(String brand, String color, int speed) {
        this.brand = brand;
        this.color = color;
        this.speed = speed;
    }

    public Car copy() {
        return new Car(
            this.brand,
            this.color,
            this.speed
        );
    }
}
```

Usage:

```java
Car original = new Car(
    "BMW",
    "Black",
    250
);

Car copy = original.copy();
```

Now:

```text
original

brand = BMW
color = Black
speed = 250


copy

brand = BMW
color = Black
speed = 250
```

But:

```java
System.out.println(original == copy);
```

Output:

```text
false
```

They contain the same values but are **different objects**.

---

# 5. Prototype vs Singleton

Don't confuse them.

## Singleton

```text
Object
  ↑
Everyone uses SAME object
```

```java
Singleton.getInstance();
```

Goal:

> Create only **one object**.

---

## Prototype

```text
Original
   |
 clone()
   ↓
New Object
```

Goal:

> Create **many objects efficiently by copying**.

So:

```text
Singleton → reuse same object

Prototype → reuse object's configuration
            but create new object
```

---

# 6. Prototype Interface

A common implementation introduces a Prototype interface.

```java
interface Prototype {

    Prototype clone();
}
```

Classes implement it:

```java
class Car implements Prototype {

    String brand;
    String color;
    int speed;

    public Car(
        String brand,
        String color,
        int speed
    ) {
        this.brand = brand;
        this.color = color;
        this.speed = speed;
    }

    @Override
    public Car clone() {

        return new Car(
            brand,
            color,
            speed
        );
    }
}
```

Usage:

```java
Car car1 =
    new Car("BMW", "Black", 250);

Car car2 = car1.clone();
```

---

# 7. Why Use an Interface?

Without Prototype:

```text
Client
   ↓
Knows exactly how Car is created
```

With Prototype:

```text
Client
   ↓
Prototype.clone()
   ↓
Concrete object handles copying
```

The client doesn't need to understand all the internal details required to reproduce the object.

---

# 8. Main Problem: Shallow Copy vs Deep Copy

This is probably the **most important Prototype interview topic**.

Suppose:

```java
class Engine {

    int horsepower;

    Engine(int horsepower) {
        this.horsepower = horsepower;
    }
}
```

And:

```java
class Car {

    String name;
    Engine engine;
}
```

Now:

```java
Car car1 = ...
Car car2 = car1.clone();
```

What happens to `engine`?

There are two possibilities:

```text
Shallow Copy
Deep Copy
```

---

# 9. Shallow Copy

A shallow copy creates a new outer object but **shares referenced nested objects**.

Example:

```java
class Car {

    String name;
    Engine engine;

    public Car clone() {

        Car copy = new Car();

        copy.name = this.name;
        copy.engine = this.engine;

        return copy;
    }
}
```

Notice:

```java
copy.engine = this.engine;
```

Both cars point to the **same Engine object**.

```text
car1 ──────┐
           ↓
         Engine
           ↑
car2 ──────┘
```

Therefore:

```java
car2.engine.horsepower = 500;
```

also affects what `car1` sees:

```java
System.out.println(
    car1.engine.horsepower
);
```

Output:

```text
500
```

Because:

```java
car1.engine == car2.engine
```

is:

```text
true
```

---

# 10. Deep Copy

Deep copy also copies nested mutable objects.

```java
public Car clone() {

    Car copy = new Car();

    copy.name = this.name;

    copy.engine =
        new Engine(this.engine.horsepower);

    return copy;
}
```

Now:

```text
car1
 ↓
Engine A


car2
 ↓
Engine B
```

They are completely independent.

```java
car2.engine.horsepower = 500;
```

doesn't affect:

```java
car1.engine.horsepower
```

---

# 11. Shallow vs Deep Copy

| Shallow Copy                | Deep Copy                     |
| --------------------------- | ----------------------------- |
| Copies outer object         | Copies outer object           |
| Nested references shared    | Nested mutable objects copied |
| Faster                      | Usually more expensive        |
| Less memory                 | More memory                   |
| Changes may affect original | Objects are independent       |

Remember:

```text
SHALLOW

Object A ──┐
           ↓
        Nested
           ↑
Object B ──┘


DEEP

Object A → Nested A

Object B → Nested B
```

---

# 12. Important Detail: Immutable Objects

Not every referenced object needs to be deep-copied.

For example:

```java
String name;
```

Java `String` is immutable.

Sharing:

```java
copy.name = original.name;
```

is generally safe because the String itself cannot be modified.

Deep copying matters primarily for **mutable nested objects**.

---

# 13. Real Example — Game Enemies

Imagine creating enemies:

```java
class Enemy {

    String type;
    int health;
    int damage;
    Weapon weapon;
    Armor armor;
    Ability ability;
}
```

Creating each enemy manually:

```java
Enemy e1 = new Enemy(...);
Enemy e2 = new Enemy(...);
Enemy e3 = new Enemy(...);
```

requires repeating lots of configuration.

Instead create a prototype:

```text
Zombie Prototype

health = 100
damage = 30
weapon = Axe
armor = Basic
ability = Poison
```

Then:

```java
Enemy zombie1 = zombiePrototype.clone();
Enemy zombie2 = zombiePrototype.clone();
Enemy zombie3 = zombiePrototype.clone();
```

And customize them:

```java
zombie1.health = 120;

zombie2.weapon = sword;

zombie3.damage = 50;
```

This is a very natural use case for Prototype.

---

# 14. Another Example — Documents

Suppose an application creates reports.

Every report has:

```text
Company logo
Header
Footer
Font settings
Page configuration
Company details
Legal disclaimer
```

Creating all of that every time is unnecessary.

Create:

```text
StandardReport Prototype
```

Then:

```java
Report january =
    standardReport.clone();

Report february =
    standardReport.clone();
```

And customize:

```java
january.setMonth("January");

february.setMonth("February");
```

---

# 15. Prototype Registry

Sometimes applications maintain multiple predefined prototypes.

For example:

```text
Prototype Registry

"basic-zombie" → Zombie Prototype

"boss-zombie"  → Boss Prototype

"archer"       → Archer Prototype
```

Example:

```java
Map<String, Enemy> prototypes =
    new HashMap<>();

prototypes.put(
    "zombie",
    zombiePrototype
);

prototypes.put(
    "boss",
    bossPrototype
);
```

Then:

```java
Enemy enemy =
    prototypes.get("zombie").clone();
```

This is called a **Prototype Registry**.

It allows objects to be created from predefined templates.

---

# 16. Java's `Cloneable`

Java provides:

```java
Cloneable
```

Example:

```java
class Car implements Cloneable {

    String name;
    int speed;

    @Override
    public Car clone() {

        try {
            return (Car) super.clone();
        }
        catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

Usage:

```java
Car car2 = car1.clone();
```

However, `Object.clone()` performs a **shallow copy** by default.

So nested mutable objects still need special handling.

---

# 17. Why Java `Cloneable` Is Often Avoided

Although Java provides `Cloneable`, many developers prefer:

```java
copy constructors
```

or:

```java
copy()
```

methods.

Because `Cloneable` / `Object.clone()` can be awkward:

* Shallow copying by default
* Constructor isn't normally called during `Object.clone()`
* Handling nested objects is easy to get wrong
* The API design is unusual
* Checked exception complications in some implementations

So Prototype **doesn't mean you must use `Cloneable`**.

Prototype is the **design idea**.

You can implement it however you want.

---

# 18. Copy Constructor

A very clean Java alternative:

```java
class Car {

    String brand;
    String color;
    int speed;

    public Car(
        String brand,
        String color,
        int speed
    ) {

        this.brand = brand;
        this.color = color;
        this.speed = speed;
    }

    public Car(Car other) {

        this.brand = other.brand;
        this.color = other.color;
        this.speed = other.speed;
    }
}
```

Usage:

```java
Car car1 =
    new Car("BMW", "Black", 250);

Car car2 =
    new Car(car1);
```

This is effectively implementing the same copying idea.

---

# 19. Deep Copy with Copy Constructor

Suppose:

```java
class Engine {

    int horsepower;

    Engine(int horsepower) {
        this.horsepower = horsepower;
    }

    Engine(Engine other) {
        this.horsepower = other.horsepower;
    }
}
```

Then:

```java
class Car {

    String name;
    Engine engine;

    Car(Car other) {

        this.name = other.name;

        this.engine =
            new Engine(other.engine);
    }
}
```

Now:

```java
Car car2 = new Car(car1);
```

creates:

```text
car1 → Engine A

car2 → Engine B
```

Deep copy achieved.

---

# 20. Prototype vs Copy Constructor

Both can copy objects.

### Copy Constructor

```java
Car copy = new Car(original);
```

Simple and explicit.

But the client knows:

```text
"I am creating a Car."
```

---

### Prototype

```java
Prototype copy =
    original.clone();
```

The client can work through an abstraction.

```java
Prototype prototype = ...;

Prototype copy =
    prototype.clone();
```

This becomes useful when the exact concrete type isn't known or shouldn't matter to the client.

---

# 21. Prototype and SRP

Prototype **can affect SRP**, but it does not inherently violate it.

Example:

```java
class Car {

    void drive() {
    }

    Car clone() {
        // copying logic
    }
}
```

The class now handles:

```text
Car behavior
+
Car copying
```

You could argue these are two responsibilities.

However, unlike a badly designed Singleton, copying itself is often closely related to the object's construction/lifecycle.

So this is usually less concerning.

If copying becomes very complicated:

```text
clone()
 ├── copy engine
 ├── copy wheels
 ├── copy owner
 ├── copy configuration
 ├── copy metadata
 └── copy permissions
```

then extracting copying logic into a dedicated:

```text
Factory
Mapper
Copy Service
Builder
```

may improve the design.

---

# 22. Prototype and OCP

Prototype can help with the **Open/Closed Principle**.

Suppose the client does:

```java
Prototype prototype = ...;

Prototype copy =
    prototype.clone();
```

Now we introduce:

```java
Car
Bike
Truck
Plane
```

Each implements:

```java
clone()
```

The client code doesn't necessarily need to change.

```text
Prototype
   ↑
   ├── Car
   ├── Bike
   ├── Truck
   └── Plane
```

This allows new prototype types to be added without rewriting generic cloning code.

---

# 23. Prototype and Coupling

Without Prototype:

```java
new ComplexEnemy(
    health,
    damage,
    weapon,
    armor,
    ability,
    ...
);
```

Client needs to understand the object's construction.

With Prototype:

```java
enemyPrototype.clone();
```

The client only needs an existing configured prototype.

Therefore Prototype can reduce coupling between:

```text
Client
```

and:

```text
Complex object creation logic
```

---

# 24. Advantages

### 1. Avoid expensive initialization

Instead of:

```text
Create
↓
Load
↓
Configure
↓
Initialize
```

clone an existing configured object.

---

### 2. Reduce repetitive creation code

Instead of:

```java
new Enemy(...20 parameters...);
```

use:

```java
prototype.clone();
```

---

### 3. Create variations easily

```java
Enemy normal =
    prototype.clone();

Enemy strong =
    prototype.clone();

strong.health = 500;
```

---

### 4. Hide complex creation logic

The client doesn't need to know how the object was originally constructed.

---

### 5. Runtime object creation

Prototypes can be selected dynamically:

```java
prototypeRegistry
    .get(enemyType)
    .clone();
```

---

# 25. Disadvantages

## 1. Deep copying can become complicated

Suppose:

```text
Object
 ↓
Object
 ↓
List
 ↓
Object
 ↓
Map
 ↓
Object
```

You need to carefully decide what should be copied and what should be shared.

---

## 2. Circular References

Imagine:

```text
Person A → Person B
   ↑          ↓
   └──────────┘
```

Deep copying such graphs can become complicated.

---

## 3. Shared Mutable State Bugs

If you accidentally shallow-copy something:

```text
Prototype
    \
     Shared List
    /
Clone
```

changing the clone can unexpectedly change the original.

---

## 4. Clone Logic Maintenance

When new fields are added:

```java
String address;
```

you may also need to update:

```java
clone()
```

Otherwise copies may be incomplete.

---

# 26. How to Make Prototype Less Brittle

### 1. Clearly Decide Shallow vs Deep Copy

Don't blindly copy references.

Ask:

```text
Is this nested object mutable?

YES → probably copy it

NO → sharing may be safe
```

---

### 2. Prefer Immutability

Immutable objects are safe to share.

For example:

```java
String
Integer
LocalDate
```

and your own immutable value objects.

This greatly simplifies copying.

---

### 3. Prefer Explicit Copy Methods

Instead of mysterious cloning:

```java
clone()
```

sometimes prefer:

```java
copy()
```

or:

```java
new Car(existingCar)
```

The copying behavior becomes easier to understand.

---

### 4. Keep Object Graphs Simple

If cloning requires copying 50 nested objects, that's often a sign that the model itself may be too complicated.

---

### 5. Test Independence

For deep copies, test:

```java
Car copy = original.copy();

copy.engine.setPower(500);

assert original.engine.getPower() != 500;
```

This catches accidental shallow copies.

---

# 27. When Should You Use Prototype?

Use Prototype when:

```text
Object creation is expensive
          OR
Object configuration is complicated
          OR
Many similar objects are required
          OR
Exact object type is determined at runtime
```

Good examples:

```text
Game characters
Document templates
UI components
Configured simulations
Graphic objects/shapes
Preconfigured objects
```

---

# 28. When Should You NOT Use Prototype?

Avoid it when:

```text
Object creation is already simple
```

For example:

```java
User user =
    new User("John", 20);
```

Creating a Prototype for this:

```java
user.clone();
```

probably adds unnecessary complexity.

Also reconsider Prototype when deep copying the object's graph is extremely complicated.

---

# 29. Prototype vs Factory

This is a common interview comparison.

### Factory

Factory **constructs** the object.

```java
Enemy enemy =
    EnemyFactory.create("zombie");
```

Conceptually:

```text
Factory
  ↓
new Object
```

---

### Prototype

Prototype **copies** an existing object.

```java
Enemy enemy =
    zombiePrototype.clone();
```

Conceptually:

```text
Existing Object
      ↓
    clone()
      ↓
 New Object
```

Remember:

```text
Factory   → CREATE

Prototype → COPY
```

---

# 30. Prototype vs Builder

### Builder

Used when object construction has many steps/options.

```java
Computer computer =
    new ComputerBuilder()
        .cpu("i9")
        .ram(32)
        .storage(1000)
        .build();
```

Think:

```text
Builder
→ Construct complex object step-by-step
```

---

### Prototype

```java
Computer computer2 =
    computer.clone();
```

Think:

```text
Prototype
→ Already have configured object
→ Copy it
```

---

# 31. Prototype vs Singleton

```text
Singleton

"I want ONE instance."

        Object
       /  |  \
      ↓   ↓   ↓
    User User User


Prototype

"I want NEW instances based on
an existing instance."

      Prototype
       /   |   \
      ↓    ↓    ↓
    Copy Copy Copy
```

---

# 32. Interview Questions

## What is Prototype?

> Prototype is a creational design pattern that creates new objects by copying an existing object instead of constructing them from scratch.

---

## Why use Prototype?

> It is useful when creating an object is expensive or complicated, or when many objects with similar configurations need to be created.

---

## What is the biggest challenge?

> Correctly handling shallow and deep copying, especially when the object contains mutable nested objects.

---

## Shallow vs Deep Copy?

> Shallow copy creates a new outer object but shares references to nested objects, while deep copy creates independent copies of mutable nested objects as well.

---

## Does Prototype require Java `Cloneable`?

> No. Prototype is a design pattern, not a Java feature. It can be implemented using `Cloneable`, copy constructors, factory methods, or custom `copy()` methods.

---

# 33. 30-Second Revision

```text
PROTOTYPE = Create object by copying
            an existing object.

Problem:
→ Object expensive to create
→ Complex initialization
→ Many similar objects needed

Solution:
→ Create configured prototype
→ clone/copy it
→ Modify copy if necessary

Main concept:
→ clone()

Most important interview topic:
→ SHALLOW vs DEEP COPY

Shallow:
→ new outer object
→ nested references shared

Deep:
→ new outer object
→ mutable nested objects copied

Java:
→ Cloneable exists
→ Object.clone() is shallow
→ Copy constructors / copy() often clearer

Advantages:
→ Faster/easier object creation
→ Less repeated configuration
→ Hides complex construction
→ Easy variations

Problems:
→ Deep copy complexity
→ Shared mutable state
→ Circular references
→ Clone logic maintenance

Less brittle:
→ Prefer immutable nested objects
→ Explicit copy semantics
→ Carefully choose shallow/deep copy
→ Test that copies are independent
```

# 34. One-Line Mental Model

```text
Singleton
→ "Give me THE object."

Factory
→ "Create an object for me."

Builder
→ "Build this object step-by-step."

Prototype
→ "Give me a COPY of that object."
```

### One-line interview answer

> **Prototype is a creational design pattern used to create new objects by cloning an existing configured object, which is especially useful when construction is expensive or many similar objects are required; the key implementation concern is correctly handling shallow versus deep copies.**
