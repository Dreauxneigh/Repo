## Declarative Programming

### Definition

**Declarative Programming** is a programming paradigm that focuses on *what* the program should accomplish, rather than *how* it should accomplish it. In contrast to imperative programming, which describes control flow step-by-step, declarative programming expresses the logic of a computation without describing its control flow. The system (language runtime, framework, database engine) then figures out the "how."

This paradigm emphasizes expressing the desired result or properties of the computation, leaving the execution details to the underlying system.

### Key Concepts of Declarative Programming

1.  **Focus on "What," Not "How":**
    *   You describe the desired outcome or state.
    *   The system determines the sequence of operations to achieve that outcome.

2.  **No Side Effects (Ideally):**
    *   Declarative programs often strive for immutability and pure functions, similar to functional programming, to make reasoning about the system easier.

3.  **High-Level Abstractions:**
    *   Relies on powerful abstractions that hide the underlying implementation details.

4.  **Domain-Specific Languages (DSLs):**
    *   Often uses or resembles DSLs tailored to a specific problem domain, making the code more readable and expressive for that domain.

### Why Use Declarative Programming?

*   **Easier to Reason About:** By focusing on the desired result, code becomes more concise and easier to understand, as you don't need to trace complex control flows.
*   **Reduced Bugs:** Less imperative code means fewer opportunities for common programming errors related to state management and control flow.
*   **Improved Maintainability:** Changes to the "how" (implementation) can often be made without altering the "what" (declaration).
*   **Better Parallelization:** The underlying system can often optimize and parallelize declarative code more effectively because it has more freedom in choosing the execution strategy.
*   **Higher Productivity:** Developers can express complex logic with less code.

### Common Examples of Declarative Programming

*   **SQL (Structured Query Language):** You declare *what* data you want (`SELECT * FROM users WHERE age > 30`), not *how* the database should retrieve it (e.g., which index to use, how to join tables).
*   **HTML (HyperText Markup Language):** You declare *what* content should be on a web page (`<p>Hello</p>`), not *how* the browser should render it.
*   **CSS (Cascading Style Sheets):** You declare *what* the style of an element should be (`color: blue;`), not *how* the browser should apply that style.
*   **Regular Expressions:** You declare *what* pattern to match, not *how* the matching algorithm should work.
*   **Functional Programming (often declarative):** Using `map`, `filter`, `reduce` to describe transformations on data without specifying loops.
*   **Logic Programming:** Prolog.
*   **Configuration Languages:** YAML, JSON, XML (when used to describe desired states, like in Kubernetes manifests or Terraform).
*   **Reactive Programming:** Describing data flows and transformations.

### Rust and Declarative Programming

Rust, while primarily an imperative language, strongly supports and encourages declarative programming styles through several features:

*   **Iterators:** Rust's iterator system is highly declarative. You describe the transformations (`map`, `filter`, `fold`, `for_each`) you want to apply to a collection, and the compiler optimizes the underlying loops.
*   **Pattern Matching (`match`):** A powerful declarative construct for handling different data shapes or enum variants.
*   **Macros:** Rust's macro system allows for creating powerful domain-specific languages (DSLs) that can be highly declarative.
*   **Traits:** Define behavior declaratively.
*   **`Option` and `Result`:** Encourage declarative error handling and optional value handling, focusing on the possible outcomes rather than imperative checks.
*   **`serde`:** A declarative serialization/deserialization framework where you derive traits to describe how data should be converted.
*   **Web Frameworks (e.g., Actix Web, Axum):** Use declarative routing and handler definitions.
*   **Infrastructure as Code (IaC) Tools (e.g., Pulumi with Rust):** Define desired infrastructure state declaratively.

#### Rust Example: Declarative Data Transformation with Iterators and Pattern Matching

Let's filter and transform a list of strings using a declarative style.

**1. Setup `Cargo.toml`:**

```toml
[package]
name = "rust_declarative_example"
version = "0.1.0"
edition = "2021"

[dependencies]
```

**2. Example Code (`src/main.rs`)**

```rust
// src/main.rs

#[derive(Debug, PartialEq)]
enum Status {
    Active,
    Inactive,
    Pending,
}

#[derive(Debug)]
struct User {
    id: u32,
    name: String,
    status: Status,
    is_admin: bool,
}

fn main() {
    let users = vec![
        User { id: 1, name: "Alice".to_string(), status: Status::Active, is_admin: false },
        User { id: 2, name: "Bob".to_string(), status: Status::Inactive, is_admin: false },
        User { id: 3, name: "Charlie".to_string(), status: Status::Active, is_admin: true },
        User { id: 4, name: "David".to_string(), status: Status::Pending, is_admin: false },
        User { id: 5, name: "Eve".to_string(), status: Status::Active, is_admin: false },
    ];

    println!("Original users: {:?}", users);

    // --- Declarative approach: Find active non-admin users and get their names ---
    let active_non_admin_names: Vec<String> = users.iter()
        // Filter: only active users
        .filter(|user| matches!(user.status, Status::Active))
        // Filter: only non-admin users
        .filter(|user| !user.is_admin)
        // Map: transform to just their names
        .map(|user| user.name.clone())
        // Collect: gather results into a new Vec
        .collect();

    println!("\nActive non-admin user names: {:?}", active_non_admin_names);

    // --- Declarative approach: Count users by status ---
    let active_count = users.iter()
        .filter(|user| matches!(user.status, Status::Active))
        .count();
    println!("\nNumber of active users: {}", active_count);

    // --- Declarative approach: Process a command using pattern matching ---
    let command = "START_SERVICE"; // Imagine this comes from user input

    let result = match command {
        "START_SERVICE" => "Service started successfully.",
        "STOP_SERVICE" => "Service stopped.",
        "RESTART_SERVICE" => "Service restarted.",
        _ => "Unknown command.",
    };
    println!("\nCommand result: {}", result);
}
```
To run: `cargo run`.

### Conclusion

Declarative Programming is a powerful paradigm that emphasizes expressing *what* a program should achieve, rather than *how*. This leads to more concise, readable, and maintainable code, often with better opportunities for optimization and parallelization by the underlying system. Rust, while primarily imperative, provides strong support for declarative styles through its rich iterator API, powerful pattern matching, and macro system, enabling developers to write expressive and efficient code that clearly communicates its intent.