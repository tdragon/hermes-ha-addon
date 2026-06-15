# Hermes Agent add-on

Runs [Hermes Agent](https://hermes-agent.nousresearch.com/docs/) (Nous Research) as a
Home Assistant add-on: the **gateway daemon** plus the **web dashboard** (in the HA
sidebar via Ingress). All state lives in the add-on's `/data` volume, so it's captured
by Home Assistant backups.

Hermes needs no GPU and runs no local LLM — it calls an external model provider you log
into once (see "Choosing a model provider").

## Configuration

| Option | Required | Purpose |
| --- | --- | --- |
| `api_server_key` | For HA Assist / API | Bearer token clients must send to use the OpenAI-compatible API on `8642`. **Must be a strong random secret** (`openssl rand -hex 32`) — weak/placeholder values are rejected and the API refuses to start. Empty → that API stays **disabled** (the dashboard still works). |
| `extra_env` | No | List of `KEY=VALUE` strings written into Hermes' `.env`. Use for API-key providers (e.g. `OPENROUTER_API_KEY=...`) or settings like `HERMES_PROFILE=default`. |

After changing options, **Restart** the add-on — they're re-applied to `.env` on boot.

## First run

1. Install, set `api_server_key` (optional), **Start**.
2. **Log** tab → wait for a clean gateway start.
3. Open the **Hermes** sidebar panel (the Ingress dashboard).
4. Log into a model provider (next section) — required before Hermes can answer.

## Choosing a model provider

Pick whichever you want Hermes to use as its brain:

- **API-key providers** (OpenAI `sk-...`, Anthropic, OpenRouter): add the key via
  `extra_env`, e.g. `OPENROUTER_API_KEY=sk-or-...`, then Restart. Non-interactive.
- **Subscription / OAuth providers** (OpenAI **Codex / "Sign in with ChatGPT"**, Nous
  Portal, Claude Max): a one-time device-code login, below.

### One-time OAuth login (e.g. Codex / ChatGPT)

Easiest: open the **Hermes** sidebar panel (Ingress dashboard) and use its built-in
"add provider / login" flow for `openai-codex`.

CLI fallback — install the **Advanced SSH & Web Terminal** add-on, then run on the host:

```sh
# container name is addon_<repo-slug>_hermes — confirm with: docker ps | grep hermes
docker exec -it $(docker ps --format '{{.Names}}' | grep hermes) \
  hermes auth add openai-codex --type oauth --no-browser
```

Codex uses a **device-code** flow: it prints a verification URL and a code — open the
URL in any browser, enter the code, authorize. (`--no-browser` just stops it trying to
launch a browser inside the container.) The credential is saved under `HERMES_HOME`
(`/data`), so it **persists across restarts, updates, and HA backups**. Then pick the
default model:

```sh
docker exec -it $(docker ps --format '{{.Names}}' | grep hermes) hermes model
```

Verify with `hermes auth status` / `hermes auth list`, then **Restart** the add-on and
confirm it's still logged in.

> Note: `hermes login` was removed in newer Hermes releases — use `hermes auth` /
> `hermes model` as above.

## Using Hermes as Home Assistant's Assist brain

Once `api_server_key` is set and Hermes has a model:

1. From the SSH add-on, sanity-check the API:
   ```sh
   curl -H "Authorization: Bearer <api_server_key>" http://localhost:8642/v1/models
   ```
2. In HA, install the HACS integration **Extended OpenAI Conversation** (the built-in
   "OpenAI Conversation" is locked to api.openai.com).
3. Configure it with:
   - **Base URL:** `http://<HA-host-IP>:8642/v1` (or `http://homeassistant.local:8642/v1`)
   - **API key:** your `api_server_key`
   - **Model:** the model from `/v1/models`
4. **Settings → Voice assistants → Assist pipeline → Conversation agent =** Extended
   OpenAI Conversation. Keep **"prefer handling commands locally"** ON so fast local
   intents (lights, etc.) don't round-trip through the agent.

Note: requests to `8642` run through the **full Hermes agent** (memory/skills/tools) —
richer answers, but slower/pricier than a plain model call. That's why local intents
should stay local.

## Updating / pinning

The Dockerfile is pinned to a concrete release (tag **and** digest), e.g.
`nousresearch/hermes-agent:v2026.6.5@sha256:...`. Released images use CalVer
(`vYYYY.M.D`). **Do not pin to `latest` or `main`** — they are rolling tags that
change without notice, which makes builds non-reproducible and the add-on's version
number meaningless.

To move to a newer Hermes:

1. Pick a tag from <https://hub.docker.com/r/nousresearch/hermes-agent/tags> and note
   its digest (the tag page shows it, or `docker buildx imagetools inspect <tag>`).
2. Edit `hermes/Dockerfile` → `FROM nousresearch/hermes-agent:<vYYYY.M.D>@sha256:<digest>`.
3. Set `version` in `hermes/config.yaml` to the same `YYYY.M.D` and add a `CHANGELOG.md`
   entry. For add-on-only changes without an image bump, append a 4th segment, e.g.
   `2026.6.5.1` (a `-N` suffix is treated as a pre-release and won't trigger an update).
4. Push to the repo → HA shows an **Update** button that rebuilds against the new image.

## Troubleshooting

- **"Refusing to bind dashboard to 0.0.0.0 ... no auth providers registered".** Hermes
  guards non-loopback dashboard binds with an OAuth gate. This add-on sets
  `HERMES_DASHBOARD_INSECURE=1` (in the Dockerfile) to skip it, because HA Ingress is the
  auth layer and 9119 isn't published to the host. If you see this message, you're on an
  image built before that env was added — Rebuild the add-on.
- **Dashboard assets 404 under Ingress (panel blank, CSS/JS fail to load).**
  Fixed since `2026.6.5.2`: a small bundled nginx translates HA's `X-Ingress-Path`
  header into the `X-Forwarded-Prefix` Hermes uses to serve its SPA under the
  Ingress subpath. If assets still 404, you're on an older build — Rebuild the
  add-on. Last-resort bypass: expose Hermes directly with `"9119/tcp": 9119`
  under `ports`, Rebuild, and open `http://<HA-host-IP>:9119` (no sidebar).
- **Need to hand-edit a config file.** `vim-tiny` is bundled — from the SSH
  add-on: `docker exec -it $(docker ps --format '{{.Names}}' | grep hermes) vi /data/.env`.
- **Add-on won't start with an AppArmor / permission error.** The image does
  `usermod`/`chown` at boot. If a default-profile denial blocks it, add
  `apparmor: false` to `config.yaml` and Rebuild.
- **API on 8642 unreachable from HA.** Confirm `api_server_key` is set (empty disables
  the API), the add-on is running, and you're using the **host IP**, not `localhost`
  (HA Core is a separate container).
- **`/data/options.json` not applied.** Options are written to `.env` only on
  (re)start — Restart after editing Configuration.

## What this add-on does and doesn't do

- **Does:** package the official image, persist state to `/data`, expose the dashboard
  via Ingress (with a bundled nginx that adapts it to the Ingress subpath), and turn
  add-on options into Hermes `.env` settings.
- **Doesn't:** fork or rebuild Hermes, manage your provider login (one-time, by you),
  or give the agent control of HA entities (that's a separate Assist/function-calling
  choice you can make later).
