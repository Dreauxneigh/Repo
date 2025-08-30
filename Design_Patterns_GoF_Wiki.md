## Design Patterns (Gang of Four)

### Definition

**Design Patterns** are reusable solutions to common problems encountered in software design. They are templates for how to solve a problem that can be used in many different situations. The term "Gang of Four" (GoF) refers to the authors of the book "Design Patterns: Elements of Reusable Object-Oriented Software", which cataloged 23 classic design patterns.

### Categories of Design Patterns

1.  **Creational Patterns:** Deal with object creation mechanisms (e.g., Singleton, Factory Method, Builder).
2.  **Structural Patterns:** Deal with the composition of classes and objects (e.g., Adapter, Decorator, Facade).
3.  **Behavioral Patterns:** Deal with the communication and interaction between objects (e.g., Strategy, Observer, Command).

### When to Use Design Patterns

Design patterns should be used judiciously. Over-engineering a solution with unnecessary patterns can lead to more complexity. Use a pattern when you have a recurring problem that the pattern is designed to solve.

### Rust and Design Patterns

While Rust is not a traditional object-oriented language, its features like traits, enums, and structs allow for the implementation of most GoF design patterns in a safe and idiomatic way.

### Rust Example: Strategy Pattern

The Strategy pattern allows you to define a family of algorithms, put each of them into a separate class, and make their objects interchangeable.

```rust
// The Strategy trait
trait ShippingStrategy {
    fn calculate_cost(&self, weight: f64) -> f64;
}

// Concrete strategies
struct StandardShipping;
impl ShippingStrategy for StandardShipping {
    fn calculate_cost(&self, weight: f64) -> f64 {
        weight * 1.5
    }
}

struct ExpressShipping;
impl ShippingStrategy for ExpressShipping {
    fn calculate_cost(&self, weight: f64) -> f64 {
        weight * 2.5
    }
}

// The context
struct ShippingCalculator<S: ShippingStrategy> {
    strategy: S,
}

impl<S: ShippingStrategy> ShippingCalculator<S> {
    fn new(strategy: S) -> Self {
        Self { strategy }
    }

    fn calculate_cost(&self, weight: f64) -> f64 {
        self.strategy.calculate_cost(weight)
    }
}

fn main() {
    let standard_calculator = ShippingCalculator::new(StandardShipping);
    println!("Standard shipping cost: {}", standard_calculator.calculate_cost(10.0));

    let express_calculator = ShippingCalculator::new(ExpressShipping);
    println!("Express shipping cost: {}", express_calculator.calculate_cost(10.0));
}
```

### Rust Example: Decorator Pattern

The Decorator pattern allows you to dynamically add new functionality to an object without changing its implementation.

```rust
// The Component trait
trait Coffee {
    fn cost(&self) -> f64;
    fn description(&self) -> String;
}

// Concrete component
struct SimpleCoffee;
impl Coffee for SimpleCoffee {
    fn cost(&self) -> f64 {
        2.0
    }

    fn description(&self) -> String {
        "Simple coffee".to_string()
    }
}

// Decorator
struct MilkDecorator<C: Coffee> {
    coffee: C,
}

impl<C: Coffee> Coffee for MilkDecorator<C> {
    fn cost(&self) -> f64 {
        self.coffee.cost() + 0.5
    }

    fn description(&self) -> String {
        format!("{}, milk", self.coffee.description())
    }
}

fn main() {
    let simple_coffee = SimpleCoffee;
    println!("Cost: {}, Description: {}", simple_coffee.cost(), simple_coffee.description());

    let milk_coffee = MilkDecorator { coffee: simple_coffee };
    println!("Cost: {}, Description: {}", milk_coffee.cost(), milk_coffee.description());
}
```

### Conclusion

Design Patterns provide a valuable toolkit for solving common software design problems. While Rust's approach to object-oriented programming differs from traditional languages, its powerful features enable the effective and idiomatic implementation of most GoF patterns, contributing to robust and well-structured Rust applications.