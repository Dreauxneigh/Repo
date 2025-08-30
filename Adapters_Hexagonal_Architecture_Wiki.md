## Adapters (in Hexagonal Architecture)

### Definition

In Hexagonal Architecture, **Adapters** are concrete implementations of the **Ports**. They serve as the crucial intermediaries that translate between the technology-agnostic interfaces defined by the application's core business logic (the "hexagon") and the specific technologies of the external world.

### Purpose and Importance

*   **Implement Ports:** Provide concrete implementations for the abstract contracts defined by the ports.
*   **Translate Data and Calls:** Convert data formats and method calls between the domain-specific language of the application core and the technology-specific language of the external system.
*   **Isolate Technology Details:** Encapsulate all the technology-specific code, ensuring that the core business logic remains clean and pure.
*   **Enable Swappability:** Allow different technologies to be used interchangeably.

### Types of Adapters

1.  **Driving Adapters (Primary/Inbound):** These adapters drive the application core. They receive input from the outside world and call the application's driving ports.
    *   Examples: Web controllers, CLI parsers, message queue consumers.
2.  **Driven Adapters (Secondary/Outbound):** These adapters are driven by the application core. They implement the application's driven ports and interact with external systems.
    *   Examples: Database repositories, external service clients, message queue producers.

### Rust Example: A Simple User Service

Let's create a simple user service with a web adapter (driving) and an in-memory database adapter (driven).

#### 1. The Ports (the contract)

```rust
// src/ports.rs
pub trait UserUseCase {
    fn create_user(&self, name: &str) -> String;
}

pub trait UserRepository {
    fn save_user(&self, id: &str, name: &str);
}
```

#### 2. The Application Core (the hexagon)

```rust
// src/domain.rs
use crate::ports::{UserUseCase, UserRepository};

pub struct UserService<R: UserRepository> {
    user_repository: R,
}

impl<R: UserRepository> UserService<R> {
    pub fn new(user_repository: R) -> Self {
        Self { user_repository }
    }
}

impl<R: UserRepository> UserUseCase for UserService<R> {
    fn create_user(&self, name: &str) -> String {
        let id = uuid::Uuid::new_v4().to_string();
        self.user_repository.save_user(&id, name);
        id
    }
}
```

#### 3. The Adapters (the implementations)

**Driving Adapter (Web)**

```rust
// src/adapters/web.rs
use crate::ports::UserUseCase;
use std::sync::Arc;
use axum::{extract::State, routing::post, Json, Router};
use serde::Deserialize;

#[derive(Deserialize)]
pub struct CreateUserPayload {
    name: String,
}

async fn create_user<U: UserUseCase>(State(user_service): State<Arc<U>>, Json(payload): Json<CreateUserPayload>) -> String {
    user_service.create_user(&payload.name)
}

pub fn create_router<U: UserUseCase + Send + Sync + 'static>(user_service: Arc<U>) -> Router {
    Router::new()
        .route("/users", post(create_user))
        .with_state(user_service)
}
```

**Driven Adapter (In-Memory Database)**

```rust
// src/adapters/db.rs
use crate::ports::UserRepository;
use std::collections::HashMap;
use std::sync::Mutex;

#[derive(Default)]
pub struct InMemoryUserRepository {
    users: Mutex<HashMap<String, String>>,
}

impl UserRepository for InMemoryUserRepository {
    fn save_user(&self, id: &str, name: &str) {
        let mut users = self.users.lock().unwrap();
        users.insert(id.to_string(), name.to_string());
    }
}
```

#### 4. Wiring it all together

```rust
// src/main.rs
mod adapters;
mod domain;
mod ports;

use adapters::{db::InMemoryUserRepository, web::create_router};
use domain::UserService;
use std::sync::Arc;

#[tokio::main]
async fn main() {
    let user_repo = Arc::new(InMemoryUserRepository::default());
    let user_service = Arc::new(UserService::new(user_repo.clone()));
    let app = create_router(user_service);

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    axum::serve(listener, app).await.unwrap();
}
```

### Conclusion

Adapters are the practical manifestation of the Hexagonal Architecture's promise of decoupling. By encapsulating technology-specific details, they ensure that the core business logic remains pure, testable, and adaptable to changing external environments.