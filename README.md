# opencode-go-clpx

A [CLIProxyAPI](https://help.router-for.me/plugin/development) plugin that exposes [OpenCode Go](https://opencode.ai) as a single `opencode-go` provider with shared key pooling, multi-protocol translation, and a per-key quota page; a personal fork of [massiveits/opencode-go-cliproxyapi](https://github.com/massiveits/opencode-go-cliproxyapi) with the changes listed below.

## Install

Add this repo as a third-party store source in CLIProxyAPI's `config.yaml`:

```yaml
plugins:
  enabled: true
  store-sources:
    - "https://raw.githubusercontent.com/tympom/opencode-go-cliproxyapi/main/registry.json"
```

Then install from the Management Center's Plugin Store page (or `POST /v0/management/plugin-store/opencode-go-clpx/install`) and restart CLIProxyAPI.

## Configuration

Only `api-keys` is needed. In `config.yaml` under `plugins.configs.opencode-go-clpx`:

```yaml
plugins:
  configs:
    opencode-go-clpx:
      api-keys:
        - value: "sk-..."
          label: "work"              # optional; names the quota card and auth file
        - "${OPENCODE_GO_API_KEY}"   # a bare string or ${ENV_VAR} also works
```

The same list can be entered in the Management Center (Plugins → Edit config), as JSON: `["sk-..."]` or `[{"value": "sk-...", "label": "work"}]`. A fresh store install registers without keys, so the editor works before any key is set; with no keys the plugin serves no models. Duplicate key values are rejected.

Everything else is optional:

| Key | Default | Meaning |
| --- | --- | --- |
| `base-url` | `https://opencode.ai/zen/go/v1` | Upstream base URL |
| `catalog-url` | `{base-url}/models` | Catalog endpoint override |
| `model-prefix.enabled` | `true` | `true` -> `opencode-go/<model>`, `false` -> bare `<model>` |
| `model-prefix.value` | `opencode-go` | Prefix name |
| `catalog.refresh-interval` | `15m` | Discovery cadence, minimum `1m` |
| `catalog.stale-while-unavailable` | `true` | Keep the last good catalog when a refresh fails |
| `protocols.chat-completions`, `.messages`, `.responses` | `true` | Enable models routed to `/v1/chat/completions`, `/v1/messages`, `/v1/responses` |
| `route-overrides` | none | Per-model route, wins over built-in prefix routing |
| `request-timeout` | `5m` | Upstream request timeout |
| `max-response-bytes` | `67108864` (64 MiB) | Max non-streaming response body |
| `allow-http` | `false` | Allow `http://` URLs, for local mocks and tests |

```yaml
      route-overrides:
        "custom-model":
          protocol: "messages"       # chat-completions | messages | responses
          endpoint: "/v1/messages"   # must start with /
```

### Notes

- **Auth files**: each key gets a credential file named after its label (`opencode-go-work.json`); unlabeled keys fall back to a masked key suffix (`opencode-go-key-Xf9a.json`). Renaming a key's label creates a new auth file; remove the old one in Auth Files manually (the host plugin ABI has no delete callback).
- **Quota page**: Management Center → OpenCode Go Quota. Cards are titled with each key's label; reset times show local date/time plus a countdown, e.g. `09/23, 16:35 · in 4d 23h` (same formatting as CPA's native quota UI). Page load never contacts upstream; refresh each card manually or all at once with Refresh All.
- Built and tested against CLIProxyAPI `v8.0.10`.

## Changes vs upstream (`massiveits/opencode-go-cliproxyapi`)

- **Plugin ID renamed** `opencode-go-cliproxyapi` → `opencode-go-clpx` so this fork can coexist with the official store listing instead of colliding on the same name.
- **Per-key labels**: `api-keys[].label` names a key's quota card and its auth file (upstream shows an opaque hash of the key).
- **Readable auth file names**: `opencode-go-<label>.json` instead of upstream's `opencode-go-key-<64-hex>.json`; same-name keys get a 12-hex disambiguator instead of overwriting each other.
- **Native reset-time formatting on the quota page**: `09/23, 16:35 · in 4d 23h` style (ported from CPA's Management Center); upstream prints the raw `resets_at` ISO string.
- **Registers without API keys**, so the config editor works right after a Store install; upstream fails registration until a key is configured.
- **Plain-string API keys**: `api-keys` accepts `["sk-..."]` as well as `{value, label}` objects.
- **CI**: Linux-only release builds (amd64/arm64), no test job; `registry.json` published for the plugin store.

Everything else (provider behavior, protocol translation, catalog discovery, scheduling, model prefixing) is unchanged from upstream (merged through upstream `v0.1.10`, incl. its Codex CLI support); merge `upstream/main` to pick up its fixes.
