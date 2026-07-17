# Family Travel — Claude Code plugin marketplace

A [Claude Code](https://code.claude.com) plugin that turns a trip into a **polished,
single-file interactive HTML travel guide** — from your flight tickets, hotel(s), and a few
details about the trip.

The generated guide includes a day-by-day itinerary (with verified restaurants and
attractions, maps, hours and prices), a public-transport tab, a multi-currency budget
calculator with a live currency converter, a shopping tab, tips, a flights tab built from your
tickets, and an interactive packing checklist — plus a **live weather** widget, favorites,
filter tags, mobile swipe, and print-to-PDF. It handles multi-city trips with several hotels.

## What's inside

```
.
├── .claude-plugin/marketplace.json     # marketplace catalog
├── plugins/
│   └── trip-guide-builder/             # the plugin
│       ├── .claude-plugin/plugin.json
│       └── skills/interactive-trip-guide/   # the skill (SKILL.md + references + example)
├── LICENSE                             # MIT
└── README.md
```

## Install

In Claude Code:

```
/plugin marketplace add DebbWass/interactive-trip-guide
/plugin install trip-guide-builder@interactive-trip-guide
```

After installing, run `/reload-plugins` — or start a new session — and the skill loads.

To try it without installing, from a clone of this repo:

```
claude --plugin-dir ./plugins/trip-guide-builder
```

## Use

Once installed, just describe your trip and attach your flight tickets — the skill triggers on
its own. For a full walkthrough and an **example prompt that gets the best results**, see
[`plugins/trip-guide-builder/skills/interactive-trip-guide/HOW-TO-USE.md`](plugins/trip-guide-builder/skills/interactive-trip-guide/HOW-TO-USE.md).

Example:

> טסים לרומא באפריל עם הילדים (בני 4 ו-7). מצרפת את כרטיסי הטיסה ואת המלון — תבנה לנו מדריך טיול אינטראקטיבי.

## Requirements

- Claude Code (the skill runs there, not in the claude.ai chat).
- Internet access while building (the skill researches and verifies real places and links).
- The finished guide fetches weather and exchange rates live; everything else works offline
  once the file has loaded.

## Notes

- **Privacy:** the flights tab a guide produces contains passenger names, seats, and a booking
  reference — great for a family's private file. If you share a generated guide publicly, ask
  for the personal details to be removed. The example bundled with the skill uses fictional
  data.
- **License:** MIT — see [LICENSE](LICENSE). © 2026 Dorit Wasserman (DebbWass).
