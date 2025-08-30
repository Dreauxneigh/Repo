## SOLID Principles

### Definition

**SOLID** is an acronym for five fundamental design principles in object-oriented programming and design. These principles, popularized by Robert C. Martin (Uncle Bob), are intended to make software designs more understandable, flexible, and maintainable.

### The Five Principles

1.  **S - Single Responsibility Principle (SRP):** A class should have only one reason to change.
2.  **O - Open/Closed Principle (OCP):** Software entities should be open for extension, but closed for modification.
3.  **L - Liskov Substitution Principle (LSP):** Subtypes must be substitutable for their base types.
4.  **I - Interface Segregation Principle (ISP):** No client should be forced to depend on methods it does not use.
5.  **D - Dependency Inversion Principle (DIP):** High-level modules should not depend on low-level modules. Both should depend on abstractions.

### Benefits of SOLID

*   **Improved Maintainability:** Code is easier to understand, modify, and maintain.
*   **Improved Testability:** Code is easier to test in isolation.
*   **Improved Flexibility:** Code is easier to extend with new functionality.
*   **Reduced Coupling:** Code is less dependent on other parts of the system.

### Relationship with Other Design Principles

*   **DRY (Don't Repeat Yourself):** SOLID helps to achieve DRY by promoting the extraction of common functionality into separate classes or modules.
*   **KISS (Keep It Simple, Stupid):** SOLID helps to achieve KISS by promoting small, focused classes and modules.
*   **YAGNI (You Ain't Gonna Need It):** SOLID helps to achieve YAGNI by promoting the design of flexible systems that can be easily extended with new functionality when it is needed.

### Rust Examples

#### Single Responsibility Principle (SRP)

**Bad:**

```rust
struct User {
    name: String,
    email: String,
}

impl User {
    fn save_to_database(&self) { /* ... */ }
    fn send_welcome_email(&self) { /* ... */ }
}
```

**Good:**

```rust
struct User {
    name: String,
    email: String,
}

trait UserRepository {
    fn save(&self, user: &User);
}

trait EmailService {
    fn send_welcome_email(&self, user: &User);
}
```

#### Open/Closed Principle (OCP)

**Bad:**

```rust
enum Shape {
    Circle(f64),
    Square(f64),
}

fn area(shape: &Shape) -> f64 {
    match shape {
        Shape::Circle(r) => std::f64::consts::PI * r * r,
        Shape::Square(s) => s * s,
    }
}
```

**Good:**

```rust
trait Shape {
    fn area(&self) -> f64;
}

struct Circle { radius: f64 }
impl Shape for Circle { /* ... */ }

struct Square { side: f64 }
impl Shape for Square { /* ... */ }
```

#### Liskov Substitution Principle (LSP)

**Bad:**

```rust
trait Bird {
    fn fly(&self);
}

struct Penguin;
impl Bird for Penguin {
    fn fly(&self) { panic!("Penguins can't fly!"); }
}
```

**Good:**

```rust
trait Bird {}

trait FlyingBird: Bird {
    fn fly(&self);
}
```

#### Interface Segregation Principle (ISP)

**Bad:**

```rust
trait Worker {
    fn work(&self);
    fn eat(&self);
}

struct Robot;
impl Worker for Robot {
    fn work(&self) { /* ... */ }
    fn eat(&self) { /* Does nothing */ }
}
```

**Good:**

```rust
trait Workable {
    fn work(&self);
}

trait Eatable {
    fn eat(&self);
}
```

#### Dependency Inversion Principle (DIP)

**Bad:**

```rust
struct HighLevelModule {
    low_level_module: LowLevelModule,
}

struct LowLevelModule;
```

**Good:**

```rust
trait LowLevelModule {
    fn do_something(&self);
}

struct HighLevelModule<T: LowLevelModule> {
    low_level_module: T,
}
```

### Conclusion

The SOLID principles are a set of guidelines that can help you to write better, more maintainable code. By following these principles, you can create systems that are easier to understand, modify, and extend over their lifetime.