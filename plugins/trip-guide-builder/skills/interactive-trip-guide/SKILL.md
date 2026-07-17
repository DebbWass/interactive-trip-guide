---
name: interactive-trip-guide
description: >-
  Build a polished, single-file interactive HTML travel guide for a multi-day trip (great for
  families) — from flight tickets, one or more hotels, and a few trip parameters. It extracts
  the flights, researches and verifies real restaurants and attractions, and outputs a
  shareable HTML with a day-by-day itinerary, maps, a multi-currency budget calculator,
  packing checklist, flights tab, live weather, and print-to-PDF; it handles multi-city trips
  with several hotels. Use it whenever someone wants to plan a vacation and turn it into a
  shareable interactive itinerary/guide — especially when they upload flight-ticket PDFs, name
  a hotel or hotels, or ask for a "day-by-day plan", "trip guide", "itinerary", "מדריך טיול",
  or "תוכנית טיול". Trigger it even if they only say "help us plan our trip to X" and provide
  tickets or a hotel, since the interactive guide is the natural deliverable. Do NOT use it for
  one-off questions that aren't a request to build a whole guide — a single restaurant or
  attraction lookup, booking one flight or hotel, a weather or currency check, a standalone
  packing list, translating a menu, or a non-travel HTML dashboard or budget tracker; answer
  those directly instead.
---

# Interactive Trip Guide Builder

## What this produces

A **single self-contained HTML file** the family opens in any browser. It is styled with
Tailwind (CDN), Chart.js (CDN), and a web font. On load it also pulls two things live —
the **weather forecast** and the **currency exchange rates** — and it degrades gracefully to
honest fallback values if there's no network (see `references/live-data.md`). Everything else
works offline once rendered; only the outbound links (Google Maps, official sites) need the
internet at click time.

The finished guide has these tabs:

- **📅 Daily itinerary** — one card per meal (breakfast/lunch/dinner) and per attraction,
  each with rating, address (Google Maps link), opening hours, prices, a family tip, and
  official-site + TripAdvisor links. Each day ends with a visual route timeline and a
  single "open the whole day in Google Maps" directions link.
- **🚇 Public transport** — local ticketing, child-fare rules, must-have apps, tips.
- **💰 Budget** — an interactive calculator (adults/kids/days/transport) with a Chart.js
  doughnut, shown in **multiple currencies** (local / home / USD / EUR) plus a small
  **currency converter**. Rates are fetched **live** (ECB) on load and are still hand-editable.
- **🛍️ Shopping**, **💡 Tips & warnings**.
- **✈️ Flights** — built from the uploaded tickets: flight cards, a seat table per
  passenger, booking reference, baggage, and airport→hotel logistics + timing advisory.
- **🧳 Packing checklist** — interactive, state saved in `localStorage`.

Plus these cross-cutting features: a **live weather widget** in the header, **favorites**
(a star on each card, saved in `localStorage`, with a floating panel), **print-to-PDF**
(a button + `@media print` that unfolds every tab into a clean brochure), **filter tags**
(highlight Kids / Shopping / Outdoors / Rainy-day cards), and **mobile polish** (48px touch
targets, no horizontal scroll, swipe between days).

**The reference example is `assets/example-warsaw-guide.html`.** Study it for the exact
HTML structure, Tailwind classes, tab/day JavaScript, and the currency / checklist /
favorites / filter / swipe / print code. Replicate those patterns and swap in the new
trip's content. Don't reinvent the structure — copy it and re-fill it.

## Inputs you need

1. **Flight ticket PDFs** (one per passenger, or a single itinerary). Source of the
   flights tab and of the arrival/departure timing logic.
2. **Hotel(s) / accommodation** — the trip may have **one base or several** (multi-city, or
   changing hotels mid-trip). Capture each hotel with the **nights it covers**. Whichever
   hotel they sleep at on a given night is that day's anchor: breakfast is a short walk from
   it, and the day starts and ends there. Map hotel → date range and confirm it with the user.
3. **Trip parameters** — gathered via guiding questions (see
   `references/guiding-questions.md`): destination, exact dates, travelers and their ages,
   interests, budget style, transport preference, home currency, language/direction.

If some inputs are missing, ask for them before building. Never invent flights or a hotel.

## Workflow

Follow these steps in order. Each one has a "why" — internalize it rather than doing it
mechanically.

### 1. Gather inputs and ask guiding questions
Read `references/guiding-questions.md` and ask the open questions in one batch. The ages of
the children drive almost every recommendation (a 4-year-old needs playgrounds, free
admission, short transit, play-areas in restaurants), so pin them down. Convert any relative
dates to absolute calendar dates, and work out the **day of week** for each date — it
determines opening days (many museums/palaces close one weekday) and affects the plan.

### 2. Extract the flights from the ticket PDFs
Read each PDF with the Read tool **without the `pages` parameter** (passing `pages` tries to
render an image via `pdftoppm`, which may not be installed; the plain read extracts the text
layer, which is what you want). If the user gives the flight details as **text** rather than
PDFs, just use those directly — the PDF step is only about getting at the same facts. From each ticket collect: passenger name and type
(adult/child), flight numbers, operating carrier, airports + terminals, dates, departure and
arrival times, seat (outbound + return), baggage, and the booking reference (PNR). All
passengers usually share the same flights — read them all to build the per-passenger seat
table, and confirm the flights match.

### 3. Research and verify every place and link
This is what separates a real guide from a plausible-looking one. For each candidate
restaurant/attraction:
- Confirm it is **currently open** (search "<name> <city> open <year>" / local-language
  equivalent). Restaurants close; a live-looking website is not proof. Flag anything
  shaky and offer a backup.
- Get the real address, opening **hours and days**, price, and official site.
- **Verify links actually resolve.** Use `curl -s -o /dev/null -w "%{http_code}" -L -A
  "<browser UA>" <url>` for every external link. Interpret results with judgment:
  `200/202` = fine; `404` = fix the URL (find the correct one); `000` / SSL errors = the
  domain may be dead or parked (cross-check with WebFetch/search, replace if broken);
  `403` on TripAdvisor / some official sites = **bot-blocking, not a broken link** — a human
  clicking it in a browser is fine, so leave those. Google Maps search links
  (`maps.google.com/?q=...`) are always valid.
- **Never fabricate ratings, review counts, prices, or hours.** Use verified values, or
  clearly mark something as approximate. A wrong "4.6 ⭐ (2,200 reviews)" erodes trust.

See `references/build-methodology.md` for the deeper reasoning on itinerary design,
timing, and data accuracy.

### 4. Design a geographically-optimized itinerary
- **Breakfast** within a 1–10 minute walk of the hotel. **Lunch and dinner** in that day's
  touring zone, not a detour. Cluster each day around one area to minimize dragging tired
  kids across town.
- Match attractions to the children's ages, note free admission for young kids, and prefer
  places with a genuine kid angle (playgrounds, science centers, zoos, play-corners in
  restaurants).
- Respect opening days (check the weekday of each date against each venue's closed day).
- **Multiple hotels / multi-city.** Anchor each day to the hotel they sleep at *that* night,
  not one fixed base. On a **changeover day** (check out of one hotel, into the next), plan
  the morning around the first hotel, then build in checkout + luggage transfer + the journey
  to the next hotel/city (with the right transport — intercity train/flight/drive, from the
  user's booked tickets if they gave them), and plan the afternoon/evening around the new
  hotel. If the trip spans cities, give each city its own block of days and its own transport
  and (if relevant) currency notes.
- **Account for the flight days.** The arrival day is not a full day: after landing they go
  to the hotel, drop luggage, and only then tour — so its morning starts late. The
  departure day is short: work back from "be at the airport 3 hours before an international
  flight" to a hotel-departure time, and only plan a relaxed breakfast + packing. Add these
  as clear notes, and if the trip has a dedicated departure day, give it its own short
  "day" card.

### 5. Generate the HTML from the reference example
Open `assets/example-warsaw-guide.html` and reproduce its structure exactly, swapping in the
new content:
- Header (dates, travelers, hotel, a flight summary line, and the weather widget). For a
  multi-hotel trip, show each day's **anchor hotel** in that day's summary, and list all the
  stays with their date ranges somewhere visible (e.g. a small "stays" strip in the header),
  so it's always clear where they're based on any given day.
- The tab nav + `showTab()` / `showDay()` machinery (unchanged).
- One day-container per day, each with meal/attraction cards, a route timeline, and a
  Google-Maps directions link that chains the day's stops (URL-encode place names).
- The transport, budget, shopping, tips, flights, and checklist tabs.
- The JavaScript blocks for the budget calculator, **multi-currency + converter**, the
  **live weather** and **live FX rate** loaders, checklist (localStorage), favorites, filters,
  swipe, and print. These are destination-agnostic — copy them and adjust only the labels,
  the coordinates and trip dates (weather), the currency codes, and the checklist items.
  **Read `references/live-data.md` before wiring the weather or the rates** — it has the exact
  verified endpoints, the required **DOM id contract** the copied JS depends on (drop an id and
  the feature silently no-ops), the reason the weather needs a historical-average fallback
  (forecast APIs only reach ~16 days ahead, and trips are planned months out), and the
  multi-city (segmented) weather pattern for multi-base trips.
- **Watch the destination-specific constants when you copy the JS.** A few values are baked in
  and must be updated per trip, or a feature quietly misbehaves: the swipe handler caps the day
  number (`if (nd > 6)` in the template's `setupSwipe` — set it to *your* day count), the
  weather coordinates/dates, and the currency set (changing the **base** currency is a
  multi-function edit, not a label swap — see `references/live-data.md`). Safest is to copy the
  header and budget markup verbatim from the template and swap only the text, so the ids and
  structure come along unchanged.
- Keep it **RTL** (`dir="rtl"`, Hebrew) or **LTR** to match the user's language.

### 6. Verify before delivering
- Extract the page's `<script>` and run `node --check` on it to catch any syntax error.
- **Hard requirement:** every id the JS references (each `getElementById('…')` and the
  `wx-strip` / `fx-note` / `res-*` / `ccy-btn-*` / `conv-*` / `rate-*` contract in
  `references/live-data.md`) must resolve to **exactly one** element in the HTML. This grep is
  what catches a hand-edited header that dropped a live-data hook — the failure is silent
  otherwise, so don't skip it. Also confirm `<section>` open/close counts and tab/day counts
  balance, and that the swipe day-cap matches the real number of days.
- The in-app browser can't open `file://` URLs, so you usually can't screenshot the result —
  tell the user to open the file in their own browser and spot-check the interactive parts
  (currency toggle, converter, favorites, filters, checklist persistence, print).

### 7. Deliver
Write the finished guide next to the user's trip materials (or where they ask), as one
`.html` file. Summarize what you built and flag anything that needs their decision (a shaky
venue, an approximate rate, a date that lands on a venue's closed day).

## Guardrails
- **Preserve functionality** if editing an existing guide — add features on top, don't break
  the tab/day/calculator machinery.
- **Offline**: by default the guide loads Tailwind/Chart.js/font from CDN, so it needs
  internet the first time it renders. If the user needs true offline use, inline those three
  libraries into the file (this makes it large, ~3 MB, but self-contained).
- **Privacy**: the flights tab contains passenger names, seats, and the booking reference.
  That's fine for the family's private file; if they plan to share it publicly, offer to
  remove or mask the personal details. Include flight numbers/times/seats but not full
  ticket numbers or payment details.
