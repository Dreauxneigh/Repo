## Space Complexity

### Definition

**Space Complexity** is a measure of the amount of memory (or space) an algorithm needs to run as a function of the length of its input. It describes how the memory usage of an algorithm grows as the input size (`n`) increases. Like time complexity, space complexity is typically expressed using **Big O Notation**, focusing on the worst-case scenario.

Space complexity considers both the auxiliary space (temporary space used by the algorithm) and the input space (space taken by the input itself). However, often when discussing space complexity, especially in interviews, the focus is on the *auxiliary space* required by the algorithm, excluding the input size.

### Why is Space Complexity Important?

*   **Resource Constraints:** Critical for environments with limited memory (e.g., embedded systems, mobile devices, serverless functions).
*   **Performance:** Excessive memory usage can lead to slower execution due to cache misses, page faults, or garbage collection overhead (in languages with GC).
*   **Scalability:** Helps determine if an algorithm can handle large inputs without running out of memory.
*   **Cost:** In cloud environments, memory usage directly impacts billing.
*   **Algorithm Comparison:** Provides another dimension for comparing the efficiency of different algorithms.

### Common Space Complexities

1.  **O(1) - Constant Space:**
    *   **Description:** The amount of memory used remains constant, regardless of the input size.
    *   *Example:* In-place sorting algorithms (e.g., Bubble Sort, Insertion Sort, Quick Sort if implemented in-place).

2.  **O(log n) - Logarithmic Space:**
    *   **Description:** The amount of memory used grows logarithmically with the input size.
    *   *Example:* Recursive algorithms that divide the problem (e.g., binary search's call stack depth).

3.  **O(n) - Linear Space:**
    *   **Description:** The amount of memory used grows linearly with the input size.
    *   *Example:* Creating a new array of the same size as the input, storing all elements of a linked list.

4.  **O(n log n) - Linearithmic Space:**
    *   **Description:** The amount of memory used grows proportionally to `n` multiplied by the logarithm of `n`.
    *   *Example:* Merge Sort (if not implemented in-place).

5.  **O(n^2) - Quadratic Space:**
    *   **Description:** The amount of memory used grows proportionally to the square of the input size.
    *   *Example:* Storing a 2D matrix where dimensions depend on `n`, adjacency matrix for a graph.

### Factors Affecting Space Complexity

*   **Input Size (`n`):** The primary factor.
*   **Auxiliary Data Structures:** Any additional data structures created by the algorithm (e.g., temporary arrays, hash maps).
*   **Recursion Stack:** The depth of recursive calls can consume stack space.
*   **Variables:** The number and size of variables used.

### Rust and Space Complexity

Rust's focus on explicit memory management and control makes space complexity a very tangible concept for developers.

*   **No Garbage Collector:** Rust's ownership system and borrow checker manage memory without a GC, leading to predictable memory usage and avoiding GC overhead.
*   **Value vs. Reference Semantics:** Rust encourages passing data by reference (`&T`) or borrowing, which often reduces memory copies and improves space efficiency.
*   **`Box<T>`:** Explicit heap allocation using `Box` makes it clear when memory is being allocated on the heap.
*   **Standard Library Data Structures:** The standard library provides efficient data structures, and their space complexities are well-understood.
*   **`no_std`:** For deeply embedded systems, Rust can be used without the standard library, giving developers full control over memory usage.

#### Rust Example: Analyzing Space Complexity

Let's look at a few Rust functions and determine their Big O space complexity.

**1. Setup `Cargo.toml`:**

```toml
[package]
name = "rust_space_complexity_example"
version = "0.1.0"
edition = "2021"

[dependencies]
```

**2. Example Code (`src/main.rs`)**

```rust
// src/main.rs

// O(1) - Constant Space
// This function uses a fixed amount of memory regardless of the input array size.
pub fn sum_first_two_elements(arr: &[i32]) -> i32 {
    if arr.len() >= 2 {
        arr[0] + arr[1]
    } else if arr.len() == 1 {
        arr[0]
    } else {
        0
    }
}

// O(n) - Linear Space
// This function creates a new vector whose size is proportional to the input array size.
pub fn double_elements(arr: &[i32]) -> Vec<i32> {
    let mut result = Vec::with_capacity(arr.len()); // Pre-allocate for efficiency
    for &num in arr {
        result.push(num * 2);
    }
    result
}

// O(n^2) - Quadratic Space
// This function creates a 2D vector (matrix) whose size is n*n.
pub fn create_n_by_n_matrix(n: usize) -> Vec<Vec<i32>> {
    let mut matrix = Vec::with_capacity(n);
    for i in 0..n {
        let mut row = Vec::with_capacity(n);
        for j in 0..n {
            row.push((i * n + j) as i32);
        }
        matrix.push(row);
    }
    matrix
}

// O(log n) - Logarithmic Space (due to recursion stack)
// Recursive factorial (auxiliary space is stack depth)
pub fn factorial_recursive(n: u64) -> u64 {
    if n == 0 {
        1
    } else {
        n * factorial_recursive(n - 1)
    }
}


fn main() {
    let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

    println!("Sum of first two: {}", sum_first_two_elements(&numbers)); // O(1) space

    let doubled = double_elements(&numbers);
    println!("Doubled elements: {:?}", doubled);

    let matrix = create_n_by_n_matrix(3);
    println!("3x3 Matrix: {:?}", matrix);

    println!("Factorial of 5: {}", factorial_recursive(5));
}
```
To run: `cargo run`.

### Conclusion

Space Complexity is a crucial aspect of algorithm analysis, measuring how an algorithm's memory usage scales with its input size. Understanding space complexity is vital for optimizing resource consumption, especially in memory-constrained environments, and for designing scalable systems. Rust's explicit memory management, lack of a garbage collector, and focus on control make space complexity a very tangible and manageable concern for developers, enabling them to write highly efficient and resource-conscious applications.