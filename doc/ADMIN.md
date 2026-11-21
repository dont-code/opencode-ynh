# Admin notes for OpenCode

## YunoHost config panel

The YunoHost webadmin tab **Apps → __APP__ → Config panel** has these actions:

- **Providers → Re-detect local providers** rechecks whether the YunoHost
  Ollama app is installed, rewrites the generated provider config and restarts
  OpenCode. The selected external provider and its stored API key are
  preserved. This is the preferred way to refresh local providers.
- **Providers → Apply provider change** writes the selected external provider
  (`none`, `openai`, `anthropic` or `mistral`) together with its API key into
  the generated config, then restarts OpenCode. Leaving the API key field empty
  keeps the key that is already stored. Selecting `none` clears it.
- **Remote access → Apply remote access change** switches between `web`,
  `proxy` and `vpn`, updates nginx/SSOwat or the private bind address, and
  regenerates the OpenTunnel route when `proxy` is selected.

Changing exposure invalidates existing pairing links. Pair again after the
change. Both actions can also be called from the CLI, for example:

```bash
sudo yunohost app action run __APP__ remote_access.settings.apply_remote_access \
  --args "exposure=proxy"
```

## Useful commands

Run CLI commands as the app user with its XDG environment:

```bash
sudo -u __APP__ env HOME=__DATA_DIR__ \
  XDG_CONFIG_HOME=__DATA_DIR__/.config \
  XDG_DATA_HOME=__DATA_DIR__/.local/share \
  XDG_STATE_HOME=__DATA_DIR__/.local/state \
  XDG_CACHE_HOME=__DATA_DIR__/.cache \
  __DATA_DIR__/.opencode/bin/opencode service status
```

Replace `service status` with any OpenCode subcommand (`pair`, `service get`,
`upgrade`, ...).

## Rotate the password

```bash
NEW_PASSWORD=$(openssl rand -hex 24)
sudo -u __APP__ env HOME=__DATA_DIR__ \
  XDG_CONFIG_HOME=__DATA_DIR__/.config \
  XDG_DATA_HOME=__DATA_DIR__/.local/share \
  XDG_STATE_HOME=__DATA_DIR__/.local/state \
  __DATA_DIR__/.opencode/bin/opencode service set password "$NEW_PASSWORD"
sudo -u __APP__ env HOME=__DATA_DIR__ \
  XDG_CONFIG_HOME=__DATA_DIR__/.config \
  XDG_DATA_HOME=__DATA_DIR__/.local/share \
  XDG_STATE_HOME=__DATA_DIR__/.local/state \
  __DATA_DIR__/.opencode/bin/opencode service start
sudo yunohost app setting __APP__ service_password -v "$NEW_PASSWORD"
```

Changing the password signs out every paired browser.

## Disable or re-enable the proxy

Prefer the **Remote access** tab of the YunoHost config panel. From the CLI:

```bash
# disable
.../opencode service set remote false
# re-enable and get a new link
.../opencode pair --remote --yes
```

## Refresh providers and set the external API key

Install, upgrade and restore detect an installed YunoHost Ollama app and write
the combined provider config to
`__DATA_DIR__/.config/opencode/ynh-ollama.json`. Use the **Providers** tab of
the YunoHost config panel to re-detect local providers or to change the external
provider/API key at any time. The equivalent CLI commands are:

```bash
sudo yunohost app action run __APP__ providers.actions.refresh_providers
sudo yunohost app action run __APP__ providers.external.apply_providers \
  --args "external_provider=openai" --args "external_api_key=YOUR_API_KEY"
```

The generated file is loaded via `OPENCODE_CONFIG`, so any `opencode.json` or
`opencode.jsonc` you manage yourself is preserved. It is owned by the app user
with mode `0600` because it contains the API key.

## Update OpenCode

YunoHost upgrades re-run the official installer and fetch the latest release:

```bash
sudo yunohost app upgrade __APP__
```

You can also update only the binary with `opencode upgrade` as the app user,
but a package upgrade will replace it again with the current upstream release.
