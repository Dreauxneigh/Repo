## Event Storming

### Definition

**Event Storming** is a highly collaborative, workshop-based technique used to quickly discover and model the business processes and domain events within a complex software system. Developed by Alberto Brandolini, it brings together domain experts and technical people to create a shared understanding of how a system works by focusing on the "events" that occur within the business.

### The Different Types of Event Storming

*   **Big Picture Event Storming:** A high-level exploration of the entire business domain.
*   **Process Modeling Event Storming:** A more detailed exploration of a specific business process.
*   **Software Design Event Storming:** A detailed exploration of a specific part of the system, with a focus on designing the software.

### How to Run an Event Storming Workshop

1.  **Preparation:**
    *   Invite the right people (domain experts, developers, testers, etc.).
    *   Find a large room with a big wall.
    *   Get a lot of sticky notes of different colors.
2.  **The Workshop:**
    *   Start with a brief introduction to Event Storming.
    *   Ask the participants to write down all the domain events they can think of on orange sticky notes.
    *   Arrange the events in chronological order on the wall.
    *   Identify the commands that cause the events (blue sticky notes).
    *   Identify the aggregates that receive the commands and produce the events (yellow sticky notes).
    *   Identify the external systems that interact with the system (pink sticky notes).
    *   Identify the read models that are used to display data (green sticky notes).
    *   Identify the policies that are triggered by events (purple sticky notes).
3.  **Follow-up:**
    *   Take pictures of the wall.
    *   Transcribe the sticky notes into a digital format.
    *   Use the output of the workshop to inform the design of the software.

### Benefits of Event Storming

*   **Rapid Domain Discovery:** Quickly uncovers complex business processes.
*   **Shared Understanding:** Creates a common mental model and Ubiquitous Language.
*   **Identifies Bounded Contexts:** Helps in naturally discovering the boundaries of different subdomains.
*   **Reveals Business Rules:** Makes implicit business rules explicit.

### Common Event Storming Pitfalls

*   **Not having the right people in the room.**
*   **Not having a skilled facilitator.**
*   **Getting bogged down in details too early.**
*   **Not following up on the output of the workshop.**

### Rust and Event Storming

Event Storming is a design technique, not a coding one. However, the output of an Event Storming session directly informs the structure of a Rust application, especially one built with Domain-Driven Design, Hexagonal Architecture, or Event-Driven Architecture.

*   **Domain Events:** Directly translate to Rust `enum`s or `struct`s.
*   **Commands:** Translate to Rust `struct`s.
*   **Aggregates:** Map to Rust `struct`s.
*   **Policies/Reactions:** Can be implemented as event handlers.

### Conclusion

Event Storming is an invaluable technique for understanding complex business domains and designing software systems that accurately reflect them. By fostering collaboration and focusing on the flow of events, it provides a powerful visual language that bridges the gap between business and technology, leading to more effective and maintainable software solutions.