## Graph Algorithms

### Definition

**Graph Algorithms** are a set of computational procedures designed to solve problems on **graphs**. A graph is a non-linear data structure consisting of a finite set of nodes (or vertices) and a set of edges (or links) that connect pairs of nodes. Graphs are incredibly versatile and can model a wide range of real-world relationships and networks, such as social networks, road maps, computer networks, and dependencies between tasks.

### Key Concepts in Graph Theory

*   **Vertex (Node):** A fundamental entity in a graph.
*   **Edge (Link/Arc):** A connection between two vertices.
*   **Directed Graph (Digraph):** Edges have a direction (e.g., one-way street).
*   **Undirected Graph:** Edges have no direction (e.g., two-way street).
*   **Weighted Graph:** Edges have associated values (weights or costs), representing distance, time, capacity, etc.
*   **Path:** A sequence of distinct vertices connected by edges.
*   **Cycle:** A path that starts and ends at the same vertex.
*   **Connected Graph:** A graph where there is a path between every pair of vertices.
*   **Tree:** A connected, acyclic (no cycles) undirected graph.
*   **Adjacency Matrix:** A square matrix used to represent a finite graph. An entry `A[i][j]` is 1 if there is an edge from vertex `i` to vertex `j`, and 0 otherwise.
*   **Adjacency List:** An array of lists, where the `i`-th list contains the neighbors of vertex `i`. More space-efficient for sparse graphs.

### Common Graph Problems and Algorithms

1.  **Graph Traversal:** Visiting all vertices in a graph.
    *   **Breadth-First Search (BFS):** Explores all the neighbor nodes at the present depth prior to moving on to the nodes at the next depth level. Uses a queue.
    *   **Depth-First Search (DFS):** Explores as far as possible along each branch before backtracking. Uses a stack (or recursion).

2.  **Shortest Path Algorithms:** Finding the path with the minimum weight between two vertices.
    *   **Dijkstra's Algorithm:** Finds the shortest paths from a single source vertex to all other vertices in a graph with non-negative edge weights. Uses a priority queue.
    *   **Bellman-Ford Algorithm:** Finds the shortest paths from a single source vertex to all other vertices in a graph that can have negative edge weights (and detects negative cycles).
    *   **Floyd-Warshall Algorithm:** Finds the shortest paths between all pairs of vertices in a graph.

3.  **Minimum Spanning Tree (MST) Algorithms:** Finding a subset of the edges of a connected, edge-weighted undirected graph that connects all the vertices together, without any cycles and with the minimum possible total edge weight.
    *   **Prim's Algorithm:** Builds an MST by adding the cheapest edge from the current tree to a new vertex.
    *   **Kruskal's Algorithm:** Builds an MST by adding edges in increasing order of weight, as long as they don't form a cycle.

4.  **Topological Sort:** Ordering of vertices in a directed acyclic graph (DAG) such that for every directed edge `u -> v`, vertex `u` comes before `v` in the ordering.
    *   **Examples:** Task scheduling, course prerequisites.

5.  **Connectivity Algorithms:**
    *   **Connected Components:** Finding sets of vertices that are connected to each other.
    *   **Strongly Connected Components (SCCs):** For directed graphs, finding maximal subgraphs where every vertex is reachable from every other vertex within the subgraph.

### Rust and Graph Algorithms

Rust is an excellent language for implementing graph algorithms due to its performance, control over memory, and safety features.

*   **Performance:** Rust's native compilation and control over memory allow for highly efficient graph implementations and algorithms.
*   **Safety:** The borrow checker helps prevent common graph-related bugs like use-after-free or data races when manipulating graph structures.
*   **Explicit Data Structures:** Rust's explicit nature means you clearly define how your graph is represented (e.g., adjacency list using `Vec<Vec<usize>>` or `HashMap<usize, Vec<usize>>`).
*   **Generics:** Allows for writing generic graph algorithms that work with various types of vertex data or edge weights.
*   **`Rc<T>` and `RefCell<T>`:** Can be used (carefully) to manage shared ownership and mutable references in complex graph structures where cycles might exist.
*   **Ecosystem:** While not built-in, there are excellent third-party crates for graph theory:
    *   **`petgraph`:** A powerful and flexible graph data structure library.
    *   **`graph`:** Another graph library.

#### Rust Example: Breadth-First Search (BFS)

BFS is a graph traversal algorithm that explores a graph level by level.

**1. Setup `Cargo.toml`:**

```toml
[package]
name = "rust_graph_algorithms_example"
version = "0.1.0"
edition = "2021"

[dependencies]
# No specific dependencies for this simple example
```

**2. Example Code (`src/main.rs`)**

```rust
// src/main.rs
use std::collections::VecDeque; // For the queue in BFS

/// Represents a graph using an adjacency list.
/// `adj[i]` contains a list of vertices adjacent to vertex `i`.
pub struct Graph {
    num_vertices: usize,
    adj: Vec<Vec<usize>>,
}

impl Graph {
    pub fn new(num_vertices: usize) -> Self {
        Graph {
            num_vertices,
            adj: vec![Vec::new(); num_vertices],
        }
    }

    /// Adds an undirected edge between two vertices.
    pub fn add_edge(&mut self, u: usize, v: usize) {
        self.adj[u].push(v);
        self.adj[v].push(u); // For undirected graph
    }

    /// Performs a Breadth-First Search (BFS) starting from a given source vertex.
    ///
    /// # Arguments
    /// * `start_node` - The vertex to start the BFS from.
    ///
    /// # Returns
    /// A vector representing the order in which nodes were visited.
    ///
    /// # Examples
    /// ```
    /// let mut graph = rust_graph_algorithms_example::Graph::new(5);
    /// graph.add_edge(0, 1);
    /// graph.add_edge(0, 2);
    /// graph.add_edge(1, 3);
    /// graph.add_edge(2, 4);
    /// let visited_order = graph.bfs(0);
    /// assert_eq!(visited_order, vec![0, 1, 2, 3, 4]);
    /// ```
pub fn bfs(&self, start_node: usize) -> Vec<usize> {
        let mut visited = vec![false; self.num_vertices];
        let mut queue = VecDeque::new();
        let mut visited_order = Vec::new();

        visited[start_node] = true;
        queue.push_back(start_node);

        while let Some(u) = queue.pop_front() {
            visited_order.push(u);

            for &v in &self.adj[u] {
                if !visited[v] {
                    visited[v] = true;
                    queue.push_back(v);
                }
            }
        }
        visited_order
    }

    /// Performs a Depth-First Search (DFS) starting from a given source vertex.
    ///
    /// # Arguments
    /// * `start_node` - The vertex to start the DFS from.
    ///
    /// # Returns
    /// A vector representing the order in which nodes were visited.
    ///
    /// # Examples
    /// ```
    /// let mut graph = rust_graph_algorithms_example::Graph::new(5);
    /// graph.add_edge(0, 1);
    /// graph.add_edge(0, 2);
    /// graph.add_edge(1, 3);
    /// graph.add_edge(2, 4);
    /// let visited_order = graph.dfs(0);
    /// assert_eq!(visited_order, vec![0, 1, 3, 2, 4]); // Or similar order depending on adjacency list order
    /// ```
pub fn dfs(&self, start_node: usize) -> Vec<usize> {
        let mut visited = vec![false; self.num_vertices];
        let mut visited_order = Vec::new();
        let mut stack = Vec::new();

        stack.push(start_node);
        visited[start_node] = true;

        while let Some(u) = stack.pop() {
            visited_order.push(u);

            // Iterate over neighbors in reverse to maintain consistent order for example
            // For actual DFS, order of neighbors doesn't strictly matter
            for &v in self.adj[u].iter().rev() {
                if !visited[v] {
                    visited[v] = true;
                    stack.push(v);
                }
            }
        }
        visited_order
    }
}

fn main() {
    // Create a graph:
    // 0 -- 1 -- 3
    // |    | 
    // 2 -- 4
    let mut graph = Graph::new(5);
    graph.add_edge(0, 1);
    graph.add_edge(0, 2);
    graph.add_edge(1, 3);
    graph.add_edge(2, 4);

    println!("--- Graph Traversal ---");
    println!("BFS from node 0: {:?}", graph.bfs(0)); // Expected: [0, 1, 2, 3, 4]
    println!("DFS from node 0: {:?}", graph.dfs(0)); // Expected: [0, 2, 4, 1, 3] (or [0, 1, 3, 2, 4] depending on adj list order)
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_bfs() {
        let mut graph = Graph::new(5);
        graph.add_edge(0, 1);
        graph.add_edge(0, 2);
        graph.add_edge(1, 3);
        graph.add_edge(2, 4);
        assert_eq!(graph.bfs(0), vec![0, 1, 2, 3, 4]);
    }

    #[test]
    fn test_dfs() {
        let mut graph = Graph::new(5);
        graph.add_edge(0, 1);
        graph.add_edge(0, 2);
        graph.add_edge(1, 3);
        graph.add_edge(2, 4);
        // DFS order depends on adjacency list order.
        // For this specific graph and add_edge implementation,
        // it will likely be [0, 2, 4, 1, 3] or [0, 1, 3, 2, 4]
        // Let's test for one of the expected valid paths.
        let result = graph.dfs(0);
        assert!(result == vec![0, 2, 4, 1, 3] || result == vec![0, 1, 3, 2, 4]);
    }

    #[test]
    fn test_bfs_disconnected() {
        let mut graph = Graph::new(5);
        graph.add_edge(0, 1);
        graph.add_edge(2, 3); // Disconnected component
        assert_eq!(graph.bfs(0), vec![0, 1]);
    }
}
```
To run: `cargo run`. To run tests: `cargo test`.

### Conclusion

Graph Algorithms are essential for solving problems that can be modeled as networks of interconnected entities. From finding the shortest path to optimizing network flows, these algorithms provide powerful tools for analyzing complex relationships. Rust's performance, control over memory, and safety features make it an excellent language for implementing efficient and reliable graph algorithms, enabling developers to tackle a wide range of computational challenges in areas like social networks, logistics, and data analysis.