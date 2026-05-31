# 🌙 Hearo Astrology Gateway (Node.js Math Engine)

Welcome to the **Astrology Mathematical Gateway** documentation. Built with **Node.js (v22) and Express.js**, this high-throughput microservice acts as the deterministic computational backbone of the PodSphere ecosystem. It isolates heavy mathematical modeling, Solar/Lunar calendar conversions, and I-Ching (Kinh Dịch) combinatorial logic from the main .NET business layer.

---

## 🛠️ 1. Architecture & Tech Stack
* **Framework:** Node.js + Express.js (Stateless API design for high horizontal scalability).
* **Core Libraries:** `lunar-javascript` (High-precision Ephemeris and Calendar Engine), `axios` (HTTP Inter-service communication).
* **Design Patterns Implemented:** * **API Gateway / BFF (Backend for Frontend):** Aggregates and formats astrology data.
  * **Interceptor Pattern:** Middle-layer sanitization of AI/External payloads.

---

## ⚙️ 2. Deep Dive: Core Subsystems & Algorithms

### 2.1. Advanced Bazi (Four Pillars) Algorithm Pipeline (`baziService.js`)
This module dynamically calculates the absolute destiny pillars using precision Solar Terms (Tiết Khí) rather than basic lunar dates.

* **Input:** Standard Gregorian datetime and geographical timezone.
* **Processing Pipeline:**
  1. **Pillar Construction:** Derives the Heavenly Stems (Thiên Can) and Earthly Branches (Địa Chi) for the Year, Month, Day, and Hour pillars.
  2. **Dynamic Ten Gods Matrix (Ma trận Thập Thần):** Automatically maps the Day Master (Nhật Chủ) against the other 7 characters to derive critical interpersonal and psychological relationships (*e.g., Tỷ Kiên, Kiếp Tài, Thực Thần, Thương Quan, Chính/Thiên Tài, Chính Quan/Thất Sát, Chính/Thiên Ấn*).
  3. **Day Master Strength Evaluation (Độ Vượng Suy):** Computes the exact weight of the elements based on the Month Command (Lệnh tháng - Seasonal influence) to programmatically determine if the chart is *Thân Vượng* (Strong) or *Thân Nhược* (Weak).
  4. **Symbolic Stars Traversal (Thần Sát):** Executes array-scanning algorithms across the Earthly Branches to identify auxiliary stars such as *Đào Hoa* (Peach Blossom), *Thiên Ất Quý Nhân* (Noble Guardian), *Văn Xương* (Academic Star), and *Dịch Mã* (Traveling Horse).

### 2.2. Deterministic I-Ching Divination Engine (`ichingService.js`)
A highly complex module handling two distinct forms of I-Ching divination logic with zero AI hallucination:

1. **Plum Blossom Divination (Mai Hoa Dịch Số):**
   * Employs chronological modulo arithmetic.
   * `(Year + Month + Day) % 8` generates the Upper Trigram (Ngoại Quái).
   * `(Year + Month + Day + Hour) % 8` generates the Lower Trigram (Nội Quái).
   * `(Year + Month + Day + Hour) % 6` determines the moving line (Hào Động).

2. **Six-Coin Toss Divination (Lục Hào):**
   * **Input:** An array of 6 integer tosses (e.g., `[6, 7, 8, 9, 7, 8]` representing Old/Young Yin and Yang).
   * **Output Generation:** Dynamically builds the Primary Hexagram (Quẻ Chính), calculates the Mutual Hexagram (Quẻ Hỗ), and derives the Mutated Hexagram (Quẻ Biến).
   * **Advanced Hexagram Mapping:** Automatically calculates highly specific metaphysical traits:
     * *Palace Identification (Tầm Cung)*
     * *Subject/Object lines (Thế/Ứng)*
     * *Missing Spirits (Phục Thần)*
     * *Empty Branches (Tuần Không)*
     * *The Six Beasts (Lục Thú - Thanh Long, Chu Tước, Câu Trận, Đằng Xà, Bạch Hổ, Huyền Vũ)* based on the day's Heavenly Stem.

### 2.3. The Python Interceptor & Data Sanitization Flow
To ensure 100% data integrity, this Node.js service intercepts responses from the Python AI microservice to patch missing or mistranslated esoteric data.

* **The Problem:** Third-party libraries or AI LLMs occasionally return miscalculated or hallucinated Na-Yin (Nạp Âm) strings due to contextual limitations.
* **The Solution:** The `/api/astrology/info` route acts as a middleware interceptor. It makes an outbound call to the Python service. Upon receiving the JSON payload, Node.js recalculates the Na-Yin using a strict, internal hardcoded dictionary mapping (`NAP_AM_MAP`).
* **The Result:** It forcefully overwrites any anomalies, ensuring the .NET Backend and the Frontend receive perfectly sanitized and astrologically accurate data.

---

## 📡 3. Internal API Routing

| Endpoint | Method | Role |
| :--- | :--- | :--- |
| `/api/bazi/calculate` | `POST` | Returns the fully mapped 4-Pillar chart with Ten Gods and Symbolic Stars. |
| `/api/iching/toss` | `POST` | Processes the 6-integer array and returns the Hexagram JSON structure. |
| `/api/astrology/info` | `GET` | Interceptor route fetching from Python, patching Na-Yin, and returning sanitized data. |

---
*Developed & Maintained by the PodSphere Core Engineering Team.*