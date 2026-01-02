# Operator Reference (minimal)

This is the only reference doc. Everything else should be driven by the Agent‑First Guide.

## Module options (Home Manager)

- `programs.clawdis.enable` (bool, default: false)
- `programs.clawdis.package` (package, default: `pkgs.clawdis-gateway`)
- `programs.clawdis.stateDir` (string, default: `~/.clawdis`)
- `programs.clawdis.workspaceDir` (string, default: `~/.clawdis/workspace`)

Telegram:
- `programs.clawdis.providers.telegram.enable` (bool, default: false)
- `programs.clawdis.providers.telegram.botTokenFile` (string path, required if enabled)
- `programs.clawdis.providers.telegram.allowFrom` (list of ints, required if enabled)
- `programs.clawdis.providers.telegram.requireMention` (bool, default: false)

Routing:
- `programs.clawdis.routing.queue.mode` (enum: queue|interrupt, default: interrupt)
- `programs.clawdis.routing.queue.bySurface` (attrset, defaults to telegram=interrupt, discord/webchat=queue)
- `programs.clawdis.routing.groupChat.requireMention` (bool, default: false)

macOS service:
- `programs.clawdis.launchd.enable` (bool, default: true)
- launchd label: `com.nix-clawdis.gateway`

## Verification commands

```bash
launchctl print gui/$UID/com.nix-clawdis.gateway | grep state
tail -n 50 ~/.clawdis/logs/clawdis-gateway.log
```

Smoke test:
- Send a Telegram message in an allowlisted chat; bot must reply.

## Secrets wiring (recommended)

- Use agenix or an equivalent secrets tool to place the bot token on disk.
- Configure `programs.clawdis.providers.telegram.botTokenFile` to point at that file.
- Do not inline tokens in Nix configs.
