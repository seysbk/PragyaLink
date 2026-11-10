# PragyaLink — Architecture and Product Decisions

**Document status:** Initial decision register  
**Last updated:** 10 November 2026

This file records decisions that affect product scope, architecture, data and implementation. It is intended to prevent different contributors or AI agents from making conflicting assumptions.

Decision statuses:
- **Proposed:** current recommendation, not yet validated.
- **Accepted:** agreed project direction.
- **Pending research:** cannot be finalised until evidence is collected.
- **Superseded:** replaced by a later decision.

---

## DEC-001 — Initial deployment focus

**Status:** Accepted

**Decision:** Build the first release for the University of Cape Coast (UCC).

**Reason:** A bounded initial area makes it possible to validate routes, locations, fares and operating rules with a manageable group of students and riders.

**Consequence:** Do not build multi-university tenancy into the MVP. Keep route and location records configurable so expansion remains possible.

## DEC-002 — Product type

**Status:** Accepted

**Decision:** PragyaLink is a campus Pragya ride coordination platform, not an unrestricted ride-hailing clone.

**Reason:** Actual operations are constrained by designated routes, road restrictions, pickup rules and local fare practices.

**Consequence:** The backend must validate journey eligibility against administrator-managed operating data. A student must not be able to request an arbitrary road journey simply by typing two locations.

## DEC-003 — Application architecture

**Status:** Accepted

**Decision:** Use a modular monolith: React frontend, Django REST Framework API and one PostgreSQL database.

**Reason:** This is sufficient for the expected project scale and is easier to develop, explain, test and deploy than microservices.

**Consequence:** Keep business modules separate within the Django project. Do not introduce microservices without a demonstrated need.

## DEC-004 — Preferred technology stack

**Status:** Accepted, subject to deployment verification

**Decision:**
- React.js + Vite + JavaScript.
- Django + Django REST Framework.
- PostgreSQL.
- Neon as a candidate managed database.
- Render for backend hosting.
- Vercel for frontend hosting.
- No third-party media storage (Cloudinary removed for MVP; verification via physical ID / station checks).
- GitHub for version control.

**Reason:** This matches the developer's preferred stack and eliminates third-party media storage costs.

**Consequence:** Agents should not replace the stack without presenting a concrete technical reason and obtaining agreement. Verify current hosting limits and production suitability before pilot deployment.

## DEC-005 — Two booking types

**Status:** Accepted

**Decision:** The first release should support:
1. `REGULAR` — ordinary/shared Pragya service.
2. `DROPPING` — exclusive hire for the booking party.

**Reason:** These represent materially different local service arrangements and pricing behaviour.

**Consequence:** Booking type is a first-class field in the request model and must be visible to students and riders.

## DEC-006 — Dropping group booking

**Status:** Accepted

**Decision:** One student acts as the booking organiser and can request one Dropping vehicle for a group.

**Reason:** Friends should be able to travel together without creating separate vehicle requests.

**Consequence:** One group booking creates one ride request and one trip. Store passenger count. Do not create one trip per passenger. Individual in-platform payment splitting is out of MVP scope.

## DEC-007 — Proposed Dropping multiplier

**Status:** Pending research

**Decision under consideration:** Calculate Dropping as the regular fare for the exact journey stretch multiplied by four.

`Dropping total = regular fare × configured multiplier`

**Reason:** The project discussion indicates that Dropping may compensate the rider for the four passenger seats/revenue opportunities reserved for the booking party.

**Unresolved:** The team has not yet interviewed operators or confirmed that the multiplier is consistently four across routes and conditions.

**Consequence:** If implemented, store the multiplier in an administrator-managed, effective-dated configuration. Do not hardcode `4` in frontend or backend logic. Do not present the formula as a verified local rule until research confirms it.

**Fallback if research contradicts the formula:** Store a separately managed Dropping fare per route stretch, or introduce a rider-confirmed quote workflow. Select the simplest model supported by evidence.

## DEC-008 — Passenger capacity is separate from pricing multiplier

**Status:** Accepted

**Decision:** Do not infer vehicle passenger capacity from the Dropping multiplier.

**Reason:** A pricing convention based on four theoretical fares does not prove that four passengers may legally or safely travel in a vehicle.

**Consequence:** Validate the actual permitted capacity. Store capacity at vehicle level if it varies by vehicle; otherwise use a verified service-level configuration. Enforce the limit on the backend.

## DEC-009 — Route and location administration

**Status:** Accepted

**Decision:** Administrators manage routes, route directions, ordered stops and recognised pickup/drop-off locations. Riders select from approved routes.

**Reason:** Operators have designated routes and restrictions. Free-text route creation would undermine the platform's central operational rule.

**Consequence:** Do not hardcode the initial route list in React. The system must support administrators adding, editing and deactivating routes and locations without a code deployment.

## DEC-010 — Route representation

**Status:** Proposed; validate through research

**Decision:** Model route directions explicitly with ordered stops. Treat the reverse direction as a separate direction-specific route record when its operating path or permissions differ.

**Reason:** A route may not be symmetric. The return journey may use different roads, stops or restrictions.

**Consequence:** Validate origin/destination order against the selected route. Add multi-route connections only if research demonstrates they are required for the MVP.

## DEC-011 — Exact journey-stretch fares

**Status:** Accepted as design direction; fare units pending research

**Decision:** Store fares for explicit origin/destination stretches on a route rather than deriving every long journey by adding smaller fares.

**Reason:** A direct stretch may have its own established fare.

**Unresolved:** Research must confirm whether regular fares are per passenger, per seat or per vehicle.

**Consequence:** The fare model must support exact-stretch records and effective dates. Final calculation rules depend on the research result.

## DEC-012 — Effective-dated pricing and fare history

**Status:** Accepted

**Decision:** Fare changes create new effective-dated records. Preserve previous fare configurations.

**Reason:** Fares can change, and historical bookings must remain understandable.

**Consequence:** Administrators can schedule a new fare and review previous fares. Existing requests and trips retain their saved pricing snapshots.

## DEC-013 — Fare quote snapshot

**Status:** Accepted

**Decision:** Save the fare inputs and total quote with the ride request at submission time.

**Reason:** Later fare changes must not silently alter a student's existing request.

**Consequence:** The backend recalculates and validates the fare on submission. Client-supplied prices are never authoritative. If the price changes between quote and submission, require the student to review the updated amount.

## DEC-014 — Payment handling in MVP

**Status:** Proposed

**Decision:** Do not integrate MoMo or card payment processing in the first release. Start with the locally accepted direct payment arrangement, likely payment to the rider, after confirming it through research.

**Reason:** Payment integration introduces reconciliation, refunds, provider integration and security requirements that are not necessary to validate ride coordination.

**Consequence:** The platform records quoted/final fare but must not falsely represent that payment was collected. If useful, add a simple payment status only after the actual cash/MoMo workflow is established.

## DEC-015 — Rider assignment

**Status:** Accepted

**Decision:** Match riders using approval status, assigned route, journey eligibility, availability, vehicle capacity and conflicting active trips.

**Reason:** A rider must be operationally eligible, not merely online.

**Consequence:** The backend is authoritative. Use a database transaction/locking or conditional update so only one rider can accept a request.

## DEC-016 — Offer responses

**Status:** Proposed

**Decision:** Track rider offers and responses separately from the overall ride request.

**Reason:** One rider declining should not automatically cancel a request if other eligible riders can still accept it.

**Consequence:** Use a `RiderOffer` record or equivalent. A request becomes unfulfilled/expired only when the offer process ends without acceptance.

## DEC-017 — Notifications

**Status:** Proposed

**Decision:** Start with database-backed in-app notifications and modest API polling.

**Reason:** This is simpler to build and deploy than real-time infrastructure while still supporting the first pilot.

**Consequence:** Consider WebSockets/push notifications only if real usage demonstrates a need. The API remains the source of truth for current status.

## DEC-018 — No live GPS in MVP

**Status:** Accepted

**Decision:** Do not include live rider GPS tracking in the first release.

**Reason:** Route-based eligibility and request coordination can be tested without continuous location collection. GPS introduces permission, privacy, battery and accuracy concerns.

**Consequence:** Do not show a live map or claim precise rider arrival information. Rider status updates are manual in the MVP.

## DEC-019 — Authentication and identity data

**Status:** Pending research

**Decision under consideration:** Use Django authentication and JWT for the React client. Decide whether login is phone-based, email-based or both after checking student preferences and practical verification options.

**Reason:** The system needs accounts but should not collect unnecessary personal data.

**Consequence:** Public registration must never permit self-assigned admin privileges. Collect only information required for operation and verification.

## DEC-020 — Rider verification workflow

**Status:** Accepted for MVP

**Decision:** Rider verification will be an administrator-managed manual workflow based on physical credential inspection (e.g. Ghana Card, Driver's License number, UCC Pragya Station / Union registration number) rather than third-party document/image upload platforms like Cloudinary.

**Reason:** Eliminates third-party media storage costs, webhook complexity, and API dependencies. Admins or student executives can verify operators in person at local stations and toggle `verification_status` directly.

**Consequence:** Riders submit identity numbers and station affiliation in registration forms. Admin verifies and updates status via `/admin/riders/{id}/approve/`. Cloudinary is removed from MVP scope.

## DEC-021 — Complaints and cancellation rules

**Status:** Accepted for MVP

**Decision:** The platform supports request and trip cancellations with structured attribution (`cancelled_by`: `STUDENT`, `RIDER`, `SYSTEM_TIMEOUT`) and mandatory reason logging. A basic support complaint model handles post-trip reporting without automatic financial penalties.

**Reason:** Clean cancellation attribution is essential for timeout automation and operational audits.

## DEC-022 — Deterministic eligibility filtering

**Status:** Accepted

**Decision:** Use deterministic eligibility filtering rather than AI-based rider assignment.

**Reason:** The initial problem has clear operational constraints and does not require opaque or difficult-to-explain matching.

**Consequence:** Matching should be explainable: approved rider + assigned route + available + no conflicting trip + sufficient capacity.

## DEC-023 — Documentation as source of truth

**Status:** Accepted

**Decision:** Repository documentation is the shared source of truth for all human and AI contributors.

**Reason:** Multiple agents may work across sessions and may not share conversation history.

**Consequence:** Agents must read the project context and relevant design files before changes, inspect existing code, report changes and update documentation only after verification.

## DEC-024 — Landmark-based dispatch with pickup hints (No GPS/Map APIs)

**Status:** Accepted

**Decision:** Rely entirely on administrator-managed landmark stops and optional student pickup hints (`pickup_landmark_note`) rather than GPS coordinates, geolocation APIs, or interactive map tiles (Leaflet, Mapbox, Google Maps).

**Reason:** Eliminates paid map API keys, reduces client battery and mobile data consumption, and matches local Ghanaian campus transit where students orient by known landmarks (e.g., "Ayensu Gate", "Behind Casely Hayford", "Science Taxi Rank").

**Consequence:** `Location` entities use human-readable names and landmark descriptions. No mapping libraries or map components in frontend or backend.

## DEC-025 — Single-corridor journeys only (No multi-route transfer graphs)

**Status:** Accepted for MVP

**Decision:** All ride requests must be satisfied within a single approved route corridor. De-scope `RouteConnection` and graph-based multi-leg route transfers.

**Reason:** Pragya passengers in UCC do not expect multi-leg digital transfers on a single booking. Graph-based routing adds unnecessary complexity and points of failure.

**Consequence:** Remove `RouteConnection` from the active schema and API endpoints. If a journey requires multiple routes, passengers book separate rides.

## DEC-026 — Direct click-to-call voice linkage (No in-app chat engine)

**Status:** Accepted for MVP

**Decision:** Connect student and rider directly via native browser `tel:` links ("Click-to-Call") upon trip confirmation instead of building an in-app text chat engine.

**Reason:** Pragya drivers operate tricycles in transit and cannot safely type or read text messages. Direct phone calls are standard practice in Ghana.

**Consequence:** Phone numbers are exposed between matched student and rider strictly during active trips (`ACCEPTED`, `RIDER_ARRIVING`, `ARRIVED`, `IN_PROGRESS`). No chat models or chat UI.

## DEC-027 — Request broadcast timeout and anti-stall fallback

**Status:** Accepted for MVP

**Decision:** Unaccepted ride requests automatically expire after a configurable countdown (default 90 seconds) and transition to `UNFULFILLED`.

**Reason:** Prevents students from being stranded indefinitely when no online rider accepts the offer.

**Consequence:** The student UI shows an active matching countdown. If expired, the student can retry or cancel. The backend marks stale requests `EXPIRED`/`UNFULFILLED`.

## DEC-028 — Simplified direct payment (Cash / Driver MoMo on arrival)

**Status:** Accepted for MVP

**Decision:** All transactions are settled directly between passenger and rider upon arrival via physical Cash or the Rider's personal Mobile Money (MoMo). No payment gateway SDK (Paystack, Flutterwave) is integrated.

**Reason:** Eliminates gateway transaction fees, KYC merchant registration hurdles, settlement reconciliation, and webhook vulnerabilities.

**Consequence:** The platform quotes and displays the binding fare, prompting: *"Pay Cash or Rider MoMo upon completion"*. No card or gateway processing code.

## DEC-029 — High-contrast, large-target Rider web UI ("Driver Mode")

**Status:** Accepted for MVP

**Decision:** The Rider web portal must be optimized for bright sunlight, handlebar-mounted devices, and rough transit, using high contrast, large typography, and minimum 56px touch targets for core actions (Accept, Arrived, Start, Complete).

**Reason:** Pragya riders operate in bright daylight with one hand on the handlebar; delicate UI layouts lead to operational failure.

**Consequence:** The frontend component library must include a streamlined "Driver Mode" layout for active rider screens.

---

## Decision change procedure

When a decision changes:
1. Add a new decision entry or mark the old one superseded.
2. Record the reason and evidence.
3. Identify affected files, models, endpoints and tests.
4. Update `PROJECT_CONTEXT.md`, `ARCHITECTURE.md`, `DATABASE_DESIGN.md`, `API_SPEC.md` and `TODO.md` where necessary.
5. Do not silently change a pending research assumption into a confirmed fact.
