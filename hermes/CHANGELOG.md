# Changelog

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
