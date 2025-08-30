## Domain-Driven Design (DDD) Tactical Patterns: Aggregates

### Definition

In Domain-Driven Design (DDD), an **Aggregate** is a cluster of associated Entities and Value Objects that are treated as a single unit for data changes. It defines a consistency boundary around a group of related objects. Each Aggregate has a single **Aggregate Root**, which is an Entity that serves as the primary entry point for all operations on the Aggregate.

### Purpose and Importance

*   **Consistency Boundary:** An Aggregate ensures that all invariants (business rules that must always be true) within its boundary are maintained.
*   **Transactional Unit:** All changes within an Aggregate are committed as a single transaction.
*   **Simplifies Relationships:** External objects only hold references to Aggregate Roots.

### Rules for Designing Aggregates

1.  **Protect business invariants within the Aggregate boundary.**
2.  **Design small Aggregates.**
3.  **Reference other Aggregates by identity only.**
4.  **Update other Aggregates using eventual consistency.**

### Rust Example: `Order` Aggregate

Let's model an `Order` Aggregate. The `Order` entity will be the Aggregate Root. It will contain `OrderItems` (Entities) and a `ShippingAddress` (Value Object).

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
```

### Aggregates and Repositories

Repositories are used to persist and retrieve Aggregates. A repository should only work with Aggregate Roots. It should not be possible to retrieve an Entity or Value Object from the repository directly.

### Conclusion

Aggregates are a cornerstone of tactical DDD, providing a powerful mechanism for encapsulating complex business rules and ensuring data consistency within a defined boundary. By channeling all modifications through a single Aggregate Root, they simplify transaction management, reduce coupling, and make the domain model more robust and easier to reason about.