## Object-Oriented Programming (OOP)

### Definition

**Object-Oriented Programming (OOP)** is a programming paradigm based on the concept of "objects," which can contain data (attributes or properties) and code (methods or behaviors). OOP aims to model real-world entities as objects, making software more modular, reusable, and easier to understand and maintain.

OOP is one of the most popular programming paradigms and is widely used in various applications, from desktop software to web development and enterprise systems.

### Core Concepts of OOP

OOP is typically characterized by four fundamental concepts:

1.  **Encapsulation:**
    *   **Definition:** Bundling data (attributes) and the methods that operate on that data within a single unit (an object/class). It also involves restricting direct access to some of an object's components, preventing external code from directly manipulating internal state.
    *   **Purpose:** To hide the internal implementation details of an object from the outside world, exposing only a well-defined interface. This protects data integrity and reduces complexity.
    *   **Example:** A `BankAccount` object encapsulates its `balance` and provides `deposit()` and `withdraw()` methods, but the `balance` itself might be private.

2.  **Abstraction:**
    *   **Definition:** Showing only essential information and hiding the complex implementation details. It focuses on *what* an object does rather than *how* it does it.
    *   **Purpose:** To reduce complexity, improve maintainability, and allow for easier changes to underlying implementations.
    *   **Example:** A `Car` object might have a `start()` method, but the user doesn't need to know the intricate details of how the engine starts.

3.  **Inheritance:**
    *   **Definition:** A mechanism where a new class (subclass/derived class) is created from an existing class (superclass/base class), inheriting its attributes and methods.
    *   **Purpose:** To promote code reuse and establish "is-a" relationships between classes.
    *   **Example:** A `Car` class might inherit from a `Vehicle` class, gaining common properties like `speed` and `color`.

4.  **Polymorphism:**
    *   **Definition:** The ability of an object to take on many forms. In OOP, it often refers to the ability of objects of different classes to be treated as objects of a common type (their base class or interface), and to respond to the same method call in different ways.
    *   **Purpose:** To enable flexible and extensible code, allowing for generic programming and dynamic behavior.
    *   **Example:** A `draw()` method called on a collection of `Shape` objects (e.g., `Circle`, `Square`) will execute the appropriate `draw()` implementation for each specific shape.

### SOLID Principles in an OOP Context

SOLID is a mnemonic acronym for five design principles intended to make software designs more understandable, flexible, and maintainable.

*   **S - Single-responsibility Principle:** A class should have only one reason to change.
*   **O - Open-closed Principle:** Software entities should be open for extension, but closed for modification.
*   **L - Liskov Substitution Principle:** Subtypes must be substitutable for their base types.
*   **I - Interface Segregation Principle:** No client should be forced to depend on methods it does not use.
*   **D - Dependency Inversion Principle:** High-level modules should not depend on low-level modules. Both should depend on abstractions.

### Rust and Object-Oriented Programming

Rust is not an object-oriented language in the traditional sense (it doesn't have classes, inheritance hierarchies, or virtual methods in the same way C++ or Java do). However, it provides powerful features that allow developers to achieve OOP principles in an idiomatic and often safer way:

*   **Encapsulation:** Achieved through `pub` and `private` visibility modifiers for struct fields and module items. Methods are defined directly on structs.
*   **Abstraction:** Primarily achieved through **Traits**. Traits define shared behavior (interfaces) that different types can implement.
*   **Inheritance:** Rust does not have class inheritance. Instead, it favors **Composition** (structs containing other structs) and **Trait Objects** (dynamic dispatch) or **Generics** (static dispatch) for code reuse and polymorphism.
*   **Polymorphism:** Achieved through **Traits** and **Trait Objects** (`Box<dyn Trait>`) for dynamic dispatch, or **Generics** (`<T: MyTrait>`) for static dispatch.

#### Composition Over Inheritance in Rust

Instead of inheriting behavior, Rust encourages composing functionality by having a struct own another struct that provides the desired behavior. This is often more flexible and avoids the tight coupling of inheritance hierarchies.

#### Static vs. Dynamic Dispatch in Rust

*   **Static Dispatch (Generics):** The compiler knows the concrete type at compile time. This is fast because there is no runtime overhead. It's achieved using generics (`<T: MyTrait>`).
*   **Dynamic Dispatch (Trait Objects):** The compiler doesn't know the concrete type at compile time. The method to call is determined at runtime. This has a small runtime cost but is more flexible. It's achieved using trait objects (`&dyn MyTrait` or `Box<dyn MyTrait>`).

#### Rust Example: OOP Concepts in Rust

Let's illustrate encapsulation, abstraction, and polymorphism in Rust.

```rust
// Encapsulation
pub struct BankAccount {
    account_number: String, // Private
    balance: f64,           // Private
}

impl BankAccount {
    pub fn new(account_number: String, initial_balance: f64) -> Self {
        BankAccount { account_number, balance: initial_balance }
    }

    pub fn deposit(&mut self, amount: f64) {
        if amount > 0.0 {
            self.balance += amount;
        }
    }

    pub fn get_balance(&self) -> f64 { // Public getter
        self.balance
    }
}

// Abstraction and Polymorphism
pub trait Drawable {
    fn draw(&self);
}

pub struct Button {
    pub label: String,
}

impl Drawable for Button {
    fn draw(&self) {
        println!("Drawing a button with label: {}", self.label);
    }
}

pub struct Image {
    pub src: String,
}

impl Drawable for Image {
    fn draw(&self) {
        println!("Drawing an image from src: {}", self.src);
    }
}

// Static Dispatch with Generics
fn draw_item<T: Drawable>(item: &T) {
    item.draw();
}

// Dynamic Dispatch with Trait Objects
fn draw_items(items: &[Box<dyn Drawable>]) {
    for item in items {
        item.draw();
    }
}

fn main() {
    let mut account = BankAccount::new("12345".to_string(), 100.0);
    account.deposit(50.0);
    println!("Balance: {}", account.get_balance());

    let button = Button { label: "Click me".to_string() };
    let image = Image { src: "/path/to/image.png".to_string() };

    println!("
--- Static Dispatch ---");
    draw_item(&button);
    draw_item(&image);

    println!("
--- Dynamic Dispatch ---");
    let items: Vec<Box<dyn Drawable>> = vec![
        Box::new(button),
        Box::new(image),
    ];
    draw_items(&items);
}
```

### When to Use OOP Patterns in Rust

While Rust isn't a traditional OOP language, OOP patterns can be useful in specific scenarios, especially when dealing with GUIs, game development, or other systems where you have a collection of heterogeneous objects that need to be treated uniformly.

### Conclusion

Object-Oriented Programming (OOP) is a powerful paradigm that helps model real-world entities and build modular, reusable, and maintainable software. While Rust does not implement OOP in the traditional class-based inheritance model, its robust features like encapsulation (via visibility), abstraction (via traits), and polymorphism (via traits and generics) allow developers to achieve the benefits of OOP in a safe and idiomatic way. Rust's unique approach often leads to designs that are more explicit, less prone to common OOP pitfalls, and inherently more reliable.