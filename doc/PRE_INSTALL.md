Before installing, please note:

- The installer downloads the **latest OpenCode V2 release** from
  `opencode.ai` (about 200 MB) and needs outbound HTTPS access.
- OpenCode can run shell commands, read files and edit files as its own
  system user. Anyone who can authenticate to its HTTP API effectively has
  that user's privileges.
- In `web` mode the app is reverse-proxied by YunoHost nginx and protected by
  the SSO group you select. In `opentunnel` mode access is through OpenCode's
  end-to-end encrypted OpenTunnel address, protected by the server password.
  In `vpn` mode the service stays on a private address and is reachable only
  from your VPN or through an SSH tunnel.
- If the YunoHost **Ollama** app is installed, OpenCode is automatically
  configured to use its local API. Pull at least one model in Ollama before
  selecting it in OpenCode.
- Supported architectures are `amd64` and `arm64`.
