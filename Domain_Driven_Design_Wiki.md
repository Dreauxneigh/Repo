## Domain-Driven Design (DDD)

### Definition

**Domain-Driven Design (DDD)** is an approach to software development that centers the development process around a rich understanding of the business domain. It emphasizes creating a shared **Ubiquitous Language** between domain experts and developers, and then translating that understanding into a software model.

### Core Concepts

*   **Ubiquitous Language:** A common language used by all team members to discuss the project.
*   **Domain Model:** An object-oriented model of the business domain.
*   **Entities:** Objects defined by their identity, not their attributes.
*   **Value Objects:** Objects defined by their attributes, not by a unique identity.
*   **Aggregates:** A cluster of associated objects that are treated as a single unit for data changes.
*   **Repositories:** Provide methods for retrieving and persisting aggregates.
*   **Domain Services:** For operations that don't naturally belong to a single entity or value object.
*   **Bounded Contexts:** A logical boundary within which a particular domain model is defined and applicable.
*   **Context Mapping:** Defines the relationships and communication patterns between different bounded contexts.

### When to Use DDD

DDD is most beneficial for large, complex projects with a rich business domain. For smaller, simpler projects, it may be overkill.

### Rust Example: A Simple Order Aggregate

```rust
// The Aggregate Root
#[derive(Clone)]
pub struct Order {
    pub id: String,
    pub items: Vec<OrderItem>,
    pub status: OrderStatus,
}

#[derive(Clone)]
pub struct OrderItem {
    pub product_id: String,
    pub quantity: u32,
}

#[derive(Clone, PartialEq)]
pub enum OrderStatus {
    Pending,
    Confirmed,
    Shipped,
}

impl Order {
    pub fn new(id: String) -> Self {
        Self { id, items: Vec::new(), status: OrderStatus::Pending }
    }

    pub fn add_item(&mut self, product_id: String, quantity: u32) {
        self.items.push(OrderItem { product_id, quantity });
    }

    pub fn confirm(&mut self) -> Result<(), &'static str> {
        if self.status == OrderStatus::Pending {
            self.status = OrderStatus::Confirmed;
            Ok(())
        } else {
            Err("Only pending orders can be confirmed")
        }
    }
}

// The Repository trait
pub trait OrderRepository {
    fn save(&self, order: &Order);
    fn find_by_id(&self, id: &str) -> Option<Order>;
}

// A concrete implementation of the repository
use std::collections::HashMap;
use std::sync::Mutex;

#[derive(Default)]
pub struct InMemoryOrderRepository {
    orders: Mutex<HashMap<String, Order>>,
}

impl OrderRepository for InMemoryOrderRepository {
    fn save(&self, order: &Order) {
        let mut orders = self.orders.lock().unwrap();
        orders.insert(order.id.clone(), order.clone());
    }

    fn find_by_id(&self, id: &str) -> Option<Order> {
        let orders = self.orders.lock().unwrap();
        orders.get(id).cloned()
    }
}

fn main() {
    let repo = InMemoryOrderRepository::default();
    let mut order = Order::new("123".to_string());
    order.add_item("p1".to_string(), 2);
    repo.save(&order);

    let mut retrieved_order = repo.find_by_id("123").unwrap();
    retrieved_order.confirm().unwrap();
    repo.save(&retrieved_order);

    let final_order = repo.find_by_id("123").unwrap();
    assert_eq!(final_order.status, OrderStatus::Confirmed);
}
```

### Common DDD Pitfalls

*   **Anemic Domain Model:** The domain objects have no business logic, only getters and setters.
*   **Leaky Abstractions:** The infrastructure details leak into the domain model.
*   **Big Ball of Mud:** The boundaries between bounded contexts are not well-defined.

### Conclusion

DDD is a powerful paradigm for tackling complex business domains, leading to software that is not only functional but also truly reflects the intricacies of the business it serves. It is often implemented in conjunction with architectural patterns like Hexagonal Architecture or Clean Architecture.