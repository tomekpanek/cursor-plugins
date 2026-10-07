---
name: sabre-tomek-traveler-form
description: Collect traveler details for Tomek's bundled Sabre Flights `checkout` tool by rendering a structured Markdown form, validating the answers against Sabre's field rules, and filling the returned `travelersInformation` template. Use only when the current `checkout` response contains a `travelersInformation` requirement, or the traveler asks what data checkout needs.
---

# Sabre Tomek traveler form

Use this skill during the Sabre Flights `checkout` workflow, at the step where
the latest response's `requirements[]` contains an entry with
`type: "travelersInformation"`. This skill covers presentation and validation
only. It does not start checkout, confirm a trip proposal, or handle payment.

## When to show the form

Show the form only when the latest `checkout` response actually contains a
`travelersInformation` requirement. Do not ask for traveler details earlier,
for example right after `add_flight_to_trip_plan` or during the `confirmation`
step.

Do not answer any other requirement type together with
`travelersInformation`. Answer only the types present in the response you just
received.

If the traveler only asks what data will be needed, you may show the empty form
as a preview, clearly labelled as a preview, without calling `checkout`.

## Reading the template

The requirement's `content` is a template:

- `primaryTraveler` has `travelerId` and `type` filled in and every other field
  `null`,
- `additionalTravelers[]` holds one template per extra traveler, with the same
  shape.

Render one form section per traveler, in this order: the primary traveler, then
the additional travelers in array order. Label each section with its
`travelerId` and `type.value` (for example `Pasażer 1 — Adult`). Never add or
remove travelers, and never change `travelerId` or `type`.

## Form layout

Answer in the traveler's language. Start with one short sentence saying that
Sabre needs traveler details to continue, and how many travelers the form
covers. Then render each traveler as a Markdown table:

```markdown
**Pasażer 1 — Adult (główny pasażer)**

| # | Pole | Wymagane | Format / przykład | Wartość |
|--:|------|----------|-------------------|---------|
| 1 | Imię | Tak | litery łacińskie, bez polskich znaków, np. `Tomasz` | |
| 2 | Drugie imię | Nie | np. `Jan`, można pominąć | |
| 3 | Nazwisko | Tak | litery łacińskie, np. `Kowalski` | |
| 4 | Data urodzenia | Tak | `RRRR-MM-DD`, np. `1985-10-30` | |
| 5 | Płeć | Tak | `Mężczyzna`, `Kobieta` lub `Nie podano` | |
| 6 | Kraj zamieszkania | Tak | kod 2-literowy, np. `PL` | |
| 7 | E-mail | Tak | np. `jan.kowalski@example.com` | |
| 8 | Telefon | Tak | z numerem kierunkowym, np. `+48 600 123 456` | |
| 9 | Paszport | Nie | numer, kraj wydania, data ważności | |
| 10 | Known Traveler Number | Nie | do 15 znaków alfanumerycznych | |
| 11 | Redress Number | Nie | do 15 znaków alfanumerycznych | |
```

Leave the `Wartość` column empty. For an additional traveler, mark e-mail and
phone as optional (`Nie`). They are required only on the primary traveler.

After the table, tell the traveler how to answer, for example:

```markdown
Odpowiedz w jednej wiadomości, w formacie `numer: wartość`, np.:
`1: Tomasz`, `3: Kowalski`, `4: 1985-10-30` …
Pola opcjonalne możesz pominąć.
```

If the client supports option buttons, you may ask for gender as a separate
choice instead of free text. Ask one question at a time and do not use buttons
for free-text fields.

Do not pre-fill values from memory, earlier conversations, or account
profiles. You may suggest a value you already know and ask the traveler to
confirm it, but never send an unconfirmed value.

## Validation rules

Validate every answer before calling `checkout`. When a value fails, show the
field, the problem, and an example of a correct value, and ask only for the
failing fields.

| Field | Template key | Rule |
|-------|--------------|------|
| First name | `givenName` | 1–30 characters, `^[a-zA-Z ]{1,30}$` |
| Middle name | `middleName` | Optional; may be an empty string |
| Surname | `surname` | 2–30 characters, `^[a-zA-Z ]{2,30}$` |
| Date of birth | `dateOfBirth` | Real date in `YYYY-MM-DD`, not in the future |
| Gender | `gender` | One of `Male`, `Female`, `Infant Male`, `Infant Female`, `Unspecified` |
| Country of residence | `countryOfResidenceCode` | ISO 3166-1 alpha-2, upper case, e.g. `PL`; `XK` for Kosovo |
| E-mail | `emailAddress` | Simple address such as `name@domain.tld`; required on the primary traveler |
| Phone | `phone.number` | `^\+?[0-9][0-9 ]{0,73}$`, with the international calling code; required on the primary traveler |
| Known Traveler Number | `knownTravelerNumber` | Optional, `^[A-Za-z0-9]{0,15}$` |
| Redress number | `redressNumber` | Optional, `^[A-Za-z0-9]{0,15}$` |

### Names and diacritics

Sabre accepts only Latin letters and spaces in names. If a name contains
diacritics, propose a transliteration and ask the traveler to confirm it before
sending. Never transliterate silently. Common Polish mappings:

```text
ą→a  ć→c  ę→e  ł→l  ń→n  ó→o  ś→s  ź→z  ż→z
Ą→A  Ć→C  Ę→E  Ł→L  Ń→N  Ó→O  Ś→S  Ź→Z  Ż→Z
```

Hyphens and apostrophes are not accepted by the pattern. Suggest replacing a
hyphen with a space (for example `Nowak-Kowalska` → `Nowak Kowalska`) and ask
the traveler to confirm. The name should match the travel document as closely
as these rules allow.

### Gender mapping

Map the traveler's wording to Sabre values:

- `Mężczyzna` / `M` → `Male`
- `Kobieta` / `K` → `Female`
- `Nie podano` / `Inna` → `Unspecified`
- for a traveler of type `INF`, use `Infant Male` or `Infant Female`

### Dates and age

Accept common local formats (for example `30.10.1985`) and convert them to
`YYYY-MM-DD`. Show the converted value in the confirmation summary. If the age
on the first departure date clearly contradicts the traveler type (for example
an `ADT` traveler under 12, or an `INF` traveler aged 2 or older), point it out
and ask before continuing. Do not change the traveler type yourself.

### Passport (optional)

Collect passport details only if the traveler wants to provide them. When
provided, send all of:

| Field | Key | Rule |
|-------|-----|------|
| First name | `passportDetails.givenName` | As in the passport |
| Middle name | `passportDetails.middleName` | Optional |
| Surname | `passportDetails.surname` | As in the passport |
| Date of birth | `passportDetails.dateOfBirth` | `YYYY-MM-DD` |
| Gender | `passportDetails.gender` | `Female`, `Male`, `Undisclosed`, `Unspecified` |
| Nationality | `passportDetails.nationalityCountryCode` | ISO 3166-1 **alpha-3**, e.g. `POL` |
| Issuing country | `passportDetails.issuingCountryCode` | ISO 3166-1 **alpha-3**, e.g. `POL` |
| Number | `passportDetails.number` | As printed |
| Expiration | `passportDetails.expirationDate` | `YYYY-MM-DD`, after the last travel date |

Note the difference: country of residence uses alpha-2 (`PL`), passport
countries use alpha-3 (`POL`). If the passport expires before the last
departure or arrival date in the template, warn the traveler.

## Confirmation before sending

Before calling `checkout`, show a read-back table of the normalized values
exactly as they will be sent:

```markdown
| Pole | Wartość do wysłania |
|------|---------------------|
| Imię | Tomasz |
| Nazwisko | Kowalski |
| Data urodzenia | 1985-10-30 |
| Płeć | Male |
| Kraj zamieszkania | PL |
| E-mail | tomasz@example.com |
| Telefon | +48 600 123 456 |
| Paszport | — |
```

In the read-back, show only the last 3 characters of a passport number (for
example `•••••456`). Ask for an explicit confirmation such as `tak` or
`wysyłaj`. Any correction means a new read-back.

## Sending the answer

Fill the `null` fields of the template you received. Do not build the object
from scratch. Keep `travelerId`, `type`, `nameNumber`, and any trip date fields
(`firstDepartureDate`, `lastDepartureDate`, `lastArrivalDate`) exactly as
returned. Leave optional fields the traveler skipped as `null`. Send
`middleName` as `null` or `""` when absent.

```json
{
  "tripplanCheckoutId": "<id from the current workflow>",
  "requirements": [
    {
      "type": "travelersInformation",
      "content": {
        "primaryTraveler": {
          "travelerId": 1,
          "type": {"code": "ADT", "value": "Adult"},
          "givenName": "Tomasz",
          "middleName": null,
          "surname": "Kowalski",
          "dateOfBirth": "1985-10-30",
          "gender": "Male",
          "countryOfResidenceCode": "PL",
          "emailAddress": "tomasz@example.com",
          "phone": {"number": "+48 600 123 456"},
          "passportDetails": null,
          "knownTravelerNumber": null,
          "redressNumber": null
        },
        "additionalTravelers": []
      }
    }
  ]
}
```

Call `checkout` once with this answer. Do not repeat the call automatically if
the result is uncertain.

## After sending

- Surface every item in `warnings[]` from the response.
- If `errors[]` is non-empty and `requirements[]` asks for
  `travelersInformation` again, show the errors next to the affected fields and
  ask only for those fields.
- If `errors[]` is non-empty and there are no further `requirements[]`, the
  checkout cannot be resumed. Report the failure and do not retry.
- Otherwise continue with whatever the new `requirements[]` asks for. Do not
  assume `ancillariesData` or `payment` comes next.

## Privacy

- Do not save traveler details to memory, notes, or files.
- Do not repeat a full passport number, Known Traveler Number, or redress
  number in chat after it has been sent.
- Never ask for card or payment details in chat. Payment happens only through
  the `paymentUrl` returned by `checkout`.
