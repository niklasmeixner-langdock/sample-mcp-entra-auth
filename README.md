# Sample MCP Server with Microsoft Entra ID Auth (DCR)

A sample [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) server demonstrating Microsoft Entra ID (Azure AD) OAuth integration with Dynamic Client Registration (DCR).

## How It Works

This server acts as an OAuth 2.0 Authorization Server that proxies authentication to Microsoft Entra ID:

1. MCP client discovers OAuth metadata via `/.well-known/oauth-authorization-server`
2. MCP client registers dynamically via DCR at `/register`
3. MCP client sends user to `/authorize` with PKCE
4. Server redirects user to Entra ID for authentication
5. User authenticates with Entra ID
6. Entra ID redirects back to `/entra/callback` with an auth code
7. Server exchanges the Entra code for tokens, generates its own auth code
8. Server redirects back to the MCP client with the auth code
9. MCP client exchanges the code at `/token` for an access token
10. MCP client uses the access token for authenticated MCP requests at `/mcp`

## Prerequisites

- Node.js 18+
- pnpm
- A Microsoft Entra ID (Azure AD) app registration with:
  - **Client secret** configured
  - `User.Read` delegated permission (Microsoft Graph)
  - Redirect URI set to `http://localhost:3333/entra/callback` (or your server URL)

## Setup

1. Install dependencies:
   ```bash
   pnpm install
   ```

2. Copy and configure environment variables:
   ```bash
   cp .env.example .env
   ```

   Fill in your Entra ID app registration details:
   - `ENTRA_TENANT_ID` - Your Azure AD tenant ID
   - `ENTRA_CLIENT_ID` - App registration client ID
   - `ENTRA_CLIENT_SECRET` - App registration client secret

## Running

Development mode (with hot reload):
```bash
pnpm dev
```

Production:
```bash
pnpm build
pnpm start
```

The server starts on `http://localhost:3333`.

## Endpoints

| Endpoint | Description |
|----------|-------------|
| `/.well-known/oauth-authorization-server` | OAuth 2.0 Authorization Server Metadata |
| `/register` | Dynamic Client Registration (RFC 7591) |
| `/authorize` | Authorization endpoint (redirects to Entra ID) |
| `/token` | Token exchange endpoint |
| `/entra/callback` | Entra ID OAuth callback |
| `/mcp` | MCP endpoint (protected by bearer auth) |

## MCP Tools

| Tool | Description |
|------|-------------|
| `get-current-user` | Returns the authenticated user's profile from Microsoft Graph (id, displayName, mail, userPrincipalName, jobTitle) |

## Testing

Use the MCP Inspector:
```bash
pnpm inspector
```

## MCP Client Configuration

Configure your MCP client to connect to this server:

```json
{
  "mcpServers": {
    "entra-auth-sample": {
      "type": "streamable-http",
      "url": "http://localhost:3333/mcp"
    }
  }
}
```

The client will automatically discover the OAuth metadata and handle the authentication flow including DCR.
