# EV Rivals — shared car data

This repository holds the car list for **EV Rivals**.

- `cars.json` — every built-in car and its figures.
- `scoring.json` — the scoring rules as data, so every app scores the same way.
- `scoring-tests.json` — expected results for checking an app's scoring.
- `SCORING.md` — the scoring rules in plain English.

The user app only **reads** these files. The admin app (Nick only) is the only thing that **writes** `cars.json`.
Users' own choices (owned cars, priorities, appearance) are **never** stored here. They stay on each user's device.

## Top of `cars.json`

| Field | Meaning |
|---|---|
| `schemaVersion` | Format version. Bump it only if the layout changes, so older apps can tell. |
| `updated` | Date the file last changed (YYYY-MM-DD). |
| `app`, `market`, `currency` | `EV Rivals`, `UK`, `GBP`. |
| `specDefinitions` | Every figure the app shows: key, label, unit, and whether `higher` or `lower` is better. Apps read labels from here, so a new figure can be added without rewriting the app. |
| `derived` | Figures the app works out itself (efficiency = range ÷ battery). Not stored per car. |
| `infoDefinitions` | Figures that are shown but never compared, ticked or scored. Currently just `length` (mm). |
| `segments` | The segment letters a car can be in, with a short label (A City car … F Luxury). |
| `bodyTypes` | The body types a car can have: `hatchback`, `saloon`, `estate`, `suv`, `mpv`, `coupe`, `convertible`. |
| `cars` | The list of cars. |

## Each car

| Field | Meaning | Example |
|---|---|---|
| `id` | **Permanent** ID. Never change or reuse it: users' Owned ticks are saved against it. Lowercase letters, numbers, dashes. | `m3-26std` |
| `make` | Brand (first dropdown). | `Tesla` |
| `model` | Model (second dropdown). | `Model 3` |
| `version` | Trim or version (third dropdown). | `Standard RWD` |
| `modelYear` | Year shown as ’26. | `2026` |
| `onSale` | Years it was sold new. `to: null` means still on sale. | `{ "from": 2019, "to": 2020 }` |
| `status` | `current` or `discontinued`. Only `current` cars set the scoring scale. | `current` |
| `classification` | Segment and body type (see below). | |
| `priceBasis` | `current` = today's UK price (re-checked). `when-new` = historic price, never re-checked. | `when-new` |
| `source` | Where the figures came from. | `EV Database #3403, Carwow, Nov 2026` |
| `checked` | When the figures were last checked. | `2026-11-01` |
| `specs` | One entry per figure (see below). | |

## Segment and body type (`classification`)

```json
"classification": { "segment": "B", "bodyType": "suv", "status": "unconfirmed", "note": "4.30 m long: sources split between B-SUV and C-SUV" }
```

| Field | Meaning |
|---|---|
| `segment` | `A` to `F`. Cars in the same segment are scored against each other for the "for its class" score. |
| `bodyType` | One of `bodyTypes`. |
| `status` | `confirmed` if a manufacturer or most sources agree, `unconfirmed` if sources disagree. |
| `note` | Optional. Why a borderline car was put in this segment. |

Segments are a convention, not an official rating, so a borderline car is a judgement call. Length alone can't decide it: SUVs are shorter than hatchbacks of the same class.

## Each figure (inside `specs`)

```json
"torque": { "value": 420, "status": "unconfirmed", "confirmation": null }
```

| Field | Meaning |
|---|---|
| `value` | The number, or `null` if not known. |
| `status` | `confirmed`, `unconfirmed` (shown with **?**), or `unknown` (no figure). |
| `confirmation` | Filled in when Nick confirms a **?** figure in the admin app, e.g. `{ "on": "2026-11-03", "source": "Tesla UK", "by": "Nick" }`. Otherwise `null`. |

## Rules

1. Never change or reuse an `id`. To replace a car, add a new one and mark the old one `discontinued`.
2. Prefer marking a car `discontinued` over deleting it, because someone may own it.
3. Keep units as listed in `specDefinitions` and `infoDefinitions` (miles, kWh, kW, hp, Nm, seconds, mph, litres, kg, pounds, insurance group 1–50, drag coefficient, mm).
4. When a figure is confirmed, set `status` to `confirmed` **and** fill in `confirmation`.
5. Update `updated` whenever the file changes.
