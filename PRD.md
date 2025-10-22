Here’s a ready-to-paste **`PRD.md`** for your *Digital Advent Calendar (Next.js + Postgres)* project.
You can copy this directly into your repo or Notion canvas for review and iteration.

---

# Digital Advent Calendar – PRD

## 1. Overview

A modern, minimalist, full-screen **digital Advent calendar** that unlocks one tile per day from **30 Nov – 24 Dec 2025**.
Each tile opens with an animation and confetti, revealing an Advent-themed Bible verse and encouragement for the season.

### Primary Goals

* Deliver a faith-centred, interactive Advent experience for family and friends.
* Encourage daily reflection and consistency through streaks and progress tracking.
* Provide secure, seamless login and analytics to understand engagement.

### Core Details

| Item               | Value                                             |
| ------------------ | ------------------------------------------------- |
| **Framework**      | Next.js (React App Router)                        |
| **Runtime**        | Node.js LTS                                       |
| **Database**       | PostgreSQL (Prisma ORM)                           |
| **Auth**           | Better Auth (email + password; OAuth future)      |
| **Analytics**      | Google Tag Manager → GA4                          |
| **Hosting (Prod)** | Raspberry Pi (ARM64) + Docker + Cloudflare Tunnel |
| **Hosting (Dev)**  | Windows Docker Desktop + WSL                      |
| **Launch Target**  | **30 Nov 2025 (Australia/Sydney)**                |

---

## 2. Functional Requirements

### Calendar & Unlock Logic

* Grid of **25 tiles** (30 Nov → 24 Dec).
* Tiles unlock automatically based on **Australia/Sydney** date.
* Locked tiles show dimmed state; current day unlocks with confetti.
* Past days remain accessible; future days remain locked.

### Content

* Each tile references a JSON file in `/content/2025/`:

  ```json
  {
    "date": "2025-12-01",
    "title": "Hope",
    "verse_ref": "Isaiah 9:6",
    "verse_text": "For unto us a child is born…",
    "encouragement_md": "In this season of **hope** …"
  }
  ```
* Content rendered with Markdown; optional image and CTA.

### Authentication

* Users sign up / login with Better Auth.
* Anonymous participation possible later (via donation unlock).
* Session stored securely (JWT + cookies).

### Gamification

* **Streak counter** increments on first open per day; resets if a day missed.
* **Progress bar** = unlocked days / 25.
* Both values computed from `DayOpen` records in Postgres.

### Analytics

* Events pushed to GTM:
  `signup`, `login`, `tile_open`, `streak_increment`, `progress_update`.

---

## 3. Non-Functional Requirements

| Category          | Target                                         |
| ----------------- | ---------------------------------------------- |
| **Performance**   | P95 page load < 1.2 s (broadband)              |
| **Uptime**        | ≥ 99 % during Advent season                    |
| **Accessibility** | WCAG AA (minimum)                              |
| **Privacy**       | Minimal PII (email only)                       |
| **Security**      | HTTPS via Cloudflare Tunnel, strong secrets    |
| **Localization**  | English only (MVP); future multi-lang possible |

---

## 4. Technical Architecture

### System Diagram

```
Browser (React UI)
      │
      ▼
Cloudflare Tunnel (TLS)
      │
Docker on Pi
 ├─ app (Next.js + Node)
 │   ├─ API routes (auth, streak, content)
 │   └─ Prisma client → Postgres
 ├─ postgres (16-alpine)
 └─ cloudflared (service)
```

### Key Components

| Component       | Description                                       |
| --------------- | ------------------------------------------------- |
| **Next.js App** | Front-end + API routes (`/api/streak`, `/api/me`) |
| **Prisma ORM**  | Connects Next.js to Postgres database             |
| **Postgres**    | Stores users, sessions, day opens                 |
| **Cloudflared** | Exposes port 3000 securely to public Internet     |

---

## 5. Database Schema (Prisma)

```prisma
model User {
  id        String   @id @default(uuid())
  email     String   @unique
  createdAt DateTime @default(now())
  dayOpens  DayOpen[]
  sessions  Session[]
}

model Session {
  id         String   @id @default(uuid())
  userId     String
  createdAt  DateTime @default(now())
  expiresAt  DateTime
  user       User     @relation(fields: [userId], references: [id])
}

model DayOpen {
  id        String   @id @default(uuid())
  userId    String
  dateISO   String
  openedAt  DateTime @default(now())
  user      User     @relation(fields: [userId], references: [id])
  @@unique([userId, dateISO])
}
```

---

## 6. Deployment Configuration

### Docker files

* `postgres:16-alpine` ARM64 → shared between dev & prod.
* `cloudflare/cloudflared:latest` for HTTPS tunnel.
* Separate compose files: `docker-compose.dev.yml` and `docker-compose.prod.yml`.

### Environment Variables

| Key                                                   | Description                  |
| ----------------------------------------------------- | ---------------------------- |
| `DATABASE_URL`                                        | Postgres connection string   |
| `POSTGRES_USER` / `POSTGRES_PASSWORD` / `POSTGRES_DB` | DB credentials               |
| `BETTER_AUTH_SECRET`                                  | Auth secret                  |
| `BETTER_AUTH_ISSUER`                                  | Issuer URL (dev / prod)      |
| `NEXT_PUBLIC_GTM_CONTAINER_ID`                        | Google Tag Manager container |
| `CLOUDFLARED_TUNNEL_TOKEN`                            | Tunnel token                 |

---

## 7. User Experience & Design

* **Style:** Modern minimalist, white + black palette with soft Christmas accents.
* **Layout:** Full-screen responsive grid (3–5 columns).
* **Animation:** Smooth CSS scale/opacity on open; confetti burst.
* **Accessibility:** Keyboard navigable, aria-labels for tiles.

---

## 8. Milestones & Timeline

| Date       | Deliverable                                    |
| ---------- | ---------------------------------------------- |
| **Nov 1**  | Finalize Prisma schema, content JSON structure |
| **Nov 10** | Auth + streak API functional in dev            |
| **Nov 17** | GTM tracking events implemented                |
| **Nov 24** | Prod build on Pi with Postgres + Cloudflared   |
| **Nov 29** | QA + content review                            |
| **Nov 30** | 🎄 Launch (First Sunday of Advent 2025)        |

---

## 9. Risks & Mitigation

| Risk                          | Mitigation                                           |
| ----------------------------- | ---------------------------------------------------- |
| Time-zone unlock errors       | All date logic via `Australia/Sydney` in server code |
| DB corruption or loss         | Daily `pg_dump` to mounted backup volume             |
| Raspberry Pi performance      | Use SSR for auth only, SSG for grid                  |
| Content typos or missing days | Pre-launch validation script checks 25 JSON files    |
| Launch traffic spike          | CDN caching for static assets via Cloudflare         |

---

## 10. Future Enhancements (Post-MVP)

* Anonymous access via donation unlock.
* Yearly reset (auto-archive previous Advent).
* Push notifications / email reminders.
* PWA offline mode for opened days.
* Theming / dark mode toggle.
* Admin dashboard to upload content for next year.

---

## 11. Success Metrics

* **Activation:** ≥ 70 % of signed-up users open Day 1.
* **Engagement:** Median streak ≥ 5 days; ≥ 40 % open ≥ 15 tiles.
* **Completion:** ≥ 20 % open all tiles by Dec 24.
* **Reliability:** ≥ 99 % uptime; < 1.2 s page load (P95).

---

### ✅ Status Next

* [ ] Confirm UI mock and tile animation assets.
* [ ] Generate 25 JSON content stubs.
* [ ] Prepare `docker-compose` for Pi deployment test.
* [ ] Verify Prisma migrations and backup job.

---

*Prepared for review by James Follent — Digital Advent Calendar 2025 (Project advent.follent.net).*
