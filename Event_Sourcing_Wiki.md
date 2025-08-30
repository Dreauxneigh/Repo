## Event Sourcing

### Definition

**Event Sourcing** is an architectural pattern where the complete state of an application is not stored directly, but rather derived from a sequence of immutable **Domain Events**. Instead of storing the current state of an entity, every change to the entity's state is captured as an event and appended to an event log (the **Event Store**).

### Key Concepts

*   **Domain Events:** Immutable facts about what *has happened* in the business domain.
*   **Event Store:** An append-only log where all Domain Events are stored.
*   **Aggregates:** Responsible for handling commands and producing new Domain Events.
*   **Projections:** Components that consume Domain Events and transform them into read-optimized representations (Query Models).
*   **Snapshots:** Periodically save the current state of an Aggregate to speed up reconstruction.

### Event Store

The Event Store is the heart of an Event Sourcing system. It can be implemented using different technologies:

*   **Relational Databases:** Simple to set up, but can be slow for large event streams.
*   **NoSQL Databases:** Good for storing large volumes of data, but may not have the transactional guarantees of a relational database.
*   **Dedicated Event Stores:** Specialized databases designed for Event Sourcing (e.g., EventStoreDB, Axon Server).

### Projections

Projections are used to create read models from the event stream. A projection is a component that listens for events and updates a read model in response. The read model can be a simple key-value store, a relational database, or a search index.

### Event Schema Evolution

As an application evolves, the schema of the events may need to change. This can be a challenging problem in Event Sourcing. Some common strategies for handling schema evolution include:

*   **Upcasting:** Converting old events to the new schema on the fly when they are read from the event store.
*   **Lazy Transformation:** Transforming old events to the new schema only when they are needed.
*   **Copy and Transform:** Creating a new event store with the new schema and migrating the data from the old store.

### Rust Example: Simple Event Sourcing with In-Memory Event Store

Let's illustrate Event Sourcing with a basic `BankAccount` aggregate.

**1. Domain Events**

```rust
#[derive(Clone)]
pub enum BankAccountEvent {
    AccountOpened { initial_balance: f64 },
    FundsDeposited { amount: f64 },
    FundsWithdrawn { amount: f64 },
}
```

**2. The `BankAccount` Aggregate**

```rust
#[derive(Default)]
pub struct BankAccount {
    pub balance: f64,
}

impl BankAccount {
    pub fn apply(&mut self, event: &BankAccountEvent) {
        match event {
            BankAccountEvent::AccountOpened { initial_balance } => self.balance = *initial_balance,
            BankAccountEvent::FundsDeposited { amount } => self.balance += *amount,
            BankAccountEvent::FundsWithdrawn { amount } => self.balance -= *amount,
        }
    }

    pub fn handle_command(&self, command: &BankAccountCommand) -> Result<Vec<BankAccountEvent>, &'static str> {
        match command {
            BankAccountCommand::OpenAccount { initial_balance } => Ok(vec![BankAccountEvent::AccountOpened { initial_balance: *initial_balance }]),
            BankAccountCommand::DepositFunds { amount } => Ok(vec![BankAccountEvent::FundsDeposited { amount: *amount }]),
            BankAccountCommand::WithdrawFunds { amount } => {
                if self.balance >= *amount {
                    Ok(vec![BankAccountEvent::FundsWithdrawn { amount: *amount }])
                } else {
                    Err("Insufficient funds")
                }
            }
        }
    }
}
```

**3. Commands**

```rust
pub enum BankAccountCommand {
    OpenAccount { initial_balance: f64 },
    DepositFunds { amount: f64 },
    WithdrawFunds { amount: f64 },
}
```

**4. The Event Store**

```rust
pub struct EventStore {
    events: Vec<BankAccountEvent>,
}

impl EventStore {
    pub fn new() -> Self {
        Self { events: Vec::new() }
    }

    pub fn append(&mut self, events: &[BankAccountEvent]) {
        self.events.extend_from_slice(events);
    }

    pub fn get_events(&self) -> &[BankAccountEvent] {
        &self.events
    }
}
```

### Conclusion

Event Sourcing is a powerful architectural pattern that offers significant benefits in terms of auditability, resilience, and flexibility. By treating events as the primary source of truth, it enables a deeper understanding of business processes and provides unparalleled capabilities for historical analysis and system evolution. However, it also introduces complexity, so it should be used judiciously.