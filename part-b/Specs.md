# Feature Specifications — Part B

The following are six feature specifications derived from Part A `Problems.md`.

## Feature Spec 1: Reliable Tatkal Virtual Queue and Booking Feedback

### Problem Statement
During Tatkal release windows the site becomes slow or unresponsive and users receive no clear feedback about queue position or whether their booking request is processing, causing failed bookings and session expirations (see `part-a/Problems.md`, Problem 1).

### Current State (from Part A)
The booking flow stalls at the moment of submission (steps 6–8 in Problem 1): users click `Book Now`, the site slows, they refresh, sessions time out, and bookings fail with no clear queueing or retry guidance.

### Proposed Solution
Introduce a virtual queue and explicit booking state UI that shows queue position, estimated wait time, and clear outcomes (queued, processing, confirmed, failed). Provide automatic retry for transient failures and persistent booking state so users don't need to re-submit during peak load.

### Proposed User Flow — Step by Step
1. User prepares booking and clicks `Book Now` at release time.
2. System immediately places request into a virtual queue and shows a `Queued` screen with position and ETA.
3. Periodic updates refresh position; user may continue preparing passengers or view booking status.
4. When request reaches front, attempt booking; show `Processing` state and a spinner with timeout guidance.
5. On success show confirmation with PNR; on failure show clear error with suggested next steps (retry or save passengers).

### Technical Implementation Plan
**System components affected:**
- Booking API gateway, queue service, frontend booking pages, session management

**New data requirements:**
- `queued_requests` table: id, user_id, request_payload (hashed/pointer), status, position_estimate, created_at, updated_at
- Booking state field on user session: `last_booking_request_id`

**API changes:**
- POST `/api/book` → returns immediate 202 + queue_id when under load
- GET `/api/queue/:id/status` → returns position, eta, and status
- Webhook or push endpoint for booking result (or server-sent events)

**Frontend changes:**
- New `Queued` screen component, queue position indicator, cancel request control, persisted booking status view
- Replace immediate success/failure modal with queued + processing flow

**Third-party services (if any):**
- Optional message queue (Redis Streams / RabbitMQ) and in-memory store (Redis) for position estimation

### Success Metrics
- Booking completion rate during Tatkal windows: increase from baseline → +20% relative
- Number of repeated refreshes per session: decrease by 50%
- User-reported clarity (survey): clarity score +15 points

### Edge Cases and Constraints
- Race conditions in position estimation; approximate ETA only
- IRCTC transaction time limits and payment gateway timeouts
- Graceful degradation: if queue service unavailable, fall back to existing synchronous booking with a clear warning and disable auto-retry

---

## Feature Spec 2: Persistent Search Filters

### Problem Statement
Search filters (class, departure time, availability) are lost when users navigate away and return, forcing repeated configuration and increasing friction (see `part-a/Problems.md`, Problem 2).

### Current State (from Part A)
Filters are applied on the train listing but cleared when users open details and return (steps 7–8). The UI does not persist filter state in route or local storage.

### Proposed Solution
Persist search filters in the URL and local/session storage so they survive navigation and page reloads. Restore filters on return and add a visual indicator that filters are active.

### Proposed User Flow — Step by Step
1. User searches for trains and applies filters.
2. The URL updates with query params encoding filters; local storage also stores the filter set.
3. User opens train details and returns — the search results reload with same filters and highlighted filter chips.
4. If user opens a new search, UI resets only when user clears filters explicitly or starts a new search.

### Technical Implementation Plan
**System components affected:**
- Frontend search page, routing, state management

**New data requirements:**
- No server-side data required; client-side storage only (sessionStorage/localStorage)

**API changes:**
- None required; ensure server respects query params for filtering

**Frontend changes:**
- Encode filters in URL query string and mirror to `sessionStorage`.
- On mount, read query params or session storage and restore filter state.
- Add a `Clear filters` control and visible active filter chips.

**Third-party services (if any):**
- None

### Success Metrics
- % of sessions where filters persist after navigation: target 95%
- Time to find desired train (median): reduce by 20%
- User satisfaction with search UX (survey): +10 points

### Edge Cases and Constraints
- Large query strings could exceed URL length limits — use short keys or base64-encoded JSON when needed
- Private/incognito sessions may not allow localStorage — degrade to URL-only persistence
- Ensure filters do not leak private data via shared URLs

---

## Feature Spec 3: Persist Seat Preference Through Booking

### Problem Statement
Passengers' berth/seat preferences reset between booking steps, forcing re-entry and causing frustration (see `part-a/Problems.md`, Problem 3).

### Current State (from Part A)
Users select berth preference on passenger details but the selection disappears in review stage (steps 5–7). Preference isn't persisted or shown in summary.

### Proposed Solution
Persist seat/berth preferences in the booking payload and surface them in the review and payment screens with an explicit `Preference saved` indicator. Allow editing without losing previously-entered data.

### Proposed User Flow — Step by Step
1. User fills passenger details and selects berth preference.
2. Preference is saved to the booking draft (client and server draft endpoint).
3. On review screen the selected preference is displayed per passenger.
4. User may edit preference and changes persist back to the draft.

### Technical Implementation Plan
**System components affected:**
- Passenger form, booking draft API, review page

**New data requirements:**
- Add `seat_preference` field to booking draft and passenger objects

**API changes:**
- POST/PUT `/api/booking-draft` to save intermediate booking state

**Frontend changes:**
- Update passenger form component to write preference into draft
- Display preference in booking summary and confirmation

**Third-party services (if any):**
- None

### Success Metrics
- % of bookings where preference persisted to confirmation: target 99%
- Decrease in preference re-entry actions per booking: -90%

### Edge Cases and Constraints
- Railway allocation rules may ignore preferences (e.g., only RAC/available). Surface this constraint in UI.
- If preference cannot be honored, show fallback explanation and offered alternatives

---

## Feature Spec 4: Prominent PNR and Journey Status Access

### Problem Statement
PNR status and journey-related actions are hard to discover after booking, requiring multiple navigation steps (see `part-a/Problems.md`, Problem 4).

### Current State (from Part A)
PNR checking lives in a separate area; users must hunt through menus to find it (steps 4–5), resulting in poor discoverability.

### Proposed Solution
Add a persistent `My Trips / PNR` quick-access widget on the logged-in homepage and booking confirmation pages, plus contextual links in the booking confirmation email and SMS. Provide a single-entry PNR lookup on the homepage with suggested recent PNRs.

### Proposed User Flow — Step by Step
1. After booking the confirmation screen shows a `View PNR & Journey` card and saves PNR to `Recent trips` in user profile.
2. On homepage a persistent `My Trips` card lists recent PNRs and quick actions (view status, cancel, share).
3. From any page, user can click `My Trips` and see PNR statuses without navigating deep into menus.

### Technical Implementation Plan
**System components affected:**
- User profile service, booking confirmation flow, homepage UI

**New data requirements:**
- `recent_pnrs` per user with timestamp and cached status

**API changes:**
- GET `/api/user/pnrs` → returns recent PNRs and cached status
- GET `/api/pnr/:pnr` → fetch latest status

**Frontend changes:**
- New `My Trips` component and `PNR card` on confirmation/home pages

**Third-party services (if any):**
- Existing PNR/status backend or railway APIs

### Success Metrics
- Time-to-PNR-access (median clicks): reduce to 1
- % of users using `My Trips` within 24h of booking: target 60%

### Edge Cases and Constraints
- PNR status updates depend on backend / railway API cadence — cache and show last-updated timestamp
- For guests (not logged in), allow one-time PNR lookup with short-term local storage

---

## Feature Spec 5: Mobile-Optimized Booking Form (Reduced Scrolling)

### Problem Statement
The mobile booking flow requires excessive vertical scrolling across passenger and contact fields, causing missed fields and validation errors (see `part-a/Problems.md`, Problem 5).

### Current State (from Part A)
On mobile the passenger form is long and stacked; users often miss required fields and face validation errors after long scrolls (steps 5–7).

### Proposed Solution
Re-structure the mobile booking form into a progressive, multi-step form with sticky action bar and field grouping (passengers, contacts, quota, payment). Use inline validation and a step summary to reduce scrolling.

### Proposed User Flow — Step by Step
1. User opens passenger form and sees step 1 (Passengers) with 1–2 passengers inline.
2. User advances to Contacts step (sticky Next/Back controls); quota selection in a compact modal.
3. Final step shows a review summary with any missing fields highlighted and direct jump links.

### Technical Implementation Plan
**System components affected:**
- Frontend mobile booking UI, form validation logic

**New data requirements:**
- No new server-side fields; local draft storage of multi-step progress

**API changes:**
- Reuse booking draft API for saving intermediate steps

**Frontend changes:**
- Multi-step form component, sticky action bar, inline validators, step-summary modal

**Third-party services (if any):**
- None

### Success Metrics
- Mobile form completion rate: increase by 25%
- Validation errors per session: reduce by 50%
- Average time-to-complete booking on mobile: reduce by 20%

### Edge Cases and Constraints
- Devices with small viewports or very slow networks — ensure form is functional with minimal JS and can fall back to single-page form

---

## Feature Spec 6: Clear Cancellation & Refund Summary

### Problem Statement
Refund rules, timelines, and expected amounts are fragmented across pages; users are unclear about refund amounts at the time of cancellation (see `part-a/Problems.md`, Problem 6).

### Current State (from Part A)
After cancellation users must search policy pages and pieces of content to learn refund expectations (steps 6–8), leaving them uncertain about deductions and timelines.

### Proposed Solution
Show an inline `Refund Summary` on the cancellation confirmation dialog that calculates estimated refund amount, lists deduction rules applied, and shows estimated processing time. Include links to full policy and appeal steps.

### Proposed User Flow — Step by Step
1. User opens a booked ticket and taps `Cancel`.
2. A `Refund Summary` modal displays the estimated refund amount, breakdown of deductions, and expected processing timeline.
3. User confirms cancellation; confirmation screen includes a link to the refund tracker and expected date.

### Technical Implementation Plan
**System components affected:**
- Cancellation service, policy engine, frontend cancellation modal

**New data requirements:**
- Ensure cancellation policy rules are available via an internal `policy` service or CDN JSON

**API changes:**
- GET `/api/cancellation/refund-estimate?ticket_id=...` → returns breakdown and ETA

**Frontend changes:**
- Cancellation modal with refund breakdown, `Appeal` link, and processing status tracker

**Third-party services (if any):**
- Payment gateway or refunds processing partner for actual timeline updates

### Success Metrics
- % user-reported clarity about refunds after cancellation: target +30 points
- Support requests about refund timing: reduce by 40%

### Edge Cases and Constraints
- Refund depends on bank/payment gateway processing times beyond IRCTC control; always show `estimated` and include last-updated timestamps
- Partial refunds for group bookings / agent bookings may require extra reconciliation; surface contact support option
