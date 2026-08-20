# 01 — OOPs in C++ (Coding Section)

> This file does double duty: it's your **coding** prep for the OOPs question, and it's half your **TCR OOPs** revision. Pair it with [../tcr/02_OOPS_THEORY.md](../tcr/02_OOPS_THEORY.md).

---

## 1. The anatomy of a C++ class

```cpp
class Student {
private:                       // accessible only inside the class
    string name;
    int roll;
    static int count;          // shared by ALL objects, not per-object

public:
    // 1. Default constructor
    Student() : name("Unknown"), roll(0) { count++; }

    // 2. Parameterised constructor (member initialiser list = preferred)
    Student(string n, int r) : name(n), roll(r) { count++; }

    // 3. Copy constructor
    Student(const Student& other) : name(other.name), roll(other.roll) { count++; }

    // 4. Destructor
    ~Student() { count--; }

    // Getters / setters = encapsulation
    string getName() const { return name; }     // const => promises not to modify
    void setName(string n) { name = n; }

    // 5. Static member function — can only touch static members
    static int getCount() { return count; }

    void display() const {
        cout << roll << " " << name << "\n";
    }
};

int Student::count = 0;        // static members MUST be defined outside the class
```

**Points examiners look for:**
- `private` data + `public` methods = **encapsulation**.
- **Member initialiser list** `: name(n)` initialises; assignment inside `{}` constructs-then-assigns (slower, and *required* for `const`/reference members).
- `const` after a method = it does not modify the object.
- WARNING: **A class definition ends with a semicolon**: `};` — the single most common compile error.

---

## 2. Constructors & destructors — order of execution

```cpp
class A { public: A(){cout<<"A ctor ";} ~A(){cout<<"A dtor ";} };
class B : public A { public: B(){cout<<"B ctor ";} ~B(){cout<<"B dtor ";} };

int main() { B b; }
// Output: A ctor B ctor B dtor A dtor
```

**Rule: construction goes base -> derived; destruction goes derived -> base (exact reverse).**
Members are constructed before the constructor body runs, and in **declaration order** — not the order you write them in the initialiser list.

### Types of constructors
| Type | Signature | When called |
|---|---|---|
| Default | `A()` | `A a;` |
| Parameterised | `A(int x)` | `A a(5);` |
| Copy | `A(const A& o)` | `A b = a;` / pass-by-value / return-by-value |
| Move (C++11) | `A(A&& o)` | from a temporary |

WARNING: If you define **any** constructor, the compiler stops generating the default one. `A a;` then fails to compile.

---

## 3. Shallow vs deep copy — the classic interview question

```cpp
class Buffer {
    int* data;
    int size;
public:
    Buffer(int s) : size(s) { data = new int[s]; }

    // SHALLOW (what the compiler gives you): copies the POINTER.
    // Both objects point to the same memory -> double free -> crash.

    // DEEP copy constructor — allocate new memory and copy contents
    Buffer(const Buffer& o) : size(o.size) {
        data = new int[size];
        for (int i = 0; i < size; i++) data[i] = o.data[i];
    }

    // Copy assignment operator
    Buffer& operator=(const Buffer& o) {
        if (this == &o) return *this;      // self-assignment guard
        delete[] data;                      // free the old
        size = o.size;
        data = new int[size];
        for (int i = 0; i < size; i++) data[i] = o.data[i];
        return *this;                       // enables a = b = c
    }

    ~Buffer() { delete[] data; }
};
```

> **Rule of Three:** if your class needs a **destructor**, it almost certainly also needs a **copy constructor** and a **copy assignment operator**. (Rule of Five in C++11 adds move constructor + move assignment.)

---

## 4. The four pillars, in code

### Encapsulation — bundle data with the methods that act on it, hide the data
```cpp
class BankAccount {
private:
    double balance;                       // nobody can set this directly
public:
    BankAccount(double b) : balance(b) {}
    void deposit(double amt) {
        if (amt > 0) balance += amt;      // the class ENFORCES the invariant
    }
    bool withdraw(double amt) {
        if (amt > 0 && amt <= balance) { balance -= amt; return true; }
        return false;
    }
    double getBalance() const { return balance; }
};
```
**The point:** balance can never go negative, because the only paths to it validate first.

### Abstraction — expose *what*, hide *how*
```cpp
class Stack {                             // user sees push/pop
public:                                   // user never sees the array/list inside
    virtual void push(int) = 0;
    virtual int  pop()     = 0;
};
```
> **Encapsulation vs Abstraction (a favourite TCR question):** encapsulation is *data hiding* — an implementation mechanism (access specifiers). Abstraction is *complexity hiding* — a design idea (show only the essential interface). Encapsulation is one of the ways you achieve abstraction.

### Inheritance — reuse and specialise
```cpp
class Vehicle {
protected:                                // derived classes CAN see protected
    string brand;
public:
    Vehicle(string b) : brand(b) {}
    void horn() { cout << "Beep!\n"; }
};

class Car : public Vehicle {
    int wheels;
public:
    Car(string b, int w) : Vehicle(b), wheels(w) {}   // MUST init the base
    void show() { cout << brand << " " << wheels << "\n"; }
};
```

| Access in base | `public` inheritance | `protected` inheritance | `private` inheritance |
|---|---|---|---|
| public | public | protected | private |
| protected | protected | protected | private |
| private | inaccessible | inaccessible | inaccessible |

**Types of inheritance:** single, multiple (`class C : public A, public B`), multilevel (A -> B -> C), hierarchical (one base, many derived), hybrid.

### Polymorphism — one interface, many behaviours
```cpp
class Shape {
public:
    virtual double area() const = 0;              // PURE virtual => abstract class
    virtual void name() const { cout << "Shape"; }
    virtual ~Shape() {}                           // ALWAYS virtual in a base class
};

class Circle : public Shape {
    double r;
public:
    Circle(double r) : r(r) {}
    double area() const override { return 3.14159 * r * r; }
    void name() const override { cout << "Circle"; }
};

class Rectangle : public Shape {
    double l, b;
public:
    Rectangle(double l, double b) : l(l), b(b) {}
    double area() const override { return l * b; }
    void name() const override { cout << "Rectangle"; }
};

int main() {
    vector<Shape*> shapes = { new Circle(2), new Rectangle(3, 4) };
    for (Shape* s : shapes) {
        s->name();
        cout << " area = " << s->area() << "\n";   // correct override chosen at RUNTIME
    }
    for (Shape* s : shapes) delete s;
}
```

**Why `virtual ~Shape()` matters:** deleting a `Circle` through a `Shape*` without a virtual destructor only runs `~Shape()` — the `Circle` part leaks. Very common TCR question.

---

## 5. Compile-time vs run-time polymorphism

| | Compile-time (static) | Run-time (dynamic) |
|---|---|---|
| Achieved by | function **overloading**, operator overloading, templates | function **overriding** with `virtual` |
| Bound at | compile time (early binding) | run time (late binding, via vtable) |
| Speed | faster | slight indirection cost |
| Needs | same name, **different parameters** | same signature, base pointer/reference |

```cpp
// OVERLOADING — same name, different parameter list (compile-time)
int add(int a, int b)             { return a + b; }
double add(double a, double b)    { return a + b; }
int add(int a, int b, int c)      { return a + b + c; }
// NOTE: differing only in RETURN TYPE is NOT overloading — it is a compile error.

// OVERRIDING — same signature, redefined in derived (run-time)
class Base    { public: virtual void f() { cout << "Base"; } };
class Derived : public Base { public: void f() override { cout << "Derived"; } };

Base* p = new Derived();
p->f();          // "Derived"  — because f is virtual
                 // "Base"     — if you remove the virtual keyword
```

**How `virtual` works (vtable):** each class with virtual functions gets a table of function pointers. Each object stores a hidden `vptr` to its class's vtable. `p->f()` looks up the address at run time — hence *dynamic dispatch*.

---

## 6. Abstract class vs interface

```cpp
// Abstract class: >= 1 pure virtual. Cannot be instantiated. Can have data + implemented methods.
class Animal {
protected:
    string name;
public:
    Animal(string n) : name(n) {}
    virtual void sound() = 0;                     // pure virtual
    void sleep() { cout << name << " sleeps\n"; } // concrete, shared
    virtual ~Animal() {}
};

// "Interface" in C++ = a class with ALL pure virtual functions and no data
class Drawable {
public:
    virtual void draw() = 0;
    virtual ~Drawable() {}
};

class Dog : public Animal, public Drawable {     // multiple inheritance
public:
    Dog(string n) : Animal(n) {}
    void sound() override { cout << "Woof\n"; }
    void draw()  override { cout << "[dog]\n"; }
};
```
WARNING: `Animal a("x");` -> **compile error**: cannot instantiate an abstract class. You may still have `Animal*` and `Animal&`.

---

## 7. Operator overloading

```cpp
class Complex {
    double re, im;
public:
    Complex(double r = 0, double i = 0) : re(r), im(i) {}

    Complex operator+(const Complex& o) const {
        return Complex(re + o.re, im + o.im);
    }
    Complex operator*(const Complex& o) const {
        return Complex(re*o.re - im*o.im, re*o.im + im*o.re);
    }
    bool operator==(const Complex& o) const {
        return re == o.re && im == o.im;
    }
    Complex& operator++() { ++re; return *this; }                  // pre-increment
    Complex operator++(int) { Complex t = *this; ++re; return t; } // post (dummy int param)

    // friend: not a member, but allowed to see privates. Needed when the
    // left operand is not your class (here it is ostream).
    friend ostream& operator<<(ostream& os, const Complex& c) {
        os << c.re << (c.im >= 0 ? "+" : "") << c.im << "i";
        return os;
    }
};

int main() {
    Complex a(1, 2), b(3, -1);
    cout << a + b << "\n";     // 4+1i
}
```
**Cannot be overloaded:** `.`  `.*`  `::`  `?:`  `sizeof`  `typeid`.

---

## 8. `this`, `static`, `const`, `friend` — quick reference

```cpp
class A {
    int x;
    static int objects;             // one copy for the whole class
public:
    A(int x) { this->x = x; }       // this-> disambiguates param vs member
    A& setX(int x) { this->x = x; return *this; }   // return *this => chaining a.setX(1).setX(2)

    int getX() const { return x; }  // const method: cannot modify members
    static int getObjects() { return objects; }     // no this inside a static method

    friend void peek(const A& a);   // free function granted private access
};
void peek(const A& a) { cout << a.x; }
```

| Keyword | Meaning |
|---|---|
| `this` | pointer to the current object |
| `static` member | shared across all objects; exists without any object |
| `static` method | callable as `A::method()`; cannot access non-static members |
| `const` method | promises not to modify the object |
| `friend` | grants a non-member access to private/protected (breaks encapsulation — use sparingly) |
| `virtual` | enables run-time dispatch |
| `override` | compiler-checks that you really are overriding (use it — catches typos) |
| `final` | forbids further overriding/inheriting |

---

## 9. The diamond problem & virtual inheritance

```cpp
class A { public: int x = 1; };
class B : virtual public A {};      // virtual => share one A
class C : virtual public A {};
class D : public B, public C {};

D d;
d.x = 5;    // unambiguous. WITHOUT virtual, D has TWO copies of A
            // and d.x is a compile error ("ambiguous"); you would need d.B::x
```
**TCR-ready phrasing:** *"Without virtual inheritance, D inherits two separate A sub-objects, making member access ambiguous. Virtual inheritance ensures a single shared A instance."*

---

## 10. Templates (generic programming)

```cpp
template <typename T>
T maxOf(T a, T b) { return a > b ? a : b; }

template <typename T>
class Box {
    T item;
public:
    Box(T i) : item(i) {}
    T get() const { return item; }
};

int main() {
    cout << maxOf(3, 7) << " " << maxOf(2.5, 1.5) << "\n";
    Box<string> b("hi");
    cout << b.get();
}
```
Templates are **compile-time polymorphism** — the compiler generates a separate version per type.

---

## 11. Exception handling

```cpp
class InsufficientFunds : public exception {
public:
    const char* what() const noexcept override { return "Insufficient funds"; }
};

double withdraw(double bal, double amt) {
    if (amt > bal) throw InsufficientFunds();
    if (amt <= 0)  throw invalid_argument("Amount must be positive");
    return bal - amt;
}

int main() {
    try {
        withdraw(100, 500);
    } catch (const InsufficientFunds& e) {
        cout << e.what() << "\n";
    } catch (const exception& e) {          // more general LAST
        cout << "Error: " << e.what() << "\n";
    } catch (...) {                          // catch-all
        cout << "Unknown error\n";
    }
}
```
**Order matters:** the first matching `catch` wins, so catch derived types before base types.

---

# THE CODING QUESTIONS

The OOPs coding question is almost always **"design a class/system with these operations."** They test structure, not algorithms. Below are the archetypes.

---

## Q1. Bank Account System  (most likely)

**Concepts covered:** encapsulation, validation, static members, inheritance, polymorphism.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Account {
protected:
    string accNo, holder;
    double balance;
    static int totalAccounts;

public:
    Account(string a, string h, double b) : accNo(a), holder(h), balance(b) {
        totalAccounts++;
    }
    virtual ~Account() { totalAccounts--; }

    void deposit(double amt) {
        if (amt <= 0) { cout << "Invalid amount\n"; return; }
        balance += amt;
        cout << "Deposited " << amt << ", balance = " << balance << "\n";
    }

    virtual bool withdraw(double amt) {           // virtual: subclasses change the rule
        if (amt <= 0 || amt > balance) { cout << "Insufficient funds\n"; return false; }
        balance -= amt;
        cout << "Withdrew " << amt << ", balance = " << balance << "\n";
        return true;
    }

    virtual void display() const {
        cout << accNo << " | " << holder << " | " << balance << "\n";
    }

    double getBalance() const { return balance; }
    static int getTotalAccounts() { return totalAccounts; }
};
int Account::totalAccounts = 0;

class SavingsAccount : public Account {
    double rate;
public:
    SavingsAccount(string a, string h, double b, double r)
        : Account(a, h, b), rate(r) {}

    void addInterest() { balance += balance * rate / 100; }

    void display() const override {
        cout << "[Savings] "; Account::display();     // call the base version explicitly
    }
};

class CurrentAccount : public Account {
    double overdraft;
public:
    CurrentAccount(string a, string h, double b, double od)
        : Account(a, h, b), overdraft(od) {}

    bool withdraw(double amt) override {              // OVERRIDDEN rule
        if (amt <= 0 || amt > balance + overdraft) {
            cout << "Overdraft limit exceeded\n"; return false;
        }
        balance -= amt;
        cout << "Withdrew " << amt << ", balance = " << balance << "\n";
        return true;
    }
    void display() const override {
        cout << "[Current] "; Account::display();
    }
};

int main() {
    vector<Account*> accounts;
    accounts.push_back(new SavingsAccount("S01", "Alice", 1000, 5));
    accounts.push_back(new CurrentAccount("C01", "Bob", 1000, 500));

    accounts[0]->withdraw(1200);      // rejected
    accounts[1]->withdraw(1200);      // allowed (overdraft)

    for (Account* a : accounts) a->display();     // polymorphic dispatch
    cout << "Total accounts: " << Account::getTotalAccounts() << "\n";

    for (Account* a : accounts) delete a;
}
```

**Say this out loud:** *"withdraw is virtual so each account type enforces its own rule while client code holds only Account*. That is run-time polymorphism, and it is why adding a new account type requires no change to main()."*

---

## Q2. Shape Hierarchy — area & perimeter (classic abstract-class question)

**Concepts:** abstract class, pure virtual, run-time polymorphism, virtual destructor, comparator.

```cpp
class Shape {
public:
    virtual double area() const = 0;
    virtual double perimeter() const = 0;
    virtual string name() const = 0;
    virtual ~Shape() {}

    void report() const {           // non-virtual, calls virtuals -> Template Method pattern
        cout << name() << ": area=" << fixed << setprecision(2) << area()
             << " perimeter=" << perimeter() << "\n";
    }
};

class Circle : public Shape {
    double r;
public:
    Circle(double r) : r(r) {}
    double area() const override      { return 3.14159265 * r * r; }
    double perimeter() const override { return 2 * 3.14159265 * r; }
    string name() const override      { return "Circle"; }
};

class Rectangle : public Shape {
protected:
    double l, b;
public:
    Rectangle(double l, double b) : l(l), b(b) {}
    double area() const override      { return l * b; }
    double perimeter() const override { return 2 * (l + b); }
    string name() const override      { return "Rectangle"; }
};

class Square : public Rectangle {         // a Square IS-A Rectangle
public:
    Square(double s) : Rectangle(s, s) {}
    string name() const override { return "Square"; }
};

class Triangle : public Shape {
    double a, b, c;
public:
    Triangle(double a, double b, double c) : a(a), b(b), c(c) {}
    double perimeter() const override { return a + b + c; }
    double area() const override {                       // Heron formula
        double s = perimeter() / 2;
        return sqrt(s * (s-a) * (s-b) * (s-c));
    }
    string name() const override { return "Triangle"; }
};

int main() {
    vector<Shape*> v = { new Circle(3), new Rectangle(4,5), new Square(4), new Triangle(3,4,5) };
    for (Shape* s : v) s->report();

    sort(v.begin(), v.end(), [](Shape* a, Shape* b){ return a->area() < b->area(); });
    cout << "Smallest: " << v[0]->name() << "\n";

    for (Shape* s : v) delete s;
}
```

---

## Q3. Employee Payroll — polymorphic salary

**Concepts:** inheritance, overriding, sorting objects by a computed value.

```cpp
class Employee {
protected:
    int id; string name; double baseSalary;
public:
    Employee(int i, string n, double s) : id(i), name(n), baseSalary(s) {}
    virtual double calculateSalary() const { return baseSalary; }
    virtual string role() const { return "Employee"; }
    virtual ~Employee() {}

    string getName() const { return name; }

    void print() const {
        cout << id << " " << name << " (" << role() << ") = "
             << calculateSalary() << "\n";
    }
};

class Manager : public Employee {
    double bonus;
public:
    Manager(int i, string n, double s, double b) : Employee(i,n,s), bonus(b) {}
    double calculateSalary() const override { return baseSalary + bonus; }
    string role() const override { return "Manager"; }
};

class Developer : public Employee {
    int overtimeHours; double rate;
public:
    Developer(int i, string n, double s, int h, double r)
        : Employee(i,n,s), overtimeHours(h), rate(r) {}
    double calculateSalary() const override { return baseSalary + overtimeHours * rate; }
    string role() const override { return "Developer"; }
};

int main() {
    vector<Employee*> e = {
        new Manager(1, "Asha", 80000, 20000),
        new Developer(2, "Ravi", 60000, 10, 500),
        new Employee(3, "Kiran", 40000)
    };

    sort(e.begin(), e.end(), [](Employee* a, Employee* b){
        return a->calculateSalary() > b->calculateSalary();     // highest first
    });

    double total = 0;
    for (Employee* p : e) { p->print(); total += p->calculateSalary(); }
    cout << "Total payroll = " << total << "\n";

    for (Employee* p : e) delete p;
}
```

---

## Q4. Build your own Stack class (templated)

**Concepts:** templates, dynamic memory, Rule of Three, exceptions, dynamic resizing.
This is the highest-value practice problem in the file — it merges **OOPs** with **Stacks**.

```cpp
template <typename T>
class MyStack {
    T* arr;
    int capacity;
    int topIdx;                          // -1 when empty

    void resize() {
        capacity *= 2;
        T* tmp = new T[capacity];
        for (int i = 0; i <= topIdx; i++) tmp[i] = arr[i];
        delete[] arr;
        arr = tmp;
    }

public:
    MyStack(int cap = 4) : capacity(cap), topIdx(-1) { arr = new T[capacity]; }

    MyStack(const MyStack& o) : capacity(o.capacity), topIdx(o.topIdx) {   // deep copy
        arr = new T[capacity];
        for (int i = 0; i <= topIdx; i++) arr[i] = o.arr[i];
    }

    MyStack& operator=(const MyStack& o) {
        if (this == &o) return *this;
        delete[] arr;
        capacity = o.capacity; topIdx = o.topIdx;
        arr = new T[capacity];
        for (int i = 0; i <= topIdx; i++) arr[i] = o.arr[i];
        return *this;
    }

    ~MyStack() { delete[] arr; }

    void push(const T& x) {
        if (topIdx == capacity - 1) resize();
        arr[++topIdx] = x;
    }
    T pop() {
        if (isEmpty()) throw runtime_error("Stack underflow");
        return arr[topIdx--];
    }
    T top() const {
        if (isEmpty()) throw runtime_error("Stack is empty");
        return arr[topIdx];
    }
    bool isEmpty() const { return topIdx == -1; }
    int  size()    const { return topIdx + 1; }
};

int main() {
    MyStack<int> s;
    for (int i = 1; i <= 6; i++) s.push(i);        // triggers a resize
    MyStack<int> t = s;                             // deep copy
    while (!t.isEmpty()) cout << t.pop() << " ";    // 6 5 4 3 2 1
    cout << "\noriginal size still " << s.size() << "\n";   // 6 -> proves deep copy
}
```

---

## Q5. Library Management System

**Concepts:** composition (has-a), collections of objects, search, state management.

```cpp
class Book {
    string isbn, title, author;
    bool issued;
public:
    Book(string i, string t, string a) : isbn(i), title(t), author(a), issued(false) {}
    string getIsbn() const  { return isbn; }
    string getTitle() const { return title; }
    bool isIssued() const   { return issued; }
    void issue()  { issued = true; }
    void ret()    { issued = false; }
    void display() const {
        cout << isbn << " | " << title << " | " << author
             << " | " << (issued ? "Issued" : "Available") << "\n";
    }
};

class Library {                       // COMPOSITION: Library HAS-A collection of Books
    vector<Book> books;
    map<string, vector<string>> memberBooks;   // member -> list of ISBNs

public:
    void addBook(const Book& b) { books.push_back(b); }

    Book* findBook(const string& isbn) {
        for (auto& b : books)
            if (b.getIsbn() == isbn) return &b;
        return nullptr;
    }

    bool issueBook(const string& isbn, const string& member) {
        Book* b = findBook(isbn);
        if (!b)            { cout << "Book not found\n"; return false; }
        if (b->isIssued()) { cout << "Already issued\n"; return false; }
        b->issue();
        memberBooks[member].push_back(isbn);
        cout << "Issued " << b->getTitle() << " to " << member << "\n";
        return true;
    }

    bool returnBook(const string& isbn, const string& member) {
        Book* b = findBook(isbn);
        if (!b || !b->isIssued()) { cout << "Invalid return\n"; return false; }
        b->ret();
        auto& lst = memberBooks[member];
        lst.erase(remove(lst.begin(), lst.end(), isbn), lst.end());
        cout << "Returned " << b->getTitle() << "\n";
        return true;
    }

    void displayAll() const { for (const auto& b : books) b.display(); }
};

int main() {
    Library lib;
    lib.addBook(Book("111", "C++ Primer", "Lippman"));
    lib.addBook(Book("222", "DSA", "Cormen"));

    lib.issueBook("111", "Alice");
    lib.issueBook("111", "Bob");        // already issued -> rejected
    lib.displayAll();
    lib.returnBook("111", "Alice");
    lib.displayAll();
}
```

---

## Q6. Matrix class with operator overloading

**Concepts:** operator overloading, 2D vectors, dimension validation, friend function.

```cpp
class Matrix {
    int rows, cols;
    vector<vector<int>> m;
public:
    Matrix(int r, int c) : rows(r), cols(c), m(r, vector<int>(c, 0)) {}

    void read() {
        for (int i = 0; i < rows; i++)
            for (int j = 0; j < cols; j++) cin >> m[i][j];
    }

    Matrix operator+(const Matrix& o) const {
        if (rows != o.rows || cols != o.cols) throw invalid_argument("Dimension mismatch");
        Matrix res(rows, cols);
        for (int i = 0; i < rows; i++)
            for (int j = 0; j < cols; j++) res.m[i][j] = m[i][j] + o.m[i][j];
        return res;
    }

    Matrix operator*(const Matrix& o) const {
        if (cols != o.rows) throw invalid_argument("Cannot multiply");
        Matrix res(rows, o.cols);
        for (int i = 0; i < rows; i++)
            for (int j = 0; j < o.cols; j++) {
                int s = 0;
                for (int k = 0; k < cols; k++) s += m[i][k] * o.m[k][j];
                res.m[i][j] = s;
            }
        return res;
    }

    Matrix transpose() const {
        Matrix res(cols, rows);
        for (int i = 0; i < rows; i++)
            for (int j = 0; j < cols; j++) res.m[j][i] = m[i][j];
        return res;
    }

    friend ostream& operator<<(ostream& os, const Matrix& x) {
        for (int i = 0; i < x.rows; i++) {
            for (int j = 0; j < x.cols; j++) os << x.m[i][j] << " ";
            os << "\n";
        }
        return os;
    }
};
```

---

## 12. If they ask for a "design a system" class question

Follow this recipe and you will never freeze:

1. **Nouns -> classes.** ("A *library* issues *books* to *members*" -> `Library`, `Book`, `Member`.)
2. **Adjectives / data -> private members.**
3. **Verbs -> public methods.** (issue, return, search)
4. **"X is a kind of Y" -> inheritance. "X has a Y" -> composition** (a member variable).
5. Add a **constructor** for every class, and a **virtual destructor** if it is a base class.
6. Add **validation** in mutators — this is where encapsulation marks live.
7. Write a `main()` that **demonstrates** each operation, **including a failure case** (withdraw too much, issue an issued book). Examiners love seeing the guard rails work.

**Prefer composition over inheritance** unless there is a genuine "is-a" relationship. A `Car` **has-an** `Engine`; a `Car` **is-a** `Vehicle`.

---

## 13. Rapid-fire self-test (cover the answers)

1. Why must a base class destructor be `virtual`? -> *Deleting a derived object through a base pointer otherwise runs only the base destructor, leaking the derived part.*
2. Can a constructor be virtual? -> *No. The vtable pointer is not set up until the object is constructed.*
3. Can a static function be virtual? -> *No. `virtual` needs an object (`this`); `static` has none.*
4. Overloading vs overriding? -> *Overloading: same name, different parameters, resolved at compile time, same scope. Overriding: identical signature, base vs derived, resolved at run time via the vtable.*
5. Rule of Three? -> *If you need a destructor, you also need a copy constructor and a copy assignment operator.*
6. What does `const` after a member function mean? -> *The function will not modify the object's state; it can be called on const objects.*
7. Can you instantiate an abstract class? -> *No, but you can hold pointers/references to it.*
8. What is object slicing? -> *Assigning a derived object to a base object **by value** copies only the base part and discards derived data. Avoid it by using pointers or references.*
9. Order of construction/destruction in inheritance? -> *Base then Derived on construction; Derived then Base on destruction.*
10. `struct` vs `class` in C++? -> *Only the default access level: `struct` defaults to public, `class` to private.*
11. Why is `friend` considered a compromise? -> *It grants outside code access to private members, weakening encapsulation; justified only when the alternative (public getters for internals) would be worse.*
12. Association vs aggregation vs composition? -> *Association = objects merely use each other. Aggregation = has-a with independent lifetimes (Department has Professors). Composition = has-a with dependent lifetime (House has Rooms; destroy the house and the rooms go).*

Next: [02_ARRAYS.md](02_ARRAYS.md)
