## Hexagonal Architecture (Ports and Adapters)

### Introduction

Hexagonal Architecture, also known as Ports and Adapters Architecture, is a powerful architectural pattern in software design that emphasizes the strict separation of an application's core business logic from its external dependencies. Its primary goal is to create highly decoupled, testable, and maintainable systems that are resilient to changes in external technologies. The "hexagonal" metaphor illustrates that the application core, represented by the hexagon, can interact with various external actors (users, databases, other applications) through a set of well-defined "ports," with these interactions facilitated by "adapters."

### Core Concepts

The architecture revolves around three fundamental concepts:

1.  **Application Core (Domain Logic):** The central part of the application, containing business rules and domain models. It has no direct dependencies on external technologies.
2.  **Ports:** Abstract definitions (traits in Rust) that specify the contract for communication between the application core and external entities.
    *   **Driving Ports (Primary/Inbound):** Interfaces that allow external actors to drive the application.
    *   **Driven Ports (Secondary/Outbound):** Interfaces that allow the application to drive external systems.
3.  **Adapters:** Concrete implementations of the ports.
    *   **Driving Adapters (Primary/Inbound):** Implement the driving ports (e.g., a web controller).
    *   **Driven Adapters (Secondary/Outbound):** Implement the driven ports (e.g., a database repository).

### Rust Example: A Simple User Service

Let's build a simple user service in Rust using Hexagonal Architecture. The project structure will be:

```
src/
├── domain.rs         // Core data structures
├── application/
│   ├── user_service.rs // Driving port implementation (use case)
│   └── ports.rs        // Driving and driven port definitions (traits)
├── infrastructure/
│   ├── cli_adapter.rs  // Driving adapter (CLI)
│   └── in_memory_user_repository.rs // Driven adapter (in-memory DB)
└── main.rs           // Application entry point (wiring)
```

#### 1. The Domain (`src/domain.rs`)

```rust
// src/domain.rs
#[derive(Debug, Clone)]
pub struct User {
    pub id: String,
    pub name: String,
}
```

#### 2. The Ports (`src/application/ports.rs`)

```rust
// src/application/ports.rs
use crate::domain::User;

// Driving Port (Use Case)
pub trait UserServicePort {
    fn create_user(&self, name: &str) -> User;
    fn find_user(&self, id: &str) -> Option<User>;
}

// Driven Port (Repository)
pub trait UserRepositoryPort {
    fn save(&self, user: &User);
    fn find_by_id(&self, id: &str) -> Option<User>;
}
```

#### 3. The Application Core (`src/application/user_service.rs`)

```rust
// src/application/user_service.rs
use super::ports::{UserServicePort, UserRepositoryPort};
use crate::domain::User;
use uuid::Uuid;

pub struct UserService<R: UserRepositoryPort> {
    user_repository: R,
}

impl<R: UserRepositoryPort> UserService<R> {
    pub fn new(user_repository: R) -> Self {
        Self { user_repository }
    }
}

impl<R: UserRepositoryPort> UserServicePort for UserService<R> {
    fn create_user(&self, name: &str) -> User {
        let user = User {
            id: Uuid::new_v4().to_string(),
            name: name.to_string(),
        };
        self.user_repository.save(&user);
        user
    }

    fn find_user(&self, id: &str) -> Option<User> {
        self.user_repository.find_by_id(id)
    }
}
```

#### 4. The Adapters (`src/infrastructure/`)

**Driven Adapter (In-Memory Repository)**

```rust
// src/infrastructure/in_memory_user_repository.rs
use crate::application::ports::UserRepositoryPort;
use crate::domain::User;
use std::collections::HashMap;
use std::sync::Mutex;

#[derive(Default)]
pub struct InMemoryUserRepository {
    users: Mutex<HashMap<String, User>>,
}

impl UserRepositoryPort for InMemoryUserRepository {
    fn save(&self, user: &User) {
        let mut users = self.users.lock().unwrap();
        users.insert(user.id.clone(), user.clone());
    }

    fn find_by_id(&self, id: &str) -> Option<User> {
        let users = self.users.lock().unwrap();
        users.get(id).cloned()
    }
}
```

**Driving Adapter (CLI)**

```rust
// src/infrastructure/cli_adapter.rs
use crate::application::ports::UserServicePort;

pub struct CliAdapter<S: UserServicePort> {
    user_service: S,
}

impl<S: UserServicePort> CliAdapter<S> {
    pub fn new(user_service: S) -> Self {
        Self { user_service }
    }

    pub fn run(&self) {
        let user = self.user_service.create_user("Alice");
        println!("Created user: {:?}", user);

        let found_user = self.user_service.find_user(&user.id);
        println!("Found user: {:?}", found_user);
    }
}
```

#### 5. Wiring it all together (`src/main.rs`)

```rust
// src/main.rs
mod application;
mod domain;
mod infrastructure;

use application::user_service::UserService;
use infrastructure::{
    cli_adapter::CliAdapter,
    in_memory_user_repository::InMemoryUserRepository,
};

fn main() {
    // Create the driven adapter (repository)
    let user_repo = InMemoryUserRepository::default();

    // Create the application core (service), injecting the repository
    let user_service = UserService::new(user_repo);

    // Create the driving adapter (CLI), injecting the service
    let cli_adapter = CliAdapter::new(user_service);

    // Run the application
    cli_adapter.run();
}
```

And the `Cargo.toml`:

```toml
[dependencies]
uuid = { version = "1.0", features = ["v4"] }
```

### Conclusion

Hexagonal Architecture provides a robust way to structure applications, separating the core logic from the outside world. This separation makes the application easier to test, maintain, and evolve over time. Rust's trait system provides a natural and powerful way to define the ports, making it a great language for implementing this architectural pattern.