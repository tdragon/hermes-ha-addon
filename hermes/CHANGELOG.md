# Changelog

## 2026.6.5.3

- Mount the Home Assistant config directory **read-only** at `/homeassistant`
  (`map: homeassistant_config`), so the agent can read `home-assistant.log`,
  `configuration.yaml`, automations, etc. Read-only by design; see DOCS.md
  ("Access to the Home Assistant config directory") for the `secrets.yaml`
  caveat and how to opt into read-write.

## 2026.6.5.2

- Fix: dashboard assets 404'd under HA Ingress — the SPA loaded but its CSS/JS
  under `/assets/...` resolved to the HA host root instead of the Ingress
  subpath. Added a small bundled **nginx** reverse proxy that translates HA's
  `X-Ingress-Path` header into the `X-Forwarded-Prefix` header Hermes reads to
  re-anchor the SPA (rewrites asset URLs and sets the runtime base path).
  Ingress now points at nginx (`ingress_port: 8099`); Hermes stays internal on
  9119. The chat/PTY tabs keep working because the `0.0.0.0` bind leaves the
  dashboard's WebSocket peer gate open.
- Added `vim-tiny` to the image for hand-editing config via `docker exec`.

## 2026.6.5.1

- Fix: dashboard refused to start under HA Ingress ("Refusing to bind dashboard to
  0.0.0.0 ... no auth providers registered"). Set `HERMES_DASHBOARD_INSECURE=1` so it
  binds for Ingress — HA Ingress is the auth layer (HA login) and 9119 is not published
  to the host.

## 2026.6.5

- Initial release: Hermes Agent **gateway + dashboard** as a Home Assistant add-on.
- Wraps the official multi-arch `nousresearch/hermes-agent` image (amd64 / aarch64),
  pinned to `v2026.6.5` by tag + digest for reproducible builds.
- State persisted under `/data` via `HERMES_HOME` — included in HA backups
  (config, `.env`, provider credentials / OAuth tokens, profiles, sessions, skills).
- Dashboard exposed via **Ingress** (9119).
- OpenAI-compatible API on **8642**, enabled when `api_server_key` is set.
- `extra_env` option for arbitrary `KEY=VALUE` passthroughs into Hermes' `.env`.
