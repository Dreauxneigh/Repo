## Command Query Responsibility Segregation (CQRS)

### Definition

**Command Query Responsibility Segregation (CQRS)** is an architectural pattern that separates the operations that change data (Commands) from the operations that read data (Queries). In a traditional CRUD system, the same data model is used for both reading and writing. CQRS advocates for using different models for these two distinct purposes.

### Key Concepts

*   **Commands:** Represent an intent to change the state of the system (e.g., `CreateOrder`).
*   **Queries:** Represent a request to retrieve data from the system (e.g., `GetProductDetails`).
*   **Command Model (Write Model):** The data model optimized for handling Commands.
*   **Query Model (Read Model):** The data model optimized for handling Queries.

### When to Use CQRS

CQRS is most beneficial for complex domains where the read and write requirements are very different. For simple CRUD applications, it may be overkill.

### CQRS and Event Sourcing

CQRS is often used with Event Sourcing. In this pattern, all changes to the application state are stored as a sequence of events. The Command Model is an append-only log of events, and the Query Model is built by replaying these events.

### Rust Example: Simple CQRS with In-Memory Stores

Let's illustrate CQRS with a very basic in-memory example for a `Product` domain.

**1. Commands and Queries**

```rust
// src/main.rs

// --- Commands ---
pub struct CreateProductCommand {
    pub id: String,
    pub name: String,
}

// --- Queries ---
pub struct GetProductQuery {
    pub id: String,
}
```

**2. The Write Side (Command Model and Handlers)**

```rust
// src/main.rs

use std::collections::HashMap;
use std::sync::{Arc, Mutex};

// The write model
#[derive(Clone)]
pub struct ProductWriteModel {
    pub id: String,
    pub name: String,
}

// The command handler
pub struct ProductCommandHandler {
    store: Arc<Mutex<HashMap<String, ProductWriteModel>>>,
}

impl ProductCommandHandler {
    pub fn new(store: Arc<Mutex<HashMap<String, ProductWriteModel>>>) -> Self {
        Self { store }
    }

    pub fn handle(&self, command: CreateProductCommand) {
        let product = ProductWriteModel { id: command.id, name: command.name };
        let mut store = self.store.lock().unwrap();
        store.insert(product.id.clone(), product);
    }
}
```

**3. The Read Side (Query Model and Handlers)**

```rust
// src/main.rs

// The read model
#[derive(Clone)]
pub struct ProductReadModel {
    pub id: String,
    pub name: String,
}

// The query handler
pub struct ProductQueryHandler {
    store: Arc<Mutex<HashMap<String, ProductReadModel>>>,
}

impl ProductQueryHandler {
    pub fn new(store: Arc<Mutex<HashMap<String, ProductReadModel>>>) -> Self {
        Self { store }
    }

    pub fn handle(&self, query: GetProductQuery) -> Option<ProductReadModel> {
        let store = self.store.lock().unwrap();
        store.get(&query.id).cloned()
    }
}
```

**4. Putting it all together**

```rust
// src/main.rs

fn main() {
    // In a real application, the read and write stores would be separate
    let write_store = Arc::new(Mutex::new(HashMap::new()));
    let read_store = Arc::new(Mutex::new(HashMap::new()));

    let command_handler = ProductCommandHandler::new(write_store.clone());
    let query_handler = ProductQueryHandler::new(read_store.clone());

    // 1. Handle a command
    let create_product = CreateProductCommand { id: "1".to_string(), name: "Laptop".to_string() };
    command_handler.handle(create_product);

    // 2. Synchronize the read store (in a real app, this would be done via events)
    let write_db = write_store.lock().unwrap();
    let mut read_db = read_store.lock().unwrap();
    for (id, product) in write_db.iter() {
        read_db.insert(id.clone(), ProductReadModel { id: product.id.clone(), name: product.name.clone() });
    }

    // 3. Handle a query
    let get_product = GetProductQuery { id: "1".to_string() };
    let product = query_handler.handle(get_product).unwrap();
    println!("Product: {} ({})", product.name, product.id);
}
```

### Common CQRS Pitfalls

*   **Over-engineering:** Applying CQRS to simple domains where it's not needed.
*   **Ignoring Eventual Consistency:** Not properly handling the fact that the read model may be stale.
*   **Leaky Abstractions:** Allowing the write model to leak into the read side, or vice-versa.

### Conclusion

CQRS is a powerful pattern for building complex, scalable, and maintainable systems. By separating the read and write concerns, it allows each side to be optimized independently, leading to improved performance and flexibility. However, it also introduces complexity, so it should be used judiciously.