# Live data: weather and exchange rates

The guide fetches two things at runtime instead of hardcoding them: the **weather forecast**
and the **currency exchange rates**. Both use free, no-API-key services that send
`Access-Control-Allow-Origin: *`, which is what makes them work from a page opened as a local
`file://` document. Both were verified working; the finished implementation lives in
`assets/example-warsaw-guide.html` (`loadWeather()` and `loadRates()`) — copy it and change
only the coordinates, dates, and currency codes.

**Design rule for both:** never let a failed network call break the page. Ship sensible
static fallback values in the HTML, overwrite them on success, and always show the user an
honest status label saying which they're looking at. A guide that silently shows stale or
invented numbers is worse than one that says "couldn't load, showing an estimate".

## Required DOM id contract (read this first)

The copied JavaScript depends on specific element ids. If you rewrite the header or the
budget/converter markup by hand and drop one, the feature **silently no-ops** — no error, it
just never fills in. This is the single most common way to break these features. Whatever
markup you write, it MUST contain these ids, and your final verification MUST confirm each
resolves to exactly one element:

- **Weather:** `wx-strip` (the container the day-cards are rendered into) and `wx-note` (the
  status line above it).
- **FX / budget:** `fx-note` (rate status line), the rate inputs `rate-<ccy>` for each
  non-base currency, the result spans `res-food` / `res-attractions` / `res-taxi` /
  `res-shopping` / `res-total` / `res-total-multi`, the currency toggle buttons `ccy-btn-<CCY>`,
  and the converter's `conv-amount` / `conv-from` / `conv-results`.

The safest path is to **copy the header and budget markup from the reference template rather
than hand-writing it**, then just swap the text — that way the ids come along for free.

## Weather — Open-Meteo

Endpoint (forecast):
```
https://api.open-meteo.com/v1/forecast
  ?latitude=<LAT>&longitude=<LON>
  &daily=weather_code,temperature_2m_max,temperature_2m_min
  &timezone=<IANA TZ, url-encoded>
  &start_date=<YYYY-MM-DD>&end_date=<YYYY-MM-DD>
```

**The gotcha that matters:** Open-Meteo only forecasts about **16 days ahead**. Trips are
usually booked months out, so a naive call returns HTTP 400 with
`"Parameter 'start_date' is out of allowed range..."` and the widget would just be broken for
most of the guide's life.

So implement two modes and let it flip automatically:

1. **Live forecast** — try the forecast endpoint for the trip dates. If it returns 200 and the
   `daily.temperature_2m_max` array has non-null values, render it and label it live.
2. **Historical average fallback** — otherwise call the **archive** API for the same calendar
   dates in each of the last 3 years and average them:
   ```
   https://archive-api.open-meteo.com/v1/archive
     ?latitude=<LAT>&longitude=<LON>
     &daily=weather_code,temperature_2m_max,temperature_2m_min
     &timezone=<IANA TZ>
     &start_date=<YYYY-1>-MM-DD&end_date=<YYYY-1>-MM-DD
   ```
   Average max/min per calendar day; for the icon take the most frequent weather code across
   those years. Label it clearly as a historical average. As the trip gets within ~16 days,
   step 1 starts succeeding and the widget upgrades itself with no code change.

This is worth the effort: for Warsaw in mid-August the real historical average came out at
**27–32 °C**, while a plausible-sounding hand-written guess was 23–27 °C. Real data changes
what the family packs.

Map WMO weather codes to emoji (`0` clear ☀️, `1-2` partly 🌤️⛅, `3` overcast ☁️, `45/48` fog
🌫️, `51-57` drizzle 🌦️, `61-67` rain 🌧️, `71-77` snow 🌨️, `80-82` showers 🌦️🌧️⛈️, `95-99`
thunderstorm ⛈️), and default to 🌤️ for anything unmapped.

Also set the static fallback cards in the HTML to the **real historical averages** you
computed, not a guess — that way even a fully offline open shows honest numbers.

### Multiple cities (multi-base trips)

The template ships a single-city loader (one `WX_LAT/WX_LON/WX_TZ`, one `TRIP_START/END`).
A multi-city trip needs weather per city, so restructure `loadWeather()` into **segments**:
one `{lat, lon, tz, startDate, endDate, cityLabel}` per city block, fetch each segment
(live-forecast-then-historical-fallback, exactly as the single-city version does), and
concatenate the day-cards in date order. Tag each card with a small city marker (e.g.
🏛️ Rome / 🌸 Florence) so it's obvious which city a day belongs to, and have the status line
report the mix (live / historical / mixed). This is the same logic as the single-city case,
just mapped over an array of segments — don't invent a new mechanism.

## Exchange rates — Frankfurter (ECB)

Endpoint:
```
https://api.frankfurter.dev/v1/latest?base=<LOCAL_CCY>&symbols=<CCY1>,<CCY2>,<CCY3>
```
Example response for `base=PLN&symbols=ILS,USD,EUR`:
```json
{"amount":1.0,"base":"PLN","date":"2026-07-16","rates":{"EUR":0.23103,"ILS":0.79702,"USD":0.26492}}
```
Free, no key, CORS-enabled, backed by European Central Bank reference rates (updated on
business days). Note the domain: `api.frankfurter.dev` — the old `api.frankfurter.app` now
301-redirects, and a browser `fetch` following that redirect is a needless failure point.

On load: fetch the rates, overwrite the rate constants, **also write them into the editable
rate inputs** so the user sees the real numbers, then re-render the budget and the converter.
On failure, keep the shipped defaults and say so. Keep the manual-override inputs either way —
rates drift and the user may want to model a worse tourist rate than the ECB mid-market one.

Store all budget figures in the **local currency** as the base, and convert on render. That
keeps one source of truth and makes switching currencies a pure display concern.

**Changing the base currency is a multi-function edit, not a label swap.** The template is
built around a PLN base (its `fmtMoney` even special-cases `zł`). Moving to, say, a EUR base
touches: `CCY_RATES` and `CCY_SYMBOL`, the `fmtMoney` base-currency formatting, the currency
toggle set, the converter's target list and the Frankfurter `symbols=` list, and it means
**removing the now-redundant rate input for the base currency** (a `rate-eur` field is
meaningless when the base already is EUR). Trace every place the old base code appears and
update all of them, then verify with `node --check` and the id/count checks.

## Choosing services for other destinations

Open-Meteo and Frankfurter are global — only the coordinates and currency codes change. If a
destination's currency isn't in the ECB set (Frankfurter returns an error for unsupported
symbols), fall back to another keyless, CORS-enabled source such as
`https://open.er-api.com/v6/latest/<CCY>` and keep the same graceful-degradation pattern.

Verify any endpoint with curl before writing code against it — check the status, the body
shape, and that `access-control-allow-origin: *` is present when an `Origin` header is sent:
```bash
curl -s -D- -o /dev/null -H "Origin: null" "<url>" | grep -iE "^HTTP|access-control-allow-origin"
```
