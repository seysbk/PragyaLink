# PragyaLink 

**Campus Pragya Ride Coordination & Management Platform**  
*A practical, zero-cost, corridor-based transportation dispatcher tailored for the University of Cape Coast (UCC), Ghana.*

---

##  Project Overview

**PragyaLink** is a full-stack, mobile-first web platform engineered to bridge the daily commuting gap between students and licensed tricycle (Pragya) operators within the University of Cape Coast community.

Unlike generic ride-hailing clones that demand expensive Google Maps API keys, battery-draining continuous GPS tracking, and complex cellular data background processes, PragyaLink operates as a **landmark- and corridor-based dispatch coordinator**. It models the actual operating reality in Cape Coast:
- Trips operate along **authorized campus corridors** with recognized landmark pickup points (e.g., *Science Taxi Rank*, *Ayensu Gate*, *Valco Flat*, *Old Site Administration*).
- Students submit on-demand hails with an optional landmark hint (e.g., *Standing in front of Kingdom Books in blue shirt*).
- Compatible, verified Pragya operators receive the request, accept it with a single tap, and coordinate directly with passengers via native **Click-to-Call** voice linkage.
- Fares are transparently calculated using administrator-managed rate tables and settled directly upon arrival via **Cash or Driver Mobile Money (MoMo)**.

PragyaLink was designed not simply as an academic software engineering capstone presentation, but as a lean, viable software solution ready for live student and operator adoption.

---

##  Core Features

###  For Students
- **Corridor & Landmark Selection:** Choose from approved UCC pickup and destination landmarks with ease.
- **Accurate Pickup Notes:** Specify concise landmark descriptions (pickup_landmark_note) eliminating the need for GPS pin dropping.
- **Flexible Booking Types:**
  - REGULAR: Standard single-passenger corridor hail.
  - DROPPING: Private hire of the entire Pragya for an individual or a group of friends (up to tricycle capacity).
- **Binding Fare Transparency:** Immediate, authoritative fare quotes calculated by the backend before submission.
- **Real-Time Request Matching & Countdown:** A 90-second anti-stall countdown timer informs students while broadcasting to available operators.
- **Instant Click-to-Call:** Matched students can tap a single button to dial their rider directly (	el:) for quick coordination.

###  For Pragya Riders (Driver Mode)
- **High-Contrast Sunlight UI:** Built specifically for handlebar-mounted phones under bright Ghanaian sunlight with minimum 56px touch targets.
- **One-Tap Operational Actions:** Quick, single-touch state transitions (ARRIVED, START TRIP, COMPLETE TRIP).
- **Active Request Feed:** Filtered strictly by the rider's authorized corridors and current availability toggle.
- **Zero In-App Typing:** Clear pickup details and instant Click-to-Call to connect directly with the passenger.

###  For Administrators
- **Corridor & Landmark Management:** Create, configure, and deactivate operating routes and pickup points dynamically without modifying code.
- **Physical Credential Verification:** In-person verification workflow for operators (Driver's License, Ghana Card, Station Union sticker) toggled securely via admin dashboard.
- **Effective-Dated Fare Engine:** Configure base fares, Dropping multipliers (default 4.0x), and passenger limits while preserving immutable historical pricing records.
- **Platform Monitoring:** Audit active requests, ongoing trips, cancellations, and support reports.

---

##  Architectural Constraints & Non-Goals (MVP)

To ensure rapid development cycles, zero cloud operational bills, and uninterrupted service on free-tier deployments:

1. **NO GPS / Geolocation APIs:** No Leaflet, Google Maps, Mapbox, or GPS background tracking. All positioning is landmark-based.
2. **NO Third-Party Payment Gateways:** No Paystack, Flutterwave, or Stripe integration. Transactions are settled offline (Cash/MoMo).
3. **NO Paid SMS Gateways:** No Twilio or Hubtel. All notifications rely on lightweight HTTP polling and in-app updates; calls use native phone links.
4. **NO In-App Text Chat:** Eliminates driver distraction during transit and removes chat moderation overhead.
5. **NO Media Upload Hosting:** No Cloudinary or AWS S3 SDKs. Operator verification is completed physically by station administrators.
6. **NO Multi-Leg Graph Transfers:** All trips are single-corridor journeys.

---

##  Technology Stack

| Layer | Technology | Rationale |
|---|---|---|
| **Frontend** | React 18 + Vite + Tailwind CSS | High-performance mobile-first SPA with responsive layouts and Rider Driver Mode. |
| **Backend** | Python 3.11+ / Django 5 / Django REST Framework | Authoritative business logic, role-based security, modular monolith architecture. |
| **Authentication** | Django SimpleJWT | Stateless token authentication with strict role permissions (STUDENT, RIDER, ADMIN). |
| **Database** | PostgreSQL (Managed via Neon) | Robust relational constraints, atomic row-locking for race-free dispatch, JSON fixtures. |
| **Hosting** | Vercel (Frontend) + Render (Backend API) | Zero-cost initial deployment with production HTTPS and CI/CD pipelines. |
| **Seed Data** | ucc_seed.json | Pre-configured baseline fixtures for University of Cape Coast routes and landmarks. |

---

##  Documentation Directory (/docs)

All comprehensive architecture, database, API, and process specifications are maintained in the docs/ folder:

-  [**ProjectContext.md**](docs/ProjectContext.md) — High-level product vision, stakeholders, functional requirements, and system boundaries.
-  [**ARCHITECTURE.md**](docs/ARCHITECTURE.md) — Modular monolith system architecture, state machines, component models, and explicit negative constraints.
-  [**DATABASE_DESIGN.md**](docs/DATABASE_DESIGN.md) — Complete relational schema, entity relationships, pricing snapshot models, and constraints.
-  [**API_SPEC.md**](docs/API_SPEC.md) — Comprehensive REST API endpoint contract, request/response payloads, and error codes.
-  [**DECISIONS.md**](docs/DECISIONS.md) — Architecture Decision Records (DEC-001 through DEC-029) documenting trade-offs, accepted scopes, and constraints.
-  [**RESEARCH_FINDINGS.md**](docs/RESEARCH_FINDINGS.md) — Field research register, operational hypotheses, and baseline UCC corridor seed defaults.
-  [**ToDo.md**](docs/ToDo.md) — Step-by-step master development checklist across all phases from foundation to deployment.

---

##  Instructions for AI Coding Agents

If you are an autonomous AI agent working on this codebase:
1. **Always read docs/ProjectContext.md and docs/ToDo.md** before writing any code.
2. **Strictly adhere to the negative constraints in docs/ARCHITECTURE.md**: Never install map libraries, GPS packages, payment SDKs, or Cloudinary.
3. **Backend is Authoritative:** Fare calculations, role permissions, and assignment locks must be executed in Django, never entrusted to the frontend.
4. **Use Atomic Transactions:** When a rider accepts a request, use select_for_update() to prevent multiple drivers from claiming the same hail.
5. **Update docs/ToDo.md** when a task is verified and functional.

---

##  Getting Started (Development Setup)

### Prerequisites
- Python 3.11+
- Node.js 18+ & npm
- PostgreSQL (local or cloud Neon instance)

### Backend Setup
`ash
cd backend
python -m venv venv
# Windows:
.\venv\Scripts\activate
# Linux/macOS:
source venv/bin/activate

pip install -r requirements.txt
python manage.py migrate
python manage.py loaddata ucc_seed.json
python manage.py runserver
`

### Frontend Setup
`ash
cd frontend
npm install
npm run dev
`

---

##  License
Academic and educational project developed for software engineering coursework at the University of Cape Coast. All rights reserved.

# PragyaLink
