# Impact vs Effort Matrix

## The Matrix

|                   | Low Effort         | High Effort        |
|-------------------|--------------------|--------------------|
| **High Impact**   | Persistent Search Filters, Refund Summary    | Reliable Tatkal Queue    |
| **Low Impact**    | Seat Preference Persistence    | Mobile Optimized Booking Form, PNR Discovery    |

## How I Scored Each Dimension

### Impact Scoring (1–5)
I scored Impact based on:
- Number of users affected (from Part A frequency)
- Whether the problem is in the core booking flow
- Severity of consequence for the user

### Effort Scoring (1–5)
I scored Effort based on:
- Number of system components touched
- Whether new infrastructure is required
- Risk of breaking existing flows
- Railway API dependencies

---

## Placement Justifications

### Reliable Tatkal Queue — High Impact / High Effort
This problem affects many users at peak times and directly changes booking success rates; implementation requires queue infrastructure, coordination with payment and gateway timeouts, and careful testing.

### Persistent Search Filters — High Impact / Low Effort
Many users lose filters frequently; changes are largely frontend-only (URL/state persistence) and produce high UX gains with low backend impact.

### Seat Preference Persistence — Low Impact / Low Effort
Important to affected passengers but limited scope (per-passenger UI + draft API); low backend complexity.

### PNR Discovery — Low Impact / High Effort
Useful for post-booking users but requires cross-cutting profile work, cached PNR storage, and homepage UI changes which need broader product changes.

### Mobile-Optimized Booking Form — Low Impact / High Effort
Mobile usability improvements have strong user benefits but require significant frontend redesign and QA across device types.

### Refund Summary — High Impact / Low Effort
Refund clarity reduces support volume and increases trust; calculating estimates is a small API addition and a UX modal — relatively low effort.

---

## Recommended Sprint Order
1. Persistent Search Filters — quick wins and visible UX improvement
2. Refund Summary — reduces support load and clarifies cancellations
3. Seat Preference Persistence — small, high-confidence change
4. Reliable Tatkal Queue — prioritized engineering effort for peak success
5. Mobile-Optimized Booking Form — larger frontend effort, schedule after critical fixes
6. PNR Discovery — follow with profile work and cross-cutting changes
