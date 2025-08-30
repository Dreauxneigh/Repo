## Strategy Pattern

### Definition

The **Strategy Pattern** is a behavioral design pattern that enables selecting an algorithm's behavior at runtime. It defines a family of algorithms, encapsulates each one as a separate object, and makes them interchangeable. The Strategy Pattern lets the algorithm vary independently from clients that use it.

### When to Use the Strategy Pattern

Use the Strategy pattern when:

*   You have multiple variants of an algorithm.
*   You need to select an algorithm at runtime.
*   You want to avoid exposing complex, algorithm-specific data structures to the client.
*   You want to replace a conditional statement with a more flexible design.

### Static vs. Dynamic Dispatch in Rust

*   **Static Dispatch (Generics):** The concrete type of the strategy is known at compile time. This is more performant as there is no runtime overhead.
*   **Dynamic Dispatch (Trait Objects):** The concrete type of the strategy is not known until runtime. This is more flexible but has a small runtime cost.

### Rust Example: Text Formatting

Let's create a text formatter that can switch between different formatting strategies (e.g., Markdown, HTML).

#### 1. The Strategy Trait

```rust
// src/lib.rs
pub trait TextFormatter {
    fn format(&self, text: &str) -> String;
}
```

#### 2. Concrete Strategies

```rust
// src/lib.rs
pub struct MarkdownFormatter;
impl TextFormatter for MarkdownFormatter {
    fn format(&self, text: &str) -> String {
        format!("**{}**", text)
    }
}

pub struct HtmlFormatter;
impl TextFormatter for HtmlFormatter {
    fn format(&self, text: &str) -> String {
        format!("<b>{}</b>", text)
    }
}
```

#### 3. The Context

```rust
// src/lib.rs
pub struct Formatter<T: TextFormatter> {
    formatter: T,
}

impl<T: TextFormatter> Formatter<T> {
    pub fn new(formatter: T) -> Self {
        Self { formatter }
    }

    pub fn format(&self, text: &str) -> String {
        self.formatter.format(text)
    }
}
```

#### 4. Usage

```rust
// src/main.rs
use strategy_pattern::{
    Formatter,
    MarkdownFormatter,
    HtmlFormatter,
};

fn main() {
    let markdown_formatter = Formatter::new(MarkdownFormatter);
    println!("{}", markdown_formatter.format("Hello, world!"));

    let html_formatter = Formatter::new(HtmlFormatter);
    println!("{}", html_formatter.format("Hello, world!"));
}
```

### Relationship with Other Patterns

*   **State Pattern:** Both patterns are similar in structure, but they solve different problems. The Strategy pattern is about having different ways of doing something, while the State pattern is about being in different states.
*   **Template Method Pattern:** The Template Method pattern uses inheritance to vary parts of an algorithm, while the Strategy pattern uses composition.

### Conclusion

The Strategy Pattern is a powerful tool for designing flexible and extensible systems where algorithms can vary. By encapsulating algorithms behind a common interface, it promotes loose coupling and makes it easy to swap behaviors at runtime, leading to cleaner and more maintainable code.