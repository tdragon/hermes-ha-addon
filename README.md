# Hermes Agent — Home Assistant Add-on

Run [Nous Research's Hermes Agent](https://hermes-agent.nousresearch.com/docs/) as a
Home Assistant OS / Supervised **add-on**: managed from the HA UI, dashboard embedded
via Ingress, state persisted in HA backups, and an OpenAI-compatible API you can point
HA's Assist voice assistant at.

This repo is a **custom add-on repository**. It wraps the official multi-arch
`nousresearch/hermes-agent` image — it does not fork or rebuild Hermes, so you ride
upstream releases.

## Install

1. Home Assistant → **Settings → Add-ons → Add-on Store**.
2. Top-right **⋮ → Repositories**, paste this repo's URL, **Add**.
3. The **Hermes Agent** add-on appears under this repository — **Install**.
4. Open the add-on's **Configuration** tab, set `api_server_key` (a bearer token of
   your choosing, only needed for the HA Assist / API integration), **Save**.
5. **Start** the add-on, watch the **Log** tab for a clean gateway start.
6. Open the **Hermes** panel in the sidebar (Ingress dashboard).

See [`hermes/DOCS.md`](hermes/DOCS.md) for the one-time model login (e.g. "Sign in with
ChatGPT" / Codex), wiring HA Assist, pinning versions, and troubleshooting.

## Requirements

- Home Assistant OS or Supervised (has the Add-on Store / Supervisor).
- 64-bit host: `amd64` or `aarch64` (the Hermes image has no 32-bit ARM build, so a
  Raspberry Pi needs 64-bit HAOS).
- ~1 GB RAM minimum, 2–4 GB recommended (2 GB+ if you use browser automation).
- No GPU and no local LLM — Hermes calls external model providers over the network.
