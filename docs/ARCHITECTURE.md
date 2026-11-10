# PragyaLink — System Architecture

**Document status:** Initial architecture baseline  
**Project stage:** Pre-research / requirements refinement  
**Initial operating area:** University of Cape Coast (UCC), Ghana  
**Architecture style:** Modular monolith with a separate React frontend and Django REST API

> This document describes the proposed architecture, not a claim that these components have already been implemented. Operational details marked as assumptions must be validated through student and Pragya-operator research.

---

## 1. Purpose and scope

PragyaLink is a mobile-friendly web platform for coordinating campus Pragya journeys. Students can request a regular/shared ride or hire a Pragya exclusively through a Dropping booking. Approved riders receive requests that match their approved operating routes and current availability. Administrators manage riders, vehicles, routes, locations, fares and platform activity.

The first release is intended for UCC. It must reflect the actual local operating rules rather than behave like an unrestricted ride-hailing application.

The first release should provide reliable request coordination and transparent fare estimates. It does **not** depend on live GPS tracking, online payments, automated route optimisation or a native mobile application.

## 2. Architectural principles

1. **Backend is authoritative.** Django validates permissions, routes, availability, passenger limits, prices and state transitions.
2. **Configuration is data.** Routes, stops, pickup locations, fares, Dropping multiplier and capacity must not be hardcoded in React.
3. **Keep the first release understandable.** Use one Django project and one PostgreSQL database, organised into focused Django apps/modules.
4. **Preserve history.** A fare or route configuration change must not silently rewrite an existing booking or completed trip.
5. **Mobile-first.** Student and rider workflows must be usable on ordinary phone browsers and modest network connections.
6. **No false live-location promise.** The initial system coordinates requests; it does not claim to know a rider's exact physical location.
7. **Build for actual operations.** Only administrator-approved route coverage, stops and pickup rules may be used for matching.
8. **Privacy by default.** Expose only the personal and vehicle information needed for a confirmed booking.

## 3. High-level components

```text
+--------------------------+        HTTPS / JSON        +---------------------------+
| React + Vite frontend    | <------------------------> | Django REST Framework API |
|                          |                            |                           |
| Student interface        |                            | Authentication & roles    |
| Rider "Driver Mode" UI   |                            | Request/trip workflows    |
| Admin interface          |                            | Route validation          |
| Click-to-Call linkage    |                            | Fare calculation          |
| Request timeout countdown|                            | Rider eligibility         |
| Shared UI/API client     |                            | Polling/notifications     |
+--------------------------+                            | Admin operations          |
                                                        +-------------+-------------+
                                                                      |
                                                                      | Django ORM
                                                                      |
                                                        +-------------v-------------+
                                                        | PostgreSQL                |
                                                        | Users, routes, fares,     |
                                                        | requests, trips, etc.     |
                                                        +---------------------------+

Core deployment services:
- Vercel: React frontend hosting.
- Render: Django API hosting.
- Neon: managed PostgreSQL.
- No third-party media storage: Cloudinary removed for MVP; verification uses in-person physical checks & ID numbers.
```

The frontend and backend are separate deployable applications. The backend remains a modular monolith: one API and one database, not separate microservices.

## 4. Suggested repository layout

```text
PragyaLink/
├── backend/
│   ├── config/                 # Django settings, URLs, WSGI/ASGI
│   ├── apps/
│   │   ├── accounts/           # Custom user, profiles, authentication
│   │   ├── operations/          # Locations, routes, stops, rider-route links
│   │   ├── pricing/             # Fare records and pricing configuration
│   │   ├── bookings/            # Ride requests, assignment, trips
│   │   ├── notifications/       # In-app notifications
│   │   └── support/             # Complaints/reports and audit records
│   ├── manage.py
│   └── requirements.txt
├── frontend/
│   ├── src/
│   │   ├── api/
│   │   ├── app/
│   │   ├── components/
│   │   ├── features/
│   │   │   ├── auth/
│   │   │   ├── student/
│   │   │   ├── rider/
│   │   │   └── admin/
│   │   ├── layouts/
│   │   └── styles/
│   └── package.json
├── PROJECT_CONTEXT.md
├── ARCHITECTURE.md
├── DATABASE_DESIGN.md
├── API_SPEC.md
├── RESEARCH_FINDINGS.md
├── DECISIONS.md
└── TODO.md
```

This is a suggested structure. Inspect the existing repository before moving or renaming anything.

## 5. Main application modules

### 5.1 Accounts and access

Responsibilities:
- User registration and login.
- Student and rider profiles.
- Rider application and verification state.
- Administrator permissions.
- Profile updates and account status.

Use a custom Django user model from the beginning if the project has not already committed to a different user model. Roles must be enforced on the backend. Never trust a role value submitted by the browser.

Suggested roles: `STUDENT`, `RIDER`, `ADMIN`. A rider's approval status is separate from their role.

### 5.2 Operations: locations and route network

Responsibilities:
- Recognised locations and pickup points.
- Administrator-managed routes and ordered stops.
- Route direction and permitted journey stretches.
- Rider-to-route assignments.
- Operational restrictions and route activation.

A **location** is a named physical place such as a hall, hostel entrance or campus landmark. A **route** is an approved operating corridor with ordered stops. A location is not automatically an authorised route stop.

Model the initial service as a set of administrator-defined route directions and ordered stops. Multi-route connections (`RouteConnection`) and complex graph-based transfers are explicitly excluded from the MVP. All journeys must be satisfied along a single approved route corridor. If a trip requires a transfer, the passenger books the next leg separately.

Private hostels may be registered as pickup points or approved access points without being treated as permission to travel along a prohibited road. Research must establish whether a rider may enter a hostel road, meet at its entrance, or only collect from a nearby approved point.

### 5.3 Pricing

Responsibilities:
- Regular passenger fares for specific journey stretches.
- Effective-dated fare changes.
- Configurable Dropping multiplier.
- Configurable permitted passenger capacity.
- Fare quote calculation and immutable booking snapshots.

The fare calculator belongs in the backend. React may display a quote returned by the API but must not be trusted to calculate or submit the authoritative amount.

### 5.4 Bookings and trips

Responsibilities:
- Student ride requests.
- Booking type and passenger count.
- Rider eligibility and request offers.
- Acceptance/rejection and cancellation.
- Trip progress and completion.
- Trip and request history.

Keep `RideRequest` and `Trip` conceptually distinct. A request is a student's demand for a ride. A trip is the operational journey after a rider accepts. The exact implementation can use a one-to-one trip created on acceptance, with clear state transitions.

### 5.5 Notifications

Start with database-backed in-app notifications. The frontend can poll for updates at a modest interval or refresh when the user returns to the page. Use WebSockets only if testing shows polling is inadequate and the deployment environment can support the extra operational complexity.

Notifications are an update channel, not the source of truth. Always retrieve the current request/trip state from the API.

### 5.6 Support and administration

Responsibilities:
- Complaints and reports.
- Administrative activity records.
- Dashboard summaries.
- Monitoring requests and trips.
- Managing users, routes, locations and pricing.

Do not expose sensitive rider verification documents to ordinary users or other riders.

## 6. Core workflows

### 6.1 Student request

1. Student signs in.
2. Student chooses an approved pickup point and destination.
3. Backend checks whether the journey is supported by the approved route network.
4. Student chooses `REGULAR` or `DROPPING`.
5. For Dropping, student selects the number of passengers.
6. Backend returns a fare quote with a short-lived or versioned reference to the pricing inputs.
7. Student reviews the quote and submits the request.
8. Backend recalculates/validates the quote at submission and stores a fare snapshot.
9. Matching service finds approved, active riders who serve the requested journey and are not committed to an incompatible trip.
10. Eligible riders receive the request.
11. The first valid acceptance wins through an atomic database operation.
12. Student sees the assigned rider/vehicle details and request progress.

### 6.2 Regular/shared ride

A regular booking is priced per passenger for the selected stretch, subject to local fare research. The rider may continue the normal shared-service practice only as permitted by the confirmed operating rules. The system must not promise that other passengers will be picked up or that a particular waiting time is guaranteed.

If the operational practice requires the rider to fill seats before departing, the research team must document how this affects request acceptance, trip start and student expectations.

### 6.3 Dropping/private hire

A Dropping booking reserves the vehicle for the organiser's group for the journey. The rider must not pick up unrelated passengers during that trip if that is the confirmed local definition of Dropping.

Proposed calculation:

`dropping_total = regular_fare_for_exact_stretch × configured_dropping_multiplier`

The proposed initial multiplier is 4, but it is **unverified** and must be confirmed with operators. Passenger capacity is a separate setting. The group size must not exceed the approved capacity for the assigned vehicle/service.

Show:
- Total quoted fare.
- Passenger count.
- Indicative per-person share (`total / passenger_count`).

The platform does not need to collect each passenger's contribution in the first release. The organiser and rider can settle payment directly, subject to research confirming the local payment practice.

## 7. Route and rider matching

The matching service should consider, at minimum:

- Rider account is approved and active.
- Rider is currently marked available.
- Rider is assigned to the relevant active route/direction.
- Pickup and destination are covered by a valid permitted journey.
- Rider has no conflicting active trip.
- Vehicle/service capacity is sufficient for the booking.
- Any configured time or operating restriction is satisfied.

The initial version may notify all eligible riders rather than implement complex ranking. If several riders accept at nearly the same time, the backend must atomically assign only one. Losing accept attempts should receive a clear conflict response.

Do not use straight-line distance or GPS proximity as a substitute for route eligibility.

## 8. Request and trip state management

Suggested request states:

- `PENDING`: request created and awaiting rider offers/response.
- `OFFERED`: request has been made available to eligible rider(s).
- `ACCEPTED`: a rider has accepted and been assigned.
- `CANCELLED`: cancelled by an authorised party.
- `EXPIRED`: no rider accepted within the configured request window.
- `UNFULFILLED`: no eligible rider could be found or the offer window ended.

A rider's rejection is best recorded as an offer response, not necessarily as the overall request becoming `REJECTED`, because other riders may still be eligible.

Suggested trip states:

- `ASSIGNED`
- `RIDER_ARRIVING`
- `ARRIVED`
- `IN_PROGRESS`
- `COMPLETED`
- `CANCELLED`

Enforce allowed transitions in backend services. A client must not be able to jump directly from `ASSIGNED` to `COMPLETED`.

## 9. Fare integrity and concurrency

- Use `DecimalField`/decimal arithmetic for GHS amounts; never binary floating point for money.
- Store the regular fare, multiplier (where applicable), currency, passenger count, total quoted amount, pricing record/version and quote timestamp with the request.
- Store the final agreed/charged amount on the trip if it can differ from the original quote. Record who authorised a change and why.
- Do not edit historical fare records in place when changing a fare. Create a new effective-dated record or version.
- Use database transactions and row locking/conditional updates for rider acceptance and cancellation races.
- Add database constraints for valid fare ranges, unique route-stop ordering and other invariants where practical.

## 10. Authentication, security and privacy

- Use Django authentication with JWT access/refresh tokens if appropriate for the chosen frontend session model.
- Store secrets only in environment variables.
- Configure HTTPS, allowed hosts, CORS and secure production cookies/token handling.
- Apply object-level permissions: students see their own requests/trips; riders see requests offered to them and their own trips; administrators have authorised management access.
- Validate every protected action on the backend.
- Restrict rider verification documents to authorised administrators and the relevant rider.
- Avoid collecting student index numbers or identity documents unless research or an actual verification requirement demonstrates a need.
- Establish retention and deletion rules for personal data and uploaded documents.
- Rate-limit authentication and request creation endpoints.
- Log important administrative changes without logging passwords, tokens or unnecessary sensitive data.

## 11. Deployment and operations

Proposed low-cost deployment:
- React/Vite frontend: Vercel.
- Django REST API: Render.
- PostgreSQL: Neon or another compatible managed PostgreSQL provider.
- Media: No third-party media hosting in MVP; driver verification uses physical ID check and registration numbers.

Confirm free-tier limits, sleep behaviour, database connection limits, backups, media privacy and production suitability before launch. A student pilot should not depend on a service configuration that silently loses data or exposes private documents.

## 12. Testing strategy

Minimum test layers:
- Unit tests for route eligibility, fare calculation and state transitions.
- API tests for authentication, permissions and validation.
- Integration tests for request → offer → acceptance → trip completion.
- Concurrency tests for two riders accepting the same request.
- Frontend tests for key forms and error states.
- Manual mobile-browser tests on real devices and weak/intermittent connections.

Important scenarios:
- Unsupported or prohibited route.
- Inactive location or fare.
- Fare changes after quote.
- Dropping passenger count above capacity.
- Rider unapproved/offline/on another trip.
- No eligible rider.
- Duplicate submissions.
- Simultaneous accept attempts.
- Student/rider cancellation at different trip stages.
- Network loss while updating trip status.

## 13. Explicitly out of scope for the first release (Prohibited for MVP)

To keep development time predictable and operating costs at zero, the following are strictly prohibited in the MVP:

- **NO GPS tracking or map APIs**: Do not import Leaflet, OpenStreetMap, Google Maps, Mapbox, or use HTML5 Geolocation continuous tracking. Location is handled strictly via named landmark stops.
- **NO Online payment gateway SDKs**: Do not integrate Paystack, Flutterwave, or Stripe. Payment is settled directly via physical Cash or Rider's personal MoMo upon trip completion.
- **NO Paid SMS gateways**: Do not integrate Twilio or Hubtel. Voice communication is initiated through native browser `tel:` links ("Click-to-Call"), and status updates use in-app polling.
- **NO In-app text chat**: Voice calls via Click-to-Call are used exclusively during active trips.
- **NO Media storage platforms**: Do not integrate Cloudinary or AWS S3. Driver verification is handled via physical inspection of credentials and admin dashboard toggles.
- **NO Dynamic pooling or multi-route transfer graphs**: No `RouteConnection` models or multi-leg journeys. All trips belong to a single corridor.
- **NO AI-based matching**: Dispatch is deterministic based on route assignment, active status, availability, and vehicle capacity.
- **NO Native Android/iOS apps**: The platform is a mobile-responsive web application with a dedicated high-contrast "Driver Mode" for riders.
- **NO Multi-university tenancy**: UCC only.

Revisit these only after the core coordination workflow has been validated with real users on campus.
