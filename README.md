# ai-lab-resources

Backend catalog for the **BizlinAI** marketplace. The app fetches
[`catalog.json`](./catalog.json) from this repo at runtime to populate the
Marketplace tabs (Tools, Skills, MCPs, Models & Providers).

## Raw URL

The app's default catalog endpoint is:

```
https://raw.githubusercontent.com/bizlinsolutions/ai-lab-resources/main/catalog.json
```

Users can override it in the app via `MarketplaceService.setCatalogUrl`.

## Files

| File | Purpose |
|---|---|
| `catalog.json` | The live catalog. Edit this to add/remove entries. |
| `catalog.schema.json` | JSON Schema (Draft-07) describing the catalog shape. Use it for validation/CI. |

## Schema overview

The catalog has 5 top-level arrays. Full field definitions are in
`catalog.schema.json`; a quick summary:

### `tools[]` — tool bundles backed by an MCP server
- `id`, `name`, `description`, `category`, `provider`
- `mcpServer` — an MCP server config (see below); installed when the user clicks "Add"
- `icon` — Material icon name (e.g. `search`, `code`)
- `popularity` — optional int for ranking

### `skills[]` — prompt-based agent capabilities
- `id`, `name`, `description`, `category`, `author`
- `promptTemplate` — the system prompt injected when the skill is active
- `requiredTools` — list of tool ids the skill expects (informational)
- `icon`

### `mcps[]` — MCP server presets
- `id`, `name`, `description`, `category`
- `server` — an MCP server config (see below)
- `icon`, `popularity`

### `providers[]` — OpenAI-compatible API providers
- `id`, `name`, `description`, `category`
- `baseUrl`, `defaultModel`, `website`
- `requiresApiKey` — if `false`, the app creates the config with no key (e.g. Ollama)
- `icon`

### `models[]` — on-device (edge) models
- `id`, `displayName`, `description`
- `url` — direct download URL (HuggingFace `resolve/main/...`)
- `filename`, `sizeLabel`
- `modelType` — one of `general`, `gemmaIt`, `gemma4`, `deepSeek`, `qwen`, `qwen3`, `llama`, `hammer`, `functionGemma`, `phi`
- `fileType` — one of `task`, `binary`, `litertlm`, `builtIn`
- `needsAuth`, `supportImage`, `maxTokens`, `icon`

### `category` enum
`productivity` | `developer` | `data` | `research` | `media` | `utilities` | `communication` | `other`

### MCP server config (shared by `tools` and `mcps`)
- `name`, `transport` (`http` | `sse` | `stdio`)
- For `http`/`sse`: `url`, `headers`
- For `stdio`: `command`, `args`, `env`
- `isEnabled` (default `true`)

## Editing the catalog

1. Edit `catalog.json` (keep it valid JSON).
2. Optionally validate against the schema:
   ```bash
   npx ajv validate -s catalog.schema.json -d catalog.json
   ```
3. Commit to `main`. The app picks up changes on next refresh — no app release needed.

## Notes

- The app ships with **bundled fallback defaults** that mirror this catalog,
  so the marketplace is never empty even when offline or before this repo is
  reachable. Keep `catalog.json` in sync with the bundled defaults in
  `ai_lab_app/lib/services/marketplace_service.dart` to avoid drift.
- Gated models (Gemma) require the user to set a HuggingFace token in the app
  before download succeeds.
