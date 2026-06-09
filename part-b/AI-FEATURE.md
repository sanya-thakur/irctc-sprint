# AI Feature Specification: Tatkal Success Predictor & Recommendation

## Problem It Solves
Addresses Problem 1 (Tatkal Booking Failure During Peak Release Time) by predicting the probability of a successful booking for a given train/quota at the time of request and recommending optimal trains/quotas to improve user success chances (see `part-a/Problems.md`, Problem 1).

## Proposed Feature — User Perspective
When a user prepares a Tatkal booking, the UI shows a `Success Probability` badge (e.g., `65%`) and a short recommendation list: `Top 3 alternatives` (same route, nearby departure, alternate quota). The badge appears on train results and on the queued booking screen before submission. If probability is low, the UI suggests actions (select alternate train, wait for queue, or pre-fill passengers for fast checkout).

## Model or API Choice
- Primary inference: OpenAI GPT-4 (text-based ranking) for human-readable explanations plus a lightweight on-premise model (XGBoost or LightGBM) for numeric probability scoring trained on historical booking events.
- Rationale: GPT-4 provides contextual recommendations and natural language suggestions; the gradient-boosted model provides fast, reproducible probability scores suitable for low-latency inference.

## Training or Input Data
- Historical booking logs: timestamps, train_id, quota, source/destination, class, seat availability, success/failure, user device, network latency — from IRCTC historical DB (requires access).
- Real-time signals: current seat availability snapshot, recent queue depth, recent success/failure rates per train/quota.
- User context: user membership level, saved passengers, payment method latency estimates.
- Data availability: historical booking logs and availability snapshots exist internally; if not available, start with 30 days of archive data and collect live telemetry for 2 weeks before model retraining.

## How Output Is Shown to the User
- UI component: `Success Probability` pill displayed on each train row and on the queued/confirmation screen. Example:

- [Train 12645] — Success: 65% — Top alternatives: 12647 (78%), 12650 (72%)

- Clicking the pill expands a small card showing the factors (e.g., `High demand`, `Low available seats in 1A`) and suggested actions.
- Wireframe reference: see `../assets/wireframes/tatkal-success-predictor.txt` for layout and copy.

## Confidence Threshold and Fallback
- Numeric model confidence threshold: only show recommendations when probability >= 55% or when model provides an explanation that meets clarity rules.
- If model confidence is low (<55%) or model unavailable, hide probability and display a deterministic fallback: show recent success trends (last 5 minutes success rate) and a neutral message `High demand — expect longer waits`.

## Success Metrics
- Conversion lift for Tatkal bookings when model is shown: +10% relative
- Reduction in failed booking retries per session: -30%
- Click-through on recommended alternatives: >15%

## Limitations and Risks
- Model bias: historical logs may underrepresent certain routes or user classes, producing skewed recommendations.
- Wrong recommendation risk: users may switch to a higher-probability train which is less convenient; always show alternatives with ETA and explicit trade-offs.
- Operational risk: model access latency during Tatkal peaks — ensure fast on-prem scoring and asynchronous GPT-4 calls for explanations only.
