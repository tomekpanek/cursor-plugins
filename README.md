# Cursor travel plugins

This repository is structured as a multi-plugin Cursor marketplace. Additional
plugins can be added under `plugins/` and registered in
`.cursor-plugin/marketplace.json`.

## Included plugins

### `sabre-flights-table`

Adds skills that tell Cursor and Grok Bot how to render flight-search,
trip-plan, and traveler-information workflows from the existing
**Sabre Flights (unstable)** MCP server.

The plugin intentionally does not contain:

- an MCP server definition,
- an MCP URL,
- an OAuth Client ID,
- an OAuth Client Secret,
- Sabre credentials.

The Sabre MCP must already be configured and enabled in the Cursor dashboard.
The authenticated MCP connection remains managed by Cursor.

### `sabre-flights-tomek`

Provides the same flight-search, Trip Plan, and traveler-form workflows, but
also installs a remote MCP connection to:

```text
https://grok-air-mcp-tomek-884114690077.us-central1.run.app/mcp
```

OAuth Client ID and Client Secret are configured as protected plugin variables
in Cursor. Their values are not stored in this repository.

## Repository layout

```text
.
├── .cursor-plugin/
│   └── marketplace.json
└── plugins/
    ├── sabre-flights-table/
    │   ├── .cursor-plugin/
    │   │   └── plugin.json
    │   ├── README.md
    │   └── skills/
    │       ├── sabre-flight-results-table/
    │       ├── sabre-trip-plan-result/
    │       └── sabre-traveler-form/
    └── sabre-flights-tomek/
        ├── .cursor-plugin/
        │   └── plugin.json
        ├── mcp.json
        ├── README.md
        └── skills/
            ├── sabre-tomek-flight-results-table/
            ├── sabre-tomek-trip-plan-result/
            └── sabre-tomek-traveler-form/
```

## Adding another plugin

1. Create `plugins/<plugin-name>/`.
2. Add `.cursor-plugin/plugin.json` inside it.
3. Add the plugin to the `plugins` array in
   `.cursor-plugin/marketplace.json`.

Do not commit OAuth secrets or Sabre credentials to this repository.
