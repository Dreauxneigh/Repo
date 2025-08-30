## Procedural Programming

### Definition

**Procedural Programming** is a programming paradigm that organizes code into procedures (also known as routines, subroutines, or functions). These procedures contain a series of computational steps to be carried out. The program's execution flow is typically controlled by sequential execution of these procedures, often with the use of control structures like loops, conditionals, and jumps.

In procedural programming, data and functions are generally separate. Functions operate on data that is often passed to them as arguments or accessed from a shared global state.

### Key Concepts of Procedural Programming

1.  **Procedures/Functions/Routines:**
    *   **Definition:** Blocks of code designed to perform a specific task. They can take input parameters and return output values.
    *   **Purpose:** To break down a program into smaller, manageable, and reusable units.

2.  **Sequential Execution:**
    *   **Definition:** Instructions are executed one after another in the order they appear in the code.
    *   **Purpose:** Provides a clear, step-by-step flow of control.

3.  **Control Structures:**
    *   **Definition:** Constructs that alter the sequential flow of execution.
    *   **Examples:** `if-else` statements (conditionals), `for` loops, `while` loops (iteration), `goto` statements (jumps).

4.  **Variables:**
    *   **Definition:** Named storage locations for data.
    *   **Types:** Local variables (within a procedure), global variables (accessible throughout the program).

5.  **Modularity:**
    *   **Definition:** The ability to break down a program into separate, independent modules (files or compilation units).
    *   **Purpose:** To manage complexity and facilitate code reuse.

### Why Use Procedural Programming?

*   **Simplicity:** Often straightforward to understand the flow of control, especially for smaller programs.
*   **Efficiency:** Can be very efficient as it directly maps to how computer hardware executes instructions.
*   **Modularity:** Procedures allow for breaking down complex tasks into smaller, reusable components.
*   **Ease of Debugging:** The sequential nature can make it easier to trace execution flow.
*   **Good for General-Purpose Programming:** Suitable for a wide range of applications, from system utilities to scientific computing.

### Limitations of Procedural Programming

*   **Global State Issues:** Reliance on global variables can lead to side effects, making code harder to reason about and prone to bugs.
*   **Tight Coupling:** Data and functions are separate, which can lead to tight coupling if functions directly manipulate global data.
*   **Difficulty in Managing Complexity:** For very large and complex systems, managing the interactions between many procedures and shared data can become challenging.
*   **Less Reusability (for complex data structures):** Procedures are often tied to specific data structures, making them less reusable with different data.

### Rust and Procedural Programming

Rust is a multi-paradigm language that fully supports procedural programming. In fact, much of Rust's core functionality and many common Rust programs are written in a procedural style, especially when dealing with command-line tools, scripts, or performance-critical algorithms.

*   **Functions:** Rust's `fn` keyword defines functions, which are the core building blocks of procedural programming. 
*   **Control Flow:** Rust provides standard control flow constructs (`if`, `else`, `for`, `while`, `match`).
*   **Variables:** Rust has both mutable (`mut`) and immutable variables.
*   **Modules:** Rust's module system (`mod`) allows for excellent code organization and modularity, enabling developers to group related functions and data.
*   **Performance:** Rust's performance makes it ideal for procedural tasks where speed is critical.
*   **Safety:** Even in a procedural style, Rust's ownership and borrowing rules prevent common memory safety bugs and data races, making procedural code much safer than in languages like C.

#### Rust Example: Procedural Style (File Processing)

Let's write a simple Rust program in a procedural style to read a file, process its lines, and write to another file.

**1. Setup `Cargo.toml`:**

```toml
[package]
name = "rust_procedural_example"
version = "0.1.0"
edition = "2021"

[dependencies]
```

**2. Example Code (`src/main.rs`)**

```rust
// src/main.rs
use std::fs::{self, File};
use std::io::{self, BufReader, BufWriter, BufRead, Write};
use std::path::Path;

// Procedure 1: Reads lines from a file
fn read_lines_from_file(file_path: &Path) -> io::Result<Vec<String>> {
    let file = File::open(file_path)?;
    let reader = BufReader::new(file);
    let mut lines = Vec::new();
    for line_result in reader.lines() {
        lines.push(line_result?);
    }
    Ok(lines)
}

// Procedure 2: Processes a single line (e.g., converts to uppercase)
fn process_line(line: &str) -> String {
    line.to_uppercase()
}

// Procedure 3: Writes processed lines to a file
fn write_lines_to_file(file_path: &Path, lines: &[String]) -> io::Result<()> {
    let file = File::create(file_path)?;
    let mut writer = BufWriter::new(file);
    for line in lines {
        writeln!(writer, "{}", line)?;
    }
    writer.flush()?;
    Ok(())
}

// Main procedure: Orchestrates the file processing
fn main() -> Result<(), Box<dyn std::error::Error>> {
    let input_file_path = Path::new("input.txt");
    let output_file_path = Path::new("output.txt");

    // Create a dummy input file for demonstration
    fs::write(input_file_path, "hello world\nRust programming\nexample")?;

    println!("---" + "Starting File Processing" + "---");

    // Step 1: Read lines
    let lines = read_lines_from_file(input_file_path)?;
    println!("Read {} lines from input.txt", lines.len());

    // Step 2: Process lines
    let mut processed_lines = Vec::new();
    for line in lines {
        let processed = process_line(&line);
        processed_lines.push(processed);
    }
    println!("Processed lines.");

    // Step 3: Write lines
    write_lines_to_file(output_file_path, &processed_lines)?;
    println!("Wrote {} lines to output.txt", processed_lines.len());

    println!("---" + "File Processing Complete" + "---");

    // Clean up dummy file
    fs::remove_file(input_file_path)?;
    Ok(())
}
```
To run: `cargo run`. Check `output.txt` for the processed content.

### Conclusion

Procedural Programming is a fundamental and widely used programming paradigm that organizes code into a sequence of steps and procedures. It offers simplicity, efficiency, and modularity, making it suitable for a broad range of applications. Rust, while supporting multiple paradigms, excels in procedural programming by providing powerful functions, control structures, and a robust module system, all while ensuring memory safety and high performance. This makes Rust an excellent choice for building reliable and efficient procedural applications, from system utilities to data processing scripts.