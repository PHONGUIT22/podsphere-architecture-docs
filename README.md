# 🌌 PodSphere Polyglot Microservices Ecosystem

Welcome to the ultimate architectural and technical specification registry for the **PodSphere Backend Ecosystem**.

PodSphere is an advanced, high-performance, AI-powered social podcast and metaphysical analysis platform. Due to the intellectual property and proprietary nature of the production source code, this repository acts as an exhaustive **Technical Showcase** mapped out to demonstrate system design patterns, distributed microservices communications, multi-language integration, and complex algorithmic processing pipelines.

---

## 🏗️ 1. Global Architecture Mapping

The ecosystem operates on a **Polyglot Microservices Design**, deliberately selecting the optimal programming language and runtime framework for distinct computational domains:

```text
                                    ┌───────────────────────────────────┐
                                    │       Client Application          │
                                    │      (Mobile iOS/Android)         │
                                    └─────────────────┬─────────────────┘
                                                      │
                                                      │ HTTPS (REST / JWT Auth / Rate-Limited)
                                                      ▼
    ┌───────────────────────────────────────────────────────────────────────────────────────────────────┐
    │                                   .NET 8 CORE API ENGINE [BFF & Core]                             │
    │  - Business Domain Workflows    - Identity & Security (JWT/OAuth)   - Data Storage & S3 Management │
    │  - Financial Transactions (Stripe) - Automated IMAP VietQR Scanning  - In-Memory Job Channels     │
    └─────────────────┬───────────────────────────────────────────────┬─────────────────────────────────┘
                      │                                               │
                      │ Async Queue Processing                        │ Synchronous HTTP REST
                      │ (Throttled by SemaphoreSlim)                  │ (Internal JSON Payload)
                      ▼                                               ▼
    ┌──────────────────────────────────────────────────┐    ┌──────────────────────────────────────────────────┐
    │          PYTHON AI MICROSERVICE [Inference]      │    │         NODE.JS ASTROLOGY GATEWAY [Math]         │
    │  - FastAPI Endpoints                             │    │  - Express.js Engine (v22 Runtime)               │
    │  - CrewAI Multi-Agent Orchestration              │    │  - Lunar-JavaScript Calendar Transformations      │
    │  - Google Gemini Flash Semantic Analysis         │◄───┤  - Deterministic BaZi & I-Ching Combo Math       │
    └─────────────────┬────────────────────────────────┘    │  - Outbound Interceptor (Na-Yin Correction)      │
                      │                                     └──────────────────────────────────────────────────┘
                      │ Read Live Context
                      ▼
    ┌──────────────────────────────────────────────────┐
    │           FIREBASE REALTIME DATABASE             │
    │  - Synchronized User Mental State / Health Logs  │
    └──────────────────────────────────────────────────┘
Core Engineering Components.NET 8 Core API Engine (C#): Governs the overarching application context. It serves as the Backend-for-Frontend (BFF), securing endpoints, managing transactional continuity, persisting data via Entity Framework Core, and queuing asynchronous heavy processing.Python AI Microservice (Python): Operates exclusively as an isolated AI inference server. It encapsulates generative AI logic, utilizes multi-agent systems, and handles semantic generation using large language models (LLMs).Node.js Astrology Gateway (JavaScript): Operates as a deterministic mathematical calculation server, rendering high-speed, zero-hallucination astrological and calendar data transformations.📡 2. Inter-Service Communication MatrixThe three microservices communicate through a hybrid model combining Synchronous HTTP REST Protocols for immediate data fetching and Asynchronous Thread-Throttled Channels for heavy workloads.2.1. The Inter-Service Communication RegistrySource ServiceTarget ServiceProtocolPayload FormatCommunication PatternPurpose.NET CoreNode.js GatewayHTTP POST / GETApplication/JSONSynchronous (Blocking)Fetching deterministic BaZi charts and I-Ching coin-toss layouts..NET CorePython AI ServiceHTTP POSTApplication/JSONAsynchronous (Queued Producer)Offloading AI reading requests (TuVi, BaZi, IChing).Node.js GatewayPython AI ServiceHTTP GETApplication/JSONSynchronous InterceptorFetching raw insights and overwriting error-prone Na-Yin fields.Python AI ServiceFirebase RTDBNative SDKDocument/JSON StreamReal-time SynchronizationMerging live psychological states (stress/mood) into AI prompts.🔄 3. End-to-End Core Workflows (System Flowcharts)3.1. Asynchronous Throttled AI Reading FlowThis pipeline ensures that a sudden surge in consumer requests does not crash the internal resources or exceed Google Gemini API limits.Plaintext[Client App]             [.NET Controller]          [In-Memory Channel]         [AiWorkerService]        [Python FastAPI]
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
Ingress: Client requests an intensive AI Reading (e.g., Tử Vi, Bát Tự, Kinh Dịch).Decoupling: The .NET Controller instantly encapsulates metadata into an AiJobMessage, pushes it to an in-memory Channel<AiJobMessage> (bounded at 1000 items), and issues an immediate HTTP 202 Accepted back to the UI.Throttling Control: A C# background worker (AiWorkerService) drains the channel. Before making an outbound API call, it requests a token from a SemaphoreSlim(3) instance, ensuring a maximum of 3 concurrent AI requests execute simultaneously.Execution & Persistence: The Python microservice invokes its multi-agent orchestration layer, processes the text using Gemini, and returns the Markdown output. The .NET thread releases the semaphore slot, saves the text directly into the database via ExecuteUpdateAsync, and dispatches an in-app system notification to the client.3.2. Automated Bank Deposit Flow (VietQR Processing)An completely autonomous payment settlement layer removing manual verification errors.Plaintext[Bank System]              [Gmail Server]           [EmailScannerWorker]        [HearoDbContext]          [Client App]
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
Trigger: The consumer scans a dynamically generated VietQR code containing the transaction string format BILL [USER_CODE]. The bank fires an transaction confirmation email to the platform's mailbox.Scanning Loop: The .NET EmailScannerWorker queries imap.gmail.com using a 15-second delay circuit-breaker. It filters for unread messages sent specifically by the bank's official address.Data Extraction & Anti-Fraud: The worker utilizes strict Regular Expressions (\+([\d,\.]+) VND and BILL ([A-Z0-9]{8,12})) to parse the financial value and targeting code. It maps the parsed ID into a string format MAIL_{unique_email_uid} and verifies it against the Payment unique constraints to maintain idempotency and block duplicate injection attacks.Settlement: The wallet balance is programmatically credited using EF Core's database engine, ensuring real-time transaction finality.3.3. The Data Sanitization Interceptor PipelineAn architecture designed to enforce 100% accurate data payloads before business state tracking occurs.Plaintext[.NET Core Engine]              [Node.js Gateway]              [Python AI Engine]              [Database]
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

🧮 4. Algorithmic Domain BreakdownThe system incorporates several complex computational models to ensure data validity across the microservices:4.1. The Modulo Modulating Model (Plum Blossom / Mai Hoa Dịch Số)The Node.js gateway calculates Hexagram indexing numbers programmatically without relying on structural static arrays. It applies modular arithmetic directly on targeted timestamp clusters:$$\text{UpperTrigram} = \left( \sum \text{Lunar Year} + \text{Lunar Month} + \text{Lunar Day} \right) \pmod 8$$$$\text{LowerTrigram} = \left( \sum \text{Lunar Year} + \text{Lunar Month} + \text{Lunar Day} + \text{Lunar Hour Branch} \right) \pmod 8$$$$\text{MovingLine (Hào Động)} = \left( \sum \text{Lunar Year} + \text{Lunar Month} + \text{Lunar Day} + \text{Lunar Hour Branch} \right) \pmod 6$$4.2. Ten Gods (Thập Thần) Assignment Mapping MatrixThe Node.js microservice instantiates an explicit mapping array evaluating the polarity relationship between the Day Master Heavenly Stem ($DM$) and all surrounding elements ($E$):$$\text{Relationship Matrix} = f(Stem_{DM} \times Stem_{E})$$This dynamically evaluates factors such as Generation (Sinh), Control (Khắc), and Polarity Alignment (Same Polarity = Biến Thể / Opposite Polarity = Chính Thể) to generate variables like Thương Quan versus Thực Thần.4.3. High-Traffic Database Constraints OptimizationTo ensure optimal query performance under high load, the relational schema enforces advanced database indexing configurations:VIP Authorization Loop: Subscription maintains a composite index on [UserId, EndDate], allowing the authentication middleware to determine premium access eligibility in $O(1)$ time.Cyclic Delete Mitigation: Recursive relationships within the Comment entity (Replies map back to ParentCommentId) are explicitly configured with DeleteBehavior.Restrict to prevent SQL Server cascade replication lockups.Verified Deployment Blueprint for the PodSphere Distributed Core Infrastructure Framework.