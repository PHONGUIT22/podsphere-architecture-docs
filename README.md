# 🌌 PodSphere Polyglot Microservices Ecosystem

Welcome to the ultimate architectural and technical specification registry for the **PodSphere Backend Ecosystem**.

PodSphere is an advanced, high-performance, AI-powered social podcast and metaphysical analysis platform. Due to the intellectual property and proprietary nature of the production source code, this repository acts as an exhaustive **Technical Showcase** mapped out to demonstrate system design patterns, distributed microservices communications, multi-language integration, and complex algorithmic processing pipelines.

---

## 🏗️ 1. Global Architecture Mapping

The ecosystem operates on a **Polyglot Microservices Design**, deliberately selecting the optimal programming language and runtime framework for distinct computational domains:

```text
┌───────────────────────────────────┐
                               │        Client Application         │
                               │       (Mobile iOS/Android)        │
                               └─────────────────┬─────────────────┘
                                                 │
                                                 │ HTTPS (REST / JWT Auth / Rate-Limited)
                                                 ▼
    ┌─────────────────────────────────────────────────────────────────────────────────────┐
    │                        .NET 8 CORE API ENGINE [BFF & Core]                          │
    │  - Workflows | Identity & Security (JWT/OAuth) | Data Storage & S3 Management       │
    │  - Financial Transactions (Stripe) | Automated IMAP VietQR | In-Memory Job Channels │
    └─────────────────┬─────────────────────────────────────────┬─────────────────────────┘
                      │                                         │
                      │ Async Queue Processing                  │ Synchronous HTTP REST
                      │ (Throttled by SemaphoreSlim)            │ (Internal JSON Payload)
                      ▼                                         ▼
    ┌───────────────────────────────────┐     ┌───────────────────────────────────┐
    │     PYTHON AI MICROSERVICE        │     │     NODE.JS ASTROLOGY GATEWAY     │
    │          [Inference]              │     │              [Math]               │
    │ - FastAPI | CrewAI Orchestration  │     │ - Express.js (v22) | Lunar-JS     │
    │ - Gemini Flash Semantic Analysis  │◄────┤ - BaZi & I-Ching Combo Math       │
    │                                   │     │ - Outbound Interceptor (Na-Yin)   │
    └─────────────────┬─────────────────┘     └───────────────────────────────────┘
                      │
                      │ Read Live Context
                      ▼
    ┌───────────────────────────────────┐
    │    FIREBASE REALTIME DATABASE     │
    │ - Synchronized User Mental State  │
    │ - Health Logs                     │
    └───────────────────────────────────┘

### Core Engineering Components
1. **[.NET 8 Core API Engine (C#)](./dotnet-core-api/):** Governs the overarching application context. It serves as the Backend-for-Frontend (BFF), securing endpoints, managing transactional continuity, persisting data via Entity Framework Core, and queuing asynchronous heavy processing.
2. **[Python AI Microservice (Python)](./python-ai-engine/):** Operates exclusively as an isolated AI inference server. It encapsulates generative AI logic, utilizes multi-agent systems, and handles semantic generation using large language models (LLMs).
3. **[Node.js Astrology Gateway (JavaScript)](./nodejs-astrology-gateway/):** Operates as a deterministic mathematical calculation server, rendering high-speed, zero-hallucination astrological and calendar data transformations.

---

## 📡 2. Inter-Service Communication Matrix

The three microservices communicate through a hybrid model combining **Synchronous HTTP REST Protocols** for immediate data fetching and **Asynchronous Thread-Throttled Channels** for heavy workloads.

### 2.1. The Inter-Service Communication Registry

| Source Service | Target Service | Protocol | Payload Format | Communication Pattern | Purpose |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **.NET Core** | **Node.js Gateway** | HTTP POST / GET | Application/JSON | Synchronous (Blocking) | Fetching deterministic BaZi charts and I-Ching coin-toss layouts. |
| **.NET Core** | **Python AI Service** | HTTP POST | Application/JSON | Asynchronous (Queued Producer) | Offloading AI reading requests (`TuVi`, `BaZi`, `IChing`). |
| **Node.js Gateway** | **Python AI Service** | HTTP GET | Application/JSON | Synchronous Interceptor | Fetching raw insights and overwriting error-prone Na-Yin fields. |
| **Python AI Service** | **Firebase RTDB** | Native SDK | Document/JSON Stream | Real-time Synchronization | Merging live psychological states (stress/mood) into AI prompts. |

---

## 🔄 3. End-to-End Core Workflows (System Flowcharts)

### 3.1. Asynchronous Throttled AI Reading Flow
This pipeline ensures that a sudden surge in consumer requests does not crash the internal resources or exceed Google Gemini API limits.

```text
[Client App]             [.NET Controller]          [In-Memory Channel]         [AiWorkerService]        [Python FastAPI]
     │                           │                           │                          │                        │
     │─── POST /api/reading ────►│                           │                          │                        │
     │                           │─── Enqueue Job Message ──►│                          │                        │
     │◄── HTTP 202 Accepted ─────│                           │                          │                        │
     │                           │                           │─── Dequeue (Task.Run) ──►│                        │
     │                           │                           │                          │── Acquire Semaphore ──►│ (Slot 1 of 3)
     │                           │                           │                          │─── POST /ai-reading ──►│
     │                           │                           │                          │                        │─── Processing
     │                           │                           │                          │◄── Return Markdown ────│    (20-40s)
     │                           │                           │                          │── Release Semaphore ───│
     │                           │                           │                          │                        │
     │                           │◄─── Persist Data & Notif ────────────────────────────│                        │
     │◄── Realtime Notification ─│                           │                          │                        │
```

### 3.2. Automated Bank Deposit Flow (VietQR Processing)
An completely autonomous payment settlement layer removing manual verification errors.

```text
[Bank System]              [Gmail Server]           [EmailScannerWorker]        [PodsphereDbContext]          [Client App]
      │                           │                          │                          │                      │
      │── Cash Deposit Email ────►│                          │                          │                      │
      │                           │                          │─── Poll IMAP (15s) ─────►│                      │
      │                           │                          │                          │                      │
      │                           │                          │─── Regex Match Valid ───►│                      │
      │                           │                          │─── Check Idempotency ──►│                      │
      │                           │                          │    (Transaction ID)      │                      │
      │                           │                          │                          │                      │
      │                           │                          │─── ExecuteUpdateAsync ──►│                      │
      │                           │                          │    (Credit WalletBalance)│                      │
      │                           │                          │                          │                      │
      │                           │                          │─── Add Success Payment ─►│                      │
      │                           │                          │                          │◄── Pull New Wallet ──│
```

### 3.3. The Data Sanitization Interceptor Pipeline
An architecture designed to enforce 100% accurate data payloads before business state tracking occurs.

```text
[.NET Core Engine]              [Node.js Gateway]              [Python AI Engine]              [Database]
        │                               │                               │                           │
        │─── HTTP GET /astrology/info ─►│                               │                           │
        │                               │── Outbound Gateway Request ──►│                           │
        │                               │                               │── (Fetch Raw Logic)       │
        │                               │◄── Returns Payload Json ──────│                           │
        │                               │                                                           │
        │                               │─── [Calculates Na-Yin]                                    │
        │                               │─── [Overwrites Errors via NAP_AM_MAP]                     │
        │                               │                                                           │
        │◄── Returns Sanitized JSON ────│                                                           │
```

---

## 🧮 4. Algorithmic Domain Breakdown

The system incorporates several complex computational models to ensure data validity across the microservices:

### 4.1. The Modulo Modulating Model (Plum Blossom / Mai Hoa Dịch Số)
The Node.js gateway calculates Hexagram indexing numbers programmatically without relying on structural static arrays. It applies modular arithmetic directly on targeted timestamp clusters:

**Upper Trigram (Ngoại Quái):**
$$UpperTrigram = (\sum LunarYear + LunarMonth + LunarDay) \pmod 8$$

**Lower Trigram (Nội Quái):**
$$LowerTrigram = (\sum LunarYear + LunarMonth + LunarDay + LunarHour) \pmod 8$$

**Moving Line (Hào Động):**
$$MovingLine = (\sum LunarYear + LunarMonth + LunarDay + LunarHour) \pmod 6$$

### 4.2. Ten Gods (Thập Thần) Assignment Mapping Matrix
The Node.js microservice instantiates an explicit mapping array evaluating the polarity relationship between the Day Master Heavenly Stem ($DM$) and all surrounding elements ($E$):

$$	ext{Relationship Matrix} = f(Stem_{DM} \times Stem_{E})$$

This dynamically evaluates factors such as Generation (Sinh), Control (Khắc), and Polarity Alignment (Same Polarity = Biến Thể / Opposite Polarity = Chính Thể) to generate variables like *Thương Quan* versus *Thực Thần*.

### 4.3. High-Traffic Database Constraints Optimization
To ensure optimal query performance under high load, the relational schema enforces advanced database indexing configurations:

* **VIP Authorization Loop:** `Subscription` maintains a composite index on `[UserId, EndDate]`, allowing the authentication middleware to determine premium access eligibility in $O(1)$ time.
* **Cyclic Delete Mitigation:** Recursive relationships within the `Comment` entity (Replies map back to `ParentCommentId`) are explicitly configured with `DeleteBehavior.Restrict` to prevent SQL Server cascade replication lockups.

---
*Verified Deployment Blueprint for the PodSphere Distributed Core Infrastructure Framework.*
