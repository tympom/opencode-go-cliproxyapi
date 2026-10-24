# opencode-go-clpx

A [CLIProxyAPI](https://help.router-for.me/plugin/development) plugin exposing [OpenCode Go](https://opencode.ai) as `opencode-go`, with key pooling, protocol translation and per-key quotas. Fork of [massiveits/opencode-go-cliproxyapi](https://github.com/massiveits/opencode-go-cliproxyapi) (upstream `v0.2.0`) under its own plugin ID, so both can be installed.

## Install

```yaml
plugins:
  enabled: true
  store-sources:
    - "https://raw.githubusercontent.com/tympom/opencode-go-cliproxyapi/main/registry.json"
```

Install **opencode-go-clpx** from the Management Center's Plugin Store and restart CLIProxyAPI.

## Configuration

Under `plugins.configs.opencode-go-clpx` (or Plugins -> Edit config), list your raw OpenCode Go keys:

```yaml
api-keys:
  - value: "sk-..."
    name: "Work"            # optional; labels the account and names its auth file
  - "${OPENCODE_GO_API_KEY}"
```

Without keys the plugin serves no models; duplicate keys are rejected.

| Optional key | Default |
| --- | --- |
| `base-url` | `https://opencode.ai/inference/go/v1` |
| `catalog-url` | `{base-url}/models` |
| `model-prefix.enabled` / `.value` | `true` / `opencode-go` |
| `catalog.refresh-interval` | `15m` (min `1m`) |
| `catalog.stale-while-unavailable` | `true` |
| `protocols.chat-completions` / `.messages` / `.responses` | `true` |
| `route-overrides` | none; per model `protocol` and `endpoint` |
| `request-timeout` | `5m` |
| `max-response-bytes` | 64 MiB |
| `allow-http` | `false` |

Quotas (rolling, weekly, monthly) use CLIProxyAPI's generic quota API and need a Management Center build that supports it; reset is unsupported. Tested with CLIProxyAPI `v8.0.38` and Go `1.27.3`.
