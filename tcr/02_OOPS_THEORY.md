# TCR 02 — OOPs Theory

> You have already met most of this in [../coding/01_OOPS_CODING.md](../coding/01_OOPS_CODING.md). This file is the **conceptual / language-agnostic** layer that TCR tests, with ready-made reasoning sentences.

---

## 1. The four pillars — with the distinctions examiners probe

### Encapsulation
Bundling data and the methods that operate on it into one unit, and restricting direct access to the data.
**Achieved by:** private fields + public accessor/mutator methods.
**Why it matters:** the class can enforce invariants (a balance can never go negative), and the internal representation can change without breaking callers.

### Abstraction
Exposing only the essential features of an object and hiding the implementation details.
**Achieved by:** abstract classes and interfaces.
**Why it matters:** callers depend on *what* an object does, not *how*, so implementations can be swapped freely.

> **The distinction they love to test:** *"Encapsulation hides **data**; abstraction hides **complexity**. Encapsulation is an implementation mechanism (access modifiers), abstraction is a design principle (interfaces). Encapsulation is one of the ways abstraction is achieved."*

### Inheritance
A class acquires the properties and behaviour of another class — an **is-a** relationship.
**Benefits:** code reuse, and a type hierarchy that enables polymorphism.
**Costs:** tight coupling between base and derived; a change in the base can break every subclass (the "fragile base class" problem). **Prefer composition where the relationship is really "has-a".**

### Polymorphism
One interface, many implementations. "Many forms."
- **Compile-time (static):** method overloading, operator overloading, templates/generics. Resolved by the compiler from the argument types.
- **Run-time (dynamic):** method overriding through a base-class pointer or reference. Resolved at execution via the vtable.

---

## 2. Overloading vs Overriding — the most-asked comparison

| | Overloading | Overriding |
|---|---|---|
| Definition | same name, **different parameter list** | derived class redefines a base method with the **identical signature** |
| Scope | same class (usually) | across base and derived classes |
| Binding | compile time (early/static) | run time (late/dynamic) |
| Return type | may differ, but cannot be the *only* difference | must match (or be covariant) |
| Needs inheritance? | No | **Yes** |
| C++ keyword | none | `virtual` on the base method |
| Also called | ad-hoc polymorphism | subtype polymorphism |

**Reasoning sentence:** *"Overloading is resolved at compile time from the argument types, so it is static polymorphism. Overriding is resolved at run time through the object's actual type via the virtual table, which is why a base-class pointer can invoke derived behaviour."*

**The gotcha — name hiding:** in C++, declaring *any* method named `f` in a derived class hides **all** base overloads of `f`. Restore them with `using Base::f;`.

---

## 3. Abstract class vs Interface

| | Abstract class | Interface |
|---|---|---|
| Contains | abstract **and** concrete methods | (classically) only method signatures |
| Fields | yes, including state | constants only |
| Multiple inheritance | usually not (Java: no) | yes — a class may implement many |
| Constructor | yes | no |
| Access modifiers | any | implicitly public |
| Represents | an **is-a** relationship with shared implementation | a **capability / contract** (can-do) |
| Use when | subclasses share code and state | unrelated classes must share a contract |

In **C++** there is no `interface` keyword: an interface is simply a class whose methods are all pure virtual (`= 0`) and which holds no data. In **Java**, `interface` is a keyword; since Java 8 interfaces may also carry `default` and `static` methods, which narrows the gap — but a Java interface still cannot hold instance state.

**Reasoning sentence:** *"Use an abstract class when subclasses genuinely share implementation and state; use an interface when otherwise unrelated classes must guarantee the same capability. Interfaces also sidestep the multiple-inheritance restriction."*

---

## 4. Static vs dynamic binding, and how virtual works

- **Static (early) binding:** the compiler decides which function runs, from the declared type. Applies to non-virtual functions, overloaded functions, and all `static` and `private` methods.
- **Dynamic (late) binding:** the decision is deferred to run time, based on the object's actual type. Applies to virtual functions accessed through a pointer or reference.

**The vtable mechanism:** every class with virtual functions gets a **virtual table** of function pointers. Every object of such a class stores a hidden `vptr` pointing to its class's vtable. A call through a base pointer follows the `vptr` and dispatches to the correct override. The cost is one extra indirection and a pointer per object.

```
Base* p = new Derived();
p->virtualMethod();     // Derived's version  (dynamic binding)
p->normalMethod();      // Base's version     (static binding, chosen from p's type)
```

**Facts worth memorising:**
- A **constructor cannot be virtual** — the vptr is only set up *during* construction, so there is nothing to dispatch through yet.
- A **destructor should be virtual** in any class intended as a base — otherwise `delete basePtr` skips the derived destructor and leaks.
- Calling a virtual function **from inside a constructor** dispatches to the *base* version, because the derived part does not exist yet.
- `static` and `friend` functions **cannot be virtual** — they have no `this` pointer.

---

## 5. Object slicing

```cpp
Derived d;
Base b = d;        // SLICING: only the Base part is copied; derived data is discarded
Base& r = d;       // fine — no copy, polymorphism preserved
Base* p = &d;      // fine
```
**Reasoning:** *"Assigning a derived object to a base object by value copies only the base sub-object, discarding derived members and any polymorphic behaviour. Using a reference or pointer avoids the copy and preserves dynamic dispatch."*

---

## 6. Relationships between classes

| Relationship | Meaning | Lifetime | Example |
|---|---|---|---|
| **Inheritance** | is-a | — | Car **is a** Vehicle |
| **Composition** | has-a, **strong** | child dies with the parent | House has Rooms; Human has a Heart |
| **Aggregation** | has-a, **weak** | child outlives the parent | Department has Professors |
| **Association** | uses-a | independent | Doctor and Patient |
| **Dependency** | temporarily uses | transient | a method taking a Logger parameter |

**Composition over inheritance:** inheritance is compile-time and permanent; composition is run-time and swappable. Composition avoids the fragile-base-class problem and deep hierarchies. Modern design guidance is to **inherit only for genuine is-a substitutability, and compose everything else.**

---

## 7. SOLID principles — expect at least one question

| Letter | Principle | One-line meaning | Violation smell |
|---|---|---|---|
| **S** | Single Responsibility | a class should have exactly one reason to change | a `User` class that also formats reports and sends email |
| **O** | Open/Closed | open for extension, closed for modification | a `switch (shapeType)` that you must edit for every new shape |
| **L** | Liskov Substitution | a subclass must be usable anywhere its base is, without surprising the caller | `Square extends Rectangle` and `setWidth` also changes the height |
| **I** | Interface Segregation | many small specific interfaces beat one fat one | an interface forcing `Robot` to implement `eat()` |
| **D** | Dependency Inversion | depend on abstractions, not concretions | a service that constructs `new MySQLDatabase()` internally instead of receiving a `Database` interface |

**Reusable reasoning sentences:**
- *"Adding a new shape by creating a subclass rather than editing a switch statement satisfies the Open/Closed Principle: behaviour is extended without modifying tested code."*
- *"Injecting an interface instead of instantiating a concrete class inverts the dependency, so the high-level module no longer depends on a low-level implementation and becomes testable with a mock."*
- *"The classic Liskov violation is Square inheriting from Rectangle: code that relies on width and height varying independently breaks, so the subtype is not truly substitutable."*

---

## 8. Design patterns worth recognising

### Creational
- **Singleton** — exactly one instance, with a global access point. Private constructor, static instance. Used for configuration, logging, connection pools. **Criticism:** it is effectively global state and makes unit testing harder; thread safety needs care (double-checked locking, or a static local in C++11+).
- **Factory Method** — a method decides which concrete class to instantiate, so client code depends only on the interface.
- **Abstract Factory** — creates families of related objects.
- **Builder** — constructs a complex object step by step; avoids telescoping constructors.
- **Prototype** — creates new objects by cloning an existing one.

### Structural
- **Adapter** — converts one interface into another the client expects (a wrapper around a legacy or third-party API).
- **Decorator** — adds responsibilities to an object dynamically by wrapping it (Java's `BufferedReader(new FileReader(...))`).
- **Facade** — one simplified interface over a complicated subsystem.
- **Proxy** — a stand-in controlling access (lazy loading, access control, caching, remote calls).
- **Composite** — treat individual objects and compositions uniformly (a file-system tree).

### Behavioural
- **Observer** — one-to-many notification; when the subject changes, all subscribers are told. This is the model behind event listeners, pub/sub, and React state updates.
- **Strategy** — encapsulate interchangeable algorithms behind a common interface and select one at run time (different sorting or payment strategies).
- **Template Method** — a base class fixes the skeleton of an algorithm and lets subclasses override individual steps.
- **Iterator** — sequential access to a collection without exposing its internals.
- **Command** — wrap a request as an object; enables undo, queuing, and logging.

**MVC (Model-View-Controller)** — an architectural pattern, not a GoF one. **Model** = data and business rules; **View** = presentation; **Controller** = handles input and coordinates the two. The benefit is separation of concerns: the UI can change without touching business logic. Express, Spring MVC, Django (as MVT) and Rails all follow it.

```cpp
// Singleton, the C++11 thread-safe way
class Logger {
    Logger() {}
public:
    Logger(const Logger&) = delete;                 // no copying
    Logger& operator=(const Logger&) = delete;
    static Logger& getInstance() {
        static Logger instance;                      // initialised once, thread-safe in C++11+
        return instance;
    }
    void log(const string& msg) { cout << msg << "\n"; }
};
```

---

## 9. Cohesion and coupling

- **Cohesion** — how focused a single module is. **High cohesion is good**: everything in the class serves one purpose.
- **Coupling** — how dependent modules are on each other. **Low coupling is good**: a change in one module does not ripple through the rest.

**The rule:** *high cohesion, low coupling*. Interfaces and dependency injection lower coupling; the Single Responsibility Principle raises cohesion.

---

## 10. Constructors and destructors — theory points

- A **constructor** initialises an object; it has the class's name, no return type, and can be overloaded.
- If you declare no constructor, the compiler supplies a default one. **Declare any constructor and the implicit default disappears.**
- Construction order: base class -> members (in declaration order) -> the constructor body. Destruction is the exact reverse.
- A **copy constructor** takes `const ClassName&`. Taking it by value would require a copy to make the copy — infinite recursion, which is why the compiler rejects it.
- **Shallow copy** duplicates pointer values (two objects sharing one buffer -> double free). **Deep copy** allocates new memory and copies the contents.
- **Rule of Three:** needing a destructor implies needing a copy constructor and a copy assignment operator. **Rule of Five** adds the move constructor and move assignment. **Rule of Zero:** prefer types that manage their own resources (`std::string`, `std::vector`, smart pointers) so you need none of them.

---

## 11. Access modifiers

| Modifier | Same class | Derived class | Elsewhere |
|---|---|---|---|
| `private` | Yes | No | No |
| `protected` | Yes | Yes | No |
| `public` | Yes | Yes | Yes |

(Java adds a package-private default: visible within the same package.)

**`friend` in C++** grants a specific external function or class access to private members. It deliberately breaks encapsulation and is justified only when the alternative — exposing internals publicly — would be worse, for example when overloading `operator<<`, whose left operand is an `ostream` and therefore cannot be a member.

---

## 12. Memory: stack vs heap

| | Stack | Heap |
|---|---|---|
| Allocation | automatic, on scope entry | manual (`new`/`malloc`) or by a smart pointer |
| Deallocation | automatic at scope exit | manual (`delete`/`free`) or by RAII |
| Speed | very fast (pointer bump) | slower (allocator bookkeeping) |
| Size | limited (a few MB) | large |
| Fragmentation | none | possible |
| Typical failure | stack overflow from deep recursion | memory leak, dangling pointer, double free |

**RAII (Resource Acquisition Is Initialisation)** is the C++ answer: bind a resource's lifetime to an object's lifetime so the destructor releases it automatically, even when an exception unwinds the stack. `std::unique_ptr`, `std::shared_ptr` and `std::lock_guard` are RAII types.
- `unique_ptr` — sole ownership, cannot be copied, only moved.
- `shared_ptr` — reference-counted shared ownership; freed when the last one goes.
- `weak_ptr` — a non-owning observer, used to break `shared_ptr` reference cycles.

**Garbage collection (Java, Python, C#) vs manual/RAII (C++):** GC removes whole classes of leak and dangling-pointer bugs at the cost of unpredictable pause times and higher memory overhead. RAII gives deterministic, immediate release with no runtime collector, but the programmer must design ownership correctly.

---

## 13. Exception handling — theory

- `try` marks the guarded region, `catch` handles a thrown type, `throw` raises, and `finally` (Java/C#) always runs. **C++ has no `finally`** — RAII destructors serve that role.
- Catch blocks are tested **in order**, so catch derived types before base types; otherwise the base handler swallows everything.
- **Checked vs unchecked (Java):** checked exceptions must be declared or handled at compile time (`IOException`); unchecked ones (`RuntimeException`, `NullPointerException`) need not be. C++ makes no such distinction.
- **Never throw from a destructor** — if it fires during stack unwinding from another exception, the program terminates.
- Prefer `catch (const std::exception& e)` by **reference**: catching by value slices the exception object and loses the derived type.

---

## 14. Multiple inheritance and the diamond problem

```
    A
   / \
  B   C
   \ /
    D
```
Without intervention, `D` contains **two** copies of `A`, so `d.x` is ambiguous. C++ solves this with **virtual inheritance** (`class B : virtual public A`), which makes `B` and `C` share one `A` sub-object. Java avoids the problem entirely by forbidding multiple class inheritance while allowing multiple interfaces — since interfaces classically carry no state, there is nothing to duplicate.

**Reasoning sentence:** *"Multiple inheritance risks ambiguity when a class inherits the same base along two paths. C++ resolves it with virtual inheritance so only one shared base sub-object exists; Java sidesteps it by permitting only single class inheritance plus multiple stateless interfaces."*

---

## 15. Generic terms you should be able to define in one line

| Term | Definition |
|---|---|
| **Class** | a blueprint defining state and behaviour |
| **Object** | an instance of a class, occupying memory |
| **Method** | a function defined inside a class |
| **Constructor** | a special method that initialises a new object |
| **Instance variable** | per-object state |
| **Class (static) variable** | one copy shared by all objects |
| **this** | a reference to the current object |
| **super / base** | a reference to the parent class part |
| **Message passing** | objects interacting by calling each other's methods |
| **Data hiding** | restricting direct access to internal state |
| **Late binding** | resolving which method to call at run time |
| **Covariant return type** | an override may return a type derived from the base method's return type |
| **Immutable object** | state cannot change after construction (`String` in Java, `const` objects in C++) |
| **Interface segregation** | clients should not be forced to depend on methods they never use |

---

## 16. Procedural vs Object-Oriented programming

| | Procedural (C, Pascal) | Object-Oriented (C++, Java) |
|---|---|---|
| Unit of organisation | functions | objects |
| Data and behaviour | separate | bundled together |
| Data access | typically global/shared | encapsulated |
| Approach | top-down | bottom-up |
| Reuse | functions | inheritance, composition, polymorphism |
| Scales to large systems | poorly | well |
| Modelling real-world entities | awkward | natural |

**Reasoning sentence:** *"Procedural code separates data from the functions acting on it, so any function can corrupt shared state. OOP binds data to its behaviour and restricts access, which localises change and makes large systems maintainable."*

---

## 17. Self test (cover the answers)

1. Encapsulation vs abstraction? -> *Encapsulation hides data via access modifiers (implementation); abstraction hides complexity via interfaces (design). Encapsulation is a means of achieving abstraction.*
2. Why can a constructor not be virtual? -> *The vptr is established during construction, so no dispatch mechanism exists yet.*
3. Why must a base destructor be virtual? -> *So `delete basePtr` runs the derived destructor too; otherwise the derived part leaks.*
4. Compile-time vs run-time polymorphism? -> *Overloading/templates resolved by the compiler from argument types, versus overriding resolved at run time through the vtable.*
5. When would you choose an interface over an abstract class? -> *When unrelated classes must share a contract without shared implementation, or when a class needs several such contracts.*
6. What is the Liskov Substitution Principle, with an example violation? -> *Subtypes must be substitutable for their base types; `Square extends Rectangle` breaks callers that set width and height independently.*
7. What problem does the Singleton pattern solve, and what is its main criticism? -> *It guarantees a single instance with global access; the criticism is that it is disguised global state, hindering testing and concurrency.*
8. What is object slicing? -> *Copying a derived object into a base object by value discards the derived members and polymorphic behaviour.*
9. High cohesion, low coupling — why? -> *Cohesive modules are easy to understand and change; loose coupling stops a change in one module rippling through the system.*
10. Composition over inheritance — why? -> *Composition is run-time flexible and avoids the fragile-base-class problem, whereas inheritance is permanent and tightly couples subclasses to base internals.*

Next: [03_FSD.md](03_FSD.md)
