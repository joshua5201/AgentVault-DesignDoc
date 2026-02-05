# Technical Specification: AgentVault
## Zero-Knowledge Asynchronous Secret Management for Autonomous Agents

**Author:** Tsung-en Hsiao (@joshua5201)
**Date:** February 6, 2026

---

## 1. Abstract
This document specifies a secure, zero-knowledge architecture for autonomous agents (e.g., LLM-based agents) to request and retrieve credentials. The system combines an **Asynchronous Out-of-Band Fulfillment Pattern** with **Client-Side Encryption** to ensure that neither the AI model's context window nor the central storage server ever possesses the plaintext secrets. This establishes a "security air-gap" where human administrators fulfill agent requests using local cryptography, sharing only pre-encrypted data blobs with the server.

## 2. Technical Field
The invention relates to distributed systems security, zero-knowledge architectures, automated credential management, and human-in-the-loop (HITL) security protocols for artificial intelligence.

## 3. Background and Problem Statement
Autonomous agents frequently require access to credentials that were not provisioned a priori. Existing solutions suffer from two critical vulnerabilities:
1.  **Context Leakage:** Users often paste secrets directly into chat interfaces, exposing them to model logs and third-party providers.
2.  **Server-Side Vulnerability:** Traditional secret managers hold the decryption keys server-side. A compromise of the server exposes all secrets.

There is a technical need for a system that allows agents to "ask" for secrets dynamically while guaranteeing that the storage provider (the Vault Server) remains mathematically incapable of accessing the credentials.

## 4. System Architecture

### 4.1 Zero-Knowledge "Blind" Storage
The Vault Server acts exclusively as a coordinator and encrypted blob store. It does not possess any keys to decrypt user data.
*   **Secrets:** Stored as encrypted binary blobs.
*   **Encryption:** Performed strictly on the client side (Admin Web UI or Agent CLI).
*   **Keys:** Private keys never leave the client devices.

### 4.2 Public Key Exchange Protocol
To facilitate secure sharing between the Admin (Secret Owner) and the Agent (Secret User):
1.  **Agent Identity:** Upon initialization, an Agent generates a key pair and registers its **Public Key** with the Vault Server.
2.  **Admin Identity:** The Tenant Admin registers their **Public Key** for secure recovery flows.
3.  **Trust:** The server creates an immutable binding between the Agent's identity and its Public Key.

## 5. Core Innovation: The Zero-Knowledge "Ask" Pattern

This protocol defines how a human securely provisions a secret to an agent without the server or the chat interface ever seeing the raw data.

### 5.1 The Workflow
1.  **Request:** An Agent encounters a missing credential and posts a structured request to the API (e.g., "I need AWS keys").
2.  **Link Generation:** The server generates a unique **Fulfillment URL** and returns it to the Agent.
3.  **Out-of-Band Handshake:** The Agent displays the URL to the user in the chat: *"I need credentials. Please provide them securely here: [URL]"*.
4.  **Client-Side Encryption (The Innovation):**
    *   The Human Administrator accesses the URL via a trusted, local Web UI.
    *   The Web UI fetches the **Agent's Public Key** from the server.
    *   The Admin enters the secret. The browser **locally encrypts** the secret using the Agent's Public Key.
    *   Only the *encrypted blob* is uploaded to the server.
5.  **Retrieval:** The Agent downloads the encrypted blob and decrypts it locally using its held **Private Key**.

## 6. Security Claims for Defensive Publication
This disclosure specifically claims the following technical improvements to establish prior art:

1.  **Context-Isolated Fulfillment:** A method for preventing credential leakage into AI interaction logs by decoupling the secret input interface from the agent's primary communication channel using a unique, request-specific fulfillment URL.
2.  **Zero-Knowledge Lease Exchange:** A protocol where a central server brokers the temporary lease of a credential between a human and an autonomous agent by storing only data encrypted with the agent's public key, ensuring the server implies no trust.
3.  **Agentic Schema Definition:** A mechanism where an autonomous agent defines the technical schema (required fields and metadata) of a missing secret to guide the human's client-side encryption process.

## 7. Implementation Details
*   **API:** RESTful API designed for machine consumption.
*   **Cryptography:** Standard Asymmetric Encryption (e.g., RSA-OAEP or ECIES) for secret exchange; Symmetric Encryption (AES-GCM) for local storage.
*   **Database:** NoSQL (MongoDB) for storing encrypted blobs and metadata.
*   **Clients:**
    *   **Agent:** CLI tool handling local key generation and decryption.
    *   **Admin:** Web Application using Web Crypto API for zero-knowledge operations.

---
*This document is intended to serve as prior art under applicable patent laws.*