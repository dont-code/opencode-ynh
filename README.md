# OpenCode for YunoHost

A [YunoHost](https://yunohost.org/) v2 application package for
[OpenCode](https://opencode.ai/), the open-source AI coding agent.

The package downloads the **latest OpenCode V2 release** with the official
installer at install/upgrade time, runs it as a dedicated system user under
systemd, and lets the administrator choose how it is reached. If the YunoHost
**Ollama** app is already installed, OpenCode is automatically configured to use
its local OpenAI-compatible API. An optional external provider (OpenAI,
Anthropic or Mistral) can be configured at install time with its API key and
combined with local Ollama. A YunoHost config panel can re-detect providers,
change the external provider/key, switch the exposure mode between `web`,
`opentunnel` and `vpn`, and create an OpenTunnel pairing link/QR at any time.

| Mode | What it does |
| --- | --- |
| `web` | Nginx reverse proxy at the root of the chosen YunoHost domain, protected by YunoHost SSO and automatically connected to the local server (no pairing link). |
| `opentunnel` | OpenCode's built-in end-to-end encrypted **OpenTunnel** address. No YunoHost nginx exposure. |
| `vpn` | Service bound to a private address only (127.0.0.1 by default, or a VPN IP you provide). No nginx exposure. |

> [!WARNING]
> OpenCode can run shell commands and edit files as its system user. Treat
> anyone who can reach its HTTP API as having that user's privileges. Keep a
> long password, avoid exposing it on a public network unnecessarily, and
> review OpenCode's permission settings in its configuration.

## Install

From the YunoHost webadmin, install this repository, or from the CLI:

```bash
sudo yunohost app install /home/papou/devs/opencode-ynh
```

Answer the install questions:

- domain (used only in `web` mode); keep the path at `/` because the web UI serves assets and API requests from the domain root. Use a dedicated domain for each web instance,
- `exposure`: `web`, `opentunnel` or `vpn`,
- `web_access_group`: YunoHost group allowed to open the web UI (web mode only),
- `vpn_bind_address`: optional VPN IP of the server (vpn mode only),
- `external_provider`: `none`, `openai`, `anthropic` or `mistral`,
- `external_api_key`: the API key (required when an external provider is selected).

## Layout

- `manifest.toml` — package metadata, install questions and resources.
- `config_panel.toml` — YunoHost config panel with provider/external-key settings and remote-access actions.
- `scripts/` — install, remove, upgrade, backup, restore, change-url and config-panel logic.
- `conf/nginx.conf` — reverse-proxy template used in `web` mode.
- `conf/systemd.service` — systemd unit template running `opencode serve --service` and loading the generated provider config file.
- `assets/logo.png` — upstream OpenCode icon applied to the YunoHost app tile (replaceable in the standard Permissions panel).
- `doc/` — text shown in the YunoHost admin interface.
- `scripts/_common.sh` — shared helpers, including the latest-release installer.

## Notes for maintainers

- The downloader uses `https://opencode.ai/v2/install`, which resolves the
  current V2 release from `opencode.ai`. The manifest version (`2.3.2~ynh11`)
  describes the *package*, not the upstream binary; bump it when the package
  changes so YunoHost offers an upgrade.
- Binary state, sessions, service config and logs live under `$data_dir`
  (`/home/yunohost.app/opencode` by default). The binary itself is not backed
  up; it is re-downloaded on restore.
- `opencode serve --service --hostname <bind> --port <port>` is used instead of a detached `opencode service
  start` so systemd supervises a foreground process. The bind address and port
  come from the install answers/YunoHost port resource; the password and
  OpenTunnel route are read from OpenCode's own service config. This is also
  the mode in which OpenCode starts the OpenTunnel client.
- Local Ollama detection and the optional external provider (with its API key)
  are regenerated together on install, upgrade, restore and from the **Providers**
  tab of the config panel. The generated config lives at
  `$data_dir/.config/opencode/ynh-ollama.json` and is loaded through
  `OPENCODE_CONFIG`, so admin-owned `opencode.json` / `opencode.jsonc` files are
  left untouched. The file is owned by the app user with mode `0600`.
- The **Remote access** config panel action reuses the existing service password,
  regenerates the systemd unit and nginx/permission configuration as needed,
  and re-creates the OpenTunnel route when switching to `opentunnel`. The
  **Pair OpenTunnel** action runs `opencode pair --remote --yes` and shows the
  one-time link and QR code in the panel. Switching modes invalidates previous
  pairing links.
- Only `amd64` and `arm64` are declared, matching the upstream Linux releases.

## License

MIT — see `LICENSE`.
