# Sabre Flights table

This skills-only plugin uses the Sabre flight tools already provided through
the Cursor dashboard MCP configuration named **Sabre Flights (unstable)**.

It does not install, start, or configure the MCP server. Its skills define how
an agent should:

- turn the raw `air_search` response into a compact Markdown comparison table,
- preserve the selected option and call `add_flight_to_trip_plan`,
- render the resulting Trip Plan status, validation, warnings, ID, and checkout
  link without overstating what Sabre returned.

## Prerequisite

The user or team must have the Sabre Flights MCP enabled and authenticated in
Cursor. The MCP must expose the `air_search` tool.

## Current behavior

- renders up to five relevant offers by default,
- shows total price, route, dates and times, duration, connections, and airlines,
- supports one-way and round-trip results,
- preserves offer identifiers internally without displaying them,
- surfaces warnings and errors,
- never invents absent baggage or fare-rule information,
- renders `ADDED` and `UNAVAILABLE` trip-plan outcomes separately,
- distinguishes a price remembered from search from a price explicitly returned
  by a later MCP response.
