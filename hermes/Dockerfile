# Wrap the official multi-arch Hermes image — do not fork/rebuild Hermes.
# Pin this to a specific tag or digest for reproducible builds and bump it
# alongside `version` in config.yaml. `:latest` is used here for first-run
# convenience; see DOCS.md ("Updating / pinning").
FROM nousresearch/hermes-agent:latest

# Relocate Hermes' entire home — config, .env, provider credentials / OAuth
# tokens (incl. Codex), profiles, sessions, skills, memories — onto the add-on's
# persistent /data volume, which Home Assistant includes in backups. Overrides
# the image default of /opt/data (which under an add-on would be an ephemeral
# anonymous volume, lost on update/rebuild).
ENV HERMES_HOME=/data

# Always run the dashboard service; it is exposed via HA Ingress on 9119.
ENV HERMES_DASHBOARD=1

# Render HA add-on options (/data/options.json) into $HERMES_HOME/.env at boot.
# Runs after the image's own bootstrap (01-hermes-setup -> stage2-hook).
COPY rootfs/etc/cont-init.d/03-ha-config /etc/cont-init.d/03-ha-config
RUN chmod +x /etc/cont-init.d/03-ha-config

# Reproduce the documented persistent deployment: the foreground gateway daemon.
# This runs as /init's "main program" via the image's main-wrapper.sh, which
# routes it to `hermes gateway run`.
CMD ["gateway", "run"]

# Home Assistant add-on metadata (purely informational).
ARG BUILD_ARCH
LABEL \
  io.hass.name="Hermes Agent" \
  io.hass.type="addon" \
  io.hass.version="0.1.0" \
  io.hass.arch="aarch64|amd64"
