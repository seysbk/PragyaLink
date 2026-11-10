# PragyaLink — Database Design

**Document status:** Proposed logical design; not yet validated by field research  
**Database:** PostgreSQL  
**ORM:** Django ORM

> This is a design baseline, not a migration script. Confirm operational rules and inspect the existing code before implementing or changing models.

---

## 1. Design goals

The database must support:

- Student and rider accounts with separate profile data.
- Rider and vehicle verification.
- Administrator-managed locations, routes and route stops.
- Explicit route coverage and direction.
- Route-stretch-specific fares.
- Effective-dated pricing and historical fare preservation.
- Regular/shared and Dropping bookings.
- Group Dropping passenger counts.
- Rider eligibility, offers, assignment and trip progress.
- Notifications, complaints and administrative audit records.

Keep the first version relational and understandable. Do not create tables for hypothetical features unless a confirmed requirement needs them.

## 2. Important modelling rules

1. A **Location** is a named physical place. It is not automatically an authorised route stop.
2. A **Route** is an approved service path/direction with ordered stops.
3. A **RouteFare** prices a particular origin-to-destination stretch on a route. Do not assume a long-stretch fare equals the sum of smaller stretches.
4. A **RiderRoute** links an approved rider to a route they are permitted to serve.
5. A **Dropping multiplier** and **vehicle passenger capacity** are independent settings.
6. A **RideRequest** stores the quoted price snapshot so later fare changes do not alter it.
7. A **Trip** records the accepted operational journey and final outcome.
8. A rider's rejection should be stored as an offer response; it should not necessarily reject the whole request while other eligible riders can still respond.

## 3. Proposed entities

### 3.1 User

Use a custom Django user model if the project has not already migrated with Django's default user model.

Suggested fields:
- `id`: UUID or integer primary key.
- `email`: optional/required depending on chosen login method; unique if used.
- `phone_number`: normalised phone number; unique if phone login is selected.
- `password`: Django-managed hashed password.
- `role`: `STUDENT`, `RIDER`, `ADMIN`.
- `is_active`: account enabled flag.
- `date_joined`, `updated_at`.

Do not store plaintext passwords. Decide whether email, phone or both are required after research and usability testing.

### 3.2 StudentProfile

- `user`: one-to-one User.
- `full_name`.
- `usual_residence_location`: nullable FK to Location, if useful.
- `created_at`, `updated_at`.

Avoid collecting index number or other identity data unless a verified requirement justifies it.

### 3.3 RiderProfile

- `user`: one-to-one User.
- `full_name`.
- `phone_number` or use User phone number consistently.
- `verification_status`: `PENDING`, `APPROVED`, `REJECTED`, `SUSPENDED`.
- `availability_status`: `AVAILABLE`, `UNAVAILABLE`.
- `reviewed_by`: nullable FK to User.
- `reviewed_at`, `review_note`.
- `created_at`, `updated_at`.

Approval and availability are separate: an approved rider can be unavailable.

### 3.4 Vehicle

- `rider`: FK to RiderProfile (or a separate owner relationship if research requires shared vehicles).
- `registration_number`: unique, normalised.
- `vehicle_type` or service category, if needed.
- `passenger_capacity`: nullable until verified; preferably use an approved capacity value.
- `photo`: optional media reference.
- `verification_status`.
- `is_active`.
- timestamps.

Do not assume the legal or practical passenger capacity is four. Confirm this. If capacity differs by vehicle, store it on Vehicle; if all participating vehicles use a single verified capacity, an administrator setting may be sufficient. Avoid maintaining contradictory global and per-vehicle values.

### 3.5 Location

- `id`.
- `name`.
- `location_type`: e.g. `HALL`, `HOSTEL`, `CAMPUS_AREA`, `LANDMARK`, `PICKUP_POINT`, `OTHER`.
- `description` / `pickup_instructions`.
- `latitude`, `longitude`: nullable and only if collected/needed.
- `is_pickup_allowed`.
- `is_dropoff_allowed`.
- `is_active`.
- `created_by`, timestamps.

A location can be a hostel entrance or meeting point without granting permission to travel along every road leading to it. Store access/route rules separately.

### 3.6 Route

- `id`.
- `name`: human-readable route name.
- `code`: unique stable identifier.
- `direction_label`: e.g. `Science → Ayensu`; model the reverse direction separately if its stop order/permissions differ.
- `description`.
- `is_active`.
- `created_by`, timestamps.

Prefer an explicit direction-specific route record over assuming a route automatically works in both directions.

### 3.7 RouteStop

- `route`: FK to Route.
- `location`: FK to Location.
- `sequence`: integer ordering the stop along this route.
- `pickup_allowed`.
- `dropoff_allowed`.
- `is_active`.

Constraints:
- Unique `(route, sequence)`.
- Optionally unique `(route, location)` if a route cannot visit the same location twice.

### 3.8 RouteConnection (EXCLUDED FROM MVP)

*De-scoped per DEC-025.* All journeys must occur on a single approved route corridor. Multi-route transfer graphs are not supported in the MVP.

### 3.9 RiderRoute

- `rider`: FK to RiderProfile.
- `route`: FK to Route.
- `is_active`.
- `approved_by`: nullable FK to User.
- `approved_at`.
- timestamps.

Unique `(rider, route)`. A rider can serve multiple approved routes.

### 3.10 RouteFare

Stores a fare for an exact stretch on a direction-specific route.

- `route`: FK to Route.
- `origin_stop`: FK to RouteStop.
- `destination_stop`: FK to RouteStop.
- `regular_fare`: DecimalField (e.g. `max_digits=10`, `decimal_places=2`).
- `currency`: default `GHS`.
- `effective_from`: datetime/date.
- `effective_to`: nullable datetime/date.
- `is_active`.
- `created_by`.
- `created_at`.

Validation:
- Origin and destination must belong to the selected route.
- Destination must occur after origin for the selected direction.
- Fare must be non-negative and normally greater than zero.
- No overlapping active effective periods for the same route/origin/destination/currency.
- A new fare should normally create a new record, not overwrite the historic amount.

If the locally agreed fare is not based on route stops but on named origin/destination pairs, adapt this model after research.

### 3.11 PricingConfiguration

Versioned configuration for rules that may change.

- `id`.
- `name` or `version`.
- `dropping_multiplier`: DecimalField.
- `currency`: `GHS`.
- `effective_from`.
- `effective_to`: nullable.
- `is_active`.
- `created_by`.
- timestamps.

The initial proposed multiplier is 4 but is **not confirmed**. Do not hardcode it.

Passenger capacity should normally live on the Vehicle or approved service configuration, not be inferred from the multiplier. If a single service-wide capacity is confirmed, a separate setting can be used.

### 3.12 RideRequest

- `id`.
- `student`: FK to User/StudentProfile.
- `pickup_location`: FK to Location.
- `destination_location`: FK to Location.
- `route`: FK to Route, if one route has been validated.
- `origin_stop`, `destination_stop`: nullable FKs to RouteStop where applicable.
- `booking_type`: `REGULAR`, `DROPPING`.
- `passenger_count`: positive integer.
- `pickup_landmark_note`: optional short text (max 120 chars, e.g. "Standing in front of Kingdom Books in blue shirt" — eliminates need for GPS).
- `status`: request state (`PENDING`, `OFFERED`, `ACCEPTED`, `CANCELLED`, `EXPIRED`, `UNFULFILLED`).
- `cancelled_by`: nullable string (`STUDENT`, `RIDER`, `SYSTEM_TIMEOUT`).
- `cancellation_reason`: optional text.
- `timeout_seconds`: integer (default 90 seconds for auto-expiration countdown).
- `quoted_regular_fare`: DecimalField snapshot.
- `quoted_multiplier`: nullable DecimalField snapshot.
- `quoted_total_fare`: DecimalField snapshot.
- `currency`: default `GHS`.
- `pricing_configuration`: nullable FK to PricingConfiguration, plus snapshots for durable history.
- `route_fare`: nullable FK to RouteFare, plus fare snapshot.
- `quote_created_at`.
- `expires_at`: datetime when request times out if unaccepted.
- `created_at`, `updated_at`.

Validation:
- `passenger_count >= 1`.
- Regular fare and Dropping fare rules are applied according to booking type.
- Dropping count does not exceed the eligible vehicle/service capacity.
- Pickup/destination must be valid for the route corridor.
- Client-submitted prices are ignored; backend calculates and stores the quote.
- Existing request snapshots are not changed by later fare edits.

The exact field structure may be simplified if the project chooses to store a dedicated immutable `FareQuote` object. Avoid storing the same information in multiple places without a clear reason.

### 3.13 RiderOffer (recommended)

Tracks which riders were offered a request and their responses.

- `id`.
- `ride_request`: FK to RideRequest.
- `rider`: FK to RiderProfile.
- `status`: `PENDING`, `ACCEPTED`, `DECLINED`, `WITHDRAWN`, `EXPIRED`.
- `offered_at`.
- `responded_at`.
- `expires_at`.

Unique `(ride_request, rider)`.

This avoids treating one rider's rejection as the rejection of the whole student request and provides an audit trail of offers.

### 3.14 Trip

- `id`.
- `ride_request`: one-to-one FK to RideRequest.
- `rider`: FK to RiderProfile.
- `vehicle`: FK to Vehicle.
- `status`: `ASSIGNED`, `RIDER_ARRIVING`, `ARRIVED`, `IN_PROGRESS`, `COMPLETED`, `CANCELLED`.
- `final_fare`: DecimalField.
- `payment_method_prompt`: string (default "CASH_OR_MOMO_ON_ARRIVAL" per DEC-028).
- `currency`: default `GHS`.
- `started_at`, `completed_at`, `cancelled_at`.
- `cancelled_by`: nullable string (`STUDENT`, `RIDER`, `SYSTEM_TIMEOUT`).
- `cancellation_reason`: optional text.
- timestamps.

The accepted fare should default to the request's stored quote. If a fare adjustment is permitted, record the adjusted amount, reason and authorising party. Do not silently overwrite the quote.

### 3.15 Notification

- `id`.
- `recipient`: FK to User.
- `notification_type`.
- `title`.
- `message`.
- `related_request`: nullable FK.
- `related_trip`: nullable FK.
- `is_read`.
- `created_at`.

Do not put sensitive verification details in notification text.

### 3.16 Complaint (support module)

- `id`.
- `submitted_by`: FK to User.
- `ride_request`: nullable FK.
- `trip`: nullable FK.
- `category`.
- `description`.
- `status`: `OPEN`, `UNDER_REVIEW`, `RESOLVED`, `DISMISSED`.
- `handled_by`: nullable FK to User.
- timestamps.

### 3.17 AdminActivityLog (recommended)

- `id`.
- `actor`: FK to User.
- `action`.
- `target_type`, `target_id`.
- `summary`.
- `created_at`.

Record sensitive changes such as rider approvals, route deactivation and fare updates. Do not log credentials, tokens or unnecessary personal data.

## 4. Relationship overview

```text
User
 ├── StudentProfile (1:1)
 │    └── RideRequest (1:M)
 │          ├── RiderOffer (1:M)
 │          └── Trip (0:1)
 └── RiderProfile (1:1)
      ├── Vehicle (1:M or 1:1 after research)
      ├── RiderRoute (M:M with Route)
      ├── RiderOffer (1:M)
      └── Trip (1:M)

Location
 └── RouteStop (M:M through RouteStop) ── Route
                                           └── RouteFare (1:M)

PricingConfiguration
 └── RideRequest pricing snapshot/reference

User ── Notification (1:M)
User ── Complaint (1:M)
User ── AdminActivityLog (1:M)
```

## 5. Fare calculation rules

### Regular

`regular_total = active_route_fare_for_exact_stretch`

Confirm through research whether the listed regular fare is per passenger or per vehicle. The current proposed model assumes it is a per-passenger fare. If the local practice differs, revise the calculation before implementation.

### Dropping

`dropping_total = regular_fare_for_exact_stretch × dropping_multiplier`

The multiplier is initially proposed as 4, pending validation.

`indicative_per_person_share = dropping_total / passenger_count`

The per-person share is informational; the booking remains one vehicle hire and one payment arrangement unless later requirements specify otherwise.

### Snapshot

When the request is submitted, save:
- Fare amount used.
- Multiplier used, if Dropping.
- Total quoted amount.
- Currency.
- Passenger count.
- Fare/configuration version or IDs.
- Quote timestamp.

Never recalculate an existing request's saved quote merely because an administrator changes the active fare.

## 6. Constraints and indexes

Recommended:
- Unique user email/phone according to authentication policy.
- Unique vehicle registration number.
- Unique route code.
- Unique `(route, sequence)` for RouteStop.
- Unique `(rider, route)` for RiderRoute.
- Unique `(ride_request, rider)` for RiderOffer.
- Check constraints for positive passenger count and non-negative fares.
- Indexes on request status/created time, rider availability/verification, active route fares and notification recipient/read state.
- Database transaction/row lock or conditional update to ensure only one rider wins a request.

Avoid over-constraining data before the real operational rules are known.

## 7. Items requiring research before final schema

- Are fares per passenger or per vehicle for each ordinary stretch?
- Is Dropping always exactly four times the regular fare?
- Does the multiplier vary by stretch, time or negotiation?
- What is the actual legal/practical passenger capacity for participating vehicles?
- Are routes directional, and are return journeys treated as separate routes?
- Can a ride cross between two approved routes?
- Are private-hostel entrances valid pickup points, or must pickup occur at a nearby approved stop?
- Can one rider operate multiple vehicles?
- Is a vehicle owned by one rider or shared among operators?
- Are phone numbers sufficient for student accounts?
- What rider verification evidence is actually required?
- How long should requests remain open before expiring?
- What cancellation rules apply at each stage?
