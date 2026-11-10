# PragyaLink — Research Findings and Requirements Register

**Document status:** Research not yet conducted; this is a research plan and evidence register  
**Research area:** University of Cape Coast and surrounding student transportation environment  
**Prepared:** 10 November 2026

> **Important:** No student survey or Pragya-operator interviews have been completed or supplied for this document. Therefore, this file contains no verified survey findings. Statements about routes, fares, passenger capacity and Dropping are working hypotheses derived from the project discussion and must be confirmed before they become final system requirements.

---

## 1. Research purpose

The purpose of the research is to establish whether PragyaLink addresses a real and sufficiently frequent problem, understand how campus Pragya transportation actually operates, and gather the rules needed to design a safe and useful first release.

Research must validate both sides of the platform:
- Students who request transportation.
- Pragya operators who may receive and fulfil requests.

Where necessary, the team should also consult relevant campus authorities or transport regulators about route and pickup restrictions. Do not treat informal assumptions as legal permission.

## 2. Current understanding — unverified

The following points are the current project hypotheses:

1. Some students, particularly those living in private hostels, have difficulty finding a Pragya near their preferred pickup point.
2. Students may walk to busier or designated locations before finding a ride.
3. Pragya operators work within designated routes and may be restricted from using certain campus roads or main roads.
4. Regular fares are associated with particular journey stretches and may change over time.
5. A service locally called **Dropping** allows one person or a group to hire the Pragya exclusively for a journey.
6. The proposed Dropping calculation is four times the regular fare for the same stretch (since the seating capacity for most pragya is four).
7. A group may share one Dropping booking and divide the cost among themselves.
8. The proposed multiplier of four may represent the revenue opportunity/seat fares foregone by the operator; the exact local meaning must be confirmed.
9. Passenger capacity must be established independently of the Dropping multiplier.
10. Riders may be willing to receive and respond to digital ride requests.

These are not research findings. They are hypotheses to test.

## 3. Research questions and evidence required

### 3.1 Student experience

Find out:
- How often do students use Pragya?
- Which locations do they travel between most frequently?
- How often can they find a ride at their desired pickup point?
- How long do they wait?
- Do they walk to another pickup point?
- Which hostels or areas experience the greatest difficulty?
- What happens when no rider is available?
- What are the main concerns: waiting, price uncertainty, route availability, safety, convenience or something else?
- Would students use a web platform to request a ride?
- What would stop them from using it?

Evidence:
- Survey responses.
- Short interviews.
- Specific examples of recent transport difficulties.
- Observed usage patterns, where feasible and ethical.

### 3.2 Rider operations

Find out:
- Which routes and directions does each operator serve?
- Which roads are prohibited or restricted?
- Where may riders legally/operationally pick up and drop off passengers?
- Can a rider enter private-hostel access roads?
- Must passengers meet at a recognised entrance or designated stop?
- Can a rider operate on more than one route?
- Can a rider change route during the day?
- What are the operating hours?
- How do riders currently indicate that they are available?
- How are passengers normally collected on a shared journey?
- Can a rider accept a request and still pick up other passengers?
- What should happen if a rider becomes unavailable after accepting?

Evidence:
- Interviews with multiple operators.
- A route/stop list confirmed by operators and, where appropriate, relevant authorities.
- A simple route sketch or map reviewed by people who operate locally.

### 3.3 Regular fares

Find out:
- What are the current fares for each commonly used stretch?
- Are fares charged per passenger, per seat, or per vehicle?
- Do fares vary by direction?
- Do fares vary by time, day, demand or pickup point?
- Are there different fares for intermediate stops?
- Who determines fare changes?
- How are changes communicated?
- How much notice is normally given?
- Are there exceptions or negotiated fares?

Evidence:
- Fare table collected from several operators.
- Date and source for each fare.
- Record of disagreement or variation between operators.
- Confirmation of who is authorised to approve the fare used by the platform.

Do not publish a fare as definitive if operators give conflicting answers. Record the variation and decide whether the platform should display a standard fare, a fare range, or a rider-confirmed quote.

### 3.4 Dropping and group bookings

Find out:
- What exactly does “Dropping” mean locally?
- Does the rider transport only the booking party for the whole stretch?
- Is the rider prohibited from picking up anyone else during that journey?
- Is Dropping always four times the regular fare?
- Does the multiplier apply to every stretch?
- Is the multiplier negotiated?
- Does the price vary by distance or time?
- Is the price based on four theoretical seats, a fixed private-hire rate, or another convention?
- What is the maximum permitted passenger count?
- Can the organiser travel with three friends?
- Are there vehicle types with different capacities?
- Who pays the rider?
- Does one organiser normally pay, or do passengers contribute separately?
- What happens if the passenger count changes after booking?
- What happens if fewer people arrive than booked?

Evidence:
- Interviews with operators who regularly provide Dropping.
- Student examples of actual Dropping journeys and prices.
- Confirmation of passenger capacity and applicable rules.
- A worked fare example for several different stretches.

### 3.5 Trust, safety and accountability

Find out:
- What information do students need to identify the assigned rider and vehicle?
- What rider verification is practical and legitimate?
- Is vehicle registration sufficient, or is additional evidence required?
- Who will verify the details?
- What should a student do if the rider does not arrive?
- What should happen if the rider or passenger cancels?
- How should complaints be submitted and handled?
- What personal information is genuinely necessary?
- Is student phone-number registration acceptable?
- Are there institutional policies relevant to operating such a platform?

Evidence:
- Student and rider feedback.
- Confirmation from project supervisors or relevant university contacts where appropriate.
- A documented minimum verification process.

## 4. Proposed research methods

Use a combination of:
1. **Student Google Form:** broad collection of usage patterns, pain points, route use, fare awareness and feature interest.
2. **Rider interviews:** semi-structured conversations about routes, restrictions, fares, capacity, Dropping and willingness to participate.
3. **Route validation:** document each route direction, ordered stops, approved pickup/drop-off points and restricted segments.
4. **Fare validation:** collect current fares with date, route stretch and source; compare answers rather than assuming one answer is universal.
5. **Small prototype review:** show a simple booking flow to a few students and riders after the basic requirements are understood.

Avoid collecting names, index numbers or phone numbers in the student survey unless there is a clear reason. If follow-up interviews are optional, collect contact details in a separate form so survey answers can remain anonymous.

## 5. Student survey sections

Recommended form title: **UCC Student Transportation and Pragya Usage Survey**

Suggested sections:
1. Student context and residence area.
2. Pragya usage frequency.
3. Current pickup and waiting experience.
4. Common routes and locations.
5. Fare awareness and fare uncertainty.
6. Familiarity and experience with Dropping.
7. Group travel and payment arrangements.
8. Interest in PragyaLink.
9. Trust, privacy and safety concerns.
10. Open feedback.

Use conditional questions:
- Students who have never used Pragya should not be forced to answer questions about past fares.
- Students unfamiliar with Dropping should receive a short neutral explanation before answering hypothetical questions.
- Students who have used Dropping should be asked about actual routes and prices.
- Ask for approximate area/hostel rather than exact home address.

Avoid leading questions. For example, ask “What difficulties, if any, have you experienced?” rather than assuming every student has a problem.

## 6. Rider interview guide

Ask operators:
1. Which routes and directions are you permitted to operate?
2. Which roads or areas must you avoid?
3. Where can you legally and practically pick up passengers?
4. Can you collect from private hostel entrances?
5. What are the regular fares for the common stretches?
6. Are those fares per passenger?
7. How are fare changes decided and communicated?
8. What does Dropping mean in your day-to-day work?
9. How do you calculate Dropping fares?
10. Is the four-times calculation always used?
11. What is the permitted passenger capacity of your vehicle?
12. Can a group of four book one Dropping journey?
13. Can you accept a digital request while carrying other passengers?
14. What information would you need before accepting a request?
15. Would you use a mobile-friendly website to receive requests?
16. What would make the system inconvenient or unacceptable?
17. How should cancellations and no-shows be handled?

Record the interview date, respondent category and notes. Do not publish an operator's personal details without consent.

## 7. Evidence register

Populate this table after research. Do not invent values.

| ID | Topic | Current hypothesis | Evidence/source | Status |
|---|---|---|---|---|
| R-01 | Student pickup difficulty | Some students struggle to find pickups near their residence | Student survey/interviews | Not researched |
| R-02 | Designated routes | Operators are restricted to approved routes/roads | Rider interviews/authority confirmation | Not researched |
| R-03 | Pickup access | Some hostels may require a nearby approved meeting point | Rider interviews/route validation | Not researched |
| R-04 | Regular fare | Fare is set for particular journey stretches | Multiple operator interviews | Not researched |
| R-05 | Fare changes | Fare may change over time | Operator interviews and dated fare records | Not researched |
| R-06 | Dropping definition | Exclusive hire for one person/group | Rider and student interviews | Not researched |
| R-07 | Dropping multiplier | Four times regular fare may be used | Multiple operator interviews | Not researched |
| R-08 | Vehicle capacity | Passenger limit may differ from multiplier | Vehicle/operator/authority confirmation | Not researched |
| R-09 | Group booking | One organiser can book for a group | Student and rider interviews | Not researched |
| R-10 | Payment | Group may settle with rider directly | Student and rider interviews | Not researched |
| R-11 | Digital adoption | Riders and students may use a web platform | Prototype test/interviews | Not researched |
| R-12 | Verification | Rider/vehicle information can be verified through a practical process | Rider/authority consultation | Not researched |

## 8. Research log template

For each research activity, record:

- Activity ID:
- Date:
- Method: survey / interview / observation / document review
- Respondent group:
- Number of participants:
- Questions covered:
- Main observations:
- Conflicting answers:
- Evidence or source:
- Limitations/bias:
- Requirements affected:
- Follow-up action:

## 9. How findings become requirements

Do not move directly from one respondent's preference to a system feature.

For each proposed requirement:
1. Identify the problem or operating rule.
2. Record supporting evidence and how many sources support it.
3. Note conflicting evidence.
4. Decide whether it is essential for the MVP.
5. Write a testable requirement.
6. Update `PROJECT_CONTEXT.md`, `DATABASE_DESIGN.md`, `API_SPEC.md` and `DECISIONS.md` where relevant.

Example:

**Unvalidated hypothesis:** Dropping costs four times the normal fare.

**Evidence required:** Multiple operators confirm the calculation across several route stretches and explain exceptions.

**Possible requirement after validation:** “The system shall calculate a Dropping quote using the active administrator-configured multiplier applied to the regular fare for the selected journey stretch.”

If the multiplier is not consistent, do not force a fixed four-times rule. Consider a separately configured Dropping fare per stretch or a rider-confirmed quote workflow.

## 10. Research completion criteria

Research is sufficiently complete for MVP requirements when the team has:
- A validated list of initial route directions and restricted segments.
- A validated list of usable pickup/drop-off points.
- A fare table with source and effective date, or a documented decision to show fare ranges/require confirmation.
- A clear definition of Dropping and its pricing method.
- A confirmed passenger-capacity rule.
- A documented group-booking and payment process.
- Evidence that students experience the target problem.
- Evidence that enough riders are willing and able to participate.
- A minimum rider verification and complaint process.
- A documented list of unresolved questions and accepted limitations.

## 12. UCC MVP Baseline Defaults (for Development Agents)

To ensure coding agents do not get blocked waiting for field interviews during initial development, the following operational defaults are established for UCC seed fixtures:

- **Seating Capacity:** 3 passengers per tricycle (standard practical passenger capacity in UCC).
- **Dropping Multiplier:** 4.0x regular fare (stored in configurable `PricingConfiguration`, not hardcoded in code).
- **Default Currency:** `GHS` (Ghanaian Cedi).
- **Direct Settlement:** Direct Cash or Rider's MoMo on trip completion.
- **Request Timeout:** 90 seconds auto-expiration if unaccepted.
- **Initial UCC Corridor Fixture (`ucc_seed.json`):**
  - *Locations:* Science Taxi Rank, Old Site (Administration), New Site (NEC/Casely Hayford), Valco Flat, Ayensu Gate, Kwaprow Junction, Amamoma Entrance.
  - *Corridor 1:* Science Taxi Rank ↔ Ayensu Gate (via Valco Flat).
  - *Corridor 2:* Old Site ↔ New Site (NEC).
  - *Corridor 3:* Science Taxi Rank ↔ Kwaprow Junction.
  - *Regular Baseline Fare:* GH₵ 5.00 per person along direct corridor stretches.

