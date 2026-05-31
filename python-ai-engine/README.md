# Hearo AI & Astrology Microservice (Python Engine)

Welcome to the **Hearo Astrology & AI Microservice** documentation. This service is built with **Python (FastAPI)** and acts as a dedicated computational and AI-inference engine for the PodSphere ecosystem. It offloads heavy astrological calculations and Large Language Model (LLM) generation from the main .NET backend, adhering to a strict **Microservices Architecture**.

---

## 🚀 1. Tech Stack & Core Libraries

* **Framework:** FastAPI (High-performance asynchronous REST framework)
* **AI & LLM Orchestration:** `crewai` (Agentic workflows) & Google Gemini (`genai`)
* **Testing:** `pytest` (Strict unit testing for astrological rules)
* **Realtime Sync:** Firebase Realtime Database integration
* **Astrology Core:** Custom-built Python library (`lasotuvi`) for Ephemeris (EPHEM) mapping and Lunar Calendar transformations.

---

## ⚙️ 2. System Architecture & Workflows

This microservice is divided into two primary subsystems: **The Deterministic Astrology Engine** and **The Non-Deterministic AI Agent Service**.

### 2.1. The Deterministic Astrology Engine (`server.py` & `lasotuvi/`)
This module handles absolute mathematical calculations to construct Bazi (Bát Tự) and Zi Wei Dou Shu (Tử Vi) charts.

```
 [.NET Core Backend] 
        │ (HTTP POST Request)
        ▼ 
  [FastAPI Endpoints] ──► /api/lap-laso
        │
        ▼
 [lasotuvi Engine]
  ├── Time/Calendar Conversion (Lich_EPHEM, Lich_HND)
  ├── Thien Ban Setup (Can Chi mapping)
  └── Dia Ban Array Initialization
        │
        ▼
 [Advanced Star Placement Algorithms]
  ├── Lộc Tồn & Sát Tinh routing (Tuần/Triệt detection)
  └── Vòng Hỏa Tinh - Linh Tinh (Directional mapping based on Yin/Yang)
        │
        ▼ (JSON Payload returned)
 [.NET Core Backend]
```
* **Key Features:**
  * Maps 12 palaces (Cung) and 100+ stars accurately based on user birth data.
  * Dynamically calculates Annual Stars (Sao Lưu) like Lưu Thái Tuế, Lưu Lộc Tồn.
  * Differentiates main stars (Chính Tinh) and auxiliary stars (Phụ Tinh), identifying blocking factors like Tuần/Triệt.

### 2.2. The AI Agent Service (`agent_service.py`)
Instead of static text outputs, this module uses **CrewAI** and **Gemini Flash** to generate highly personalized, human-like metaphysical readings.

```
 [.NET Worker Service] ──► (Trigger Async Reading)
        │
        ▼
  [FastAPI AI Endpoint] ──► /api/ai-reading / /api/compatibility
        │
        ▼
 [Context Aggregator] ──► Fetches Realtime User State from Firebase DB
        │                 (e.g., current stress level, journal mood)
        ▼
 [Prompt Engineering Engine]
  ├── System Instruction: Enforces persona constraints
  └── Dynamic Prompt Injection:
      - Interpersonal Sync (Phân tích chéo Cung Phu Thê/Nô Bộc)
      - Power Dynamics & Shared Flaws
      - Karmic Resolution (Xu cát tị hung)
        │
        ▼
 [Google Gemini API] ──► Multi-turn LLM generation
        │
        ▼ (Returns rich Markdown Analysis)
 [.NET Core Backend]
```
* **Key Features:**
  * **Cross-Analysis (Compatibility):** Takes two independent user charts and cross-analyzes their interpersonal palaces (Marriage/Friendship).
  * **Actionable Output:** Focuses on real-world resolution (*Xu cát tị hung*) rather than purely theoretical astrology.
  * **Stateless Operation:** Operates independently, returning success/failure statuses seamlessly back to the C# message queue.

---

## 🧪 3. Quality Assurance & Testing (`test_laso.py`)

Astrological mapping requires extreme precision. A single miscalculated step shifts the entire chart. We employ `pytest` to run deterministic assertions on the internal logic.

* **Loc Ton & Sat Tinh Constraints:** Validates that `Đại Hao`, `Tiểu Hao`, and `Phục Binh` are correctly routed based on Yin/Yang (Âm/Dương) and gender configurations (e.g., Dương Nam rules vs Âm Nữ rules).
* **Element-Specific Start Points:** Tests the precise starting sectors for Fire (Hỏa Tinh) and Bell (Linh Tinh) stars derived from the Dần-Ngọ-Tuất triads.

---
*Maintained by the PodSphere Core Engineering Team.*
