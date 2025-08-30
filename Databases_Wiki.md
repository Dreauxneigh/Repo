## Databases (Relational and NoSQL)

### Definition

A **Database** is an organized collection of structured information, or data, typically stored electronically in a computer system. It is designed to efficiently store, manage, and retrieve large amounts of data.

### Why are Databases Important?

*   **Data Persistence:** Store data reliably.
*   **Data Organization:** Structure data for easy querying.
*   **Data Integrity:** Ensure data accuracy and consistency.
*   **Data Security:** Control access to data.

### 1. Relational Databases (SQL Databases)

#### Definition

**Relational Databases** organize data into tables of rows and columns. Tables can be related using keys. They are based on the relational model and use SQL.

#### ACID Properties

*   **Atomicity:** Transactions are all-or-nothing.
*   **Consistency:** Transactions bring the database from one valid state to another.
*   **Isolation:** Concurrent transactions appear to execute in isolation.
*   **Durability:** Once committed, a transaction remains committed even in case of system failure.

#### Database Design Principles (Normalization)

Normalization is the process of organizing the columns and tables of a relational database to minimize data redundancy and improve data integrity. It involves a series of normal forms (1NF, 2NF, 3NF, BCNF, etc.).

#### Rust Example: Interacting with PostgreSQL using `sqlx`

```rust
// src/main.rs
use sqlx::{postgres::PgPoolOptions, Pool, Postgres};
use tokio::main;
use uuid::Uuid;
use chrono::{DateTime, Utc};

#[derive(Debug, sqlx::FromRow)]
struct Product {
    id: Uuid,
    name: String,
    price: f64,
    created_at: DateTime<Utc>,
}

#[main]
async fn main() -> Result<(), sqlx::Error> {
    let database_url = "postgres://user:password@localhost:5432/mydatabase";
    let pool = PgPoolOptions::new().max_connections(5).connect(database_url).await?;

    sqlx::query(
        r#"CREATE TABLE IF NOT EXISTS products (id UUID PRIMARY KEY, name VARCHAR NOT NULL, price DOUBLE PRECISION NOT NULL, created_at TIMESTAMPTZ NOT NULL DEFAULT NOW())"#,
    ).execute(&pool).await?;

    let new_product_id = Uuid::new_v4();
    sqlx::query("INSERT INTO products (id, name, price) VALUES ($1, $2, $3)")
        .bind(new_product_id).bind("Rust Book").bind(59.99)
        .execute(&pool).await?;

    let fetched_product = sqlx::query_as::<_, Product>("SELECT id, name, price, created_at FROM products WHERE id = $1")
        .bind(new_product_id).fetch_one(&pool).await?;
    println!("Fetched product: {:?}", fetched_product);

    Ok(())
}
```

### 2. NoSQL Databases

#### Definition

**NoSQL (Not only SQL)** databases provide a mechanism for storage and retrieval of data that is modeled in means other than the tabular relations. They are designed for specific data models and have flexible schemas.

#### Database Design Principles (Denormalization)

Denormalization is the process of intentionally adding redundant data to a database to improve read performance. This is often used in NoSQL databases to optimize for specific query patterns.

#### Rust Example: Interacting with MongoDB using `mongodb` crate

```rust
// src/main.rs
use mongodb::{bson::{doc, oid::ObjectId}, Client, Collection};
use serde::{Deserialize, Serialize};
use futures::stream::TryStreamExt;
use tokio::main;

#[derive(Debug, Serialize, Deserialize)]
struct Book {
    #[serde(rename = "_id", skip_serializing_if = "Option::is_none")]
    id: Option<ObjectId>,
    title: String,
    author: String,
    year: u32,
}

#[main]
async fn main() -> Result<(), mongodb::error::Error> {
    let client = Client::with_uri_str("mongodb://localhost:27017").await?;
    let db = client.database("mydatabase");
    let collection: Collection<Book> = db.collection("books");

    let new_book = Book { id: None, title: "The Rust Programming Language".to_string(), author: "Steve Klabnik and Carol Nichols".to_string(), year: 2018 };
    let insert_result = collection.insert_one(new_book, None).await?;
    println!("Inserted new book with ID: {:?}", insert_result.inserted_id);

    let filter = doc! { "title": "The Rust Programming Language" };
    let found_book = collection.find_one(filter, None).await?;
    println!("Found book: {:?}", found_book);

    Ok(())
}
```

### Database Migrations

Database migrations are a way to manage changes to your database schema in a controlled and versioned way. Tools like `sqlx-cli` for Rust can help automate this process.

### Object-Relational Mapping (ORM) and Query Builders

*   **ORM:** A technique that maps objects in an object-oriented programming language to tables in a relational database. Popular Rust ORMs include `Diesel` and `SeaORM`.
*   **Query Builders:** Provide a programmatic way to construct SQL queries, offering more type safety and reducing the risk of SQL injection compared to raw SQL strings.

### Conclusion

Databases are the backbone of modern software, providing persistent storage and efficient management of data. Rust, with its performance, reliability, and growing ecosystem of database drivers, is an excellent choice for building robust and efficient applications that interact with both relational and NoSQL databases.