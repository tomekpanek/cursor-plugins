# Sabre Flights Tomek

This plugin installs Tomek's remote Sabre Flights MCP together with skills for:

- rendering `air_search` results as concise Markdown tables,
- selecting an offer and rendering `add_flight_to_trip_plan` results,
- collecting and validating traveler information during `checkout`.

## MCP endpoint

```text
https://grok-air-mcp-tomek-884114690077.us-central1.run.app/mcp
```

Cloud Run is publicly reachable, while the MCP endpoint itself requires a
Sabre Okta Bearer token.

## OAuth configuration

Configure these protected plugin variables in Cursor:

```text
SABRE_MCP_CLIENT_ID
SABRE_MCP_CLIENT_SECRET
```

Do not commit their values to GitHub. After installation, the user still needs
to complete the interactive Okta login.

## Coexistence with the previous plugin

The skills use unique `sabre-tomek-*` names, so this plugin can be installed
alongside `sabre-flights-table` for testing. To avoid duplicate MCP tools and
competing skill instructions during normal use:

1. enable `sabre-flights-tomek`,
2. disable `sabre-flights-table`,
3. disable the standalone **Sabre Flights (unstable)** MCP.
