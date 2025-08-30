## Domain-Driven Design (DDD) Tactical Patterns: Entities and Value Objects

### Definition

In Domain-Driven Design (DDD), **Entities** and **Value Objects** are the fundamental building blocks of the Domain Model.

*   **Entity:** An object that is defined by its unique identity, not its attributes. Its attributes can change over time, but its identity remains constant.
*   **Value Object:** An object that is defined by its attributes, not by a unique identity. They are immutable and are compared by their values.

### Key Differences

| Feature        | Entity                                     | Value Object                               |
|----------------|--------------------------------------------|--------------------------------------------|
| **Identity**   | Has a unique identifier                    | No unique identifier                       |
| **Equality**   | Compared by identity                       | Compared by value                          |
| **Mutability** | Mutable                                    | Immutable                                  |
| **Lifecycle**  | Has a lifecycle                            | No lifecycle                               |
| **Example**    | `User`, `Order`, `Product`                 | `Address`, `Money`, `DateRange`            |

### Choosing Between Entity and Value Object

*   **Does it have a unique identity?** If yes, it's probably an Entity.
*   **Do you care about the history of changes to the object?** If yes, it's probably an Entity.
*   **Is it a descriptive attribute of another object?** If yes, it's probably a Value Object.
*   **Can you replace it with another object with the same attributes?** If yes, it's probably a Value Object.

### Rust Example: `User` Entity and `Address` Value Object

```rust
// The Address Value Object
#[derive(Clone, PartialEq, Eq, Debug)]
pub struct Address {
    pub street: String,
    pub city: String,
    pub zip_code: String,
}

// The User Entity
#[derive(Clone, Debug)]
pub struct User {
    pub id: String,
    pub name: String,
    pub address: Address,
}

impl User {
    pub fn new(id: String, name: String, address: Address) -> Self {
        Self { id, name, address }
    }

    pub fn change_address(&mut self, new_address: Address) {
        self.address = new_address;
    }
}

// For Entities, we compare by identity
impl PartialEq for User {
    fn eq(&self, other: &Self) -> bool {
        self.id == other.id
    }
}
impl Eq for User {}

fn main() {
    let address1 = Address { street: "123 Main St".to_string(), city: "Anytown".to_string(), zip_code: "12345".to_string() };
    let address2 = Address { street: "123 Main St".to_string(), city: "Anytown".to_string(), zip_code: "12345".to_string() };
    assert_eq!(address1, address2); // Value Objects are equal if their attributes are equal

    let mut user1 = User::new("1".to_string(), "Alice".to_string(), address1);
    let user2 = User::new("2".to_string(), "Alice".to_string(), address2);
    assert_ne!(user1, user2); // Entities are not equal, even if their attributes are the same

    let new_address = Address { street: "456 Oak Ave".to_string(), city: "Otherville".to_string(), zip_code: "67890".to_string() };
    user1.change_address(new_address);
    println!("User 1 new address: {:?}", user1.address);
}
```

### Immutability of Value Objects

Value Objects should be immutable. This means that once a Value Object is created, its attributes cannot be changed. If you need to "change" a Value Object, you create a new one with the desired attributes. This makes the code easier to reason about and prevents unexpected side effects.

### Conclusion

Entities and Value Objects are fundamental building blocks in Domain-Driven Design. By correctly identifying and implementing them, developers can create a rich, expressive, and robust Domain Model that accurately reflects the business reality. Entities provide unique identity and lifecycle, while Value Objects encapsulate descriptive attributes and enforce immutability and self-validation.