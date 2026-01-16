# 🚀 Product Implementation & Evolution Roadmap (PIER)

Developed from the perspective of a Senior Staff Engineer / System Architect (10+ years experience). This roadmap outlines the lifecycle of the Blood Test Result Platform from MVP to a production-grade, highly resilient system.

---

## Phase 1: The Foundation (Infrastructure & Data)
*Focus: Establishing the source of truth and type safety.*

1.  **Strict Schema Migration**:
    *   Use `golang-migrate` to manage versions.
    *   Implement UUIDs for primary keys to prevent ID enumeration attacks.
    *   **Pro Tip**: Add a `metadata` JSONB column to `test_results` early for future-proofing.
2.  **The Repository Pattern**:
    *   Create a clean interface for database operations in `internals/db`.
    *   Use `pgxpool` for efficient connection management.
    *   Implement context-aware queries to ensure no "leaked" connections.
3.  **Configuration Management**:
    *   Implement a multi-environment config loader in `internals/config` (Development, Staging, Production).
    *   Strictly use Environment Variables for secrets (DB URLs, Bot Tokens).

---

## Phase 2: Core Logic (Staff API & Bot State Machine)
*Focus: Functionality and User Experience.*

1.  **Staff API (Gin)**:
    *   Implement **Idempotent** endpoints for result uploads (prevent double-entry).
    *   Standardize JSON responses (Error codes, request IDs).
    *   Add basic API Key authentication for the hospital dashboard.
2.  **Telegram Bot (State Machine)**:
    *   Design the bot as a **Finite State Machine (FSM)**.
    *   Handle "Contact Sharing" via Telegram's native button vs. "Manual Passcode" entry.
    *   **The Logic**: 
        *   `Compare(SharedPhone, RegisteredPhone)` 
        *   If mismatch -> `AWAIT_PASSCODE`.
3.  **Concurrency Layer**:
    *   Use an internal `Notifier` service. When a result is added, a background goroutine checks if a `telegram_id` exists for that patient and pushes a "New Result Available" message.

---

## Phase 3: Observability (The "Better Every Day" Foundation)
*Focus: Knowing when things break before the user does.*

1.  **Structured Logging**:
    *   Use `uber-go/zap` or `zerolog`. Log in JSON format for easier ingestion by tools like ELK or Datadog.
    *   Include `trace_id` in logs that correlate API requests to DB queries.
2.  **Health Checks & Metrics**:
    *   Add `/live` and `/ready` endpoints.
    *   Export Prometheus metrics (Request duration, error rates, active bot sessions).
3.  **Automated Testing**:
    *   Unit tests for the FSM logic.
    *   Integration tests for the Repository layer using `testcontainers-go` (running a real Postgres in Docker during tests).

---

## Phase 4: Hardening & Security
*Focus: Protecting sensitive medical data.*

1.  **At-Rest Encryption**:
    *   Evaluate if `result_data` needs field-level encryption before being stored in DB.
2.  **Rate Limiting**:
    *   Implement middleware in Gin to prevent brute-forcing the 4-digit passcode.
    *   Apply rate limits on the Telegram bot side to prevent spamming.
3.  **Audit Logs**:
    *   Track *who* (staff member) added/edited a result and *when* a user accessed it via the bot.

---

## Phase 5: Scaling & Evolution (Continuous Improvement)
*Focus: Scaling for thousands of clinics.*

1.  **Distributed Caching**:
    *   Introduce Redis for session management if the bot scales horizontally (multiple instances).
2.  **PDF Generation Worker**:
    *   Offload PDF generation to a separate background service/worker using a message queue (e.g., RabbitMQ or NATS) if the load increases.
3.  **CI/CD Pipeline**:
    *   Automate linting (`golangci-lint`) and vulnerability scanning (`govulncheck`) on every PR.
4.  **Feedback Loop**:
    *   Implement a "Rate this experience" button in the bot after result delivery to collect NPS and iterate on the UI/UX.

---

### Architect's Closing Note:
"A platform is only as good as its reliability. Do not build the shiny features before you have the logs and tests to prove they work. Medical data requires a 99.9% reliability mindset."

*Antigravity AI Architecture Team*
