## Test Doubles (Mocks, Stubs, Fakes, Spies, Dummies)

### Definition

**Test Doubles** are generic terms for objects that are used in place of real objects for testing purposes. When testing a specific component (the "System Under Test" or SUT), it often has dependencies on other components (e.g., a database, an external API, a complex service). To isolate the SUT and make tests faster, more reliable, and repeatable, these dependencies are replaced with test doubles.

The term "test double" comes from the idea of a "stunt double" in movies – someone who stands in for the real actor in dangerous or complex scenes.

### Why Use Test Doubles?

*   **Isolation:** Allows you to test the SUT in isolation, ensuring that test failures are due to issues in the SUT itself, not its dependencies.
*   **Speed:** Replaces slow dependencies (like database calls or network requests) with fast in-memory implementations.
*   **Repeatability:** Ensures tests produce consistent results, regardless of the state of external systems.
*   **Control:** Allows you to simulate specific scenarios (e.g., error conditions, specific return values) that might be difficult to achieve with real dependencies.
*   **Testing Unimplemented Features:** Can test code that interacts with components that are not yet built.

### Types of Test Doubles

The xUnit Test Patterns by Gerard Meszaros define several specific types of test doubles, each with a distinct purpose:

1.  **Dummy Objects:**
    *   **Definition:** Objects passed around but never actually used. They are typically used to fill parameter lists.
    *   **Purpose:** To satisfy the compiler or method signature requirements.
    *   **Example:** Passing `None` or an empty struct instance when a function requires an argument but its value doesn't affect the test.

2.  **Test Stubs:**
    *   **Definition:** Objects that provide canned answers to calls made during the test, without any complex logic. They don't respond to anything outside of what's programmed for the test.
    *   **Purpose:** To control the SUT's dependencies by providing specific, predefined return values.
    *   **Example:** A stub for a `UserRepository` that always returns a specific `User` object when `find_by_id` is called with a certain ID.

3.  **Test Spies:**
    *   **Definition:** Stubs that also record some information about how they were called (e.g., how many times a method was called, what arguments were passed).
    *   **Purpose:** To verify interactions with the dependency.
    *   **Example:** A spy for an `EmailService` that records whether `send_email` was called and with what arguments.

4.  **Mock Objects:**
    *   **Definition:** Objects that pre-programmed with expectations about the calls they are expected to receive during the test. If these expectations are not met, the test fails. Mocks are often used to verify interactions.
    *   **Purpose:** To verify behavior and interactions. Mocks are typically used when you want to assert that a specific method on a dependency was called with certain arguments.
    *   **Example:** A mock for a `PaymentGateway` that expects `process_payment` to be called exactly once with a specific amount.

5.  **Fakes:**
    *   **Definition:** Objects that have working implementations, but usually simplified or optimized for testing. They are a lightweight version of the real thing.
    *   **Purpose:** To provide a functional, but simplified, replacement for a dependency.
    *   **Example:** An in-memory database implementation that mimics a real database but stores data in a `HashMap`.

### Rust in Test Doubles

Rust's trait system and its emphasis on explicit dependencies make it well-suited for creating test doubles. While Rust doesn't have built-in mocking frameworks like some other languages, crates like `mockall` provide powerful capabilities.

*   **Traits as Interfaces:** Traits are perfect for defining the contracts that test doubles will implement.
*   **Dependency Injection:** Passing dependencies as trait objects (`Box<dyn Trait>`) or generic parameters (`<T: Trait>`) makes it easy to inject test doubles.
*   **`mockall` crate:** A popular crate for generating mock objects from traits, allowing you to define expectations and verify calls.
*   **Manual Fakes/Stubs:** Rust's strong type system and clear syntax make it relatively straightforward to write simple fakes and stubs manually.

#### Rust Example: Using `mockall` for Mocks and Stubs

Let's test a `UserService` that depends on a `UserRepository`.

**1. Setup `Cargo.toml`:**

```toml
[package]
name = "rust_test_doubles_example"
version = "0.1.0"
edition = "2021"

[dependencies]
# No runtime dependencies for this example

[dev-dependencies]
mockall = "0.12" # For mocking
```

**2. Define the Trait and Domain Object (`src/lib.rs`)**

```rust
// src/lib.rs
// Our domain object
#[derive(Debug, PartialEq, Clone)]
pub struct User {
    pub id: String,
    pub name: String,
    pub email: String,
}

// The trait that defines the contract for our repository
pub trait UserRepository {
    fn find_by_id(&self, id: &str) -> Option<User>;
    fn save(&self, user: User) -> Result<(), String>;
}

// The service we want to test (System Under Test - SUT)
pub struct UserService<R: UserRepository> {
    repository: R,
}

impl<R: UserRepository> UserService<R> {
    pub fn new(repository: R) -> Self {
        UserService { repository }
    }

    pub fn get_user_profile(&self, user_id: &str) -> Option<User> {
        self.repository.find_by_id(user_id)
    }

    pub fn update_user_email(&self, user_id: &str, new_email: String) -> Result<(), String> {
        let mut user = self.repository.find_by_id(user_id)
            .ok_or_else(|| "User not found".to_string())?;

        user.email = new_email;
        self.repository.save(user)
    }
}
```

**3. Write Tests with Mocks/Stubs (`src/lib.rs` - within `#[cfg(test)] mod tests`)**

```rust
// src/lib.rs (continued)
#[cfg(test)]
mod tests {
    use super::*;
    use mockall::{automock, predicate::*}; // Import mockall macros and predicates

    // Use automock to generate a mock implementation of UserRepository
    #[automock]
    impl UserRepository for MockUserRepository {
        // Mockall will generate the methods based on the trait definition
        // We can then define expectations for these methods in our tests
    }

    #[test]
    fn get_user_profile_returns_user_if_found() {
        // Create a mock repository
        let mut mock_repo = MockUserRepository::new();

        // Program the mock to return a specific user when find_by_id is called
        mock_repo.expect_find_by_id()
            .with(eq("user123")) // Expect call with "user123"
            .times(1) // Expect it to be called once
            .returning(|_| Some(User {
                id: "user123".to_string(),
                name: "Alice".to_string(),
                email: "alice@example.com".to_string(),
            }));

        // Create the service with the mock repository
        let user_service = UserService::new(mock_repo);

        // Call the method under test
        let user = user_service.get_user_profile("user123");

        // Assert the result
        assert!(user.is_some());
        assert_eq!(user.unwrap().name, "Alice");
    }

    #[test]
    fn get_user_profile_returns_none_if_not_found() {
        let mut mock_repo = MockUserRepository::new();
        mock_repo.expect_find_by_id()
            .with(eq("nonexistent"))
            .times(1)
            .returning(|_| None); // Return None for this ID

        let user_service = UserService::new(mock_repo);
        let user = user_service.get_user_profile("nonexistent");
        assert!(user.is_none());
    }

    #[test]
    fn update_user_email_succeeds_and_saves() {
        let mut mock_repo = MockUserRepository::new();
        let initial_user = User {
            id: "user123".to_string(),
            name: "Alice".to_string(),
            email: "alice@example.com".to_string(),
        };

        // Stub: find_by_id returns the initial user
        mock_repo.expect_find_by_id()
            .with(eq("user123"))
            .times(1)
            .returning(move |_| Some(initial_user.clone()));

        // Mock: expect save to be called with the updated user
        mock_repo.expect_save()
            .with(eq(User {
                id: "user123".to_string(),
                name: "Alice".to_string(),
                email: "new_alice@example.com".to_string(),
            }))
            .times(1)
            .returning(|_| Ok(()));

        let user_service = UserService::new(mock_repo);
        let result = user_service.update_user_email("user123", "new_alice@example.com".to_string());
        assert!(result.is_ok());
    }

    #[test]
    fn update_user_email_fails_if_user_not_found() {
        let mut mock_repo = MockUserRepository::new();
        mock_repo.expect_find_by_id()
            .with(eq("nonexistent"))
            .times(1)
            .returning(|_| None);
        // No expectation for `save` because it shouldn't be called

        let user_service = UserService::new(mock_repo);
        let result = user_service.update_user_email("nonexistent", "any@example.com".to_string());
        assert!(result.is_err());
        assert_eq!(result.unwrap_err(), "User not found");
    }
}
```
To run: `cargo test`.

### Conclusion

Test Doubles are indispensable tools for writing effective unit and integration tests, especially in complex applications with many dependencies. By allowing developers to isolate the System Under Test, control its environment, and verify interactions, test doubles contribute to faster, more reliable, and more maintainable test suites. Rust's strong type system, trait-based design, and powerful mocking libraries like `mockall` provide excellent support for implementing various types of test doubles, enabling robust and comprehensive testing strategies.