## Kanban

### Definition

**Kanban** is a lean methodology and visual system for managing and improving workflow. Originating from Toyota's production system, it was adapted for software development to help teams visualize their work, limit work in progress (WIP), and maximize efficiency and flow. Unlike Scrum, Kanban is not time-boxed and is more focused on continuous delivery and optimizing the flow of value.

The word "Kanban" is Japanese for "visual signal" or "card."

### Core Principles

Kanban is built on four foundational principles:

1.  **Visualize the Workflow:** Make all work items and their progress visible. This is typically done using a Kanban board with columns representing different stages of the workflow (e.g., To Do, In Progress, Testing, Done).
2.  **Limit Work in Progress (WIP):** Set explicit limits on the number of items that can be in progress at each stage of the workflow. This prevents multitasking, reduces context switching, and helps identify bottlenecks.
3.  **Manage Flow:** Optimize the flow of work through the system. This involves identifying and addressing bottlenecks, reducing lead time (time from start to finish), and ensuring a smooth, continuous delivery of value.
4.  **Make Process Policies Explicit:** Clearly define the rules and policies for how work moves through the system. This ensures everyone understands the process and can contribute to its improvement.
5.  **Implement Feedback Loops:** Regularly review the process and performance to identify opportunities for improvement.
6.  **Improve Collaboratively, Evolve Experimentally:** Encourage continuous, incremental, and evolutionary changes to the process based on data and feedback.

### The Kanban Board

The central tool in Kanban is the **Kanban board**. It's a visual representation of the workflow, typically divided into columns.

*   **Columns:** Represent stages of the workflow (e.g., Backlog, Ready, Development, Code Review, Testing, Deploy, Done).
*   **Cards:** Represent individual work items (tasks, user stories, bugs). Each card moves from left to right as work progresses.
*   **WIP Limits:** Numbers placed at the top of each "in progress" column, indicating the maximum number of cards allowed in that stage at any given time.

**Example Kanban Board Columns:**

*   **Backlog:** All potential work items.
*   **Selected for Development:** Items prioritized and ready to be worked on.
*   **In Development (WIP Limit: 3):** Items currently being coded.
*   **In Code Review (WIP Limit: 2):** Items awaiting peer review.
*   **In Testing (WIP Limit: 2):** Items being tested.
*   **Ready for Deployment:** Items that have passed all checks and are ready to be released.
*   **Done:** Completed work items.

### Key Metrics in Kanban

*   **Lead Time:** The total time it takes for a work item to go from the beginning of the workflow (e.g., "Selected for Development") to "Done."
*   **Cycle Time:** The time a work item spends in an active "in progress" state.
*   **Throughput:** The number of work items completed per unit of time.
*   **Work in Progress (WIP):** The number of items currently being worked on.

### Benefits of Kanban

*   **Flexibility:** Highly adaptable to changing priorities and requirements. No fixed iterations or deadlines.
*   **Improved Flow:** Focuses on optimizing the flow of work, leading to faster delivery and reduced bottlenecks.
*   **Reduced Waste:** Limiting WIP reduces context switching, idle time, and rework.
*   **Increased Transparency:** The visual board provides a clear, real-time overview of work status for everyone.
*   **Predictability:** By analyzing lead time and throughput, teams can make more accurate predictions about delivery.
*   **Continuous Delivery:** Naturally supports continuous delivery by emphasizing a smooth flow of completed work.
*   **Less Overhead:** Fewer prescribed meetings and roles compared to Scrum, making it suitable for maintenance teams or operations.

### Kanban vs. Scrum

| Feature             | Kanban                                        | Scrum                                             |
| :------------------ | :-------------------------------------------- | :------------------------------------------------ |
| **Time-boxing**     | No fixed iterations (continuous flow)         | Fixed-length Sprints (e.g., 2 weeks)              |
| **WIP Limits**      | Explicitly used to manage flow                | Implicitly managed by Sprint Backlog              |
| **Roles**           | No prescribed roles (can use existing)        | Defined roles (Product Owner, Scrum Master, Dev Team) |
| **Cadence**         | Event-driven, continuous delivery             | Iteration-driven, delivery at end of Sprint       |
| **Change**          | Changes can be introduced at any time         | Changes discouraged within a Sprint               |
| **Best Suited For** | Maintenance, operations, continuous flow      | Complex product development, stable teams         |

### Rust and Kanban

Rust's features and ecosystem support Kanban principles:

*   **Quality and Reliability:** Rust's focus on correctness and safety means that work items moving through the "Development" and "Testing" columns are likely to be of high quality, reducing rework.
*   **Performance:** Rust's performance allows for efficient processing of work items, contributing to faster flow.
*   **Tooling:** `cargo fmt`, `cargo clippy`, and `cargo test` ensure that code entering "Code Review" or "Testing" stages is already well-formatted, linted, and has passing tests, streamlining the flow.
*   **Modularity:** Rust's module system and crate ecosystem facilitate breaking down work into small, manageable units that fit well on Kanban cards.

### Conclusion

Kanban is a powerful and flexible lean methodology for visualizing, managing, and improving the flow of work. By emphasizing visualization, limiting WIP, and continuous improvement, it helps teams deliver value more efficiently and predictably. Its adaptability makes it suitable for a wide range of contexts, from software development to operations and beyond, providing a clear path to continuous delivery and operational excellence.