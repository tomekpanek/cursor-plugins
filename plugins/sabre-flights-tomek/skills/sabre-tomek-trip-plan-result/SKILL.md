---
name: sabre-tomek-trip-plan-result
description: Select a flight option previously returned by Tomek's bundled Sabre Flights MCP, add it to a Trip Plan with add_flight_to_trip_plan, and render the ADDED or UNAVAILABLE result as structured Markdown tables. Use when the traveler chooses an option with phrases such as "dodaj pierwszą", "wybieram opcję 2", or asks to add a displayed flight to the trip plan.
---

# Sabre Tomek Trip Plan result

Use this skill after `air_search` results have been shown and the traveler
selects a specific displayed option. Use the internal option-number-to-offer
mapping retained from the search response. Do not display the underlying offer
identifier to the traveler.

## Before calling the tool

Resolve references such as `pierwsza`, `opcja 2`, or `ten najtańszy` against the
most recent flight table. If the reference is ambiguous, ask the traveler to
choose one option. Do not silently choose.

Build the `add_flight_to_trip_plan` request from the selected
`airOffers.airOffer[]` entry:

- include every journey and schedule in travel order,
- convert local times to `HH:MM`,
- derive each segment's local dates from the leg `departureDate`,
  `departureDateAdjustment`, and `arrival.dateAdjustment`,
- preserve marketing airline, flight number, and booking class,
- use the same traveler counts as the originating search,
- pass an existing `tripPlanId` only when the traveler asked to append to it.

Call `add_flight_to_trip_plan` once. It is not idempotent. Do not repeat it
automatically after an uncertain or partial result.

## Result contract

The tool result contains:

```text
status
tripPlanId
checkoutUrl
nextAction
message
errors[]
warnings[]
offerValidation[]
```

It does not currently return the complete Trip Plan document, selected
itinerary, or authoritative repriced total. Keep the selected itinerary and
price from the preceding `air_search` result for presentation, and label the
price as coming from the search.

Never say that Sabre confirmed or revalidated a specific amount unless that
amount is explicitly present in the current tool result. The absence of a
price-change warning is not proof that a price was confirmed.

## Successful `ADDED` result

Start with a short statement that the selected option was added to a Sabre Trip
Plan. Then render a summary table:

```markdown
| Status | Plan podróży | Wybrana opcja | Cena z wyszukiwania | Walidacja | Ostrzeżenia |
|--------|---------------|----------------|----------------------|------------|-------------|
| Dodano | `LGVWDDBBXF` | 1 — Lufthansa przez Frankfurt | 1209,49 USD | Matched; Same cabin | Brak |
```

Translate labels and values to the traveler's language.

Map fields as follows:

- **Status:** `status`; render `ADDED` as `Dodano` in Polish.
- **Trip Plan:** `tripPlanId`, displayed as code.
- **Selected option:** the option number and concise route/carrier description
  retained from the prior flight table. Do not include the offer UUID.
- **Search price:** the selected offer's
  `pricingInformation.fare.totalFare.totalPrice` and `currency`; append
  `(z wyszukiwania)` in prose when needed.
- **Validation:** distinct
  `offerValidation[].bookingClassCodeValidation` values, preserving their
  returned wording.
- **Warnings:** `Brak` only when `warnings[]` is empty.

Do not assign an `offerValidation` item to a particular flight segment unless
the returned object explicitly contains a segment reference. The current
contract associates validation with an offer, not a segment. Therefore do not
claim, for example, that the first segment was `Matched` and the second was
`Same cabin` based only on the order of validation entries.

After the summary, render the selected itinerary retained from search:

```markdown
| Odcinek | Lot | Wylot | Przylot | Klasa z wyszukiwania |
|---------|-----|-------|----------|-----------------------|
| 1 | LH 1347 | WAW — 6 paź, 09:35 | FRA — 6 paź, 11:25 | K |
| 2 | LH 438 | FRA — 6 paź, 13:10 | DFW — 6 paź, 17:05 | K |
```

Use the same route and date derivation rules as the flight-results skill. This
table describes the selected search offer; it is not a substitute for a full
Trip Plan response.

If `checkoutUrl` is present, put it on its own line:

```markdown
**Dokończ rezerwację:** [Przejdź do Sabre Checkout](https://...)
```

State that opening the checkout link may continue toward payment or booking.
Do not say that a reservation or ticket already exists.

If the URL or server host indicates a test or CERT environment, label it
clearly:

```markdown
> **Środowisko testowe:** Sabre CERT — operacja nie powinna być traktowana jako
> wystawienie prawdziwego biletu.
```

Do not infer a ticketing deadline from the current date, fare basis, or prior
conversation. Show a purchase deadline only when it is explicitly returned by
an MCP response.

## Warnings and errors

Render every item in `warnings[]` below the tables:

```markdown
> **Ostrzeżenie — flight_check:** Price changed during revalidation.
```

Use `source`, `category`, `type`, and `description` when available. Do not hide
a warning behind a general `Brak ostrzeżeń` statement.

If `errors[]` is non-empty despite `ADDED`, show them prominently and avoid
claiming that the workflow completed cleanly.

If the message or warning says the Traveler Gateway workflow status was not
`Completed`, explain that the Trip Plan ID was issued but the write should be
verified. Do not retry the add automatically.

## `UNAVAILABLE` result

Do not show the flight as added. Render:

```markdown
| Status | Wybrana opcja | Następny krok |
|--------|----------------|---------------|
| Niedostępna | 1 — Lufthansa przez Frankfurt | Ponowne wyszukanie ofert |
```

Then show all `errors[]`, `warnings[]`, and validation verdicts. When
`nextAction` is `RETRY_SEARCH`, explain that repeating
`add_flight_to_trip_plan` unchanged will not help. Offer to rerun `air_search`
with the same criteria.

## Checkout boundary

Adding a flight to a Trip Plan is not checkout. After presenting an `ADDED`
result:

- provide the returned checkout link when available,
- do not call `checkout` automatically,
- ask for explicit confirmation before starting an interactive checkout,
- do not request traveler or payment data until checkout actually requires it.
