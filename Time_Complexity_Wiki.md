## Time Complexity

### Definition

**Time Complexity** is a measure of the amount of time an algorithm takes to run as a function of the length of its input. It describes how the running time of an algorithm grows as the input size (`n`) increases. Time complexity is typically expressed using **Big O Notation**, which provides an upper bound on the growth rate of the algorithm's execution time in the worst-case scenario.

It's important to note that time complexity doesn't measure the actual execution time in seconds or milliseconds, but rather the number of elementary operations an algorithm performs.

### Why is Time Complexity Important?

*   **Performance Prediction:** Helps predict how an algorithm will perform with larger inputs, which is crucial for designing scalable systems.
*   **Algorithm Comparison:** Allows for objective comparison of different algorithms for the same problem, independent of hardware or programming language.
*   **Bottleneck Identification:** Highlights parts of the code that will become performance bottlenecks as the input size grows.
*   **Optimization Guidance:** Guides developers on where to focus optimization efforts.
*   **Resource Management:** Helps in understanding the computational resources required by an algorithm.

### Common Time Complexities (from fastest to slowest)

1.  **O(1) - Constant Time:**
    *   Execution time is constant, regardless of input size.
    *   *Example:* Accessing an array element by index.

2.  **O(log n) - Logarithmic Time:**
    *   Execution time grows logarithmically with input size. Typically involves dividing the problem in half repeatedly.
    *   *Example:* Binary search.

3.  **O(n) - Linear Time:**
    *   Execution time grows linearly with input size.
    *   *Example:* Iterating through all elements in a list, linear search.

4.  **O(n log n) - Linearithmic Time:**
    *   Execution time grows proportionally to `n` multiplied by the logarithm of `n`.
    *   *Example:* Efficient sorting algorithms (Merge Sort, Quick Sort).

5.  **O(n^2) - Quadratic Time:**
    *   Execution time grows proportionally to the square of the input size. Often involves nested loops.
    *   *Example:* Simple sorting algorithms (Bubble Sort, Insertion Sort).

6.  **O(2^n) - Exponential Time:**
    *   Execution time doubles with each addition to the input size.
    *   *Example:* Some recursive algorithms without memoization, brute-force solutions to NP-hard problems.

7.  **O(n!) - Factorial Time:**
    *   Execution time grows proportionally to the factorial of the input size. Extremely inefficient for even small `n`.
    *   *Example:* Brute-force solution to the Traveling Salesperson Problem.

### Factors Affecting Time Complexity

*   **Input Size (`n`):** The primary factor.
*   **Number of Operations:** Basic operations like arithmetic, comparisons, assignments.
*   **Control Flow:** Loops, conditionals, function calls.
*   **Data Structure Operations:** The efficiency of operations on underlying data structures (e.g., `HashMap` lookup is O(1) average, O(n) worst-case).

### Rust and Time Complexity

Rust's focus on performance and explicit control makes understanding and optimizing for time complexity very important.

*   **Predictable Performance:** Rust's lack of a garbage collector and explicit memory management lead to more predictable execution times, making Big O analysis more directly applicable to real-world performance.
*   **Standard Library Guarantees:** The Rust standard library documentation often specifies the Big O complexity for operations on its data structures (e.g., `Vec::push` is O(1) amortized, `HashMap::get` is O(1) average).
*   **Zero-Cost Abstractions:** Rust's abstractions (like iterators) are designed to compile down to highly efficient machine code, often matching the performance of hand-written loops, thus preserving the theoretical time complexity.
*   **Concurrency and Parallelism:** Rust's safe concurrency features allow developers to implement algorithms that leverage multiple cores, potentially reducing wall-clock time, but the Big O analysis still applies to the total work done.

#### Rust Example: Analyzing Time Complexity of Sorting Algorithms

Let's compare the time complexity of two sorting algorithms: Bubble Sort (O(n^2)) and a more efficient sort (like `Vec::sort` which is O(n log n)).

**1. Setup `Cargo.toml`:**

```toml
[package]
name = "rust_time_complexity_example"
version = "0.1.0"
edition = "2021"

[dependencies]
rand = "0.8" # For generating random numbers
```

**2. Example Code (`src/main.rs`)**

```rust
// src/main.rs
use rand::Rng;
use std::time::Instant;

// O(n^2) - Bubble Sort
fn bubble_sort<T: Ord>(arr: &mut [T]) {
    let n = arr.len();
    for i in 0..n {
        for j in 0..n - 1 - i {
            if arr[j] > arr[j + 1] {
                arr.swap(j, j + 1);
            }
        }
    }
}

// O(n log n) - Rust's built-in sort (Timsort/MergeSort hybrid)
fn rust_sort<T: Ord>(arr: &mut [T]) {
    arr.sort();
}

fn main() {
    let sizes = vec![100, 1000, 5000]; // Different input sizes

    for &size in &sizes {
        let mut rng = rand::thread_rng();
        let mut numbers_bubble: Vec<u32> = (0..size).map(|_| rng.gen()).collect();
        let mut numbers_rust = numbers_bubble.clone(); // Same data for fair comparison

        println!("\n--- Sorting {} elements ---", size);

        // Measure Bubble Sort
        let start = Instant::now();
        bubble_sort(&mut numbers_bubble);
        let duration = start.elapsed();
        println!("Bubble Sort (O(n^2)): {:?}", duration);

        // Measure Rust's built-in sort
        let start = Instant::now();
        rust_sort(&mut numbers_rust);
        let duration = start.elapsed();
        println!("Rust Sort (O(n log n)): {:?}", duration);

        // Verify correctness (optional)
        // assert_eq!(numbers_bubble, numbers_rust);
    }
}
```
To run: `cargo run --release` (use `--release` for accurate performance measurements).
You will observe that as `n` increases, the time taken by `bubble_sort` grows much faster than `rust_sort`, demonstrating their respective time complexities.

### Conclusion

Time Complexity is a fundamental concept in algorithm analysis, providing a crucial understanding of how an algorithm's execution time scales with its input size. Expressed using Big O Notation, it guides developers in selecting efficient algorithms, identifying performance bottlenecks, and designing scalable software systems. Rust's focus on predictable performance and its explicit standard library guarantees make time complexity analysis a highly relevant and actionable tool for Rust developers aiming to build high-performance and efficient applications.
