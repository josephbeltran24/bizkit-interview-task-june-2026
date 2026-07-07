# Notes

## Fixes

### 1. Double-booking bug
`dates_overlap` in [app.py](app.py) used `start_b <= start_a <= end_b`, which only checks whether the
*new* booking's start date falls inside the *existing* booking's range. It missed cases where the new
booking starts before the existing one but the ranges still overlap. Fixed to the standard interval-overlap
check: `start_a <= end_b and start_b <= end_a`, true whenever the two ranges share any day, regardless of
which one starts first.

**Failure example:** existing booking 2026-07-10 to 2026-07-20. The original code wrongly allowed a new
booking from 2026-07-05 to 2026-07-25 (new range starts before the existing one, so `start_b <= start_a`
was false) — the fix now correctly blocks it as a conflict.

### 2. PHP 0 booking bug
`rental_days` computed `(to_date - from_date).days`, which counts only the *nights* between two dates, not
the billable days. Per the stated business rule ("both start and end day count"), a same-day rental
(from == to) should bill 1 day, but the original formula returned `0` — hence "PHP 0" totals. Fixed by
adding `+ 1` so a same-day booking correctly bills as 1 day and any range is billed inclusively.

### 3. Maintenance equipment rule
Equipment with `status: "maintenance"` (e.g. the HD Projector) had no special handling — it showed up as
bookable in `/api/equipment` and `/api/availability`, and `create_booking` would happily book it. Added a
guard in all three: `list_equipment` and `availability` now filter out maintenance items, and
`create_booking` returns a `409` error if someone tries to book one directly by ID.

### 4. Frontend price bug
On [index.html](index.html), the `change` event listener was only wired up for the `from` date input, so
`updateTotal()` never ran when the `to` date changed — the displayed total silently went stale. Fixed by
adding the same listener to the `to` input so both date fields trigger a recalculation.

## AI use

Used Claude Code to locate the relevant code (grepping/reading `app.py` and `index.html`) and to draft the
fixes. Checked correctness by reading each diff line-by-line against the actual bug description, reasoning
through concrete date/day-count examples by hand (as above) rather than trusting the diff at face value, and
manually re-tracing the frontend event-listener fix against the DOM/JS to confirm `updateTotal()` now fires
on both date inputs.
