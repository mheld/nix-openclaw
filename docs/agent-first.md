# Agent‑First Guide (single prompt + deterministic checklist)

This repo is designed to be executed by a coding agent with **zero context**. Use the prompt below exactly as‑is.

## Copy‑paste prompt (the only supported onboarding path)

```text
You are setting up Clawdis on macOS using the public nix-clawdis repo.

Requirements:
- Everything must be declarative and reproducible.
- Do NOT touch any personal nixos-config or private repo.
- Use a fresh, local flake directory (e.g. ~/code/clawdis-local).
- Use Home Manager for user-level config.

Inputs (ask me to fill these in if missing):
- macOS version:
- Home Manager configuration name (e.g. "myuser"):
- Telegram bot token file path:
- Telegram allowFrom chat IDs (list of ints):

Steps you must perform:
1) Check if Determinate Nix is installed; install it if missing.
2) Enable flakes in ~/.config/nix/nix.conf.
3) Create a local flake at ~/code/clawdis-local (do not use any personal config repo).
4) Add nix-clawdis as a flake input and enable the Home Manager module.
5) Wire the Telegram token file path and allowFrom list declaratively.
6) Run home-manager switch for the configured user.
7) Verify launchd status and logs:
   - launchctl print gui/$UID/com.nix-clawdis.gateway | grep state
   - tail -n 50 ~/.clawdis/logs/clawdis-gateway.log
8) Smoke test: send a Telegram message in an allowlisted chat and confirm the bot replies.

Constraints:
- Telegram-first only.
- No ad-hoc JSON edits; config must be derived from Nix options.
- Secrets must be passed as a file path (agenix-style).

Deliverables:
- A working Clawdis gateway running via launchd.
- A single, reproducible local flake that can be rebuilt on another machine.
- A short summary of what was changed and how to verify it again.
```

## Deterministic checklist (agent self‑test)

The agent’s work is correct **only if** all checks pass:

- A new local flake exists at `~/code/clawdis-local`.
- That flake references `github:joshp123/nix-clawdis` as an input.
- Home Manager config enables `programs.clawdis` with Telegram enabled.
- Token is referenced by a file path (no inline secrets).
- `launchctl print gui/$UID/com.nix-clawdis.gateway | grep state` shows `state = running`.
- `~/.clawdis/logs/clawdis-gateway.log` shows startup without fatal errors.
- A real Telegram message in an allowlisted chat receives a bot response.

If any item fails, the setup is incomplete.
