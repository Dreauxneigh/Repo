## Greedy Algorithms

### Definition

**Greedy Algorithms** are a class of algorithms that follow the problem-solving heuristic of making the locally optimal choice at each stage with the hope of finding a global optimum. In many problems, a greedy strategy does not produce an optimal solution, but for some problems, it does. The key characteristic is that the algorithm makes a choice that seems best at the moment, without considering future consequences.

### Key Concepts of Greedy Algorithms

1.  **Local Optimum:** At each step, the algorithm chooses the option that appears to be the best at that moment, based on some immediate criteria.
2.  **No Backtracking:** Once a choice is made, the algorithm does not reconsider or backtrack to previous choices, even if a better overall solution might be found by doing so.
3.  **Optimal Substructure:** The problem must exhibit optimal substructure, meaning an optimal solution to the problem contains optimal solutions to its subproblems.
4.  **Greedy Choice Property:** A globally optimal solution can be arrived at by making a locally optimal (greedy) choice. This is the property that makes a greedy algorithm work for a specific problem.

### When Do Greedy Algorithms Work?

Greedy algorithms are often simpler and faster than other approaches (like Dynamic Programming) when they are applicable. They work for problems that have the **greedy choice property** and **optimal substructure**. Proving that a greedy algorithm yields an optimal solution usually involves showing that a greedy choice can always be extended to an optimal solution, and that any optimal solution can be modified to incorporate the greedy choice without becoming suboptimal.

### Common Problems Solved with Greedy Algorithms

*   **Activity Selection Problem:** Selecting the maximum number of non-overlapping activities.
*   **Coin Change Problem (Canonical Coin Systems):** Finding the minimum number of coins to make a given amount, *if* the coin denominations are "canonical" (e.g., 1, 5, 10, 25 cents). (Note: For arbitrary coin systems, DP is needed).
*   **Fractional Knapsack Problem:** Maximizing the value of items that can be put into a knapsack, where items can be broken into fractions. (Note: For 0/1 Knapsack, DP is needed).
*   **Huffman Coding:** Building optimal prefix codes for data compression.
*   **Dijkstra's Algorithm:** Finding the shortest path in a graph with non-negative edge weights.
*   **Prim's and Kruskal's Algorithms:** Finding the Minimum Spanning Tree (MST) in a graph.

### Rust and Greedy Algorithms

Rust's performance and control make it well-suited for implementing greedy algorithms, which are often designed for efficiency.

*   **Performance:** Greedy algorithms are often chosen for their speed, and Rust's native compilation ensures that these algorithms run as fast as possible.
*   **Control:** Rust provides fine-grained control over data structures and memory, allowing for efficient implementations of the local choices.
*   **Iterators:** Rust's iterator system can be used to efficiently process data for making greedy choices.
*   **Standard Library:** Provides efficient data structures like `Vec`, `HashMap`, `BinaryHeap` (for priority queues, often used in greedy algorithms like Dijkstra's).

#### Rust Example: Activity Selection Problem

Given a set of activities, each with a start and finish time, select the maximum number of non-overlapping activities. The greedy strategy is to always pick the activity that finishes earliest among the remaining compatible activities.

**1. Setup `Cargo.toml`:**

```toml
[package]
name = "rust_greedy_example"
version = "0.1.0"
edition = "2021"

[dependencies]
```

**2. Example Code (`src/main.rs`)**

```rust
// src/main.rs

#[derive(Debug, PartialEq, Eq, Clone)]
pub struct Activity {
    pub start: u32,
    pub finish: u32,
}

/// Solves the Activity Selection Problem using a greedy algorithm.
///
/// Given a list of activities with start and finish times,
/// selects the maximum number of non-overlapping activities.
///
/// The greedy choice is to always pick the activity that finishes earliest.
///
/// # Arguments
/// * `activities` - A vector of `Activity` structs.
///
/// # Returns
/// A vector of selected activities.
///
/// # Examples
/// ```
/// use rust_greedy_example::Activity;
/// use rust_greedy_example::select_activities;
///
/// let activities = vec![
///     Activity { start: 1, finish: 4 },
///     Activity { start: 3, finish: 5 },
///     Activity { start: 0, finish: 6 },
///     Activity { start: 5, finish: 7 },
///     Activity { start: 3, finish: 9 },
///     Activity { start: 5, finish: 9 },
///     Activity { start: 6, finish: 10 },
///     Activity { start: 8, finish: 11 },
///     Activity { start: 8, finish: 12 },
///     Activity { start: 2, finish: 14 },
///     Activity { start: 12, finish: 16 },
/// ];
///
/// let selected = select_activities(activities);
/// // Expected: [Activity { start: 1, finish: 4 }, Activity { start: 5, finish: 7 }, Activity { start: 8, finish: 11 }, Activity { start: 12, finish: 16 }]
/// // Or: [Activity { start: 1, finish: 4 }, Activity { start: 5, finish: 7 }, Activity { start: 8, finish: 11 }, Activity { start: 12, finish: 16 }]
/// // The exact order might vary based on initial sort stability for equal finish times,
/// // but the count and set of activities should be optimal.
/// assert_eq!(selected.len(), 4);
/// ```
pub fn select_activities(mut activities: Vec<Activity>) -> Vec<Activity> {
    if activities.is_empty() {
        return Vec::new();
    }

    // Sort activities by their finish times in ascending order.
    // This is the crucial greedy choice.
    activities.sort_by_key(|a| a.finish);

    let mut selected_activities = Vec::new();
    selected_activities.push(activities[0].clone()); // Select the first activity (earliest finish time)

    let mut last_finish_time = activities[0].finish;

    // Iterate through the remaining activities
    for i in 1..activities.len() {
        // If the current activity's start time is greater than or equal to
        // the last selected activity's finish time, select it.
        if activities[i].start >= last_finish_time {
            selected_activities.push(activities[i].clone());
            last_finish_time = activities[i].finish;
        }
    }

    selected_activities
}

fn main() {
    let activities = vec![
        Activity { start: 1, finish: 4 },
        Activity { start: 3, finish: 5 },
        Activity { start: 0, finish: 6 },
        Activity { start: 5, finish: 7 },
        Activity { start: 3, finish: 9 },
        Activity { start: 5, finish: 9 },
        Activity { start: 6, finish: 10 },
        Activity { start: 8, finish: 11 },
        Activity { start: 8, finish: 12 },
        Activity { start: 2, finish: 14 },
        Activity { start: 12, finish: 16 },
    ];

    println!("Original activities: {:?}", activities);
    let selected = select_activities(activities);
    println!("Selected activities: {:?}", selected);
    println!("Number of selected activities: {}", selected.len());
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_select_activities_basic() {
        let activities = vec![
            Activity { start: 1, finish: 2 },
            Activity { start: 3, finish: 4 },
            Activity { start: 0, finish: 6 },
            Activity { start: 5, finish: 7 },
            Activity { start: 8, finish: 9 },
            Activity { start: 5, finish: 9 },
        ];
        let selected = select_activities(activities);
        assert_eq!(selected.len(), 4);
        assert!(selected.contains(&Activity { start: 1, finish: 2 }));
        assert!(selected.contains(&Activity { start: 3, finish: 4 }));
        assert!(selected.contains(&Activity { start: 5, finish: 7 }));
        assert!(selected.contains(&Activity { start: 8, finish: 9 }));
    }

    #[test]
    fn test_select_activities_empty() {
        let activities: Vec<Activity> = vec![];
        let selected = select_activities(activities);
        assert!(selected.is_empty());
    }

    #[test]
    fn test_select_activities_single() {
        let activities = vec![Activity { start: 10, finish: 20 }];
        let selected = select_activities(activities);
        assert_eq!(selected.len(), 1);
        assert!(selected.contains(&Activity { start: 10, finish: 20 }));
    }

    #[test]
    fn test_select_activities_complex() {
        let activities = vec![
            Activity { start: 1, finish: 4 },
            Activity { start: 3, finish: 5 },
            Activity { start: 0, finish: 6 },
            Activity { start: 5, finish: 7 },
            Activity { start: 3, finish: 9 },
            Activity { start: 5, finish: 9 },
            Activity { start: 6, finish: 10 },
            Activity { start: 8, finish: 11 },
            Activity { start: 8, finish: 12 },
            Activity { start: 2, finish: 14 },
            Activity { start: 12, finish: 16 },
        ];
        let selected = select_activities(activities);
        assert_eq!(selected.len(), 4);
        // The exact activities might vary if multiple have the same earliest finish time,
        // but the count should be 4.
        // For example, (1,4), (5,7), (8,11), (12,16) is one valid set.
    }
}
```
To run: `cargo run`. To run tests: `cargo test`.

### Conclusion

Greedy Algorithms offer a simple and often efficient approach to solving optimization problems by making locally optimal choices. While they don't guarantee a globally optimal solution for all problems, they are highly effective for a specific class of problems that exhibit the greedy choice property and optimal substructure. Rust's performance, control over data structures, and efficient iterators make it an excellent language for implementing greedy algorithms, enabling developers to create fast and practical solutions for suitable problems.