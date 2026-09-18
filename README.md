# opencode-go-clpx

A [CLIProxyAPI](https://help.router-for.me/plugin/development) plugin that exposes [OpenCode Go](https://opencode.ai) as a single `opencode-go` provider with shared key pooling, multi-protocol translation, and a per-key quota page — a personal fork of [massiveits/opencode-go-cliproxyapi](https://github.com/massiveits/opencode-go-cliproxyapi) with the changes listed below.

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

In `config.yaml` under `plugins.configs.opencode-go-clpx`:

```yaml
plugins:
  configs:
    opencode-go-clpx:
      # Upstream base URL (default: "https://opencode.ai/zen/go/v1")
      base-url: "https://opencode.ai/zen/go/v1"

      # Optional catalog endpoint override (default: "{base-url}/models")
      # catalog-url: "https://opencode.ai/zen/go/v1/models"

      # Client-facing model ID prefix configuration
      model-prefix:
        enabled: true           # true -> "opencode-go/<model>", false -> bare "<model>" (default: true)
        value: "opencode-go"    # prefix name (default: "opencode-go")

      # OpenCode Go API keys (at least one required). Supports ${ENV_VAR} expansion.
      api-keys:
        - value: "sk-opencode-key-1"
          label: "work"                # optional display name; also names the auth file
        - value: "sk-opencode-key-2"
          label: "personal"
        - value: "${OPENCODE_GO_API_KEY}"

      # Catalog discovery settings
      catalog:
        refresh-interval: "15m"          # discovery refresh cadence, min "1m" (default: "15m")
        stale-while-unavailable: true    # retain last good catalog snapshot on refresh failure (default: true)

      # Protocol enable/disable switches (all default to true)
      protocols:
        chat-completions: true   # enables models routed to /v1/chat/completions
        messages: true           # enables models routed to /v1/messages
        responses: true          # enables models routed to /v1/responses

      # Explicit route overrides per model (takes priority over built-in prefix routing)
      route-overrides:
        "custom-model":
          protocol: "messages"           # "chat-completions" | "messages" | "responses"
          endpoint: "/v1/messages"       # must start with /

      # Execution settings
      request-timeout: "5m"              # upstream request timeout (default: "5m")
      max-response-bytes: 67108864       # max non-streaming response body size in bytes (default: 64 MiB)
      allow-http: false                  # allow http:// scheme for local mock/testing (default: false)
```

All fields are also editable through the Management Center's plugin config UI (registration publishes `ConfigFields`).

### Notes

- **Auth files**: each key gets a credential file named after its label (`opencode-go-work.json`); unlabeled keys fall back to a masked key suffix (`opencode-go-key-Xf9a.json`). Renaming a key's label creates a new auth file — remove the old one in Auth Files manually (the host plugin ABI has no delete callback).
- **Quota page**: Management Center → OpenCode Go Quota. Cards are titled with each key's label; reset times render as `MM-DD HH:mm · in Xd Yh` (same formatting as CPA's native quota UI). Page load never contacts upstream; each card refreshes manually.
- Requires CLIProxyAPI `v7.2.138+` and a CGO-enabled build (Go 1.24+).

## Changes vs upstream (`massiveits/opencode-go-cliproxyapi`)

- **Plugin ID renamed** `opencode-go-cliproxyapi` → `opencode-go-clpx` so this fork can coexist with the official store listing instead of colliding on the same name.
- **Per-key labels**: `api-keys[].label` names a key's quota card and its auth file (upstream shows an opaque hash of the key).
- **Readable auth file names**: `opencode-go-<label>.json` instead of upstream's `opencode-go-key-<64-hex>.json`; same-name keys get a 12-hex disambiguator instead of overwriting each other.
- **Native reset-time formatting on the quota page**: `09/23, 16:35 · in 4d 23h` style (ported from CPA's Management Center), including fractional-seconds tolerance and day/hour/minute precision — upstream prints the raw `resets_at` ISO string.
- **`ConfigFields` in registration metadata**: the plugin's settings render in the Management Center config UI (upstream publishes none).
- **CI**: linux-only release builds (amd64/arm64) on latest GitHub Actions versions; `registry.json` published for the plugin store.
- **Go toolchain**: `go 1.27.1`.

Everything else — provider behavior, protocol translation, catalog discovery, scheduling, model prefixing — is unchanged from upstream; merge `upstream/main` to pick up its fixes.
