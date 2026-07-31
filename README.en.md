# Jimin Jeon — Backend Developer Portfolio

**English** | [한국어/Korean Version](README.md)

**A backend developer with a data background and the ability to collaborate in English.**

I spent about 1 year and 9 months in data roles — including at National Capital Flag, a US-based company — working on data standardization, data-transformation automation, and data research. I now focus on building servers that behave correctly and reliably under pressure, with an emphasis on payment consistency and load-test-driven performance diagnosis on top of Spring.

**Email** · jeemin065@gmail.com &nbsp;|&nbsp; 💻 **GitHub** · [@jeemin65-pixel](https://github.com/jeemin65-pixel)

> This repository collects two team projects from my backend bootcamp.
> Full source for each project is available in the original team repositories linked below.

---

## Tech Stack

| Area | Details |
|------|---------|
| **Language** | Java 21, SQL (MySQL, PostgreSQL) |
| **Framework** | Spring Boot 3.x / 4.x, Spring Data JPA (JPQL) |
| **Auth / Payments** | OAuth 2.0 (Kakao, Google), Toss Payments |
| **AI / LLM** | OpenAI API |
| **Cloud (hands-on)** | GCP Cloud Functions (2023, data-transformation automation), AWS S3 (backend image-upload integration) |
| **Test / Load** | JUnit, k6 |
| **Collaboration** | Git, Jira |

---

## Projects

### 1. ShoppingFourU — E-Commerce Platform

> An e-commerce backend with cart, order, and payment domains. Using load tests that
> simulate high traffic, I diagnosed a payment-concurrency bottleneck and derived a
> structural fix.

- **Duration / Team**  Oct 2025 – Jan 2026 / 5-person team
- **Original repo**  [kt-cloud-basic-project/shopping](https://github.com/kt-cloud-basic-project/shopping)
- **Stack**  Java 21, Spring Boot 3.5.7, JPA, MySQL, JUnit, OpenAI API, k6

**My role**
- Designed the cart / order domains and implemented their REST APIs
- Integrated an OpenAI-based FAQ chatbot (system-prompt design, vector-store setup)
- Designed and ran k6 load tests

**Key troubleshooting — diagnosing a payment-concurrency deadlock**
- During a k6 load test simulating high traffic, responses stalled at the point of concurrent payment processing.

*Load-test result (Before)*
- 30 VUs (low concurrency), one payment each → **0 successful payments**, with every request timing out at 30 seconds (p95 = 30.02s).
- No stock change (150 → 150, 0 deducted).
- Server log: `Connection is not available, timed out after 30s (total=10, active=10, idle=0, waiting=14)`

*Root cause — connection-pool deadlock*
- Connection A held (`completePayment`) → optimistic-lock (`@Version`) conflict during stock deduction → rollback requests a second connection B via `REQUIRES_NEW` → with the pool (size 10) already exhausted, A is held while B waits → deadlock.
- The cause is the **rollback's `REQUIRES_NEW` structure, not load**. Enlarging the pool only defers it — higher concurrency reproduces the same failure.

*Direction* — redesign the `REQUIRES_NEW` rollback path, plus concurrency control (distributed / pessimistic locking) and a ticketing-style queue; under discussion with the team. *(design stage)*

**AI chatbot integration · test troubleshooting**
- **Resolved a `contextLoads` failure through test isolation** — when `@SpringBootTest` loads every bean, `DefaultVectorApi` (which calls the external OpenAI API) failed to initialize in CI because of a missing API key. I separated the isolation strategy by each bean's role ([PR #160](https://github.com/kt-cloud-basic-project/shopping/pull/160)):
  - `VectorApi` / `ChatApi`, which `FAQService` requires: removing them breaks context loading, so I excluded the real implementations (`DefaultVectorApi`, `DefaultChatApi`) from tests and swapped in `FakeVectorApi` / `FakeChatApi` that make no external calls.
  - Non-essential config beans (`OpenAIConfiguration`, `OpenAICustomAdvisor`): disabled in tests via `@ConditionalOnProperty`.
- **Fixed a DI initialization-order bug** — a `token` derived from `OpenAIProperties` was initialized before dependency injection when using `@RequiredArgsConstructor`; I made the constructor explicit to pin the initialization order ([PR #160](https://github.com/kt-cloud-basic-project/shopping/pull/160)).
- Wrote JUnit unit tests for the cart / order flows to stabilize the service layer.

---

### 2. MindLog AI — Counseling Chatbot Service

> A backend that manages counseling records, credits, and payments. I focused on
> handling money-related flows — payment consistency and credit consumption — correctly.

- **Duration / Team**  Jan 2026 – Apr 2026 / 15-person team
- **Original repo**  [8ocket/Backend](https://github.com/8ocket/Backend)
- **Stack**  Java 21, Spring Boot 4.0.1, JPA, PostgreSQL, OAuth 2.0, AWS S3, JUnit

**My role**
- Designed the AI counseling-record domain and backend (case summary, emotion, action-suggestion management)
- Integrated Toss Payments
- Built the credit system (paid / free credits separated; free credits consumed first)
- Implemented Kakao · Google social login via OAuth 2.0
- Implemented image upload via AWS S3

**Key point — ensuring payment consistency**
- Amount-tampering prevention: **two-stage verification (before / after the PG call) that chains the request, DB, and PG-approved amounts**.
- Duplicate-payment prevention blocks re-requests of the same payment.
- Wrote fixture-based tests for the counseling-summary / credit domains to verify core logic.

**Credit-consistency troubleshooting**
- **Fixed a credit double-deduction bug** — the balance-sum query included REFUND transactions, so credits were deducted twice. I excluded refunds from the sum (`transactionType != 'REFUND'`) to fix it ([PR #101](https://github.com/8ocket/Backend/pull/101)).

**Payment-verification reordering — splitting verification across layers (before / after the PG call)**

- **Problem** — the original flow **approved** the payment via Toss `confirm()` first and verified the amount/status afterward. Tampered amounts and duplicate requests were only caught *after* the PG approval had already gone out, which then required a separate payment cancellation (compensating transaction) — complicating the flow and leaving a consistency risk.
- **Improvement — split verification into two layers:**
  - **Before the PG call**: verify payment status (`READY`), `paymentKey` duplication, and **request amount vs DB-stored amount** *before* calling `tossClient.confirm()` → tampered or duplicate payments never reach the approval request.
  - **After the PG call**: checks that require the Toss response (`status == DONE`, `orderId` match, **DB amount vs Toss-approved amount**) run after approval.
- **Effect** — request amount `==` DB amount `==` Toss-approved amount are chained end-to-end, guaranteeing amount consistency all the way through; invalid requests are filtered before the PG call, removing unnecessary "approve → cancel" round-trips. *(personal refactoring, applied locally)*

---

## Experience

- **National Capital Flag** (Oct 2024 – May 2025): product-data entry and naming-convention design; standardized ~20,000 product records (NetSuite ERP)
- **GMU Schar School** (Feb 2024 – Oct 2024): public-source data collection, duplicate removal, and missing-data completion (Shadow Influence Project)
- **XAI Land** (Feb 2023 – Jun 2023): GCP Cloud Function–based data-transformation automation (CSV → TXT)
- **Teaching Assistant** (Jan 2024 – May 2024): SQL course TA — grading and mentoring

## Education

- George Mason University, B.S. in Computational and Data Sciences (GPA 3.76 / 4.0)
- KT Cloud Tech Up Backend Development Bootcamp (completed)

## Languages

- English (Business) · Korean (Native)
