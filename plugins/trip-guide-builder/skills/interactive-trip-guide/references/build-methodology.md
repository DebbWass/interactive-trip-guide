# Build methodology — the reasoning behind a good guide

This is the "why" behind the workflow in SKILL.md. Read it when building so the guide is
genuinely useful, not just plausible-looking.

## Itinerary design

**Anchor every day to the hotel.** Breakfast is a 1–10 minute walk away so mornings start
without transit. Lunch and dinner sit inside the day's touring zone. Each day clusters around
one district — the enemy is backtracking across the city with tired children.

**Design for the specific ages.** For ~4-year-olds: playgrounds, zoos, science centers with a
dedicated little-kids' zone, parks where they can run or feed animals, and restaurants with a
play-corner/play-room so adults can actually eat. Note free admission (many attractions are
free under a certain age). Keep walking legs short and always give a taxi/rideshare fallback.
For older kids, lean into interactive museums, viewpoints, and more walking. Re-derive this
from the ages the user gives — don't assume.

**Respect opening days and hours.** Compute the weekday of each date. Many palaces/museums
close one weekday (e.g. a palace closed Tuesdays); make sure the day it's scheduled isn't its
closed day. Put real opening hours on every card — and check that a breakfast slot isn't
before the place opens (a café that opens at 10:00 can't serve an 08:30 breakfast; use an
earlier-opening backup).

**Give a backup for anything shaky.** If a venue is financially wobbly, newly opened, or you
couldn't fully confirm it's open, add a nearby verified alternative right in the card so a
closed door doesn't derail the day.

## Flight-day realities

The tickets change the shape of the first and last days — this is one of the most valuable
things the guide gets right:

- **Arrival day.** They land, collect bags, travel to the hotel, and drop luggage *before*
  touring. So the first day's real start is often 2+ hours after landing. Shift the morning
  later (a landing at 08:10 → touring from ~10:30, breakfast becomes a late brunch at the
  hotel). Say this explicitly.
- **Departure day.** Work backwards: be at the airport ~3 hours before an international
  flight → subtract the airport transfer (train/taxi ~20–40 min) and a buffer → that's the
  hotel-departure time. Whatever's left is a short, relaxed morning (breakfast near the
  hotel + packing + checkout), not a full touring day. If the trip has a standalone
  departure day, give it its own short "day" card with a departure timeline.

Add a per-day date + weekday label so nobody miscounts which calendar day maps to which
itinerary day.

## Data accuracy (non-negotiable)

- **Verify, don't invent.** Ratings, review counts, prices, hours, and addresses must come
  from a real source (search / official site) or be marked approximate. A confident-looking
  fabricated number is worse than an honest "≈".
- **Check every link with curl.** `200/202` fine; `404` → find and use the correct URL;
  `000`/SSL cert mismatch → likely dead or parked, cross-check and replace; `403` on
  TripAdvisor and some official sites is bot-blocking (the link works for humans — keep it).
- **Weather and FX are fetched live, not hardcoded** — see `references/live-data.md` for the
  verified endpoints and the fallback logic. Hand-written weather guesses are routinely wrong
  (a plausible "23–27 °C" for Warsaw in August was really 27–32 °C), and stale rates quietly
  mislead the budget. Keep editable rate inputs anyway, and always label what's shown (live vs.
  historical average vs. offline fallback) so nobody trusts a number more than it deserves.

## HTML / build notes

- The reference example (`assets/example-warsaw-guide.html`) already contains every pattern:
  the tab and day switching (`showTab`/`showDay`), the budget calculator + Chart.js doughnut,
  the multi-currency logic (base values stored in the local currency, converted on render),
  the currency converter with editable rates, the packing checklist with `localStorage`, the
  favorites system (a star injected into each card's `<h4>`, saved to `localStorage`, with a
  floating panel), the keyword-based filter highlighter, the touch-swipe between days, and
  the `@media print` brochure mode. Copy these; only the content, labels, currency codes, and
  checklist items change per trip.
- **Cards get favorite stars via JS injection** (the script walks `#itinerary .card` and
  prepends a star to each `<h4>`). So you don't hand-add a star to every card — just keep the
  `.card` + `<h4>` structure and the script handles it. Same idea for filters (keyword match
  on card text).
- **Verify the JS** by extracting the `<script>` block and running `node --check` on it, then
  grep that referenced DOM ids exist exactly once and that `<section>`/tab/day counts balance.
  The in-app browser blocks `file://`, so ask the user to open the file to eyeball it.
- Keep the file **self-contained and single-file**. Default to CDN for Tailwind/Chart.js/font
  (needs internet on first render); inline them only if the user explicitly needs offline use.
