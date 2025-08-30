## Data Structures

### Definition

**Data Structures** are specialized ways of organizing, storing, and managing data in a computer so that it can be accessed, used, and modified efficiently. The choice of data structure significantly impacts the efficiency of algorithms.

### Common Data Structures

*   **Arrays:** A collection of elements of the same data type stored at contiguous memory locations.
*   **Linked Lists:** A sequence of nodes, where each node contains data and a reference to the next node.
*   **Stacks:** A LIFO (Last In, First Out) data structure.
*   **Queues:** A FIFO (First In, First Out) data structure.
*   **Trees:** Hierarchical data structures with a root node and child nodes.
*   **Graphs:** A collection of nodes (vertices) and edges (connections between nodes).
*   **Hash Tables (Hash Maps):** Data structures that store key-value pairs.

### Advanced Data Structures

*   **Heaps:** A specialized tree-based data structure that satisfies the heap property. Used to implement priority queues.
*   **Tries:** A tree-like data structure that is used to store a dynamic set of strings. Used for efficient prefix searches.
*   **Bloom Filters:** A probabilistic data structure that is used to test whether an element is a member of a set. Can have false positives, but no false negatives.

### Choosing the Right Data Structure

*   Use an **Array** or **`Vec`** for fast random access.
*   Use a **Linked List** for frequent insertions and deletions.
*   Use a **Stack** for LIFO scenarios.
*   Use a **Queue** for FIFO scenarios.
*   Use a **Hash Map** for fast key-value lookups.
*   Use a **Binary Search Tree** for maintaining a sorted collection.
*   Use a **Graph** to represent networks and relationships.
*   Use a **Heap** for implementing priority queues.
*   Use a **Trie** for efficient prefix searches.
*   Use a **Bloom Filter** for probabilistic set membership testing.

### Rust Example: A Simple Graph

```rust
use std::collections::HashMap;

#[derive(Debug)]
struct Graph {
    nodes: HashMap<String, Vec<String>>,
}

impl Graph {
    fn new() -> Self {
        Self { nodes: HashMap::new() }
    }

    fn add_node(&mut self, node: &str) {
        self.nodes.entry(node.to_string()).or_insert(Vec::new());
    }

    fn add_edge(&mut self, node1: &str, node2: &str) {
        self.nodes.entry(node1.to_string()).or_insert(Vec::new()).push(node2.to_string());
        self.nodes.entry(node2.to_string()).or_insert(Vec::new()).push(node1.to_string());
    }

    fn get_neighbors(&self, node: &str) -> Option<&Vec<String>> {
        self.nodes.get(node)
    }
}

fn main() {
    let mut graph = Graph::new();
    graph.add_node("A");
    graph.add_node("B");
    graph.add_node("C");
    graph.add_edge("A", "B");
    graph.add_edge("B", "C");

    println!("Graph: {:?}", graph);
    println!("Neighbors of B: {:?}", graph.get_neighbors("B"));
}
```

### Data Structures in the Rust Ecosystem

*   **`petgraph`:** A popular crate for graph data structures and algorithms.
*   **`ndarray`:** A crate for N-dimensional arrays.
*   **`im`:** A crate for immutable data structures.

### Conclusion

Data Structures are fundamental building blocks of computer science. The choice of data structure is critical for the performance and scalability of algorithms and applications. Rust's standard library offers a rich set of common data structures, and its unique ownership system enables developers to implement custom, highly performant, and safe data structures.