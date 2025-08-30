## Onion Architecture

### Definition

**Onion Architecture**, introduced by Jeffrey Palermo, is an architectural pattern that emphasizes the separation of concerns by organizing code into concentric layers. Its primary goal is to ensure that the application's core business logic (the "Domain Model") remains independent of infrastructure concerns. The fundamental rule is the **Dependency Rule**: dependencies can only point inwards.

### The Layers of the Onion

1.  **Domain Model:** The heart of the application, containing business entities and value objects.
2.  **Domain Services:** Business logic that doesn't naturally fit within an entity.
3.  **Application Services:** Orchestrate the execution of use cases and define interfaces for the infrastructure layer.
4.  **Infrastructure:** The outermost layer, containing concrete implementations of the interfaces defined in the Application Services layer (e.g., databases, UI frameworks).

### Data Transfer Objects (DTOs)

DTOs are used to transfer data between layers without leaking domain objects to the outer layers. This helps to maintain the separation of concerns and prevent the outer layers from depending on the internal representation of the domain objects.

### Rust Example: A Simple User Service

Let's build a simple user service in Rust using the Onion Architecture.

#### 1. Domain Layer

```rust
// src/domain/user.rs
#[derive(Clone)]
pub struct User {
    pub id: String,
    pub name: String,
}
```

#### 2. Application Layer

```rust
// src/application/user_service.rs
use crate::domain::user::User;

// Input Port
pub trait UserUseCase {
    fn create_user(&self, name: &str) -> User;
}

// Output Port
pub trait UserRepository {
    fn save(&self, user: &User);
}

pub struct UserServiceImpl<R: UserRepository> {
    user_repository: R,
}

impl<R: UserRepository> UserServiceImpl<R> {
    pub fn new(user_repository: R) -> Self {
        Self { user_repository }
    }
}

impl<R: UserRepository> UserUseCase for UserServiceImpl<R> {
    fn create_user(&self, name: &str) -> User {
        let user = User { id: uuid::Uuid::new_v4().to_string(), name: name.to_string() };
        self.user_repository.save(&user);
        user
    }
}
```

#### 3. Infrastructure Layer

```rust
// src/infrastructure/in_memory_user_repository.rs
use crate::application::user_service::UserRepository;
use crate::domain::user::User;
use std::collections::HashMap;
use std::sync::Mutex;

#[derive(Default)]
pub struct InMemoryUserRepository {
    users: Mutex<HashMap<String, User>>,
}

impl UserRepository for InMemoryUserRepository {
    fn save(&self, user: &User) {
        let mut users = self.users.lock().unwrap();
        users.insert(user.id.clone(), user.clone());
    }
}
```

#### 4. Wiring it all together

```rust
// src/main.rs
mod application;
mod domain;
mod infrastructure;

use application::user_service::{UserServiceImpl, UserUseCase};
use infrastructure::in_memory_user_repository::InMemoryUserRepository;

fn main() {
    let user_repo = InMemoryUserRepository::default();
    let user_service = UserServiceImpl::new(user_repo);

    let user = user_service.create_user("Alice");
    println!("Created user: {:?}", user.name);
}
```

### Testing

Each layer of the Onion Architecture can be tested in isolation. For example, you can test the Application Services layer by mocking the UserRepository.

### Conclusion

Onion Architecture provides a clear and effective way to structure applications, ensuring that business rules remain at the heart of the system, protected from the volatility of external technologies. It is very similar to Hexagonal and Clean Architecture, and the choice between them often comes down to personal preference and team conventions.