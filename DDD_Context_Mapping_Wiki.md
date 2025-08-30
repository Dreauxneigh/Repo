## Domain-Driven Design (DDD) Strategic Patterns: Context Mapping

### Definition

**Context Mapping** is a strategic DDD pattern that focuses on explicitly defining the relationships and communication patterns between different **Bounded Contexts** within a large software system. Once Bounded Contexts are identified, Context Mapping helps to understand how they interact, what dependencies exist, and how to manage those dependencies effectively. It's a crucial step in designing the overall architecture of a complex system, especially when integrating multiple subdomains.

### Purpose and Importance

Context Mapping is vital for:

*   **Visualizing System Landscape:** Provides a high-level overview of how different parts of the system fit together and interact.
*   **Managing Integration Complexity:** Helps to identify and manage the complexities of integrating different Bounded Contexts, each with its own Ubiquitous Language and Domain Model.
*   **Informing Architectural Decisions:** The relationships identified through Context Mapping directly influence architectural choices, such as whether to use shared databases, message queues, or API calls for integration.
*   **Facilitating Communication:** Creates a shared understanding among teams about how their respective Bounded Contexts interact with others.
*   **Identifying Integration Patterns:** Helps in choosing appropriate integration patterns (e.g., Shared Kernel, Customer/Supplier, Anti-Corruption Layer).

### Common Context Map Patterns

Context Mapping identifies various patterns of relationships between Bounded Contexts. Understanding these patterns helps in choosing the right integration strategy.

1.  **Partnership (P):**
    *   **Description:** Two teams (and their Bounded Contexts) have a mutual dependency and work closely together. They succeed or fail together.
    *   **Implication:** Requires strong communication and coordination between teams. Changes in one context directly impact the other.

2.  **Customer/Supplier (C/S):**
    *   **Description:** One Bounded Context (the "Supplier") provides a service or data to another Bounded Context (the "Customer"). The Customer has influence over the Supplier's roadmap.
    *   **Implication:** The Supplier team needs to consider the Customer's needs. Often involves a published API or shared library.

3.  **Conformist (CF):**
    *   **Description:** The Customer Bounded Context chooses to conform to the model and Ubiquitous Language of the Supplier Bounded Context. The Customer has no influence over the Supplier.
    *   **Implication:** Simpler integration for the Customer, but they are at the mercy of the Supplier's changes. Often seen when integrating with off-the-shelf products or external services.

4.  **Anti-Corruption Layer (ACL):**
    *   **Description:** A translation layer built by the Customer Bounded Context to isolate its own Domain Model from the foreign model of the Supplier Bounded Context. It translates concepts between the two models.
    *   **Implication:** Adds complexity but provides strong isolation, protecting the Customer's domain integrity. Useful when integrating with legacy systems or external systems with incompatible models.

5.  **Shared Kernel (SK):
    *   **Description:** A subset of the domain model that is shared and agreed upon by two or more Bounded Contexts. Changes to the Shared Kernel require coordination among all involved teams.
    *   **Implication:** Reduces duplication but creates tight coupling within the shared part. Requires strong communication and version control.

6.  **Separate Ways (SW):**
    *   **Description:** Two Bounded Contexts have no significant relationship and can evolve independently.
    *   **Implication:** Ideal scenario for maximum autonomy, but only applicable when there's genuinely no overlap or need for integration.

7.  **Published Language (PL):**
    *   **Description:** The Supplier Bounded Context publishes its interface and data format in a well-documented, standardized language (e.g., JSON Schema, OpenAPI).
    *   **Implication:** Facilitates integration for multiple Consumers, often used with Customer/Supplier or Conformist relationships.

### Creating a Context Map

Context Mapping is typically a collaborative activity involving architects, lead developers, and domain experts. It often involves:

1.  **Identifying Bounded Contexts:** (As discussed in the previous wiki).
2.  **Drawing Boundaries:** Visually representing each Bounded Context.
3.  **Identifying Relationships:** Drawing lines between contexts that interact.
4.  **Labeling Relationships:** Using the Context Map patterns (P, C/S, CF, ACL, SK, SW, PL) to label the nature of the interaction.
5.  **Discussing and Refining:** Iteratively discussing the implications of each relationship and refining the map.

### Rust Example: Illustrating Context Mapping Patterns

While Context Mapping is a high-level design activity, its implications are reflected in the Rust code's structure and dependencies.

Consider our retail system with `Sales`, `Fulfillment`, and `Billing` Bounded Contexts.

#### 1. Customer/Supplier Relationship: `Sales` (Customer) and `Fulfillment` (Supplier)

*   `Sales` needs to tell `Fulfillment` to ship an order.
*   `Fulfillment` provides an API for `Sales` to request shipments.

**`sales_context/src/application/services/order_shipping_service.rs`**
```rust
// sales_context/src/application/services/order_shipping_service.rs
// This service in Sales Context will call the Fulfillment Context's API

use reqwest::Client; // For making HTTP calls
use serde::{Deserialize, Serialize};

#[derive(Serialize)]
pub struct ShipOrderRequest {
    pub order_id: String,
    pub customer_address: String, // Simplified
    pub items: Vec<String>, // Simplified
}

#[derive(Deserialize)]
pub struct ShipOrderResponse {
    pub shipment_id: String,
    pub status: String,
}

pub struct FulfillmentGateway {
    client: Client,
    fulfillment_service_url: String,
}

impl FulfillmentGateway {
    pub fn new(fulfillment_service_url: String) -> Self {
        FulfillmentGateway {
            client: Client::new(),
            fulfillment_service_url,
        }
    }

    pub async fn request_shipment(&self, request: ShipOrderRequest) -> Result<ShipOrderResponse, String> {
        let url = format!("{}/shipments", self.fulfillment_service_url);
        let response = self.client.post(&url)
            .json(&request)
            .send()
            .await
            .map_err(|e| format!("Failed to send shipment request: {}", e))?;

        if response.status().is_success() {
            response.json::<ShipOrderResponse>().await
                .map_err(|e| format!("Failed to parse shipment response: {}", e))
        } else {
            Err(format!("Fulfillment service returned error: {}", response.status()))
        }
    }
}
```
**`fulfillment_context/src/adapters/web/shipment_controller.rs`**
```rust
// fulfillment_context/src/adapters/web/shipment_controller.rs
// This is the API exposed by the Fulfillment Context

use actix_web::{post, web, HttpResponse, Responder};
use serde::{Deserialize, Serialize};

#[derive(Deserialize)]
pub struct ShipOrderRequest {
    pub order_id: String,
    pub customer_address: String,
    pub items: Vec<String>,
}

#[derive(Serialize)]
pub struct ShipOrderResponse {
    pub shipment_id: String,
    pub status: String,
}

#[post("/shipments")]
async fn create_shipment(req: web::Json<ShipOrderRequest>) -> impl Responder {
    // In a real app, this would call a Fulfillment Application Service
    println!("Fulfillment: Received request to ship order {}. Items: {:?}", req.order_id, req.items);
    HttpResponse::Ok().json(ShipOrderResponse {
        shipment_id: uuid::Uuid::new_v4().to_string(),
        status: "Pending".to_string(),
    })
}
```

#### 2. Anti-Corruption Layer (ACL): `Billing` (Customer) and `Legacy Accounting` (Supplier)

*   `Billing` needs customer data from a `Legacy Accounting` system.
*   The `Legacy Accounting` system has an old, messy data model.
*   `Billing` builds an ACL to translate `Legacy Accounting`'s "Customer" into `Billing`'s "Customer".

**`billing_context/src/infrastructure/acl/legacy_customer_acl.rs`**
```rust
// billing_context/src/infrastructure/acl/legacy_customer_acl.rs
// This ACL translates between the legacy system's customer model and Billing's customer model

use serde::{Deserialize, Serialize};

// Legacy system's customer representation (foreign model)
#[derive(Debug, Deserialize)]
pub struct LegacyCustomerDto {
    pub legacy_cust_id: String,
    pub full_name: String,
    pub addr_line1: String,
    pub zip_code_str: String, // Legacy uses string for zip code
    pub balance_due: f64,
}

// Billing Context's Customer Entity (our domain model)
#[derive(Debug, Clone, PartialEq)]
pub struct Customer {
    pub id: String,
    pub name: String,
    pub address: String, // Simplified
    pub outstanding_balance: f64,
}

pub struct LegacyCustomerAcl;

impl LegacyCustomerAcl {
    pub fn translate_to_domain(legacy_dto: LegacyCustomerDto) -> Customer {
        // Complex translation logic here
        let customer_id = format!("BILL-{}", legacy_dto.legacy_cust_id); // Map legacy ID to our ID
        let address = format!("{}, {}", legacy_dto.addr_line1, legacy_dto.zip_code_str); // Combine address parts

        Customer {
            id: customer_id,
            name: legacy_dto.full_name,
            address,
            outstanding_balance: legacy_dto.balance_due,
        }
    }

    // Potentially methods to translate from our domain to legacy for writes
    // pub fn translate_from_domain(customer: &Customer) -> LegacyCustomerDto { ... }
}
```

### Conclusion

Context Mapping is an indispensable strategic tool in DDD for designing and managing complex software systems composed of multiple Bounded Contexts. By explicitly defining the relationships between these contexts, it helps teams make informed architectural decisions, manage integration complexities, and maintain a clear understanding of the overall system landscape. It's a living document that evolves with the system, guiding development and ensuring that the right integration patterns are applied where they are most effective.