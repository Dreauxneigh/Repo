## Domain-Driven Design (DDD) Tactical Patterns: Domain Services

### Definition

In Domain-Driven Design (DDD), a **Domain Service** is a tactical pattern used to model operations that do not naturally fit within the responsibility of an Entity or a Value Object. These are typically stateless operations that involve multiple Aggregates or perform a significant business process.

### Purpose and Importance

*   **Orchestrating Complex Business Processes:** When a business operation involves multiple domain objects.
*   **Maintaining Domain Purity:** Prevents Entities and Value Objects from becoming bloated.
*   **Encapsulating Cross-Aggregate Logic:** Handles interactions between different Aggregates.

### Domain Services vs. Application Services

*   **Domain Services:** Contain core business logic that is part of the domain. They are expressed in the Ubiquitous Language.
*   **Application Services:** Orchestrate the execution of use cases. They are the clients of the domain model and do not contain business logic.

### Rust Example: `TransferService` for Bank Accounts

Let's model a `TransferService` that handles transferring money between two `BankAccount` Aggregates.

#### 1. The `BankAccount` Aggregate

```rust
// src/domain/bank_account.rs
#[derive(Clone)]
pub struct BankAccount {
    pub id: String,
    pub balance: f64,
}

impl BankAccount {
    pub fn new(id: String, balance: f64) -> Self {
        Self { id, balance }
    }

    pub fn deposit(&mut self, amount: f64) {
        self.balance += amount;
    }

    pub fn withdraw(&mut self, amount: f64) -> Result<(), &'static str> {
        if self.balance >= amount {
            self.balance -= amount;
            Ok(())
        } else {
            Err("Insufficient funds")
        }
    }
}
```

#### 2. The `AccountRepository` Trait

```rust
// src/domain/account_repository.rs
use crate::domain::bank_account::BankAccount;

pub trait AccountRepository {
    fn save(&self, account: &BankAccount);
    fn find_by_id(&self, id: &str) -> Option<BankAccount>;
}
```

#### 3. The `TransferService` Domain Service

```rust
// src/domain/transfer_service.rs
use crate::domain::account_repository::AccountRepository;

pub struct TransferService<'a, R: AccountRepository> {
    repo: &'a R,
}

impl<'a, R: AccountRepository> TransferService<'a, R> {
    pub fn new(repo: &'a R) -> Self {
        Self { repo }
    }

    pub fn transfer(&self, from_id: &str, to_id: &str, amount: f64) -> Result<(), &'static str> {
        let mut from_account = self.repo.find_by_id(from_id).ok_or("From account not found")?;
        let mut to_account = self.repo.find_by_id(to_id).ok_or("To account not found")?;

        from_account.withdraw(amount)?;
        to_account.deposit(amount);

        self.repo.save(&from_account);
        self.repo.save(&to_account);

        Ok(())
    }
}
```

### Statelessness of Domain Services

Domain Services should be stateless. They should not hold any state of their own. Any state they need should be passed in as arguments to their methods. This makes them easier to test and reason about.

### Conclusion

Domain Services are a vital tactical pattern in DDD for encapsulating business logic that spans across multiple domain objects. By keeping them stateless and focused on domain-specific operations, they help maintain the purity and integrity of Entities and Value Objects, leading to a cleaner, more expressive, and more maintainable Domain Model.