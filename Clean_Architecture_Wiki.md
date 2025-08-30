## Clean Architecture

### Definition

**Clean Architecture** is a software architectural pattern proposed by Robert C. Martin (Uncle Bob) that aims to create systems that are independent of frameworks, databases, UI, and any external agencies. The core idea is to organize the code into concentric circles, with the innermost circles representing the highest-level policies (business rules) and the outermost circles representing the lowest-level details (frameworks, databases, UI). The fundamental rule is the **Dependency Rule**: dependencies can only point inwards.

### The Concentric Circles

1.  **Entities:** Core business objects.
2.  **Use Cases (Interactors):** Application-specific business rules.
3.  **Interface Adapters:** Convert data between the format of the use cases and the format of external agencies (e.g., web, database).
4.  **Frameworks & Drivers:** The outermost layer, containing frameworks and tools like the web framework and database.

### The Dependency Rule

Source code dependencies can only point inwards. Inner circles must not know anything about outer circles.

### Rust Example: A Simple User Service

Let's build a simple user service in Rust using Clean Architecture. The project structure will be:

```
src/
├── domain/
│   └── user.rs         // Entity
├── application/
│   ├── use_cases/
│   │   └── create_user.rs // Use Case
│   └── ports.rs        // Ports (traits)
├── infrastructure/
│   ├── web/
│   │   └── user_controller.rs // Interface Adapter (Controller)
│   └── persistence/
│       └── in_memory_user_repository.rs // Interface Adapter (Repository)
└── main.rs           // Application entry point (wiring)
```

#### 1. Domain Layer (`src/domain/user.rs`)

```rust
// src/domain/user.rs
#[derive(Debug, Clone)]
pub struct User {
    pub id: String,
    pub name: String,
}
```

#### 2. Application Layer (`src/application/`)

**Ports (`src/application/ports.rs`)**

```rust
// src/application/ports.rs
use crate::domain::user::User;

// Input Port
pub trait CreateUserInputPort {
    fn create_user(&self, name: &str) -> User;
}

// Output Port
pub trait UserRepositoryOutputPort {
    fn save(&self, user: &User);
}
```

**Use Case (`src/application/use_cases/create_user.rs`)**

```rust
// src/application/use_cases/create_user.rs
use crate::application::ports::{CreateUserInputPort, UserRepositoryOutputPort};
use crate::domain::user::User;
use uuid::Uuid;

pub struct CreateUserUseCase<R: UserRepositoryOutputPort> {
    user_repository: R,
}

impl<R: UserRepositoryOutputPort> CreateUserUseCase<R> {
    pub fn new(user_repository: R) -> Self {
        Self { user_repository }
    }
}

impl<R: UserRepositoryOutputPort> CreateUserInputPort for CreateUserUseCase<R> {
    fn create_user(&self, name: &str) -> User {
        let user = User {
            id: Uuid::new_v4().to_string(),
            name: name.to_string(),
        };
        self.user_repository.save(&user);
        user
    }
}
```

#### 3. Infrastructure Layer (`src/infrastructure/`)

**Persistence Adapter (`src/infrastructure/persistence/in_memory_user_repository.rs`)**

```rust
// src/infrastructure/persistence/in_memory_user_repository.rs
use crate::application::ports::UserRepositoryOutputPort;
use crate::domain::user::User;
use std::collections::HashMap;
use std::sync::Mutex;

#[derive(Default)]
pub struct InMemoryUserRepository {
    users: Mutex<HashMap<String, User>>,
}

impl UserRepositoryOutputPort for InMemoryUserRepository {
    fn save(&self, user: &User) {
        let mut users = self.users.lock().unwrap();
        users.insert(user.id.clone(), user.clone());
        println!("User saved: {:?}", user);
    }
}
```

**Web Adapter (`src/infrastructure/web/user_controller.rs`)**

```rust
// src/infrastructure/web/user_controller.rs
use crate::application::ports::CreateUserInputPort;

// This is a simplified controller for demonstration.
// In a real web framework, this would handle HTTP requests.
pub struct UserController<U: CreateUserInputPort> {
    create_user_use_case: U,
}

impl<U: CreateUserInputPort> UserController<U> {
    pub fn new(create_user_use_case: U) -> Self {
        Self { create_user_use_case }
    }

    pub fn create_user(&self, name: &str) {
        let user = self.create_user_use_case.create_user(name);
        println!("Controller created user: {:?}", user);
    }
}
```

#### 4. Wiring it all together (`src/main.rs`)

```rust
// src/main.rs
mod application;
mod domain;
mod infrastructure;

use application::use_cases::create_user::CreateUserUseCase;
use infrastructure::{
    persistence::in_memory_user_repository::InMemoryUserRepository,
    web::user_controller::UserController,
};

fn main() {
    // Create the repository (outer layer)
    let user_repo = InMemoryUserRepository::default();

    // Create the use case (application layer), injecting the repository
    let create_user_use_case = CreateUserUseCase::new(user_repo);

    // Create the controller (infrastructure layer), injecting the use case
    let user_controller = UserController::new(create_user_use_case);

    // Simulate a web request
    user_controller.create_user("Alice");
}
```

And the `Cargo.toml`:

```toml
[dependencies]
uuid = { version = "1.0", features = ["v4"] }
```

### Conclusion

Clean Architecture provides a robust framework for building complex, long-lived applications that are resilient to change and easy to maintain. By enforcing the Dependency Rule, it ensures that the core business logic remains isolated and testable, independent of any external concerns.