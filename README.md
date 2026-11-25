####project_video 

https://drive.google.com/file/d/17vnjTxd7HzTpCvCokZ8SG4fzpwJG8E10/view?usp=sharing

# CineFlow v2 — Multiplex Movie Ticket Reservation System

CineFlow v2 is an enterprise-grade desktop application for multiplex movie ticket management built using **Java 21**, **JavaFX 21** (Pure Java, no FXML), **Maven**, and **MySQL 8.x** via pure **JDBC**.

CineFlow v2 introduces a **modern glassmorphic design system** with smooth 60fps animations, daemon concurrency offloading (`AsyncUi`), and **5 technically deep advanced features**:
1. **Personalised Movie Recommendations & Taste Profile Onboarding** (Exponential recency decay, composite multi-attribute scoring, diversity anti-clustering re-ranking, and explainability tags).
2. **Automatic Best-Seat Allocation ("Smart Pick")** (Acoustic and sightline distance modeling, group contiguous sliding-windows, and orphan seat avoidance).
3. **Automated FIFO Waitlist System** (Two-phase seat offer holds, background expiry sweeping, in-app notification center, and cancellation trigger reallocation).
4. **Cryptographic QR-Code Digital Tickets & Entry Gate Validator** (Tamper-proof HMAC-SHA256 tokens, ZXing QR generator, and atomic gate admission with anti-double-entry auditing).
5. **Rule-Based Dynamic Pricing & Locked Quotes** (Demand surges, time-to-show discounts, weekend/prime hour stacking, bounding clamps, nearest-₹5 rounding, and live interactive manager simulator).

---

## Technology Stack & Architecture

| Component | Technology | Description |
|---|---|---|
| **Language** | Java 21 (LTS) | Modern Java with records, pattern matching, `BigDecimal` money |
| **UI Framework** | JavaFX 21.x | Desktop GUI built in pure Java code (**Zero FXML**) |
| **QR Code Engine** | ZXing 3.5.3 | High-contrast QR image matrix generation, PNG export, and decoding |
| **Database** | MySQL 8.x | Relational DBMS using InnoDB engine for ACID compliance |
| **Persistence** | JDBC (`java.sql`) | Strictly `PreparedStatement` queries and transactional Connections |
| **Concurrency** | Custom `AsyncUi` & `BackgroundJobs` | Daemon thread pools keeping the JavaFX Application Thread at 60 FPS |
| **Styling** | JavaFX Looked-up Tokens | Dark glassmorphism (`src/main/resources/css/styles.css`) |
| **Testing** | JUnit 5 | Isolated unit tests for pure algorithms + integration tests |

> **Strict Architectural Boundary**: CineFlow is a standalone desktop application. It intentionally uses no heavyweight frameworks or ORMs (no Spring Boot, Hibernate, JPA, Lombok, or web wrappers) to demonstrate mastery of core Java, JDBC, and object-oriented principles.

---

## Database Architecture (`cineflow_db`)

The database consists of **16 normalized InnoDB tables**:

```
[users] ────────┬──────────< [bookings] ──────────< [booking_seats] ──────────< [tickets]
                ├──────────< [user_preferences]                                     │
                ├──────────< [waitlist_entries]                                     ▼
                ├──────────< [notifications]                                  [entry_scans]
                └──────────< [price_quotes]
                                  │
[movies] ───────┬──────────< [showtimes] ─────────< [seat_holds]
                │                 │
[screens] ──────┴──────────< [seats]
                                  │
                             [pricing_rules]
```

1. **`users`**: Customer and Admin accounts with salted SHA-256 credentials.
2. **`movies`**: Active catalog with genre, language, duration, and ratings.
3. **`screens`**: Auditoriums and layouts (IMAX, Dolby Cinema).
4. **`seats`**: Grid seats (Regular, Premium) with unique coordinate constraints.
5. **`showtimes`**: Screenings with baseline regular and premium prices.
6. **`bookings`**: ACID reservation transactions with pricing calculation snapshots.
7. **`booking_seats`**: Unique seat allocations per showtime.
8. **`seat_holds`**: Temporary checkout holds preventing race conditions.
9. **`payments`**: Payment records (`PAID`, `REFUNDED`) via CARD, UPI, or CASH.
10. **`pricing_rules`**: Configurable dynamic pricing adjustment factors.
11. **`price_quotes`**: Locked price quotations valid during checkout.
12. **`waitlist_entries`**: FIFO customer waitlist queue for sold-out showtimes.
13. **`notifications`**: In-app notifications and actionable waitlist seat offers.
14. **`user_preferences`**: Customer taste profiles for personalized recommendations.
15. **`tickets`**: Cryptographically signed digital ticket passes with unique tokens.
16. **`entry_scans`**: Gate scan audit log recording admissions, double entries, and rejections.

---

## 5 Core Advanced Features

### 1. Personalised Movie Recommendations
- **Exponential Recency Decay:**
  $$\text{Weight}(\Delta t) = e^{-\lambda \cdot \Delta t}, \quad \lambda = \frac{\ln(2)}{30 \text{ days}}$$
- **Composite Scoring:**
  $$\text{Score}(m) = 0.40 \cdot G(m) + 0.20 \cdot L(m) + 0.20 \cdot \frac{R(m)}{10} + 0.10 \cdot T(m) + 0.10 \cdot P(m)$$
- **Diversity Anti-Clustering Re-Ranker:** Limits top recommendations to a maximum of 2 movies of the same genre.
- **Explainability:** Generates human-readable badges (e.g. *"Because you loved Interstellar"*).

### 2. Smart Pick Best-Seat Allocation
- **Sightline Scoring:**
  $$\text{Score}(s) = 0.35 \cdot S_{\text{centre}}(s) + 0.35 \cdot S_{\text{view}}(s) + 0.15 \cdot S_{\text{cat}}(s) + 0.15 \cdot S_{\text{pref}}(s)$$
- **Orphan Avoidance Penalty:** Applies a $-25$ penalty to group allocations that leave single isolated empty seats.
- **Multi-Row Fallback:** Falls back from contiguous blocks to split clusters and stacked adjacent rows.

### 3. Automated FIFO Waitlist Engine
- **Concurrency-Safe Queue:** Evaluates FIFO queue entries ordered by `position_ts ASC`.
- **Triggered Reallocation:** Reallocates seats immediately on booking cancellation and via 15-second background daemon sweeps.
- **Two-Phase Offer Holds:** Places temporary 10-minute holds in `seat_holds` and alerts the guest via the top-navigation **Notification Bell** and slide-in glass drawer.

### 4. Cryptographic QR Tickets & Entry Gate
- **Tamper-Proof Token Codec:**
  $$\text{CF1}.\langle\text{ticketId}\rangle.\langle\text{nonce}_{16}\rangle.\langle\text{HMAC-SHA256}(\text{payload}, K)_{16}\rangle$$
- **Anti-Timing Attack:** Employs constant-time signature comparison (`MessageDigest.isEqual`) before querying the database.
- **Atomic Gate Admission:** Single-query atomic state update (`UPDATE tickets SET status='USED'... WHERE status='VALID'`).

### 5. Rule-Based Dynamic Pricing & Locked Quotes
- **Additive Factor Stacking:**
  $$\text{FinalPrice} = \max\left(₹50.00, \text{roundToNearest5}\left(\text{clamp}(\text{Adjusted}, \text{Floor}, \text{Cap})\right)\right)$$
- **Pure Arithmetic:** Strict `BigDecimal` arithmetic with `RoundingMode.HALF_UP`.
- **Live Simulator:** Admin tab featuring interactive sliders for occupancy, time-to-show, and screening hours with live real-time price roll-up.

---

## Evaluation Credentials

| Role | Username | Password | Key Demo Features |
|---|---|---|---|
| **ADMIN** | `admin` | `Admin@123` | Entry Gate Scanner, Dynamic Pricing Simulator, Waitlist Monitor |
| **CUSTOMER** | `john_doe` | `Customer@123` | Personalised "For You" Carousel, Smart Pick, Digital Pass |
| **CUSTOMER** | `jane_smith` | `Customer@123` | Active Waitlist Queue Candidate |

---

## Quick Start & Verification (Team Clone Guide)

If you just cloned this repository, follow these 3 simple steps to get CineFlow running on your machine:

### 1. Prerequisites
- **Java 21 (JDK 21 LTS)** installed and set in your `JAVA_HOME`.
- **Maven 3.8+** installed (`mvn -v`).
- **MySQL 8.x** running locally on port 3306.

### 2. Database Initialization
Open your terminal in the project root folder and run:
```bash
mysql -u root -p < database/schema.sql
mysql -u root -p < database/seed_data.sql
```
*(Enter your MySQL root password when prompted. This creates `cineflow_db` with all 23 tables and demo seed data).*

### 3. Database Credentials Configuration
Copy `db.properties.example` to `db.properties` and enter your local MySQL connection settings:
```properties
db.url=jdbc:mysql://localhost:3306/cineflow_db?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=Asia/Kolkata&characterEncoding=UTF-8
db.user=root
db.password=YOUR_MYSQL_PASSWORD
```
For Razorpay gateway checkout and refunds, set `razorpay.key.id` and `razorpay.key.secret` in your private `db.properties`. The instant test payment runs locally with a demo or Razorpay test configuration.

### 4. Run Test Suite
Verify that all 81 automated tests pass on your machine:
```bash
mvn clean compile test
```

### 5. Launch the Application
```bash
mvn javafx:run
```

### 6. Quick Demo Logins
- **Admin**: `admin` / `Admin@123` (or click the 1-click **"⚡ Quick Admin Demo"** button on the login screen)
- **Customer**: `john_doe` / `Customer@123` (or click **"⚡ Quick Customer Demo"**)

For the complete under-60-second viva presentation script and faculty Q&A defenses, refer to [DEMO_SCRIPT.md](docs/DEMO_SCRIPT.md).
