## Concurrency Primitives (Mutex, Channels, Atomics)

### Definition

**Concurrency Primitives** are fundamental building blocks provided by programming languages or libraries to manage and coordinate the execution of multiple threads or concurrent tasks. They are essential for writing safe and efficient concurrent programs, helping to prevent common issues like race conditions, deadlocks, and data corruption when multiple threads access shared resources.

### Why Use Concurrency Primitives?

*   **Safety:** Prevent data races and ensure data integrity when multiple threads access shared mutable state.
*   **Coordination:** Allow threads to communicate and synchronize their actions.
*   **Resource Management:** Control access to limited resources.
*   **Performance:** Enable parallel execution and efficient utilization of multi-core processors.
*   **Responsiveness:** Keep applications responsive by performing long-running operations in the background.

### Common Concurrency Primitives in Rust

Rust provides a rich set of concurrency primitives in its standard library and through popular crates, all designed with safety in mind, leveraging the ownership and borrowing system.

1.  **`std::thread` (Threads):**
    *   **Purpose:** The most basic unit of concurrency. Allows you to run code in parallel with the main program thread.
    *   **Usage:** `thread::spawn` to create a new thread, `handle.join()` to wait for a thread to complete.
    *   **Safety:** Rust's ownership system ensures that data moved into a thread is safely owned by that thread, preventing use-after-free.

2.  **`std::sync::Mutex<T>` (Mutual Exclusion):**
    *   **Purpose:** Provides exclusive access to shared mutable data. Only one thread can acquire the lock at a time, ensuring that the data is not corrupted by simultaneous writes.
    *   **How it works:** When a thread wants to access the protected data, it must first acquire the mutex lock. If the lock is already held by another thread, the current thread blocks until the lock is released.
    *   **Safety:** In Rust, `Mutex` returns a "lock guard" (`MutexGuard`) which automatically releases the lock when it goes out of scope, preventing common lock-release errors. The data inside the `Mutex` must be `Send` (can be moved between threads) and `Sync` (can be shared between threads).
    *   **Usage:** Often combined with `Arc<T>` (Atomic Reference Counted) to allow multiple threads to own a reference to the same `Mutex`.

3.  **`std::sync::RwLock<T>` (Read-Write Lock):**
    *   **Purpose:** Allows multiple readers or a single writer to access shared data. It's more permissive than a `Mutex` for read-heavy workloads.
    *   **How it works:** Multiple threads can acquire a read lock simultaneously. If a write lock is requested, all read locks must be released first. Only one thread can hold a write lock at a time.
    *   **Safety:** Similar to `Mutex`, returns a lock guard that automatically releases the lock.

4.  **`std::sync::mpsc::channel` (Message Passing Channels):**
    *   **Purpose:** Allows threads to communicate by sending messages to each other. This is often preferred over shared mutable state as it avoids many synchronization complexities.
    *   **How it works:** Consists of a "sender" (`Sender`) and a "receiver" (`Receiver`). Messages sent by the sender are received by the receiver.
    *   **Types:**
        *   `mpsc`: Multiple Producer, Single Consumer.
        *   `crossbeam-channel`: A popular third-party crate for more advanced channel types (e.g., multi-producer multi-consumer, bounded/unbounded).
    *   **Safety:** Rust's ownership system ensures that data sent through a channel is safely transferred, preventing data races.

5.  **`std::sync::atomic` (Atomic Types):**
    *   **Purpose:** Provides primitive types (e.g., `AtomicBool`, `AtomicUsize`, `AtomicPtr`) that can be safely accessed and modified by multiple threads without using locks. They offer very fine-grained, low-overhead synchronization.
    *   **How it works:** Operations on atomic types are guaranteed to be atomic (indivisible), meaning they complete entirely without interruption from other threads.
    *   **Use Cases:** Counters, flags, simple state changes where full mutex overhead is not desired.
    *   **Safety:** Operations are safe, but complex logic built on atomics can be hard to reason about.

6.  **`std::sync::Barrier`:**
    *   **Purpose:** Allows a group of threads to wait for each other to reach a common point before proceeding.
    *   **Use Cases:** Synchronizing phases in parallel algorithms.

7.  **`std::sync::Condvar` (Condition Variable):**
    *   **Purpose:** Used in conjunction with a `Mutex` to allow threads to wait for a certain condition to become true.
    *   **Use Cases:** Producer-consumer problems, thread pools.

### Rust Example: Using Mutex, Channels, and Atomics

**1. Setup `Cargo.toml`:**

```toml
[package]
name = "rust_concurrency_primitives_example"
version = "0.1.0"
edition = "2021"

[dependencies]
tokio = { version = "1", features = ["full"] } # For async sleep in channel example
```

**2. Example Code (`src/main.rs`)**

```rust
// src/main.rs
use std::sync::{Arc, Mutex, mpsc};
use std::thread;
use std::time::Duration;
use std::sync::atomic::{AtomicUsize, Ordering};

fn main() {
    println!("---"Mutex Example---");
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for i in 0..5 {
        let counter_clone = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter_clone.lock().unwrap(); // Acquire lock
            *num += 1;
            println!("Thread {} incremented counter to {}", i, *num);
            // Lock released when `num` goes out of scope
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }
    println!("Final Mutex counter value: {}
", *counter.lock().unwrap());


    println!("---"Channel Example---");
    let (tx, rx) = mpsc::channel();
    let mut producer_handles = vec![];

    for i in 0..3 { // Three producer threads
        let tx_clone = tx.clone();
        let handle = thread::spawn(move || {
            let message = format!("Message {} from producer {}", i, thread::current().id().as_u64());
            tx_clone.send(message).unwrap();
            thread::sleep(Duration::from_millis(50));
        });
        producer_handles.push(handle);
    }
    // Drop the original tx so the receiver knows when all senders are done
    drop(tx);

    // Consumer thread
    let consumer_handle = thread::spawn(move || {
        for received in rx {
            println!("Consumer got: {}", received);
        }
        println!("Consumer finished receiving messages.");
    });

    for handle in producer_handles {
        handle.join().unwrap();
    }
    consumer_handle.join().unwrap();
    println!("\n");


    println!("---"Atomic Example---");
    let atomic_counter = Arc::new(AtomicUsize::new(0));
    let mut atomic_handles = vec![];

    for i in 0..5 {
        let atomic_counter_clone = Arc::clone(&atomic_counter);
        let handle = thread::spawn(move || {
            // Atomically increment the counter
            let old_value = atomic_counter_clone.fetch_add(1, Ordering::SeqCst);
            println!("Atomic Thread {} incremented from {} to {}", i, old_value, old_value + 1);
        });
        atomic_handles.push(handle);
    }

    for handle in atomic_handles {
        handle.join().unwrap();
    }
    println!("Final Atomic counter value: {}
", atomic_counter.load(Ordering::SeqCst));
}
