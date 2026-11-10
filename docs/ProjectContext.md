# PRAGYALINK
## Campus Pragya Ride Coordination and Management Platform

**Project Type:** Full-stack web application  
**Initial Deployment Area:** University of Cape Coast (UCC), Ghana  
**Primary Users:** Students, Pragya Operators and Administrators  
**Development Approach:** Modular Monolithic Architecture  
**Project Stage:** Requirements Refinement and Initial Development Planning

---

# 1. PROJECT OVERVIEW

PragyaLink is a mobile-friendly web application designed to improve access to Pragya transportation within the University of Cape Coast community.

The platform connects students seeking transportation with verified Pragya operators through a structured ride-request, fare estimation and trip coordination system.

The application addresses the difficulties students encounter when trying to find available Pragya operators, particularly students residing in private hostels and locations that are not immediately close to established Pragya pickup points.

Rather than requiring students to walk to a busy road or wait until they encounter an available Pragya, the platform allows them to submit a transportation request from a recognised pickup location and receive responses from compatible, available operators.

PragyaLink is not intended to function as an unrestricted ride-hailing application.

It must reflect the actual operating conditions of Pragya transportation within UCC, including designated routes, restricted roads, approved pickup locations, passenger capacity and existing fare arrangements.

The initial implementation will focus on UCC. The architecture should, however, accommodate the possibility of introducing additional routes, locations and operating areas in the future without requiring major architectural changes.

The ultimate objective is to produce a practical platform that students and Pragya operators can continue using after the academic project has been completed.

# 2. PROBLEM STATEMENT

Students within the University of Cape Coast frequently rely on Pragya transportation to move between halls, private hostels, academic areas and other destinations.

Finding an available Pragya can be inconvenient, particularly when a student is located away from a designated pickup point.

Students may have to walk to another location, wait for an available operator or depend on personal contacts to arrange transportation.

Additionally, passengers may not always know the applicable fare for a particular stretch, the availability of operators or whether a particular destination is served by an authorised Pragya route.

The existing arrangement also presents coordination challenges for Pragya operators who must identify passengers, manage their availability and communicate with potential customers.

PragyaLink seeks to address these challenges through a centralised platform for requesting, assigning and managing Pragya transportation.

# 3. CORE OBJECTIVES

The application should:

1. Make it easier for students to request Pragya transportation.
2. Connect students with verified and available Pragya operators.
3. Enforce predefined and approved operating routes.
4. Support both regular/shared rides and private Dropping services.
5. Allow groups of students to book a single Dropping service.
6. Provide transparent fare estimates before a request is submitted.
7. Allow administrators to manage routes, locations and fare structures.
8. Improve communication between students and operators.
9. Maintain records of requests, trips, cancellations and completed journeys.
10. Provide administrators with tools for monitoring and managing platform activity.

# 4. USER ROLES

The application has three primary user categories.

## 4.1 STUDENT

Students are the primary customers of the platform.

They should be able to:

- Register and log in.
- Manage their profiles.
- Select pickup locations.
- Select destinations.
- Select a booking type.
- Request regular/shared rides.
- Request private Dropping services.
- Specify the number of passengers for group bookings.
- View estimated fares.
- Submit ride requests.
- View request status.
- View assigned rider and vehicle information.
- Cancel eligible requests.
- View previous trips.
- Receive notifications.
- Submit complaints or feedback.

Students must not have access to rider verification information, administrative functionality or other students' private trip information.

## 4.2 PRAGYA RIDER

Pragya riders are transportation operators registered on the platform.

They should be able to:

- Register as a rider.
- Submit personal and vehicle information.
- Submit required verification details.
- View their verification status.
- Select approved operating routes.
- Manage their availability.
- Receive compatible ride requests.
- Accept or reject requests.
- View pickup and destination information.
- View the booking type and passenger count.
- View the quoted fare.
- Update trip progress.
- Cancel an accepted request where necessary.
- Mark trips as completed.
- View trip history.

A rider must not receive requests unless their account has been approved and they are currently available.

Riders must not be able to approve themselves, modify official fare configurations or create unauthorised routes.

## 4.3 ADMINISTRATOR

Administrators manage the overall platform.

They should be able to:

- View platform statistics.
- Approve and reject rider applications.
- Suspend and reactivate rider accounts.
- Manage student accounts.
- Manage approved routes.
- Manage locations and pickup points.
- Manage route stops and route connections.
- Manage route fares.
- Configure Dropping pricing rules.
- Monitor ride requests.
- Monitor ongoing and completed trips.
- View cancelled requests.
- Manage complaints and reports.
- View important administrative activity.

Administrator accounts must be created through authorised administrative processes rather than public registration.

# 5. BOOKING TYPES

PragyaLink supports two primary booking types.

## 5.1 REGULAR / SHARED RIDE

This represents the conventional shared Pragya arrangement.

A student requests transportation for a particular stretch and pays the established passenger fare for that journey.

Other passengers may be picked up along the route, subject to vehicle capacity and operating restrictions.

The fare is calculated using the active fare configuration for the selected journey.

Example:

Route: Science to Ayensu

Regular passenger fare: GH₵5

Passenger count: 1

Estimated fare: GH₵5

The example is illustrative. Actual fares must be established through field research.

## 5.2 DROPPING / PRIVATE HIRE

Dropping allows a student to hire the entire Pragya for a journey.

The rider agrees to transport the requesting passenger or group without picking up unrelated passengers during that trip.

The Dropping fare is initially calculated using:

Dropping Fare = Regular Route Fare × Dropping Multiplier

The initial proposed multiplier is 4, subject to confirmation through research with UCC Pragya operators.

The multiplier must be configurable by administrators and must not be permanently hardcoded.

Students must be able to request Dropping for themselves or for a group of friends.

A group booking represents one vehicle reservation and one trip, not multiple independent bookings.

The student making the request is the booking organiser.

The platform should record the number of passengers and ensure that it does not exceed the configured permitted vehicle capacity.

The actual passenger capacity and the Dropping multiplier are separate configuration values and must be independently managed.

The total fare may be displayed alongside an indicative per-person cost.

The platform does not initially need to collect individual contributions from each group member.

# 6. FARE ESTIMATION AND MANAGEMENT

Fare management is a core component of PragyaLink.

Fares must be stored in the database and managed through the administrator dashboard.

Each fare should be associated with the appropriate route or journey stretch.

Administrators should be able to:

- Create route-specific fares.
- Edit future fare configurations.
- Schedule fare changes.
- Deactivate outdated fare records.
- View previous fare configurations.
- Manage the Dropping multiplier.
- Manage passenger capacity settings.

Fare changes should support effective dates.

The application must preserve historical pricing information.

When a student receives a fare estimate, the application must calculate the price using the applicable active pricing configuration.

When the request is submitted, the quoted fare and the pricing information used to calculate it must be stored with the request.

Subsequent fare changes must not silently modify existing requests or completed trips.

All monetary values must use appropriate decimal precision rather than floating-point arithmetic.

The initial application currency is Ghanaian Cedi (GHS).

# 7. ROUTE AND LOCATION MANAGEMENT

Route management is one of the most important components of the application.

Pragya operators are subject to designated operating routes and road restrictions.

The platform must not allow unrestricted transportation requests that conflict with established operating arrangements.

Routes must be created and managed by authorised administrators.

Riders should select from approved routes rather than freely inventing operating routes.

The system should distinguish between:

- Locations.
- Approved pickup points.
- Route stops.
- Operating routes.
- Route connections.

Locations may include halls, private hostels, academic areas and other recognised destinations.

A private hostel may be a valid pickup location without being an official route.

The route system should accommodate future additions and changes without requiring developers to modify application code.

The initial routes will be established through research and stakeholder confirmation.

The application should also account for travel direction where necessary.

Where a requested journey cannot be served through an approved route or combination of permitted route segments, the application must not assign an arbitrary rider.

It should inform the student that the requested journey is currently unsupported.

# 8. RIDE REQUEST AND TRIP LIFECYCLE

The general workflow is:

1. Student logs in.
2. Student selects pickup location.
3. Student selects destination.
4. Student selects Regular Ride or Dropping.
5. Student specifies the passenger count where applicable.
6. System validates the journey against the approved route network.
7. System calculates the estimated fare.
8. Student reviews and submits the request.
9. System identifies compatible approved riders.
10. Compatible riders receive the request.
11. A rider accepts or rejects the request.
12. Student receives an assignment notification.
13. Rider proceeds to the pickup point.
14. Rider marks arrival.
15. Trip begins.
16. Rider completes the trip.
17. System records the completed journey.

The system must prevent conflicting assignments.

A rider must not accept multiple incompatible active trips.

Request cancellation, expiration and rejection must be supported.

The system should maintain a clear distinction between a ride request and an actual trip.

# 9. REQUEST STATUSES

The application should support appropriate request and trip statuses.

Suggested request statuses:

- PENDING
- OFFERED
- ACCEPTED
- REJECTED
- CANCELLED
- EXPIRED

Suggested trip statuses:

- ASSIGNED
- RIDER_ARRIVING
- ARRIVED
- IN_PROGRESS
- COMPLETED
- CANCELLED

The exact status transitions must be validated by backend business logic.

Users must not be able to arbitrarily modify request or trip statuses.

# 10. NOTIFICATIONS

The application should provide basic notifications for important events.

Examples:

- A rider receives a new request.
- A rider accepts a request.
- A rider rejects a request.
- A rider reports arrival.
- A trip begins.
- A trip is completed.
- A request is cancelled.

The first version should use database-backed in-app notifications.

Real-time communication may initially use periodic API polling.

WebSocket communication can be introduced later if required.

# 11. DATABASE DESIGN DIRECTION

The initial database should be designed around the application's actual business entities.

Suggested Django models include:

- User
- StudentProfile
- RiderProfile
- Vehicle
- Location
- Route
- RouteStop
- RiderRoute
- RouteFare
- PricingSettings
- RideRequest
- Trip
- Notification
- Complaint
- ActivityLog

These are proposed entities, not an instruction to create every model immediately.

The final schema must be developed after requirements analysis.

Important relationships:

- A user has an appropriate account role.
- A student can submit multiple requests.
- A rider can operate multiple approved routes.
- A route contains recognised stops or locations.
- A route can have multiple fare records over time.
- A ride request belongs to one student.
- An accepted request can result in one trip.
- A trip belongs to one student or booking group and one rider.
- A completed trip preserves its final fare.

The backend must protect against duplicate assignments and conflicting active trips using database transactions and appropriate constraints.

# 12. TECHNICAL STACK

The preferred technology stack is:

Frontend:
- React.js
- Vite
- JavaScript
- CSS or Tailwind CSS

Backend:
- Python
- Django
- Django REST Framework

Database:
- PostgreSQL
- Neon

Authentication:
- Django authentication system
- SimpleJWT

Frontend deployment:
- Vercel

Backend deployment:
- Render

Image/Media storage:
- None in MVP (Cloudinary removed; verification is conducted via physical station checks & identity numbers to avoid 3rd-party costs)

Version control:
- GitHub

UI/UX planning:
- Figma (optimized for mobile view and high-contrast Rider "Driver Mode")

Prohibited / Out-of-Scope for MVP:
- NO GPS live-tracking or geolocation tracking APIs.
- NO Leaflet, OpenStreetMap, Google Maps, or Mapbox mapping libraries.
- NO Payment gateway SDKs (Paystack, Flutterwave, Stripe); payment is direct cash/MoMo to rider.
- NO Paid SMS gateways (Twilio, Hubtel); communication uses Click-to-Call and in-app polling.
- NO In-app chat engines; direct phone calls are used.
- NO Graph-based multi-leg route transfers; single corridor trips only.

Potential future technologies (post-pilot):
- Django Channels / WebSockets for high-frequency real-time updates.
- Progressive Web App capabilities.
- Browser push notifications.

Do not introduce additional technologies without a clear functional requirement.

# 13. ARCHITECTURAL PRINCIPLES

The application should use a modular monolithic architecture.

The React frontend and Django backend are separate applications communicating through a REST API.

Django should contain the authoritative business logic.

React is responsible for presentation, user interaction and client-side validation.

Important principles:

1. Do not duplicate business logic between frontend and backend.
2. Do not hardcode routes or fares in frontend components.
3. Do not trust client-submitted prices.
4. Recalculate and validate fares on the backend.
5. Enforce role-based permissions on every protected backend endpoint.
6. Preserve historical pricing records.
7. Prevent conflicting rider assignments.
8. Keep the application modular and maintainable.
9. Avoid unnecessary abstractions and overengineering.
10. Design for mobile-first use.

# 14. SECURITY AND PRIVACY

The application must:

- Protect user authentication credentials.
- Enforce role-based access control.
- Restrict access to personal information.
- Restrict access to rider verification documents.
- Protect administrative endpoints.
- Validate all incoming API data.
- Use environment variables for secrets.
- Apply appropriate API rate limiting.
- Prevent users from accessing other users' private trips.
- Maintain records of important administrative actions.

A user must never gain administrative privileges by modifying frontend data or submitting a different role through the registration endpoint.

# 15. INITIAL PRODUCT SCOPE

The first functional version should include:

- Student registration and authentication.
- Rider registration and manual in-person verification workflow.
- Administrator authentication.
- Role-based dashboards (including high-contrast large-target "Driver Mode" for riders).
- Route and landmark location management (single-corridor routes).
- Pickup landmark notes for precise meetups without coordinates.
- Direct "Click-to-Call" voice connection between matched student and rider during active trips.
- Request broadcast countdown with anti-stall fallback (default 90-second timeout).
- Rider route selection.
- Rider availability management.
- Regular ride requests (single-passenger point-to-point corridor hails).
- Dropping requests.
- Group Dropping bookings (within configured passenger capacity).
- Route-based fare estimation and binding snapshots.
- Direct offline settlement prompt (Cash or Rider's MoMo on arrival).
- Configurable Dropping multiplier.
- Rider request acceptance and rejection.
- Trip status management (with dedicated arrival, start, and complete actions).
- Structured cancellation tracking (`STUDENT`, `RIDER`, `SYSTEM_TIMEOUT`) with mandatory reason logging.
- Trip history.
- Lightweight in-app notifications and polling.
- Administrative monitoring.

The first version explicitly EXCLUDES (prohibited for MVP):

- Live GPS tracking, geolocation coordinates, and map APIs (Google Maps, Leaflet, Mapbox).
- Integrated online payment gateways (Paystack, Flutterwave, Stripe).
- Paid SMS gateway notifications (Twilio, Hubtel).
- In-app text chat engines (voice calls via Click-to-Call are used).
- Multi-leg route connection graphs and dynamic pooling algorithms.
- Third-party media storage SDKs (Cloudinary; verification uses physical checks and ID numbers).
- AI-based rider allocation.
- Multiple university multi-tenancy.
- Separate native Android or iOS mobile applications.

These features may be evaluated after the core platform has been proven in campus use.

# 16. DEVELOPMENT EXPECTATIONS

The application must be developed incrementally.

Do not attempt to implement the entire platform in one development phase.

Begin with requirements analysis, database design and application architecture.

Implement and test each functional module before proceeding to the next.

Maintain consistency between the database, backend API and frontend.

Avoid unnecessary modifications to existing working functionality.

Every significant feature should have appropriate validation and test coverage.

The project should remain understandable to developers who did not implement the original feature.

# 17. DEFINITION OF SUCCESS

PragyaLink should be considered successful when:

1. A student can register and request a regular Pragya ride.
2. A student can request a private Dropping service.
3. A student can book a Dropping service for a group within the permitted passenger capacity.
4. The system calculates the correct fare using administrator-managed pricing.
5. The system respects approved routes and operating restrictions.
6. Only eligible riders receive compatible requests.
7. A rider can accept and complete a trip.
8. Students can follow the progress of their requests.
9. Administrators can manage riders, locations, routes and fares without developer intervention.
10. Historical requests and trips retain their original pricing information.
11. The application is usable on ordinary mobile devices.
12. Actual students and Pragya operators can successfully use the deployed platform.

The overall objective is not simply to demonstrate that the application works.

It is to build a practical campus transportation coordination platform that can be tested, adopted and improved through real-world usage.