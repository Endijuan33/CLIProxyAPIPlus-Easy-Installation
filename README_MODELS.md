# Adding and Testing New Models (English)

This document explains how to add new model entries (for example: Claude Opus 5, Claude Opus 5 Thinking, GPT-5.6 Sol) to the CLIProxyAPI+ installation and how to verify they are available at runtime.

## Overview
- Models are declared in two places:
  - Example config: `configs/droid-config.json.example` (repository sample)
  - Runtime factory config: `~/.factory/config.json` or the repository `.factory/config.json` example
- The management server (GUI) exposes endpoints to view and modify the runtime factory config and to query models returned by the local proxy backend.
- The proxy binary (cliproxyapi-plus) must support forwarding the model id to the correct provider. If requests for a new model return errors, check the proxy version and logs.

## Example JSON entry
Add model entries to `configs/droid-config.json.example` (already present in the repo) or add them at runtime via the management API.

Example snippet (already added to configs/droid-config.json.example):

```json
{
  "model_display_name": "Claude Opus 5 Thinking",
  "model": "claude-opus-5-thinking",
  "base_url": "http://localhost:8317/v1",
  "api_key": "sk-dummy",
  "provider": "claude"
}
```

## Add models at runtime (management API)
Use the management server API exposed by `gui-cliproxyapi.ps1` (default port 8318) to add models without editing files.

cURL example:

```bash
curl -X POST "http://localhost:8318/api/factory-config/add" \
  -H "Content-Type: application/json" \
  -d '{
    "models": ["claude-opus-5","claude-opus-5-thinking","gpt-5.6-sol"],
    "displayNames": {
      "claude-opus-5": "Claude Opus 5",
      "claude-opus-5-thinking": "Claude Opus 5 Thinking",
      "gpt-5.6-sol": "GPT-5.6 Sol"
    }
  }'
```

PowerShell example (Invoke-RestMethod):

```powershell
$body = @{
  models = @("claude-opus-5","claude-opus-5-thinking","gpt-5.6-sol")
  displayNames = @{
    "claude-opus-5" = "Claude Opus 5"
    "claude-opus-5-thinking" = "Claude Opus 5 Thinking"
    "gpt-5.6-sol" = "GPT-5.6 Sol"
  }
} | ConvertTo-Json -Depth 4

Invoke-RestMethod -Uri "http://localhost:8318/api/factory-config/add" -Method Post -Body $body -ContentType "application/json"
```

## Verify added models
- GET the runtime factory config:
  - `GET http://localhost:8318/api/factory-config`
- GET the models endpoint exposed by the proxy (models known to proxy backend):
  - `GET http://localhost:8318/api/models`
- Check that the model ids appear either in factory config or in proxy's models list.

## Restarting / Applying changes
- To restart the management server via the API (if required):
  - `POST http://localhost:8318/api/restart` (the GUI script exposes /api/restart)
- If you modify local files under `%USERPROFILE%/.cli-proxy-api` or `~/.cli-proxy-api`, restart the cliproxyapi-plus server using the GUI or `start-cliproxyapi.ps1`.

## OAuth & provider notes
- The GUI script `scripts/gui-cliproxyapi.ps1` contains a `Start-OAuthLogin` function mapping provider names to CLI flags used when launching the binary for OAuth.
- If a new provider or new login flag is required for a model, add an entry in the $flags hashtable inside `Start-OAuthLogin`.

Example (PowerShell) mapping in `Start-OAuthLogin`:

```powershell
$flags = @{
  "gemini" = "--login"
  "copilot" = "--github-copilot-login"
  "antigravity" = "--antigravity-login"
  "codex" = "--codex-login"
  "claude" = "--claude-login"
  "qwen" = "--qwen-login"
  "iflow" = "--iflow-login"
  "kiro" = "--kiro-aws-login"
}
```

- To add a new provider mapping (for example `opus5`), add an entry with the provider key and the corresponding command-line flag. The script will use this mapping when invoking the binary to start an OAuth login flow.

## Troubleshooting
- If a model call fails:
  - Check logs: `%USERPROFILE%/.cli-proxy-api/logs` (Windows) or `~/.cli-proxy-api/logs` (Linux/macOS)
  - Ensure the binary (cliproxyapi-plus) version supports forwarding the new model id. If not, upgrade the binary.
  - Ensure valid auth tokens exist for the provider (use the GUI OAuth buttons or Start-OAuthLogin mapping).

## Security
- Never commit real API keys. Example configs use `sk-dummy` placeholders.
- Store secrets locally in `~/.cli-proxy-api/config.yaml` or in OS-specific secret managers.

---

End of document.
