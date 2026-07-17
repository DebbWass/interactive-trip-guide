# trip-guide-builder

A Claude Code plugin that bundles the **interactive-trip-guide** skill: give it your flight
tickets, your hotel(s), and a few trip details, and it builds a single-file interactive HTML
travel guide (day-by-day itinerary, maps, multi-currency budget, packing checklist, flights
tab, live weather, print-to-PDF). Handles multi-city trips with several hotels.

## Install

```
/plugin marketplace add DebbWass/interactive-trip-guide
/plugin install trip-guide-builder@interactive-trip-guide
```

Then `/reload-plugins` (or a new session). The skill triggers automatically when you describe
a trip and attach tickets — see
[`skills/interactive-trip-guide/HOW-TO-USE.md`](skills/interactive-trip-guide/HOW-TO-USE.md)
for a full walkthrough and an example prompt.

Licensed under MIT (see the repository `LICENSE`).
