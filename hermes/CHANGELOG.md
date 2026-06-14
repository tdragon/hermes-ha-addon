# Changelog

## 0.1.0

- Initial release: Hermes Agent **gateway + dashboard** as a Home Assistant add-on.
- Wraps the official multi-arch `nousresearch/hermes-agent` image (amd64 / aarch64).
- State persisted under `/data` via `HERMES_HOME` — included in HA backups
  (config, `.env`, provider credentials / OAuth tokens, profiles, sessions, skills).
- Dashboard exposed via **Ingress** (9119).
- OpenAI-compatible API on **8642**, enabled when `api_server_key` is set.
- `extra_env` option for arbitrary `KEY=VALUE` passthroughs into Hermes' `.env`.
