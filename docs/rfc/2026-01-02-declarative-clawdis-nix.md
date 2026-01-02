# RFC: Declarative Clawdis as a Nix Package (nix-clawdis)

- Date: 2026-01-02
- Status: Implementing
- Audience: Nix users, agents (Codex/Claude), package maintainers, operators

## 1) Narrative: what we are building and why

Clawdis is powerful but hard to install and configure for new users, especially those who do not want to learn Nix internals. We need a batteries‑included, obvious, and safe path to get a working Clawdis instance with minimal friction. This RFC proposes a dedicated public repo, `nix-clawdis`, that packages Clawdis for Nix and provides a declarative, user‑friendly configuration layer with strong defaults, clear guardrails, and an **agent‑first onboarding flow**.

The goal is a **fully declarative bootstrap**: a user can provide a small set of inputs (token path + allowlist), and an agent performs all configuration steps via Nix, with no ad‑hoc snippets or manual tweaking.

## 1.1) Non‑negotiables

- Nix‑first installation: no global installs, no manual brew steps required for core functionality.
- Safe defaults: providers disabled unless explicitly enabled and configured.
- No secrets committed to the repo; explicit guidance for secrets wiring (agenix‑style).
- **Agent‑first docs**: no random snippets; a single prompt + deterministic steps.
- Deterministic builds and reproducible outputs.
- Documentation must be suitable for publication on the internet.

## 1.2) Scope boundaries (avoid confusion)

This RFC is only about:
- The public `nix-clawdis` repo (package + module + docs).
- A generic, end‑user Nix setup that lives outside any personal config repo.

This RFC is explicitly **not** about:
- Josh’s personal `nixos-config` or any private machine configuration.
- Editing or publishing personal settings, tokens, or machine‑specific modules.

## 2) Goals / Non‑goals

Goals:
- Provide a Nix package for Clawdis and a Home Manager module with batteries‑included defaults.
- Provide a flake app for the gateway CLI and a clear verification path.
- Make configuration technically light with explicit options and guardrails.
- Telegram‑first configuration and defaults.
- Provide a single **agent‑first onboarding flow** that is end‑to‑end declarative.
- New user can get a working bot in 10 minutes without understanding Nix internals.

Non‑goals:
- Rewriting Clawdis core functionality.
- Supporting non‑Nix install paths in this repo.
- Shipping a hosted SaaS or paid hosting.
- Replacing upstream Clawdis docs.
- Cross‑platform support (Linux/Windows) in v1.
- CI automation in v1.

## 3) System overview

`nix-clawdis` is a public repo that provides (macOS‑only in v1, no CI in v1):
- A Nix package derivation for Clawdis.
- A Home Manager module for user‑level config and service wiring.
- A nix‑darwin module for macOS users (optional, thin wrapper over HM).
- A flake with devShell and a gateway app.
- Agent‑first documentation and a declarative bootstrap flow.

## 4) Components and responsibilities

- **Package derivation**: builds Clawdis from a pinned source (tag or rev) and exposes a binary.
- **Home Manager module**: declarative config, writes `~/.clawdis/clawdis.json`, manages services.
- **Flake outputs**:
  - `packages.<system>.clawdis-gateway` (binary)
  - `apps.<system>.clawdis` (gateway CLI)
  - `devShells.<system>.default` (docs + lint + tests)
  - `homeManagerModules.clawdis`
  - `darwinModules.clawdis` (if needed)
- **Docs**: single agent‑first onboarding flow + operator reference.

## 5) Configuration model (public contract)

The Home Manager module is the public contract. It must expose a small, explicit option set (enable, token path, allowlist, queue mode) and render a deterministic `~/.clawdis/clawdis.json`.

The design constraint: **users should not have to write arbitrary JSON**. The module should remain the only supported configuration surface for v1.

## 6) Agent‑first onboarding flow (no snippets)

The onboarding experience must be a **single prompt** that an agent can execute end‑to‑end, producing a working Clawdis instance. The flow includes:

- Install Determinate Nix if missing.
- Enable flakes.
- Create a minimal local flake (in a neutral user directory, not a personal config repo).
- Add `nix-clawdis` input and HM module.
- Wire secrets declaratively (token file path + allowlist).
- Run a build and verify the service is running.

The docs must provide a **single prompt** and a **deterministic checklist**, not a pile of unrelated snippets.

## 7) Secrets handling (opinionated default)

- Recommend agenix for bot tokens on macOS.
- Default docs refer to a token file path under `/run/agenix/`.
- Provide a minimal, agent‑friendly explanation of how to create and reference the secret file.

## 8) Verification and smoke test

Verification must be explicit and minimal:
- launchd service name is `com.nix-clawdis.gateway`.
- log path is `~/.clawdis/logs/clawdis-gateway.log`.
- smoke test is a **real Telegram message** in an allowlisted chat and a reply from the bot.

## 9) Determinism and validation

- Pin Clawdis source to a known revision or release tag.
- Nix assertions validate required tokens and allowlists.
- Providers must not start unless explicitly enabled and configured.
- Strict allowlists for inbound chat IDs.
- Emit clear, actionable errors when config is invalid.

## 10) Deliverables (docs)

We will rebuild docs to be **agent‑first**:
- A single “Agent‑First Guide” (copy‑paste prompt + checklist).
- A minimal operator reference (options, defaults, verification steps).

No other docs until those two are excellent.

## 11) Definition of Done (DoD)

This RFC is complete when:
- The repo is public with a clear README and agent‑first guide.
- Telegram‑first quickstart works on macOS with a real bot token.
- `nix run .#clawdis` launches the gateway and responds in an allowlisted chat.
- Documentation includes a single copy‑paste agent prompt and explicit verification steps.
- Smoke test: user sends a Telegram message in an allowlisted chat and receives a response.
- Secrets flow documented with agenix‑style token file wiring.
- A release tag is published and referenced in the agent‑first guide.

## 12) Implementation status (current)

- Repo reset in progress to eliminate doc/code confusion.
- `nix/` is retained for audit and will be refit to the agent‑first model.
- Next milestone is rebuilding docs to match this RFC.

