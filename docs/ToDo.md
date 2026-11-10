# PRAGYALINK — DEVELOPMENT TODO

This document tracks the implementation progress of PragyaLink.

It is the central development checklist for all AI coding agents and human contributors.

IMPORTANT:

- Never assume a feature is implemented simply because it is described in documentation.
- Inspect the existing code before making changes.
- Verify implementation before marking a task complete.
- Do not overwrite existing working functionality unnecessarily.
- Update this document when a task is completed.
- Record unresolved issues and technical decisions.
- Do not silently change the agreed project requirements.

Status legend:

- [ ] Not started
- \[\~\] In progress
- [x] Completed and verified
- \[!\] Blocked or requires clarification

---

# PHASE 0: PROJECT DISCOVERY AND REQUIREMENTS

## Research

- [ ] Interview UCC students.
- [ ] Interview Pragya operators.
- [ ] Confirm approved operating routes.
- [ ] Identify restricted roads.
- [ ] Identify approved pickup and drop-off locations.
- [ ] Confirm route directions.
- [ ] Establish actual passenger capacity.
- [ ] Confirm regular fares for each route stretch.
- [ ] Confirm Dropping pricing arrangements.
- [ ] Confirm whether the Dropping multiplier is consistently four.
- [ ] Investigate group booking practices.
- [ ] Establish applicable rider verification requirements.
- [ ] Establish operating hours and availability practices.
- [ ] Determine how students and riders prefer receiving notifications.

## Requirements and documentation

- [ ] Finalise problem statement.
- [ ] Finalise project objectives.
- [ ] Prepare stakeholder analysis.
- [ ] Prepare functional requirements.
- [ ] Prepare non-functional requirements.
- [ ] Prepare use case diagram.
- [ ] Prepare activity diagrams.
- [ ] Prepare entity relationship diagram.
- [ ] Prepare system architecture documentation.
- [ ] Document unresolved requirements.

---

# PHASE 1: PROJECT FOUNDATION

## Repository and setup

- [x] Establish GitHub repository.
- [x] Create frontend application.
- [x] Create Django backend application.
- [x] Configure environment variables.
- [x] Configure PostgreSQL.
- [x] Establish local development workflow.
- [x] Configure Git ignore rules.
- [x] Establish README documentation.

## Backend foundation

- [ ] Configure Django REST Framework.
- [ ] Configure authentication.
- [ ] Configure SimpleJWT.
- [ ] Implement custom user model if required.
- [ ] Implement account roles.
- [ ] Implement role-based permissions.
- [ ] Configure CORS.
- [ ] Configure API error handling.
- [ ] Configure database migrations.

## Frontend foundation

- [ ] Configure React and Vite.
- [ ] Establish routing.
- [ ] Create shared layouts.
- [ ] Create reusable components.
- [ ] Create API communication layer.
- [ ] Configure authentication state.
- [ ] Create responsive navigation.
- [ ] Establish consistent design system.

---

# PHASE 2: USER MANAGEMENT

## Students

- [ ] Student registration.
- [ ] Student login.
- [ ] Student profile.
- [ ] Profile editing.
- [ ] Logout.
- [ ] Password management.

## Riders

- [ ] Rider registration.
- [ ] Rider profile.
- [ ] Vehicle information submission (plate, Pragya station registration number).
- [ ] Manual in-person verification workflow (physical ID/license check by admin; no Cloudinary upload).
- [ ] Verification status tracking.
- [ ] Rider profile editing.

## Administrators

- [ ] Secure administrator account creation.
- [ ] Administrator login.
- [ ] Administrator dashboard.
- [ ] Rider approval (physical credential verification toggle).
- [ ] Rider rejection.
- [ ] Rider suspension.
- [ ] Rider reactivation.
- [ ] Student account management.

---

# PHASE 3: ROUTE AND LOCATION MANAGEMENT

- [ ] Create Location model (named landmarks, e.g. "Science Taxi Rank", "Ayensu Gate"; NO GPS coordinates).
- [ ] Create Route model (approved directional corridors).
- [ ] Create RouteStop model (ordered stops along single corridor).
- [ ] Implement route management API.
- [ ] Implement location management API.
- [ ] Create initial UCC seed fixture (`ucc_seed.json`).
- [ ] Create administrator route interface.
- [ ] Create administrator location interface.
- [ ] Allow multiple routes to be created.
- [ ] Allow routes to be edited and deactivated.
- [ ] Allow locations to be added dynamically.
- [ ] Associate riders with approved routes.
- [ ] Implement route eligibility validation (single corridor only; NO multi-route transfers).
- [ ] Test unsupported journeys.

IMPORTANT:
Routes and locations must not be hardcoded in frontend components. Single-corridor journeys only.

---

# PHASE 4: PRICING ENGINE

## Regular fares

- [ ] Create RouteFare model.
- [ ] Associate fares with appropriate route stretches.
- [ ] Implement fare calculation service.
- [ ] Support active fare configurations.
- [ ] Support effective dates.
- [ ] Preserve previous fare records.
- [ ] Implement administrator fare management.
- [ ] Validate fare calculations.

## Dropping

- [ ] Create PricingSettings model or equivalent configuration.
- [ ] Configure Dropping multiplier.
- [ ] Configure passenger capacity separately.
- [ ] Implement Dropping fare calculation.
- [ ] Display total Dropping fare.
- [ ] Display indicative per-person fare.
- [ ] Support group passenger counts.
- [ ] Prevent bookings exceeding vehicle capacity.
- [ ] Preserve the quoted fare with each request.
- [ ] Test fare changes against existing requests.

## Pricing administration

- [ ] Create fare management interface.
- [ ] Allow administrators to schedule fare changes.
- [ ] Display active fares.
- [ ] Display historical fares.
- [ ] Record fare changes in administrative activity logs.

---

# PHASE 5: RIDE REQUESTS
 
- [ ] Create RideRequest model.
- [ ] Implement pickup selection (landmark stop dropdown).
- [ ] Implement destination selection (landmark stop dropdown).
- [ ] Implement booking type selection (`REGULAR`, `DROPPING`).
- [ ] Implement passenger count.
- [ ] Implement pickup landmark note input (`pickup_landmark_note` short text for precise rendezvous without GPS).
- [ ] Validate approved corridor journeys.
- [ ] Calculate fare on backend.
- [ ] Display binding fare estimate with direct settlement prompt (Cash or Rider MoMo on arrival).
- [ ] Submit ride request.
- [ ] Prevent duplicate submissions.
- [ ] Implement broadcast countdown timer (default 90-second timeout) with UI countdown bar.
- [ ] Implement automatic transition to `UNFULFILLED` / `EXPIRED` on timeout.
- [ ] Implement student retry and cancel actions.
- [ ] Implement request cancellation attribution (`cancelled_by`, `cancellation_reason`).
- [ ] Implement request status management.
 
 ---
 
 # PHASE 6: RIDER ASSIGNMENT
 
 - [ ] Identify eligible riders (approved, assigned to corridor, available, no conflicting trip).
 - [ ] Match riders using approved corridor routes.
 - [ ] Check rider verification status.
 - [ ] Check rider availability.
 - [ ] Check conflicting active trips.
 - [ ] Send compatible requests to rider active request feed.
 - [ ] Implement accept functionality (atomic lock to prevent race conditions).
 - [ ] Implement decline functionality.
 - [ ] Prevent conflicting assignments using database transactions.
 - [ ] Release riders after eligible cancellations.
 - [ ] Handle requests with no available riders (graceful fallback).
 - [ ] Test simultaneous requests.
 
 IMPORTANT:
 The backend must be authoritative when assigning riders.
 Do not rely on frontend availability checks alone.
 
 ---
 
 # PHASE 7: TRIP MANAGEMENT & DRIVER MODE
 
 - [ ] Create Trip model.
 - [ ] Convert accepted requests into trips.
 - [ ] Implement Rider "Driver Mode" web UI (high-contrast, min 56px touch targets for sunlight/handlebar use).
 - [ ] Implement direct native "Click-to-Call" (`tel:`) voice button for student and rider during active trip.
 - [ ] Implement rider-arriving status (`RIDER_ARRIVING`).
 - [ ] Implement rider-arrived action (`POST /api/v1/rider/trips/{id}/arrived/`).
 - [ ] Implement trip-start action (`POST /api/v1/rider/trips/{id}/start/`).
 - [ ] Implement trip-completion action (`POST /api/v1/rider/trips/{id}/complete/`).
 - [ ] Implement trip cancellation with attribution (`STUDENT`, `RIDER`, `SYSTEM_TIMEOUT`).
 - [ ] Prompt direct offline payment completion (Cash or Rider MoMo).
 - [ ] Record final trip fare snapshot.
 - [ ] Implement student trip history.
 - [ ] Implement rider trip history.
 - [ ] Test invalid status transitions.

---

# PHASE 8: NOTIFICATIONS

- [ ] Create Notification model.
- [ ] Notify riders of new requests.
- [ ] Notify students of accepted requests.
- [ ] Notify students of rejected requests.
- [ ] Notify students of rider arrival.
- [ ] Notify users of cancellations.
- [ ] Notify students of completed trips.
- [ ] Create notification interface.
- [ ] Implement unread notification counts.
- [ ] Implement notification read status.

Future:

- [ ] Evaluate Django Channels.
- [ ] Evaluate browser push notifications.
- [ ] Evaluate SMS or WhatsApp notifications.

---

# PHASE 9: ADMINISTRATIVE MANAGEMENT

- [ ] Dashboard statistics.
- [ ] Student management.
- [ ] Rider management.
- [ ] Vehicle management.
- [ ] Route management.
- [ ] Location management.
- [ ] Fare management.
- [ ] Request monitoring.
- [ ] Trip monitoring.
- [ ] Complaint management.
- [ ] Activity logs.

---

# PHASE 10: TESTING AND SECURITY

## Functional testing

- [ ] Student registration.
- [ ] Rider registration.
- [ ] Rider approval.
- [ ] Route selection.
- [ ] Regular ride requests.
- [ ] Dropping requests.
- [ ] Group Dropping bookings.
- [ ] Fare calculations.
- [ ] Rider assignment.
- [ ] Request cancellation.
- [ ] Trip completion.
- [ ] Trip history.

## Edge cases

- [ ] No available riders.
- [ ] Unsupported route.
- [ ] Deactivated route.
- [ ] Deactivated pickup location.
- [ ] Rider goes offline during a request.
- [ ] Rider becomes unavailable during assignment.
- [ ] Duplicate ride requests.
- [ ] Simultaneous assignment attempts.
- [ ] Fare changes after request submission.
- [ ] Passenger count exceeds capacity.
- [ ] Student cancels after acceptance.
- [ ] Rider cancels an accepted request.
- [ ] Network interruption during trip updates.

## Security

- [ ] Role-based endpoint permissions.
- [ ] Protected administrative endpoints.
- [ ] Protected rider verification information.
- [ ] User data access restrictions.
- [ ] API input validation.
- [ ] API rate limiting.
- [ ] Secure environment configuration.
- [ ] Authentication expiry handling.

---

# PHASE 11: DEPLOYMENT

- [ ] Prepare production environment.
- [ ] Configure production PostgreSQL.
- [ ] Deploy Django backend.
- [ ] Configure production environment variables.
- [ ] Configure allowed hosts.
- [ ] Configure CORS.
- [ ] Configure frontend API URL.
- [ ] Deploy React frontend.
- [ ] Configure image storage.
- [ ] Run production migrations.
- [ ] Create initial administrator account.
- [ ] Configure initial routes.
- [ ] Configure initial locations.
- [ ] Configure initial fares.
- [ ] Test deployed application.
- [ ] Test using actual mobile devices.

---

# PHASE 12: PILOT AND IMPROVEMENTS

- [ ] Recruit a small group of student testers.
- [ ] Recruit participating Pragya operators.
- [ ] Conduct controlled pilot.
- [ ] Collect feedback.
- [ ] Identify usability problems.
- [ ] Identify operational problems.
- [ ] Correct critical issues.
- [ ] Evaluate adoption.
- [ ] Prepare final project documentation.

---

# DEVELOPMENT AGENT INSTRUCTIONS

Every AI coding agent working on this repository must follow these instructions.

1. Read PROJECT\_CONTEXT.md before implementing features.
2. Read this TODO.md to understand the current development stage.
3. Inspect the existing repository before making assumptions.
4. Determine which parts of the requested feature already exist.
5. Do not recreate existing functionality unnecessarily.
6. Follow the established architecture and naming conventions.
7. Do not hardcode routes, fares, passenger limits or administrative configurations.
8. Keep business logic in Django.
9. Validate all important operations on the backend.
10. Do not introduce additional technologies without explaining their necessity.
11. Avoid unrelated modifications.
12. Test changes before declaring them complete.
13. Update TODO.md only after verifying implementation.
14. Record important architectural decisions in DECISIONS.md.
15. Report incomplete functionality and known limitations honestly.

When starting a new task, first provide a brief assessment of the existing implementation and the files that need modification.

When finishing a task, report:

- What was implemented.
- Which files were modified.
- Which database changes were introduced.
- Which tests were performed.
- What remains incomplete.
- Whether documentation needs updating.

Never claim that a feature is complete merely because its frontend interface exists.

A feature is complete only when its required frontend, backend, database operations, permissions and validation work together.