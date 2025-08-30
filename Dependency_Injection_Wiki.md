## Dependency Injection (DI)

### Definition

**Dependency Injection (DI)** is a software design pattern that allows the creation of dependent objects outside of a class and provides those objects to a class through various means (constructor, method, or property injection). It's a specific form of **Inversion of Control (IoC)**, where the control of creating and managing dependencies is inverted from the dependent class to an external entity (an "injector" or "container").

Instead of a class creating its own dependencies, it receives them from the outside. This promotes loose coupling and makes components more modular, testable, and reusable.

### Why Use Dependency Injection?

*   **Loose Coupling:** Components are not tightly bound to their concrete dependencies. They only depend on abstractions (interfaces/traits).
*   **Improved Testability:** Dependencies can be easily replaced with test doubles (mocks, stubs, fakes) during unit testing.
*   **Increased Reusability:** Components become more generic and can be reused in different contexts.
*   **Easier Maintenance:** Changes to a dependency's implementation don't require changes in the dependent class.
*   **Better Code Organization:** Encourages a clear separation of concerns.

### Types of Dependency Injection

1.  **Constructor Injection:** Dependencies are provided through the class's constructor. This is the most common and preferred method.
2.  **Setter Injection (Property Injection):** Dependencies are provided through public setter methods.
3.  **Method Injection (Interface Injection):** Dependencies are provided through specific methods.

### Rust and Dependency Injection

Rust's strong type system, ownership model, and trait system make it an excellent language for implementing DI, often without the need for complex DI containers.

*   **Traits as Interfaces:** Rust's traits are the natural way to define abstractions.
*   **Constructor Injection (Idiomatic):** Passing dependencies to `new` functions is the most common way to achieve DI in Rust.
*   **Manual DI:** For many Rust applications, manual dependency injection is sufficient and preferred.
*   **DI Containers:** For very large applications, crates like `shaku` can provide more automated DI container functionality.

#### Rust Example: Constructor Injection

```rust
// Trait for our dependency
pub trait Greeter {
    fn greet(&self) -> String;
}

// Concrete implementation
pub struct EnglishGreeter;
impl Greeter for EnglishGreeter {
    fn greet(&self) -> String {
        "Hello!".to_string()
    }
}

// Struct with a dependency on the Greeter trait
pub struct Service<G: Greeter> {
    greeter: G,
}

impl<G: Greeter> Service<G> {
    // Constructor injection
    pub fn new(greeter: G) -> Self {
        Service { greeter }
    }

    pub fn do_something(&self) {
        println!("{}", self.greeter.greet());
    }
}

fn main() {
    let greeter = EnglishGreeter;
    let service = Service::new(greeter);
    service.do_something();
}
```

#### Rust Example: Setter Injection

```rust
pub trait Logger {
    fn log(&self, message: &str);
}

pub struct ConsoleLogger;
impl Logger for ConsoleLogger {
    fn log(&self, message: &str) {
        println!("LOG: {}", message);
    }
}

pub struct Task {
    logger: Option<Box<dyn Logger>>,
}

impl Task {
    pub fn new() -> Self {
        Task { logger: None }
    }

    // Setter injection
    pub fn set_logger(&mut self, logger: Box<dyn Logger>) {
        self.logger = Some(logger);
    }

    pub fn execute(&self) {
        if let Some(logger) = &self.logger {
            logger.log("Executing task...");
        }
        println!("Task executed.");
    }
}

fn main() {
    let mut task = Task::new();
    task.execute(); // No logging

    task.set_logger(Box::new(ConsoleLogger));
    task.execute(); // With logging
}
```

### Dependency Injection vs. Service Locator

A Service Locator is another pattern for managing dependencies. However, it can hide dependencies and make code harder to test. With DI, dependencies are explicit. With a Service Locator, a class asks for its dependencies from a central locator, which can lead to a less transparent design.

### Conclusion

Dependency Injection is a powerful design pattern that promotes loose coupling, improves testability, and enhances the maintainability and reusability of software components. Rust's features make it well-suited for implementing DI in a clear and idiomatic way, leading to robust and well-structured applications.