## Cryptography and Security Fundamentals

### Definition

**Cryptography** is the practice and study of techniques for secure communication in the presence of adversarial behavior. More generally, it is about constructing and analyzing protocols that prevent third parties or the public from reading private messages. It's a fundamental pillar of **Information Security**, which aims to protect information and information systems from unauthorized access, use, disclosure, disruption, modification, or destruction.

### Why are Cryptography and Security Fundamentals Important?

*   **Data Protection:** Safeguard sensitive data (personal information, financial records, intellectual property) from theft, alteration, or unauthorized access.
*   **Trust and Integrity:** Ensure that data has not been tampered with and that communication is authentic.
*   **Privacy:** Protect user privacy by encrypting communications and data at rest.
*   **Compliance:** Meet regulatory requirements (e.g., GDPR, HIPAA, PCI DSS) that mandate data security.
*   **Business Continuity:** Prevent security breaches that can lead to financial losses, reputational damage, and service disruption.
*   **System Reliability:** Secure systems are inherently more reliable as they are less susceptible to attacks.

### Key Pillars of Information Security (CIA Triad)

1.  **Confidentiality**:
    *   **Definition:** Ensuring that information is accessible only to those authorized to have access.
    *   **Techniques:** Encryption, access controls, data masking.

2.  **Integrity**:
    *   **Definition:** Ensuring that information is accurate, complete, and has not been tampered with or altered without authorization.
    *   **Techniques:** Hashing, digital signatures, message authentication codes (MACs).

3.  **Availability**:
    *   **Definition:** Ensuring that authorized users have reliable and timely access to information and resources.
    *   **Techniques:** Redundancy, backups, disaster recovery, DDoS mitigation.

### Core Cryptographic Concepts

1.  **Encryption**:
    *   **Definition:** The process of transforming plaintext (readable data) into ciphertext (unreadable data) to protect its confidentiality.
    *   **Types**:
        *   **Symmetric-key Encryption:** Uses a single secret key for both encryption and decryption (e.g., AES, DES). Fast but key distribution is a challenge.
        *   **Asymmetric-key Encryption (Public-key Cryptography):** Uses a pair of keys: a public key (for encryption, can be shared) and a private key (for decryption, kept secret) (e.g., RSA, ECC). Slower but solves key distribution.

2.  **Hashing (Cryptographic Hash Functions)**:
    *   **Definition:** A one-way mathematical function that takes an input (or 'message') and returns a fixed-size string of bytes (a 'hash value' or 'digest'). It's computationally infeasible to reverse the process.
    *   **Properties:** Deterministic (same input always gives same output), fast to compute, collision-resistant (hard to find two different inputs that produce the same hash).
    *   **Use Cases:** Password storage (store hash, not password), data integrity verification, digital signatures.
    *   **Examples:** SHA-256, SHA-3.

3.  **Digital Signatures**:
    *   **Definition:** A mathematical scheme for verifying the authenticity and integrity of digital messages or documents. Uses asymmetric cryptography.
    *   **How it works:** Sender signs a message hash with their private key. Receiver decrypts the signature with the sender's public key and compares it to their own computed hash of the message.
    *   **Use Cases:** Software authenticity, document signing, non-repudiation.

4.  **Key Management**:
    *   **Definition:** The process of generating, distributing, storing, rotating, and revoking cryptographic keys.
    *   **Importance:** The security of cryptographic systems heavily relies on the security of their keys.

5.  **TLS/SSL (Transport Layer Security/Secure Sockets Layer)**:
    *   **Definition:** Protocols that provide secure communication over a computer network. They encrypt data exchanged between a web server and a browser (HTTPS).
    *   **How it works:** Uses a combination of symmetric and asymmetric encryption, hashing, and digital certificates.

### Common Security Vulnerabilities (Beyond Memory Safety)

Even with memory-safe languages like Rust, other vulnerabilities can exist:

*   **Injection Flaws:** SQL Injection, Command Injection (due to improper input validation).
*   **Broken Authentication/Access Control:** Weak password policies, insecure session management, improper authorization checks.
*   **Insecure Design:** Lack of threat modeling, insecure architectural patterns.
*   **Cryptographic Misuse:** Using weak algorithms, improper key management, incorrect protocol implementation.
*   **Server-Side Request Forgery (SSRF):** Exploiting a server to make requests to internal or external resources.
*   **Vulnerable Third-Party Components:** Using libraries with known security flaws.

### Rust in Cryptography and Security

Rust is an exceptionally strong choice for building secure and cryptographic applications due to its unique combination of features:

*   **Memory Safety:** Rust's borrow checker eliminates entire classes of memory-related vulnerabilities (e.g., buffer overflows, use-after-free) that are common in C/C++ and are frequently exploited in security breaches. This is a massive advantage.
*   **Concurrency Safety:** Prevents data races, which can lead to subtle security bugs.
*   **Strong Type System:** Helps prevent logical errors and ensures data integrity.
*   **Explicit Error Handling:** Forces developers to handle potential failure points, reducing the likelihood of unexpected behavior that could be exploited.
*   **Zero-Cost Abstractions:** Allows for high-performance cryptographic implementations without sacrificing safety.
*   **Cryptographic Ecosystem:** A growing number of high-quality, audited cryptographic crates:
    *   **`ring`:** Cryptographic library for Rust, often used for TLS, hashing, and signatures.
    *   **`rustls`:** A modern, safe, and fast TLS library written in Rust.
    *   **`sha2`, `md5`, `aes`:** Crates for specific cryptographic algorithms.
    *   **`rand`:** Cryptographically secure random number generation.
    *   **`zeroize`:** For securely clearing sensitive data from memory.

#### Rust Example: Hashing a Password (for secure storage)

**1. Setup `Cargo.toml`:**

```toml
[package]
name = "rust_crypto_example"
version = "0.1.0"
edition = "2021"

[dependencies]
sha2 = "0.10" # For SHA-256 hashing
hex = "0.4"   # For converting bytes to hex string
rand = "0.8"  # For generating random salt
getrandom = { version = "0.2", features = ["js"] } # Required by rand for some targets
```

**2. Example Code (`src/main.rs`)**

```rust
// src/main.rs
use sha2::{Sha256, Digest};
use rand::RngCore; // For RngCore trait

/// Hashes a password using SHA-256 with a random salt.
///
/// This is a simplified example. In a real application, you would use a
/// dedicated password hashing library like `argon2` or `scrypt` which are
/// designed to be slow and resistant to brute-force attacks.
///
/// # Arguments
/// * `password` - The plaintext password string.
///
/// # Returns
/// A tuple containing the salt (as hex string) and the hashed password (as hex string).
pub fn hash_password(password: &str) -> (String, String) {
    // Generate a random salt (e.g., 16 bytes)
    let mut salt_bytes = [0u8; 16];
    rand::thread_rng().fill_bytes(&mut salt_bytes);
    let salt_hex = hex::encode(salt_bytes);

    // Create a SHA-256 hasher
    let mut hasher = Sha256::new();

    // Hash the password combined with the salt
    hasher.update(password.as_bytes());
    hasher.update(salt_bytes); // Add salt to the hash

    let hashed_bytes = hasher.finalize();
    let hashed_hex = hex::encode(hashed_bytes);

    (salt_hex, hashed_hex)
}

/// Verifies a password against a stored hash and salt.
pub fn verify_password(password: &str, stored_salt_hex: &str, stored_hash_hex: &str) -> bool {
    let salt_bytes = hex::decode(stored_salt_hex).expect("Invalid salt hex string");

    let mut hasher = Sha256::new();
    hasher.update(password.as_bytes());
    hasher.update(salt_bytes);

    let computed_hash_bytes = hasher.finalize();
    let computed_hash_hex = hex::encode(computed_hash_bytes);

    // Compare the computed hash with the stored hash
    // Use a constant-time comparison to prevent timing attacks
    computed_hash_hex == stored_hash_hex
}

fn main() {
    let user_password = "mySecretPassword123";

    println!("--- Password Hashing Example ---");

    // Simulate user registration
    let (salt, hashed_password) = hash_password(user_password);
    println!("Original Password: {}", user_password);
    println!("Generated Salt:    {}", salt);
    println!("Stored Hash:     {}", hashed_password);

    // Simulate user login attempt
    let login_attempt_password = "mySecretPassword123";
    let is_valid = verify_password(login_attempt_password, &salt, &hashed_password);
    println!("\nLogin attempt with correct password: {}", is_valid);

    let wrong_password = "wrongPassword";
    let is_valid_wrong = verify_password(wrong_password, &salt, &hashed_password);
    println!("Login attempt with wrong password: {}", is_valid_wrong);
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_password_hashing_and_verification() {
        let password = "test_password";
        let (salt, hashed) = hash_password(password);
        assert!(verify_password(password, &salt, &hashed));
        assert!(!verify_password("wrong_password", &salt, &hashed));
    }

    #[test]
    fn test_different_salts_produce_different_hashes() {
        let password = "same_password";
        let (_, hash1) = hash_password(password);
        let (_, hash2) = hash_password(password);
        // Hashes should be different because salts are random
        assert_ne!(hash1, hash2);
    }
}
