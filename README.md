# opencode-go-clpx

A [CLIProxyAPI](https://help.router-for.me/plugin/development) plugin exposing [OpenCode Go](https://opencode.ai) as `opencode-go`, with shared key pooling, protocol translation and per-key quotas. Fork of [massiveits/opencode-go-cliproxyapi](https://github.com/massiveits/opencode-go-cliproxyapi), using the distinct plugin ID `opencode-go-clpx` so both store listings can coexist. Includes upstream `v0.2.0` with native CLIProxyAPI quotas.

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

Only `api-keys` is needed. It goes in CLIProxyAPI's `config.yaml` under `plugins.configs.opencode-go-clpx`.

### The one-key setup (copy this)

Use your raw OpenCode Go key (not a proxy client key; no `Bearer ` prefix):

```yaml
plugins:
  configs:
    opencode-go-clpx:
      api-keys:
        - value: "sk-..."     # your OpenCode Go API key, exactly as shown at opencode.ai
          name: "Work"        # optional account name
```

`api-keys` is always a list. Optional `name` labels the account and names its new auth file (`Work.json`); without it, one key shows as **OpenCode Go** and several as **OpenCode Go 1**, **OpenCode Go 2**, ... in configuration order.

Alternatively, in Management Center (Plugins -> Edit config):

```json
[{"value": "sk-...", "name": "Work"}]
```

### More examples

Add one item per key. Unnamed shorthand accepts bare strings and environment variables:

```yaml
      api-keys:
        - "sk-..."
        - "${OPENCODE_GO_API_KEY}"
```

A fresh store install registers without keys, so the editor works before any key is set; with no keys the plugin serves no models. Duplicate key values are rejected.

### Optional settings

| Key | Default | Meaning |
| --- | --- | --- |
| `base-url` | `https://opencode.ai/inference/go/v1` | Upstream base URL |
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

- **Auth files**: existing files keep their names and IDs; a changed `name` updates the label, not the file name. Same-name new files get a `-2`, `-3`, ... suffix.
- **Quotas**: rolling, weekly and monthly windows use CLIProxyAPI's generic quota API (`GET /v0/management/quota/providers`, `POST /v0/management/quota/fetch`) for the selected credential; the separate quota page is gone and the Management Center build must support generic quotas. Quota reset is unsupported.
- Built and tested against CLIProxyAPI `v8.0.38` with Go `1.27.2`.
