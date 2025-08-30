## Domain-Driven Design (DDD) Strategic Patterns: Bounded Contexts

### Definition

In Domain-Driven Design (DDD), a **Bounded Context** is a central strategic pattern that defines a logical boundary within a large software system where a particular Domain Model is defined and applicable. It's a conceptual boundary that encapsulates a specific part of the business domain and its associated Ubiquitous Language, Domain Model, and code.

### Purpose and Importance

*   **Manages Complexity:** Breaks down a large, complex domain into smaller, more manageable subdomains.
*   **Enforces Model Consistency:** Ensures that within a Bounded Context, the Ubiquitous Language and Domain Model are consistent and unambiguous.
*   **Facilitates Team Autonomy:** Allows different teams to work on different Bounded Contexts independently.
*   **Supports Microservices:** Bounded Contexts are a natural fit for microservices architecture.

### Identifying Bounded Contexts

*   **Different Meanings of Terms:** When a word has different meanings in different parts of the business.
*   **Different Business Processes:** Distinct workflows or sets of rules.
*   **Organizational Structure:** How the business is organized into departments or teams.

### Context Mapping

Context Mapping is the process of identifying and defining the relationships between different Bounded Contexts.

*   **Shared Kernel:** Two or more teams share a common part of the model.
*   **Customer-Supplier:** One team (the supplier) provides a service to another team (the customer).
*   **Anti-Corruption Layer (ACL):** A layer of code that translates between two different Bounded Contexts, preventing the concepts of one context from leaking into the other.

### Rust Example: Sales and Fulfillment Contexts

Let's create a simple example with a `Sales` context and a `Fulfillment` context. The `Sales` context will publish an `OrderPlaced` event, and the `Fulfillment` context will listen for this event and create a shipment.

#### 1. The `Sales` Context

```rust
// sales/src/lib.rs
pub mod domain {
    pub struct Order {
        pub id: String,
        pub product_id: String,
        pub quantity: u32,
    }
}

pub mod application {
    use crate::domain::Order;

    pub trait OrderRepository {
        fn save(&self, order: &Order);
    }

    pub struct OrderService<R: OrderRepository> {
        repo: R,
    }

    impl<R: OrderRepository> OrderService<R> {
        pub fn new(repo: R) -> Self {
            Self { repo }
        }

        pub fn place_order(&self, product_id: &str, quantity: u32) -> Order {
            let order = Order { id: uuid::Uuid::new_v4().to_string(), product_id: product_id.to_string(), quantity };
            self.repo.save(&order);
            order
        }
    }
}
```

#### 2. The `Fulfillment` Context

```rust
// fulfillment/src/lib.rs
pub mod domain {
    pub struct Shipment {
        pub id: String,
        pub order_id: String,
    }
}

pub mod application {
    use crate::domain::Shipment;

    pub trait ShipmentRepository {
        fn save(&self, shipment: &Shipment);
    }

    pub struct ShipmentService<R: ShipmentRepository> {
        repo: R,
    }

    impl<R: ShipmentRepository> ShipmentService<R> {
        pub fn new(repo: R) -> Self {
            Self { repo }
        }

        pub fn create_shipment(&self, order_id: &str) -> Shipment {
            let shipment = Shipment { id: uuid::Uuid::new_v4().to_string(), order_id: order_id.to_string() };
            self.repo.save(&shipment);
            shipment
        }
    }
}
```

#### 3. The Anti-Corruption Layer (ACL)

```rust
// acl/src/lib.rs
use fulfillment::application::ShipmentService;
use sales::domain::Order;

pub struct SalesToFulfillmentAcl<S: fulfillment::application::ShipmentRepository> {
    shipment_service: ShipmentService<S>,
}

impl<S: fulfillment::application::ShipmentRepository> SalesToFulfillmentAcl<S> {
    pub fn new(shipment_service: ShipmentService<S>) -> Self {
        Self { shipment_service }
    }

    pub fn translate_and_forward(&self, order: &Order) {
        self.shipment_service.create_shipment(&order.id);
    }
}
```

### Conclusion

Bounded Contexts are a cornerstone of strategic DDD, providing a powerful mechanism for managing complexity in large software systems. By explicitly defining the boundaries of consistent Domain Models and their Ubiquitous Languages, they enable modular development, facilitate team autonomy, and lay the groundwork for architectures like microservices.