# OAuth callback

Wonton connects to remote MCP servers that use OAuth. A home server has no public HTTPS address, so the
provider redirects to this static page, which hands the result to the Wonton app; the app passes it to
the user's own server, which exchanges the code for tokens.

- `client.json`: the OAuth Client ID Metadata Document. Its URL is the `client_id`.
- `callback/`: the redirect URI. It forwards the query string to `wonton://oauth/callback` and stores nothing.

The authorization code is useless without the PKCE verifier, which stays on the user's server.
