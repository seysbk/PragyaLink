# PragyaLink — API Specification

**Document status:** Proposed REST API contract  
**Backend:** Django REST Framework  
**Base path:** `/api/v1/` (proposed)

> This is a starting contract for implementation. Endpoint names and payloads may be adjusted to match the existing repository, but frontend and backend agents must agree on one contract. All examples are illustrative.

---

## 1. API conventions

- Use JSON for request and response bodies unless uploading a file.
- Use HTTPS in production.
- Prefix endpoints with `/api/v1/`.
- Use consistent plural resource names.
- Return appropriate HTTP status codes.
- Keep business rules and authoritative calculations on the backend.
- Never accept a client-calculated fare as authoritative.
- Use pagination for potentially large lists.
- Return validation errors in a consistent field-oriented format.

Suggested response shape:

```json
{
  "data": {},
  "message": "Request completed successfully"
}
```

For paginated lists, use DRF pagination or a consistent wrapper:

```json
{
  "count": 42,
  "next": null,
  "previous": null,
  "results": []
}
```

Suggested error shape:

```json
{
  "detail": "The requested journey is not supported.",
  "code": "UNSUPPORTED_JOURNEY"
}
```

Field validation may return:

```json
{
  "passenger_count": ["This number exceeds the permitted capacity."]
}
```

## 2. Authentication and roles

Suggested endpoints:

| Method | Endpoint | Purpose | Access |
|---|---|---|---|
| POST | `/auth/register/student/` | Student registration | Public |
| POST | `/auth/register/rider/` | Rider application | Public |
| POST | `/auth/token/` | Obtain access/refresh tokens | Public |
| POST | `/auth/token/refresh/` | Refresh access token | Authenticated/valid refresh |
| POST | `/auth/logout/` | Revoke refresh token if blacklist is configured | Authenticated |
| GET | `/auth/me/` | Current user and role/profile summary | Authenticated |
| PATCH | `/auth/me/` | Update permitted profile fields | Authenticated |

Rules:
- Public registration must never allow a user to submit `ADMIN` as their role.
- Rider registration creates a pending rider profile.
- A pending, rejected or suspended rider cannot receive or accept requests.
- Admin accounts are provisioned through a controlled process, not public signup.
- Choose either phone-based or email-based login after research; do not build two parallel identity systems without need.

## 3. Public reference data

| Method | Endpoint | Purpose | Access |
|---|---|---|---|
| GET | `/locations/` | List active selectable locations | Public/authenticated |
| GET | `/locations/{id}/` | Location detail | Public/authenticated |
| GET | `/routes/` | List active routes and route stops | Public/authenticated |
| GET | `/routes/{id}/` | Route detail | Public/authenticated |
| GET | `/routes/eligible-journeys/` | Validate/list supported origin-destination pairs | Public/authenticated |
| POST | `/fare-quotes/` | Calculate a fare quote | Authenticated |

Do not expose internal notes, disabled route records or private administrative fields through public serializers.

### Fare quote request

`POST /api/v1/fare-quotes/`

```json
{
  "pickup_location_id": "location-uuid",
  "destination_location_id": "location-uuid",
  "booking_type": "DROPPING",
  "passenger_count": 3
}
```

Illustrative response:

```json
{
  "data": {
    "journey_supported": true,
    "route": {
      "id": "route-uuid",
      "name": "Science to Ayensu"
    },
    "booking_type": "DROPPING",
    "passenger_count": 3,
    "currency": "GHS",
    "regular_fare": "5.00",
    "dropping_multiplier": "4.00",
    "total_fare": "20.00",
    "indicative_per_person_share": "6.67",
    "pricing_version": "pricing-config-uuid",
    "quoted_at": "2026-11-10T18:00:00Z"
  }
}
```

The numbers are examples only, not confirmed UCC prices. For a regular booking, omit or return null for Dropping-only fields. The server should return a clear error if the journey is unsupported, no active fare exists or the passenger count is invalid.

A quote endpoint is informational. On request submission, the backend must revalidate the journey and price, then store the authoritative snapshot. If the quote has expired or the price changed, return the updated quote and require the student to confirm again rather than silently changing the amount.

## 4. Student booking endpoints

| Method | Endpoint | Purpose |
|---|---|---|
| POST | `/ride-requests/` | Create a ride request |
| GET | `/ride-requests/` | List the current student's requests |
| GET | `/ride-requests/{id}/` | Retrieve own request and status |
| POST | `/ride-requests/{id}/cancel/` | Cancel an eligible request |
| GET | `/trips/` | List the current student's trips |
| GET | `/trips/{id}/` | Retrieve own trip details |
| POST | `/trips/{id}/report/` | Submit a report/complaint |

### Create ride request

`POST /api/v1/ride-requests/`

```json
{
  "pickup_location_id": "location-uuid",
  "destination_location_id": "location-uuid",
  "booking_type": "DROPPING",
  "passenger_count": 3,
  "pickup_landmark_note": "Standing in front of Kingdom Books in blue shirt",
  "quote_reference": "optional-quote-reference"
}
```

The client must not submit `total_fare`, `regular_fare` or `dropping_multiplier` as trusted values. The backend derives these from active pricing records.

Illustrative response:

```json
{
  "data": {
    "id": "request-uuid",
    "status": "PENDING",
    "booking_type": "DROPPING",
    "passenger_count": 3,
    "pickup": {"id": "location-uuid", "name": "Example pickup"},
    "destination": {"id": "location-uuid", "name": "Example destination"},
    "pickup_landmark_note": "Standing in front of Kingdom Books in blue shirt",
    "fare": {
      "currency": "GHS",
      "regular_fare": "5.00",
      "multiplier": "4.00",
      "total": "20.00",
      "payment_mode": "CASH_OR_MOMO_ON_ARRIVAL"
    },
    "timeout_seconds": 90,
    "expires_at": "2026-11-10T18:01:30Z",
    "created_at": "2026-11-10T18:00:00Z"
  },
  "message": "Ride request created. Waiting for rider acceptance."
}
```

Expected validation cases:
- Pickup/destination inactive or not selectable.
- Journey not served by an approved route.
- No active fare for the exact stretch.
- Invalid booking type.
- Passenger count below one.
- Dropping passenger count above eligible capacity.
- Duplicate/repeated submission.
- User is not a student or account is inactive.

## 5. Rider endpoints

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/rider/profile/` | Rider profile and approval status |
| PATCH | `/rider/profile/` | Update allowed profile fields |
| GET | `/rider/vehicles/` | List own vehicles |
| POST | `/rider/vehicles/` | Submit vehicle details |
| PATCH | `/rider/vehicles/{id}/` | Update own permitted vehicle details |
| GET | `/rider/routes/` | View assigned routes |
| GET | `/rider/requests/` | View requests offered to this rider |
| POST | `/rider/requests/{id}/accept/` | Accept an offered request |
| POST | `/rider/requests/{id}/decline/` | Decline an offered request |
| PATCH | `/rider/availability/` | Set available/unavailable |
| GET | `/rider/trips/` | List own trips |
| POST | `/rider/trips/{id}/arrived/` | Mark arrival |
| POST | `/rider/trips/{id}/start/` | Start trip |
| POST | `/rider/trips/{id}/complete/` | Complete trip |
| POST | `/rider/trips/{id}/cancel/` | Request/record cancellation under policy |

The backend must check rider approval, vehicle status, route eligibility, capacity and conflicting active trips on acceptance. An unavailable or unapproved rider must not accept a request.

### Accept request

`POST /api/v1/rider/requests/{id}/accept/`

No fare or rider ID is needed in the body; identity comes from the authenticated user.

Possible success response:

```json
{
  "data": {
    "request_id": "request-uuid",
    "status": "ACCEPTED",
    "trip_id": "trip-uuid"
  },
  "message": "Request accepted."
}
```

If another rider has already accepted, return `409 Conflict` with code `REQUEST_ALREADY_ASSIGNED`. Use a database transaction/locking strategy to make this safe under concurrent requests.

## 6. Trip endpoints

Student and rider views should return only fields each role is authorised to see.

A confirmed student booking returns:
- Rider display name.
- Vehicle registration number.
- **Rider phone number** (rendered as clickable `tel:<phone_number>` link for voice call).
- Pickup and destination landmark names.
- Student pickup landmark note (`pickup_landmark_note`).
- Booking type and passenger count.
- Current trip status (`ASSIGNED`, `RIDER_ARRIVING`, `ARRIVED`, `IN_PROGRESS`, `COMPLETED`, `CANCELLED`).
- Quoted binding fare and payment prompt: *"Pay Cash or Rider MoMo upon completion"*.

A matched rider view ("Driver Mode") returns:
- Student display name.
- **Student phone number** (rendered as large Click-to-Call button).
- Pickup location name and student pickup landmark note.
- Destination location name.
- Booking type and passenger count.
- Quoted fare to collect.
- One-tap status update buttons: `ARRIVED`, `START`, `COMPLETE`.

Phone numbers are exposed strictly during active trips (`ACCEPTED` through `IN_PROGRESS`) to protect privacy once the trip terminates. Do not reveal rider personal identification numbers or other riders' offers.

Trip status updates must use dedicated actions (`/arrived/`, `/start/`, `/complete/`, `/cancel/`). Do not permit arbitrary client PATCH of the status field. Cancellation endpoints must require `cancellation_reason` and record attribution (`STUDENT`, `RIDER`, or `SYSTEM_TIMEOUT`).

## 7. Notification endpoints

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/notifications/` | List current user's notifications |
| POST | `/notifications/{id}/read/` | Mark one notification read |
| POST | `/notifications/read-all/` | Mark all current user's notifications read |

The API should return unread count or support a lightweight unread-count endpoint. Notifications should reference the related request/trip where possible.

## 8. Administrator endpoints

All endpoints below require an authenticated administrator permission.

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/admin/dashboard/` | Summary statistics |
| GET | `/admin/riders/` | List/filter rider applications |
| POST | `/admin/riders/{id}/approve/` | Approve rider |
| POST | `/admin/riders/{id}/reject/` | Reject rider |
| POST | `/admin/riders/{id}/suspend/` | Suspend rider |
| POST | `/admin/riders/{id}/reactivate/` | Reactivate rider |
| CRUD | `/admin/locations/` | Manage locations |
| CRUD | `/admin/routes/` | Manage routes |
| CRUD | `/admin/routes/{id}/stops/` | Manage ordered stops |
| CRUD | `/admin/rider-route-assignments/` | Assign riders to routes |
| CRUD | `/admin/route-fares/` | Manage effective-dated fares |
| CRUD | `/admin/pricing-configurations/` | Manage multiplier/config versions |
| GET | `/admin/ride-requests/` | Monitor requests |
| GET | `/admin/trips/` | Monitor trips |
| GET/PATCH | `/admin/complaints/` | Review complaints |
| GET | `/admin/activity-logs/` | Review important admin actions |

For fare updates, prefer creating a new effective-dated record rather than editing a record already used by a booking. If an old record is corrected administratively, preserve an audit trail and do not mutate existing request snapshots.

## 9. Permissions matrix

| Resource/action | Student | Rider | Admin |
|---|---|---|---|
| Browse active routes/locations | Yes | Yes | Yes |
| Create ride request | Own | No | Only if explicitly required |
| View ride request | Own | Offered/assigned only | Yes |
| Cancel request | Own, subject to policy | No | Yes, with audit |
| Accept/decline offer | No | Own offer | No |
| Update trip progress | No | Own assigned trip | Controlled override |
| View own trip history | Yes | Yes | Yes |
| Set rider availability | No | Own, if approved | Yes |
| Manage routes/fares | No | No | Yes |
| Approve riders | No | No | Yes |
| View verification documents | No | Own documents only | Authorised admins |

## 10. Status and error codes

Suggested stable error codes:
- `UNAUTHENTICATED`
- `PERMISSION_DENIED`
- `INVALID_LOCATION`
- `UNSUPPORTED_JOURNEY`
- `NO_ACTIVE_FARE`
- `QUOTE_EXPIRED`
- `FARE_CHANGED`
- `PASSENGER_LIMIT_EXCEEDED`
- `RIDER_NOT_APPROVED`
- `RIDER_UNAVAILABLE`
- `REQUEST_ALREADY_ASSIGNED`
- `INVALID_STATUS_TRANSITION`
- `REQUEST_EXPIRED`
- `CANCELLATION_NOT_ALLOWED`

Use:
- `400` for invalid input.
- `401` for unauthenticated.
- `403` for authenticated but unauthorised.
- `404` for inaccessible/nonexistent resources where appropriate.
- `409` for state/concurrency conflicts.
- `429` for rate limiting.
- `500` only for unexpected server failures.

## 11. API implementation and documentation

Use DRF serializers for input/output validation and permissions for access control. Put multi-step business operations (fare quote, request creation, rider matching, accept, cancel, trip transitions) in service functions/classes rather than large view methods.

Generate an OpenAPI schema using a compatible DRF schema package once the endpoint contract stabilises. Keep this document aligned with the implemented schema; update both when a contract changes.
