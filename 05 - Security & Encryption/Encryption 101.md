---
tags:
  - encryption
  - security
category: Security, Identity & Compliance
---

# Encryption 101

> [!INFO] Definition
> Encryption is the process of encoding information so that only authorized parties can access it.

## Encryption Approaches

### Encryption At Rest
* **Goal**: Protects against physical theft and tampering of the media (hard drives, tapes).
* **Usage**: Generally used when only one entity is involved (e.g., an encrypted laptop).

### Encryption In Transit
* **Goal**: Protects data as it moves between systems (Network, Internet).
* **Mechanism**: Uses an encrypted tunnel (SSL/TLS).
* **Usage**: Used when multiple systems or individuals are involved.

## Core Concepts
* **Plaintext**: Unencrypted raw data (text, images, files).
* **Algorithm**: Code that performs the mathematical operations for encryption.
* **Key**: A piece of information (password or random string) used by the algorithm.
* **Ciphertext**: The encrypted output.

## Symmetric vs. Asymmetric

### Symmetric Encryption
* **Mechanism**: The **same key** is used for both encryption and decryption.
* **Best For**: Local disk encryption, database encryption (fast).

### Asymmetric Encryption
* **Mechanism**: Uses a **Key Pair** consisting of a **Public Key** and a **Private Key**.
* **Key Fact**: Data encrypted with the Public Key can **only** be decrypted by the corresponding Private Key.
* **Best For**: Secure communication (SSL/TLS), Digital Signatures.

## Digital Signing Process
Used to verify the **Identity** of the sender and the **Integrity** of the message.

1.  **Hash**: The sender creates a unique hash of the data.
2.  **Sign**: The sender encrypts the hash using their **Private Key**. This is the digital signature.
3.  **Verify**: The recipient decrypts the signature using the sender's **Public Key** to reveal the hash.
4.  **Confirm**: The recipient hashes the received data themselves. If their hash matches the decrypted signature hash, the data is authentic.

## Steganography
* **Definition**: The practice of hiding secret data within another non-secret file (like an image or an audio file) to avoid detection.
