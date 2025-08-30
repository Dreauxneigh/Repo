## Observer Pattern

### Definition

The **Observer Pattern** is a behavioral design pattern that defines a one-to-many dependency between objects so that when one object (the **Subject**) changes state, all its dependents (the **Observers**) are notified and updated automatically.

### When to Use the Observer Pattern

Use the Observer pattern when:

*   Changes to the state of one object need to be reflected in other objects without coupling them tightly.
*   You need to support an unknown number of objects that need to be notified of state changes.
*   You want to establish a one-to-many relationship between objects.

### Push vs. Pull Models

*   **Push Model:** The subject sends all the data to the observers in the `update` method. This is simple, but can be inefficient if the observers don't need all the data.
*   **Pull Model:** The subject sends a notification to the observers, and the observers then pull the data they need from the subject. This is more flexible, but can be more complex to implement.

### Rust Example: Weather Station

Let's create a weather station that notifies different displays of changes in the weather data.

#### 1. The Observer Trait

```rust
// src/lib.rs
pub trait Observer {
    fn update(&self, temperature: f32, humidity: f32, pressure: f32);
}
```

#### 2. The Subject Trait

```rust
// src/lib.rs
pub trait Subject<'a, T: Observer> {
    fn attach(&mut self, observer: &'a T);
    fn detach(&mut self, observer: &'a T);
    fn notify_observers(&self);
}
```

#### 3. The Concrete Subject

```rust
// src/lib.rs
pub struct WeatherData<'a, T: Observer> {
    temperature: f32,
    humidity: f32,
    pressure: f32,
    observers: Vec<&'a T>,
}

impl<'a, T: Observer + PartialEq> WeatherData<'a, T> {
    pub fn new() -> Self {
        Self {
            temperature: 0.0,
            humidity: 0.0,
            pressure: 0.0,
            observers: Vec::new(),
        }
    }

    pub fn set_measurements(&mut self, temperature: f32, humidity: f32, pressure: f32) {
        self.temperature = temperature;
        self.humidity = humidity;
        self.pressure = pressure;
        self.notify_observers();
    }
}

impl<'a, T: Observer + PartialEq> Subject<'a, T> for WeatherData<'a, T> {
    fn attach(&mut self, observer: &'a T) {
        self.observers.push(observer);
    }

    fn detach(&mut self, observer: &'a T) {
        if let Some(index) = self.observers.iter().position(|x| (*x) as *const _ == observer as *const _) {
            self.observers.remove(index);
        }
    }

    fn notify_observers(&self) {
        for observer in &self.observers {
            observer.update(self.temperature, self.humidity, self.pressure);
        }
    }
}
```

#### 4. Concrete Observers

```rust
// src/lib.rs
#[derive(PartialEq)]
pub struct CurrentConditionsDisplay;
impl Observer for CurrentConditionsDisplay {
    fn update(&self, temperature: f32, humidity: f32, _pressure: f32) {
        println!("Current conditions: {}F degrees and {}% humidity", temperature, humidity);
    }
}

#[derive(PartialEq)]
pub struct StatisticsDisplay;
impl Observer for StatisticsDisplay {
    fn update(&self, temperature: f32, humidity: f32, pressure: f32) {
        println!("Avg/Max/Min temperature = {}/{}/{}", temperature, humidity, pressure);
    }
}
```

#### 5. Usage

```rust
// src/main.rs
use observer_pattern::*;

fn main() {
    let mut weather_data = WeatherData::new();

    let current_conditions_display = CurrentConditionsDisplay;
    let statistics_display = StatisticsDisplay;

    weather_data.attach(&current_conditions_display);
    weather_data.attach(&statistics_display);

    weather_data.set_measurements(80.0, 65.0, 30.4);
    weather_data.set_measurements(82.0, 70.0, 29.2);

    weather_data.detach(&statistics_display);
    weather_data.set_measurements(78.0, 90.0, 29.2);
}
```

### Implementation in Rust: `Weak` References

To avoid potential memory leaks and reference cycles, especially in more complex scenarios, you can use `std::rc::Weak` to hold references to the observers. This allows the observers to be deallocated when they are no longer in use, and the subject can gracefully handle the case where an observer has been deallocated.

### Conclusion

The Observer Pattern is a fundamental behavioral pattern for implementing event-driven systems. It provides a robust mechanism for one-to-many communication, promoting loose coupling and making systems more adaptable to changes in their notification requirements.