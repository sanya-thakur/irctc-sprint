# IRCTC Problem Discovery — Part A

## Summary

* Total problems documented: 6 (3 given + 3 self-discovered)
* Platform explored: IRCTC Rail Connect Website
* Devices used: Desktop Chrome, Mobile Chrome
* Date of exploration: June 2026

---

# Problem 1: Tatkal Booking Failure During Peak Release Time [Given]

## What is broken

During Tatkal booking hours, users frequently experience slow loading, session expiration, failed booking attempts, and lack of meaningful feedback when demand spikes.

The system provides limited visibility into queue position, expected wait time, or booking status during peak traffic.

## Affected users

* Daily commuters
* Emergency travelers
* Students
* Business travelers
* Users booking Tatkal tickets

Potentially affects hundreds of thousands of users during Tatkal windows.

## Frequency

* Daily
* Most visible during Tatkal release periods
* Highest impact around booking opening times

## Current flow — step by step

1. User opens IRCTC around 9:50 AM.
2. User logs in and searches for a train.
3. User selects Tatkal quota.
4. User fills passenger details in advance.
5. User waits for booking window to open.
6. User clicks "Book Now" immediately after release.
7. Website slows significantly or becomes unresponsive.
8. User refreshes repeatedly.
9. Session may expire or request may fail.
10. User discovers tickets are no longer available.

## Where exactly it breaks

Step 6–8.

The booking request is submitted, but the platform struggles to provide timely feedback under high traffic conditions. Users are left uncertain whether the request is processing, failed, or queued.

---

# Problem 2: Search Filters Do Not Persist Reliably [Given]

## What is broken

Filters such as class type, departure time, and availability do not always behave predictably. Users may lose filter selections after navigation or receive inconsistent search results.

## Affected users

* General ticket booking users
* Users comparing train options
* Travelers with specific class preferences

## Frequency

* Frequent during train search sessions
* Occurs whenever users repeatedly modify filters

## Current flow — step by step

1. User enters source and destination stations.
2. User searches trains.
3. Train list loads.
4. User applies "Available Seats" filter.
5. User applies class filter.
6. User opens a train detail page.
7. User returns to search results.
8. Previously selected filters are removed.
9. User repeats filtering process.

## Where exactly it breaks

Step 7–8.

Filter state is not reliably preserved when users navigate away and return to search results.

---

# Problem 3: Seat Preference Selection Resets [Given]

## What is broken

Users selecting berth preferences may not see those preferences consistently reflected in later booking stages.

## Affected users

* Senior citizens
* Families traveling together
* Passengers with medical needs
* Users preferring lower berths

## Frequency

* Common during booking sessions
* More noticeable on mobile devices

## Current flow — step by step

1. User searches for a train.
2. User chooses a train and class.
3. User proceeds to passenger details.
4. User selects lower berth preference.
5. User proceeds to next screen.
6. User reviews booking information.
7. Selected berth preference appears missing or reset.
8. User returns and re-enters preference.

## Where exactly it breaks

Step 5–7.

The selected berth preference is not clearly persisted or surfaced in later booking stages.

---

# Problem 4: PNR Status and Journey Information Are Difficult to Discover [Self-Discovered]

## How I found it

While attempting to locate booking status information after a ticket search session.

## Screenshot or description

The PNR-related functionality is separated from the main booking workflow and requires additional navigation steps that are not obvious to first-time users.

## What is broken

Critical post-booking actions such as PNR status checking are not prominently integrated into the user journey.

## Affected users

* First-time railway passengers
* Elderly users
* Infrequent travelers

## Frequency

* Every time users need journey status updates

## Current flow — step by step

1. User completes booking.
2. User leaves website.
3. User later returns to check ticket status.
4. User searches homepage for PNR information.
5. User navigates through multiple menus.
6. User eventually finds PNR status page.
7. User enters PNR details.

## Where exactly it breaks

Step 4–5.

The information architecture makes important post-booking functions harder to discover than expected.

---

# Problem 5: Mobile Booking Experience Contains Excessive Scrolling [Self-Discovered]

## How I found it

Tested booking flow on mobile browser.

## Screenshot or description

Passenger details, contact information, quota selection, and booking options appear in a long vertically stacked form.

## What is broken

The booking flow becomes difficult to navigate on smaller screens because of excessive scrolling and limited visual hierarchy.

## Affected users

* Mobile browser users
* Elderly users
* Users on slower networks

## Frequency

* Every mobile booking session

## Current flow — step by step

1. User opens IRCTC on mobile browser.
2. User searches train.
3. User selects train.
4. Passenger form opens.
5. User scrolls extensively through multiple sections.
6. User misses required field.
7. Validation error appears.
8. User scrolls again to locate issue.

## Where exactly it breaks

Step 5–7.

The form structure increases cognitive load and makes error correction difficult.

---

# Problem 6: Cancellation and Refund Information Lacks Transparency [Self-Discovered]

## How I found it

While exploring ticket cancellation and refund-related pages.

## Screenshot or description

Refund expectations, timelines, and cancellation outcomes are spread across multiple screens and policy references.

## What is broken

Users often do not immediately understand refund amount, deduction rules, or expected processing timelines.

## Affected users

* Users cancelling confirmed tickets
* Waitlisted passengers
* Tatkal passengers

## Frequency

* Every cancellation workflow

## Current flow — step by step

1. User opens booked ticket.
2. User selects cancellation.
3. System asks for confirmation.
4. User confirms cancellation.
5. Ticket is cancelled.
6. User searches for refund details.
7. User reviews policy information.
8. User remains unsure about refund timeline.

## Where exactly it breaks

Step 6–8.

Refund communication is not surfaced clearly at the point where users make cancellation decisions.

---

# Conclusion

The six documented issues span:

1. High-demand booking reliability
2. Search and filtering consistency
3. Seat preference persistence
4. Information architecture
5. Mobile usability
6. Cancellation transparency

Together, these problems affect both booking success rates and overall user confidence throughout the IRCTC journey. These findings will be used in Part B to create feature specifications and product improvements.
