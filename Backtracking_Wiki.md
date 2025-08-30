## Backtracking

### Definition

**Backtracking** is a general algorithmic technique for solving problems, typically constraint satisfaction problems or optimization problems, by systematically trying all possible configurations or solutions. It builds a solution incrementally, one piece at a time. If, at any point, a partial solution is found to violate a constraint or cannot possibly lead to a valid complete solution, the algorithm "backtracks" (undoes its last choice) and tries a different option.

It's a form of depth-first search (DFS) on a state-space tree.

### Key Concepts of Backtracking

1.  **State-Space Tree:** A conceptual tree where each node represents a partial solution, and the paths from the root to the leaves represent complete solutions. Backtracking explores this tree.
2.  **Constraints:** Rules that a solution must satisfy. If a partial solution violates a constraint, the algorithm prunes that branch of the search tree.
3.  **Base Case/Goal State:** A condition where a complete, valid solution has been found.
4.  **Choices:** At each step, the algorithm has a set of choices to extend the current partial solution.
5.  **Recursion:** Backtracking algorithms are typically implemented recursively, with each recursive call exploring a new choice.

### How Backtracking Works

1.  **Start at the Root:** Begin with an empty or initial partial solution.
2.  **Make a Choice:** Select one of the available options to extend the current partial solution.
3.  **Check Constraints:** If the new partial solution violates any constraints, discard it and backtrack to the previous choice point.
4.  **Explore Recursively:** If the new partial solution is valid, recursively call the backtracking function with this extended solution.
5.  **Base Case:** If a complete, valid solution is found (base case), record it.
6.  **Backtrack:** After exploring all possibilities from the current choice, undo the choice and return to the previous state to try other options.

### Why Use Backtracking?

*   **Systematic Exploration:** Guarantees finding all (or at least one) solutions for problems that can be modeled as a search through a state-space tree.
*   **Handles Constraints:** Naturally incorporates constraints to prune the search space, making it more efficient than brute-force.
*   **Elegance for Certain Problems:** For problems like N-Queens or Sudoku, the recursive nature of backtracking can lead to elegant solutions.

### Limitations of Backtracking

*   **Time Complexity:** Can be very slow (often exponential or factorial time complexity) for large inputs, as it explores many paths.
*   **Memory Usage:** Recursive calls can consume significant stack space.
*   **Not Always Optimal:** While it finds *a* solution or *all* solutions, it doesn't inherently guarantee the *optimal* solution for optimization problems unless all paths are explored.

### Common Problems Solved with Backtracking

*   **N-Queens Problem:** Place N chess queens on an N×N chessboard so that no two queens threaten each other.
*   **Sudoku Solver:** Fill a 9×9 grid with digits so that each column, each row, and each of the nine 3×3 subgrids contains all of the digits from 1 to 9.
*   **Knight's Tour Problem:** Find a sequence of moves of a knight on a chessboard such that the knight visits every square exactly once.
*   **Permutations and Combinations:** Generating all possible permutations or combinations of a set of elements.
*   **Subset Sum Problem:** Find a subset of a given set of numbers whose sum equals a given target.
*   **Maze Solving:** Finding a path from start to end in a maze.

### Rust and Backtracking

Rust is well-suited for implementing backtracking algorithms due to its performance, control, and features that support recursion and explicit state management.

*   **Performance:** Rust's native compilation ensures that the often computationally intensive backtracking process runs as fast as possible.
*   **Recursion:** Rust fully supports recursion, which is the natural way to implement backtracking.
*   **Explicit State:** Rust's explicit mutability and ownership rules make it clear when the partial solution's state is being modified and when it's being "undone" (backtracked).
*   **`Vec` and `HashMap`:** Efficient standard library data structures are used to build and manage partial solutions.
*   **`Result` and `Option`:** Useful for signaling success/failure or the absence of a solution.

#### Rust Example: N-Queens Problem

Place N chess queens on an N×N chessboard such that no two queens threaten each other.

**1. Setup `Cargo.toml`:**

```toml
[package]
name = "rust_backtracking_example"
version = "0.1.0"
edition = "2021"

[dependencies]
```

**2. Example Code (`src/main.rs`)**

```rust
// src/main.rs

/// Solves the N-Queens problem using backtracking.
///
/// Finds all possible ways to place N queens on an N x N chessboard
/// such that no two queens attack each other.
///
/// # Arguments
/// * `n` - The size of the chessboard (N x N).
///
/// # Returns
/// A vector of vectors, where each inner vector represents a valid solution.
/// Each solution is a list of column indices for queens in each row.
///
/// # Examples
/// ```
/// let solutions = rust_backtracking_example::solve_n_queens(4);
/// assert_eq!(solutions.len(), 2);
/// // Example solution: [[1, 3, 0, 2], [2, 0, 3, 1]]
/// ```
pub fn solve_n_queens(n: usize) -> Vec<Vec<usize>> {
    let mut solutions = Vec::new();
    // `board` stores the column index of the queen in each row.
    // `board[row]` = column of queen in that row.
    let mut board = vec![0; n]; // Initialize board with dummy values

    // Start backtracking from the first row
    backtrack(0, n, &mut board, &mut solutions);

    solutions
}

// Recursive backtracking function
fn backtrack(
    row: usize,
    n: usize,
    board: &mut Vec<usize>,
    solutions: &mut Vec<Vec<usize>>,
) {
    // Base case: If all queens are placed (reached last row + 1)
    if row == n {
        solutions.push(board.clone()); // Found a valid solution, add to results
        return;
    }

    // Try placing a queen in each column of the current row
    for col in 0..n {
        // Check if placing a queen at (row, col) is safe
        if is_safe(row, col, board) {
            board[row] = col; // Make a choice: place queen at (row, col)
            backtrack(row + 1, n, board, solutions); // Recurse to the next row
            // No explicit "undo" needed for `board[row] = col` because
            // the next iteration of the loop will overwrite it, or
            // the function returns and the stack frame is popped.
            // If we were modifying a shared mutable structure, we'd need to undo.
        }
    }
}

// Helper function to check if placing a queen at (row, col) is safe
fn is_safe(row: usize, col: usize, board: &Vec<usize>) -> bool {
    // Check previous rows
    for prev_row in 0..row {
        let prev_col = board[prev_row];

        // Check if in the same column
        if prev_col == col {
            return false;
        }

        // Check if in the same diagonal
        // abs(current_row - prev_row) == abs(current_col - prev_col)
        if (row as isize - prev_row as isize).abs() == (col as isize - prev_col as isize).abs() {
            return false;
        }
    }
    true
}

fn main() {
    let n = 4; // For N=4, there are 2 solutions
    let solutions = solve_n_queens(n);

    println!("Found {} solutions for {}-Queens problem:", n, solutions.len());
    for (i, sol) in solutions.iter().enumerate() {
        println!("Solution {}: {:?}", i + 1, sol);
        // Optional: print board visually
        for r in 0..n {
            for c in 0..n {
                if sol[r] == c {
                    print!("Q ");
                } else {
                    print!(". ");
                }
            }
            println!();
        }
        println!();
    }

    let n_8 = 8; // For N=8, there are 92 solutions
    let solutions_8 = solve_n_queens(n_8);
    println!("Found {} solutions for {}-Queens problem.", n_8, solutions_8.len());
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_solve_n_queens_4() {
        let solutions = solve_n_queens(4);
        assert_eq!(solutions.len(), 2);
        // Check if the known solutions are present
        assert!(solutions.contains(&vec![1, 3, 0, 2]));
        assert!(solutions.contains(&vec![2, 0, 3, 1]));
    }

    #[test]
    fn test_solve_n_queens_1() {
        let solutions = solve_n_queens(1);
        assert_eq!(solutions.len(), 1);
        assert_eq!(solutions[0], vec![0]);
    }

    #[test]
    fn test_solve_n_queens_0() {
        let solutions = solve_n_queens(0);
        assert_eq!(solutions.len(), 1); // Empty board has one "solution" (no queens)
        assert_eq!(solutions[0], vec![]);
    }

    #[test]
    fn test_solve_n_queens_2() {
        let solutions = solve_n_queens(2);
        assert_eq!(solutions.len(), 0);
    }

    #[test]
    fn test_solve_n_queens_3() {
        let solutions = solve_n_queens(3);
        assert_eq!(solutions.len(), 0);
    }
}
```
To run: `cargo run`. To run tests: `cargo test`.

### Conclusion

Backtracking is a powerful algorithmic technique for systematically exploring all possible solutions to a problem, particularly useful for constraint satisfaction and combinatorial problems. While often leading to high time complexity, its ability to prune the search space based on constraints makes it more efficient than brute-force. Rust's performance, control over state, and strong support for recursion make it an excellent language for implementing backtracking algorithms, enabling developers to tackle complex search problems with safety and efficiency.