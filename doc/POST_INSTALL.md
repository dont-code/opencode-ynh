OpenCode is installed. Selected exposure: **__EXPOSURE__**.

### If you chose `web`

Open https://__DOMAIN____PATH__ and sign in with a YunoHost account allowed
by the `__APP__` permission. No pairing link is needed: the web UI is already
connected to this server's local OpenCode service through the reverse proxy.

### If you chose `opentunnel`

The encrypted OpenTunnel address is:

**__REMOTE_URL__**

From the YunoHost config panel, use **Remote access → Pair OpenTunnel** to
create a one-time link and QR code for the OpenCode app or another browser.

### If you chose `vpn`

The service listens on **__BIND_ADDRESS__:__PORT__**. From a device connected
to your VPN you can forward it locally:

```bash
ssh -L __PORT__:__BIND_ADDRESS__:__PORT__ root@__DOMAIN__
```

Then open the link printed by `opencode pair` in your browser.

### Server password

The HTTP API password is stored in the app setting `service_password` and in
OpenCode's own service configuration. HTTP clients use username `opencode`.
Browser clients use one-time pairing links instead.

### Local Ollama

If the YunoHost Ollama app was detected, the `ollama` provider is already
configured against its local API. Pull a model in Ollama, then select one of its
`ollama/<model>` entries from OpenCode's `/models` picker. If you install Ollama
after OpenCode, use the **Providers → Re-detect local providers** button in the
YunoHost config panel (or run `sudo yunohost app upgrade __APP__`).

### External provider

If an external provider was selected at install time, it is configured in the
same generated config file alongside local Ollama, using the API key you
provided. Models from both providers appear in OpenCode's `/models` picker.
To change the provider or its key later, use the **Providers → Apply provider
change** button in the YunoHost config panel. Leaving the key field empty keeps
the existing key; selecting `none` removes the external provider.
