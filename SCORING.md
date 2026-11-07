# EV Rivals — how scores are worked out

The same rules are in `scoring.json` as data. `scoring-tests.json` holds expected results for checking an app.

## 1. Scoring one figure (0–100)

Each figure is scored against the **reference cars**: every car in `cars.json` with `status: "current"`.
If fewer than 4 current cars have that figure, every car in the file is used instead.

1. **Middle** = the median of the reference cars' values.
2. **Step** = the interquartile range ÷ 1.349 (the typical spread).
   If that is 0, use the population standard deviation. If that is also 0, every car scores 50.
3. **Score** = 50 + 50 × tanh(direction × (value − middle) ÷ step ÷ 3), kept between 0 and 100.
   Direction is +1 when higher is better and −1 when lower is better (price, weight, 0–62).

What this means in practice:
- The middle car scores 50. Cars close to typical score close to 50.
- Scores ease towards 0 and 100 rather than stopping dead, so very fast cars stay in order.
- One extreme car doesn't move everyone else's score.

Quantiles use linear interpolation between sorted values: position = (n − 1) × p.

**Efficiency** is not stored. It is range ÷ battery (only when both are known and battery > 0).
**Battery** is not scored on its own, because range and efficiency already cover it.

## 2. Groups

| Group | Figures averaged |
|---|---|
| Eco | Efficiency, price, kerb weight |
| Comfort | Range, boot (seats up), boot (seats down), rapid charging |
| Performance | 0–62, top speed, power, torque |

A group's score is the average of its figure scores that are available.

## 3. Priorities and the overall score

Each group has a priority from 0 to 3 (Off, Low, Medium, High). The default is 1 for all three.
**Overall** = the average of the group scores, weighted by priority.
Groups at 0, or with no score, are left out. If nothing is left, there is no overall score ("–").

## 4. Head to head

In a comparison, a figure only counts if **both** cars have it.
So a car's score on its own can differ from its score in a pair. The tests include a case for this.

## 5. Radar chart (Performance profile)

Six axes, each 0–100, using the same figure scores:
Power · Acceleration (0–62) · Efficiency · Range · Practicality · Weight.
**Practicality** = the average of the two boot scores, or whichever one is known.
A missing axis is drawn at 0 but labelled "no data". It is not a real 0.

## 6. Display

- Scores are kept unrounded and shown rounded to the nearest whole number.
- The head-to-head verdict compares the **rounded** overall scores: equal means "Level", otherwise "<car> leads by N".

## 7. For its class

The Garage and Rankings also show a score **for its class**. Compare always uses the main score above.

1. A car's class is its `classification.segment` (A to F).
2. Each figure is scored exactly as in section 1, but the reference cars are only the **current cars in the same segment**.
3. If fewer than **3** of those have the figure, every car in that segment is used instead (discontinued ones too).
4. If there are still fewer than 3, that figure has no class score. If no figure has one, the car has no class score and the app shows "Not enough cars in this class yet".
5. Groups and priorities then work exactly as in sections 2 and 3.

The minimum is 3 here, not 4, because classes are much smaller than the whole list.

## 8. Not scored

Battery, insurance group, drag coefficient and length are not scored. Insurance group and drag coefficient are shown and ticked in Compare (lower is better). Length is shown only.

## 9. Testing another app (e.g. Swift)

Load `cars.json` and `scoring.json`, run every case in `scoring-tests.json`, and check each value is within 0.001.
It covers 15 single-car scores, 6 head-to-heads, 3 radar charts and 4 edge cases, plus 10 class scores, 3 class rankings and 2 class edge cases.
