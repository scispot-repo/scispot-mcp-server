# Changelog

All notable changes to this plugin will be documented here.

## 1.0.0 — initial release

- Added the Cursor plugin manifest (`.cursor-plugin/plugin.json`) and the `scispot` MCP server pointing at `https://mcp.scispot.io/` (streamable HTTP).
- Auth uses OAuth 2.0 with PKCE. Cursor opens the Scispot sign-in page; there is no API key or client ID to configure in the plugin.
- Logo: Scispot's official icon mark (the scispot.com apple-touch-icon), 256×256 PNG.
- README: added Cursor install steps and made OAuth sign-in the primary auth method.
