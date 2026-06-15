# Wrap the official multi-arch Hermes image — do not fork/rebuild Hermes.
# Pinned (tag + digest) for reproducible builds; bump this together with
# `version` in config.yaml. Released images use CalVer (vYYYY.M.D); the
# `latest`/`main` tags are ROLLING — never pin to them. See DOCS.md.
FROM nousresearch/hermes-agent:v2026.6.5@sha256:9ad3b04ec916ea2c2da22358fd43b024c788d74073210695af88bfc2e63869b4

# ---------- add-on extras layered on the Hermes image (debian:13.4) ----------
# nginx-light: tiny reverse proxy that adapts the dashboard to HA Ingress — it
#   rewrites HA's `X-Ingress-Path` header into the `X-Forwarded-Prefix` Hermes
#   uses to serve its SPA under the Ingress subpath (rootfs/etc/nginx/nginx.conf).
# vim-tiny: minimal editor for hand-editing config from a shell, e.g.
#   `docker exec -it <container> vi /data/.env`.
RUN apt-get update \
    && apt-get install -y --no-install-recommends nginx-light vim-tiny \
    && rm -rf /var/lib/apt/lists/*

# Relocate Hermes' entire home — config, .env, provider credentials / OAuth
# tokens (incl. Codex), profiles, sessions, skills, memories — onto the add-on's
# persistent /data volume, which Home Assistant includes in backups. Overrides
# the image default of /opt/data (which under an add-on would be an ephemeral
# anonymous volume, lost on update/rebuild).
ENV HERMES_HOME=/data

# Always run the dashboard service. It binds 0.0.0.0:9119 inside the container,
# but 9119 is NOT published to the host — only the bundled nginx (with HA
# Supervisor/Ingress in front) reaches it. nginx translates HA Ingress'
# `X-Ingress-Path` into the `X-Forwarded-Prefix` header Hermes uses to serve the
# SPA correctly under the Ingress subpath. See rootfs/etc/nginx/nginx.conf.
ENV HERMES_DASHBOARD=1

# Skip Hermes' OAuth auth gate on the 0.0.0.0 (non-loopback) bind: HA login via
# Ingress is the auth layer and 9119 isn't host-published. The 0.0.0.0 bind also
# keeps the dashboard's chat/PTY/events WebSocket gate open — that gate only
# rejects non-loopback peers on a *loopback* bind (Hermes' _ws_client_reason),
# so binding 0.0.0.0 is what lets the chat tab work behind the proxy.
ENV HERMES_DASHBOARD_INSECURE=1

# Render HA add-on options (/data/options.json) into $HERMES_HOME/.env at boot.
# Runs after the image's own bootstrap (01-hermes-setup -> stage2-hook).
COPY rootfs/etc/cont-init.d/03-ha-config /etc/cont-init.d/03-ha-config
RUN chmod +x /etc/cont-init.d/03-ha-config

# nginx Ingress adapter: minimal config + s6-overlay longrun service. Fronts
# the dashboard on the Ingress port (8099, see config.yaml `ingress_port`) and
# proxies to Hermes on 127.0.0.1:9119, translating the Ingress prefix header.
COPY rootfs/etc/nginx/nginx.conf /etc/nginx/nginx.conf
COPY rootfs/etc/services.d/nginx/run /etc/services.d/nginx/run
RUN chmod +x /etc/services.d/nginx/run

# Reproduce the documented persistent deployment: the foreground gateway daemon.
# This runs as /init's "main program" via the image's main-wrapper.sh, which
# routes it to `hermes gateway run`.
CMD ["gateway", "run"]

# Home Assistant add-on metadata (purely informational).
ARG BUILD_ARCH
LABEL \
  io.hass.name="Hermes Agent" \
  io.hass.type="addon" \
  io.hass.version="2026.6.5.2" \
  io.hass.arch="aarch64|amd64"
