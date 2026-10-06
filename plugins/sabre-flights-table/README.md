# Sabre Flights table

This skills-only plugin uses the Sabre flight tools already provided through
the Cursor dashboard MCP configuration named **Sabre Flights (unstable)**.

It does not install, start, or configure the MCP server. Its first skill defines
how an agent should turn the raw `air_search` response into a compact Markdown
comparison table.

## Prerequisite

The user or team must have the Sabre Flights MCP enabled and authenticated in
Cursor. The MCP must expose the `air_search` tool.

## Current behavior

- renders up to five relevant offers by default,
- shows total price, route, dates and times, duration, connections, and airlines,
- supports one-way and round-trip results,
- preserves offer identifiers for follow-up operations,
- surfaces warnings and errors,
- never invents absent baggage or fare-rule information.
