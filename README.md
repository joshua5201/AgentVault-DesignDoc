# Technical Specification: AgentVault
## Secure Asynchronous Secret Retrieval and Human-in-the-Loop Fulfillment for Autonomous Agents

**Author:** Tsung-en Hsiao (@joshua5201)
**Date:** February 4, 2026 

---

## 1. Abstract
This document specifies a novel security architecture for autonomous agents (e.g., Large Language Model-based agents) to securely retrieve existing credentials and request missing secrets. The core innovation lies in the **Asynchronous Out-of-band Fulfillment Pattern**, which establishes a security air-gap between the agent's conversational context and the administrative secret entry process.

## 2. Technical Field
The invention relates to distributed systems security, automated credential management, and human-in-the-loop (HITL) security protocols for artificial intelligence.

## 3. Background and Problem Statement
Traditional secret managers (e.g., HashiCorp Vault) assume that the calling entity (the Agent) knows the specific identifier of a secret a priori. Autonomous agents, however, frequently encounter unforeseen requirements during task execution. 

Current methods often involve:
1. **Plaintext Injection:** Users providing secrets directly in the chat, which leaks sensitive data into logs and model context.
2. **Over-privileged Access:** Granting agents broad access to all secrets, violating the principle of least privilege.

There is a technical need for a system that allows agents to request specific secrets without ever handling the raw input provided by a human administrator.

## 4. System Architecture

### 4.1 Two-Tier Key Hierarchy
To ensure tenant isolation and secure automation:
1. **System Master Key (SMK):** Abstrated via a `MasterKeyProvider`, used exclusively to encrypt/decrypt Tenant Keys.
2. **Tenant Key (TK):** A unique AES-256 key for each tenant, used to encrypt/decrypt individual secret values within that tenant's scope.

### 4.2 Flexible Metadata-Driven Search
Agents query secrets using an arbitrary metadata map (e.g., `{"service": "github", "env": "prod"}`). The system performs indexed lookups on predefined high-value keys to ensure performance while maintaining flexibility for agentic reasoning.

## 5. Core Innovation: The "Ask" Pattern (Asynchronous Fulfillment)

This protocol defines the mechanism for handling missing credentials without exposing the secret to the LLM's context.

### 5.1 The Workflow
1. **Identification:** An Agent fails to find a required secret via standard search.
2. **Request Initiation:** The Agent calls `POST /api/v1/requests` with a defined schema:
   - `required_metadata`: Target service identifiers.
   - `required_fields`: Expected keys (e.g., `["api_key", "client_secret"]`).
3. **Out-of-band URL Generation:** The Vault Server generates a unique, short-lived **Fulfillment URL** and a `request_id`.
4. **Decoupled Notification:** The Agent presents the URL to the user: *"I need credentials for [Service]. Provide them here: [URL]"*.
5. **Human Resolution:** An authenticated Administrator accesses the URL via a separate secure UI. The Admin can:
   - **Fulfill:** Create a new secret based on the requested schema.
   - **Map:** Link the request to an existing secret.
6. **Asynchronous Completion:** Once fulfilled, the request status is updated. The Agent, upon polling or webhook notification, retrieves the secured secret via its own authorized channel.

## 6. Security Claims for Defensive Publication
This disclosure specifically claims the following technical improvements:
1. **Context Isolation:** A method for preventing credential leakage into AI chat logs by decoupling the secret input interface from the agent's primary communication channel.
2. **Agentic Schema Definition:** A mechanism where an autonomous agent defines the technical requirements (metadata and fields) of a missing secret to guide human fulfillment.
3. **Lease-Based Visibility:** A visibility tier (`LEASE_REQUIRED`) where secrets are hidden from agents until a temporary, intent-based lease is requested by the agent and approved by a human administrator.

## 7. Implementation Details
- **Backend:** Java 21 / Spring Boot 3.
- **Database:** MongoDB (Sharded by `tenant_id` for horizontal scaling).
- **Encryption:** AES-GCM (Java Cryptography Architecture).
- **Authentication:** OAuth2 / JWT with tenant-scoped claims.

---
*This document is intended to serve as prior art under applicable patent laws.*
