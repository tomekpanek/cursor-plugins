# Cursor travel plugins

This repository is structured as a multi-plugin Cursor marketplace. Additional
plugins can be added under `plugins/` and registered in
`.cursor-plugin/marketplace.json`.

## Included plugin

### `sabre-flights-table`

Adds skills that tell Cursor and Grok Bot how to render flight-search and
trip-plan results from the existing **Sabre Flights (unstable)** MCP server as
Markdown tables.

The plugin intentionally does not contain:

- an MCP server definition,
- an MCP URL,
- an OAuth Client ID,
- an OAuth Client Secret,
- Sabre credentials.

The Sabre MCP must already be configured and enabled in the Cursor dashboard.
The authenticated MCP connection remains managed by Cursor.

## Repository layout

```text
.
├── .cursor-plugin/
│   └── marketplace.json
└── plugins/
    └── sabre-flights-table/
        ├── .cursor-plugin/
        │   └── plugin.json
        ├── README.md
        └── skills/
            ├── sabre-flight-results-table/
            │   └── SKILL.md
            └── sabre-trip-plan-result/
                └── SKILL.md
```

## Adding another plugin

1. Create `plugins/<plugin-name>/`.
2. Add `.cursor-plugin/plugin.json` inside it.
3. Add the plugin to the `plugins` array in
   `.cursor-plugin/marketplace.json`.

Do not commit OAuth secrets or Sabre credentials to this repository.
