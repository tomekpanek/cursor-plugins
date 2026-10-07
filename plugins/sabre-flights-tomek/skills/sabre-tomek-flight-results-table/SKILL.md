---
name: sabre-tomek-flight-results-table
description: Render flight search results from Tomek's bundled Sabre Flights MCP as a concise comparison table. Use whenever air_search returns flight offers or the user asks to search, show, compare, or summarize Sabre flight options.
---

# Sabre Tomek flight results table

Use the `air_search` tool exposed by the `sabre-flights-tomek` MCP bundled with
this plugin. This skill controls presentation only.

## Result source

Read offers from:

```text
airOffers.airOffer[]
```

The response is passed through from Sabre Traveler Gateway and may contain
additional fields. Use only values present in the response. Never infer baggage,
refundability, change conditions, seat availability, or booking status from
their absence.

## Default presentation

Answer in the language used by the traveler. Start with one short sentence that
summarizes the searched route, dates, number of offers found, and lowest total
price when those values are available.

Show at most five relevant offers unless the traveler asks for more. Prefer the
ordering returned by Sabre unless the traveler explicitly requests sorting.

Render the offers as a GitHub-flavored Markdown table:

```markdown
| Opcja | Cena | Trasa | Terminy | Czas | Przesiadki | Linie |
|------:|------|-------|---------|------|------------|-------|
| 1 | 153,40 USD | EWR → LAX | 4 gru, 16:47–19:54 | 6 h 07 min | Bezpośredni | B6 1873 |
```

Translate the column labels to the traveler's language.

## Field mapping

For each entry in `airOffers.airOffer[]`:

- **Offer identifier:** `id`.
- **Total amount:** `pricingInformation.fare.totalFare.totalPrice`.
- **Currency:** `pricingInformation.fare.totalFare.currency`.
- **Directions:** `itinerary.legs[]`.
- **Duration:** `itinerary.legs[].elapsedTime`, expressed in minutes.
- **Segments:** `itinerary.legs[].schedules[]`.
- **Departure:** `scheduleDesc.departure.airport` and
  `scheduleDesc.departure.time`.
- **Arrival:** `scheduleDesc.arrival.airport` and
  `scheduleDesc.arrival.time`.
- **Marketing airline:** `scheduleDesc.carrier.marketing`.
- **Flight number:** `scheduleDesc.carrier.marketingFlightNumber`.
- **Operating airline:** `scheduleDesc.carrier.operating`, when present.
- **Base date for a direction:**
  `itinerary.groupDescription.legDescriptions[].departureDate`.

Format monetary values as `<amount> <currency>`. Preserve the precision returned
by Sabre; do not convert currencies or recalculate the total.

## Routes and dates

Build each direction from its schedules in travel order. Join airport codes with
` → `. For example, two schedules from KRK through WAW to LHR become:

```text
KRK → WAW → LHR
```

For a round trip, put each direction on a separate line inside the same table
cell using `<br>`, for example:

```text
Tam: KRK → WAW → LHR<br>Powrót: LHR → WAW → KRK
```

Treat returned times as local airport times. Keep the supplied UTC offset out of
the compact table unless it is necessary to explain a date boundary.

Derive dates carefully:

1. Start from the matching leg description's `departureDate`.
2. Add `schedules[].departureDateAdjustment` to obtain a segment departure date.
3. Add `scheduleDesc.arrival.dateAdjustment` when obtaining its arrival date.
4. Never assume arrival occurs on the departure date.

Use an unambiguous localized short date. Mark next-day arrival with `+1 dzień`
or the equivalent phrase in the traveler's language.

## Duration and connections

Convert each `elapsedTime` value from minutes to hours and minutes:

```text
367 → 6 h 07 min
```

For round trips, label durations separately:

```text
Tam: 4 h 40 min<br>Powrót: 4 h 15 min
```

For each direction, calculate connections as:

```text
max(number of schedules - 1, 0)
```

Render zero as `Bezpośredni` in Polish or its equivalent in the response
language. Do not treat `scheduleDesc.stopCount` as a connection; if it is
greater than zero, mention it separately as a technical stop only when useful.

## Airlines

Deduplicate repeated flight designators while preserving travel order. Render a
codeshare as the marketing flight first and add the operating carrier only when
it differs:

```text
BA 123 (operowany przez AA)
```

Do not expand an airline code to a full airline name unless that name is present
in the response or otherwise available from a reliable configured tool.

## Offer references

Keep the mapping between the displayed option number and the Sabre offer `id`
internally for follow-up operations such as `add_flight_to_trip_plan`.

Do not display offer identifiers to the traveler. In particular, never append
an `Identyfikatory ofert` section or expose UUIDs after the results table. The
traveler selects offers by their displayed option number, for example
`opcja 1` or `pierwsza`.

Never invent, shorten, or modify an offer identifier when using it internally.

## Missing data and diagnostics

Use `—` for an individual unavailable table value. If a value would require
guessing, omit it or use `—`.

If no entries exist under `airOffers.airOffer`, do not render an empty table.
State that no matching offers were returned and suggest changing only relevant
constraints such as dates, cabin, carriers, direct-flight requirement, or
maximum stops.

Show response warnings below the table as:

```markdown
> **Uwaga:** warning text
```

Show errors before any partial results and clearly distinguish an upstream
system error from a valid search returning no offers.

## Safety

- Do not state that an offer is booked, reserved, held, or ticketed.
- Treat prices and availability as time-sensitive.
- Do not call a booking or trip-plan mutation tool merely to improve rendering.
- Ask for explicit confirmation before any later action that changes a trip
  plan or starts checkout.
