# Hearo Backend Architecture & Workflow Documentation

Welcome to the **Hearo Backend** repository documentation. This project is built using **.NET 8** following the **Clean Architecture** paradigm. It acts as the core engine for an AI-powered social podcast and metaphysics platform, orchestrating heavy AI jobs, automated payment scanning, media content streaming, and complex astrological data handling.

---

## 🏗️ 1. Architecture Overview

The system strictly decouples concerns into four concentric layers to maintain high testability, scalability, and independence from external frameworks:

1. **Domain:** Enterprise core business models, entities, and enums (e.g., `User`, `AstrologyProfile`, `Podcast`, `Meditation`).
2. **Application:** System workflows, repository interfaces, and core DTOs/use cases.
3. **Infrastructure:** External integrations, persistence logic (`HearoDbContext`), cloud storage (`S3StorageService`), authentication (`JwtTokenGenerator`), and background queue processors.
4. **WebAPI (Presentation):** REST Controllers, API routing, custom middleware, and rate-limiting enforcement.

---

## 🔄 2. Core System Workflows (System Luồng)

### 2.1. Throttled AI Astrology & Divination Processing Flow
To safeguard the system's performance and prevent hitting external API rate limits (e.g., Google Gemini / CrewAI limits), AI processing utilizes an **in-memory Bounded Channel Queue** coupled with a semaphore-throttled background worker.

```
 [Client Request] 
        │
        ▼ (Controller)
 [Enqueue Job into Bounded Channel] ──► (Cap: 1000 messages, non-blocking Producer)
        │
        ▼
 [AiJobQueue (In-Memory Queue)]
        │
        ▼ (Consumer loop picks up job immediately)
 [AiWorkerService] ──► [Task.Run (Fire-and-Forget Thread)]
                             │
                             ▼
                     [SemaphoreSlim(3)] ──► Max 3 concurrent jobs running
                             │
                             ▼
                  [Call Python CrewAI Endpoint] ──► (Timeout: 3 mins, takes 20-40s)
                             │
                             ▼ (On Success)
                  ┌──────────┴──────────┐
                  ▼                     ▼
       [Update Database Entity]  [Deduct Wallet Balance]
       (BaziChart / TuViChart /    (If user is not an Admin)
        IChingDivination Text)          │
                  │                     │
                  └──────────┬──────────┘
                             │
                             ▼
                 [Create In-App Notification]
```

#### Detailed Flow Steps:
1. **Initiation:** The client requests an astrological reading (Bát Tự, Tử Vi) or gieo quẻ (IChing Divination).
2. **Buffering:** The controller builds an `AiJobMessage` containing user context, metadata, and cost pricing, pushing it to the `AiJobQueue`.
3. **Throttling Strategy:** The `AiWorkerService` pulls the message from the channel. It spawns a decoupled task governed by a `SemaphoreSlim(3)`. This limits maximum concurrency to 3 parallel requests, protecting core server resources.
4. **AI Generation:** The worker uses an optimized `HttpClient` configured with a 3-minute timeout to make a POST request to the Python CrewAI microservice endpoints (`/api/tuvi-reading`, `/api/iching-reading`, or `/api/ai-reading`) passing along the requested persona configurations (*e.g., traditional, GenZ*).
5. **State Finalization:**
   * **Success:** It saves the resulting Markdown response into the targeted table (`BaziChart.AiReadingText` or `IChingDivinations.AiReadingText`), subtracts the service fee from the `User.WalletBalance`, and publishes an instant success notification.
   * **Failure:** It logs the failure stack trace, alerts the user via notifications that the transaction was canceled safely, and guarantees **zero balance deduction**.

---

### 2.2. Automated Bank Email Deposit Flow (VietQR Automation)
Instead of relying entirely on heavy third-party webhooks, the infrastructure provides an asynchronous, direct **IMAP Email Scanner Worker** that automates balance top-ups by reading encrypted official transaction alerts.

```
       [IMAP Connection] ──► Every 15 seconds to imap.gmail.com
              │
              ▼
    [Filter Unseen Emails] ──► Matches Bank Address (e.g., Vietcombank)
              │
              ▼
     [Regex Extraction] 
     ├── Amount:  @"\+([\d,\.]+) VND"
     └── Bill ID: @"BILL ([A-Z0-9]{8,12})"
              │
              ▼
  [Idempotency & Anti-Fraud Check] ──► Verifies if Transaction ID "MAIL_{id}" already exists
              │
              ▼ (If Unique)
   [Credit User Balance] ──► Matches Bill code prefix to User UUID
              │
              ▼
   [Mark Email as SEEN]  ──► Ensures zero double-processing loops
```

#### Key Implementation Logic:
* **Security:** The transaction process is securely bound to an asynchronous database execution context (`ExecuteUpdateAsync`).
* **Idempotency Safeguard:** To prevent race conditions or duplicate top-up attacks if an email is parsed twice, a unique constraint is validated on the database index of the `Payment` table.

---

### 2.3. S3 / Cloudflare R2 Media Management Flow
Streaming static audio files for podcast episodes and guided meditations demands exceptional throughput and cloud-storage cost efficiency.

```
 [File Stream Input] ──► [S3StorageService]
                                │
                                ▼
                   [TransferUtilityUploadRequest]
                                │
                                ▼ (Crucial Performance Flag)
                   [DisablePayloadSigning = true] ──► Bypasses AWS checksum calculation
                                │                     Optimized specifically for Cloudflare R2
                                ▼
                   [Return Signed CDN Public URL] ──► Format: {PublicUrl}/{FileName}
```

---

## 🗄️ 3. Database Schema & Complex Relations

The data persistence design leverages **Entity Framework Core Fluent API Configurations** inside `HearoDbContext` to control advanced relationships, custom constraints, and custom table performance indexing:

### 3.1. Cascading & Cyclic Delete Restrictions
To prevent default SQL Server cyclical reference errors during deletion cascades, specific relationships are governed by custom delete boundaries:
* **Comment Tree Hierarchy:** The `Comment` entity uses a recursive self-referencing hierarchy (`ParentCommentId` mapped to `Replies`). Deleted parent entries are restricted via `DeleteBehavior.Restrict` to protect sub-threads.
* **Social Reactions:** `CommentReaction` connects both `Comment` and `User`. The user path is configured with `DeleteBehavior.Restrict` to break cyclical delete loops.
* **Astrology Profiles:** `User` has a 1-N relationship with `AstrologyProfile`. Deleting a user account cleanly cascades (`DeleteBehavior.Cascade`) to erase all personal sub-charts (`TuViChart`, `BaziChart`) to respect absolute data privacy.
* **Astrology Daily Insights:** If an administrative operator deletes a `Podcast` or a `Meditation` that happened to be dynamically linked to a user's daily lucky recommendation block (`DailyAstrologyInsight`), the link smoothly drops to `NULL` via `DeleteBehavior.SetNull`, preventing crash errors.

### 3.2. Precision & Index Mapping Optimizations
* **Financial Data Security:** Every entity handling currency data (`Course.Price`, `Course.SalePrice`, `Payment.Amount`, `User.WalletBalance`) is strictly mapped to `decimal(18,2)` to eliminate floating-point rounding discrepancies.
* **Performance Query Indexing:** High-traffic foreign keys and lookup parameters are optimized via composite database indices:
  * Unique index on `Payment.TransactionId` to enforce strict payment integrity.
  * Unique index on `Blog.Slug` and `User.Email`.
  * Composite index on `AstrologyProfile` (`UserId`, `IsPrimaryProfile`) for lightning-fast profile switching.
  * Composite index on `Subscription` (`UserId`, `EndDate`) to optimize rapid real-time VIP membership authorization middleware.

---

## 🚀 4. Seeding Specifications (Core Media Content)
Upon database creation (`DbInitializer.Seed`), the backend instantiates default technical categories, administrative accounts, and populates **6 core targeted meditation sessions**:

| Title | Target Category | Audio URL Type | Core Purpose Description |
| :--- | :--- | :--- | :--- |
| **3 Phút Hạ Nhiệt Stress** | Quick | SoundHelix Streaming MP3 | Instant emergency calming during heavy project deadlines or academic pressure. |
| **Hít Thở Tỉnh Thức** | Quick | SoundHelix Streaming MP3 | Breath focus adjustment exercises tailored before intensive programming blocks. |
| **Chữa Lành Nội Tâm** | Deep | SoundHelix Streaming MP3 | 432Hz deep frequency structural therapy session mapping emotional stability. |
| **Mưa Đêm Trên Mái Tôn** | Sleep | SoundHelix Streaming MP3 | High-fidelity environmental noise mapping designed to improve REM cycles. |
| **Tiếng Sóng Biển Rì Rào** | Sleep | SoundHelix Streaming MP3 | Extended ocean soundscapes to handle extreme insomnia. |
| **Tư Duy Alpha Siêu Cấp** | Focus | SoundHelix Streaming MP3 | Alpha wave brainwave manipulation to boost cognitive synthesis and study focus. |

---
*Documentation Compiled & Validated for the Hearo .NET Ecosystem Deployment Framework.*
