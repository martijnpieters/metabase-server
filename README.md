# metabase-server MCP Server

A Model Context Protocol server for Metabase integration.

This is a TypeScript-based MCP server that implements integration with Metabase API. It allows AI assistants to interact with Metabase, providing access to:

- Dashboards, questions/cards, and databases as resources
- Tools for listing and executing Metabase queries
- Ability to view and interact with Metabase data

## Features

### Resources
- List and access Metabase resources via `metabase://` URIs
- Access dashboards, cards/questions, and databases
- JSON content type for structured data access

### Tools
- `list_dashboards` - List all dashboards in Metabase
- `list_cards` - List all questions/cards in Metabase
- `list_databases` - List all databases in Metabase
- `execute_card` - Execute a Metabase question/card and get results
- `get_dashboard_cards` - Get all cards in a dashboard
- `execute_query` - Execute a SQL query against a Metabase database

## Configuration

Before running the server, you need to set environment variables for authentication. The server supports three methods:

1.  **API Key (Preferred):**
    *   `METABASE_URL`: The URL of your Metabase instance (e.g., `https://your-metabase-instance.com`).
    *   `METABASE_API_KEY`: Your Metabase API key.

2.  **Session Token / Google SSO:**
    *   `METABASE_URL`: The URL of your Metabase instance.
    *   `METABASE_SESSION_TOKEN`: A Metabase session token. Use this when your organisation enforces SSO (e.g. Google OAuth) and a standalone API key does not work. After completing the Google OAuth flow in your browser, open the browser's developer tools, go to **Application → Cookies**, and copy the value of the `metabase.SESSION` cookie. Set that value as `METABASE_SESSION_TOKEN`.
    *   `METABASE_FORWARD_AUTH` *(optional)*: If your Metabase instance sits behind a forward-auth proxy (e.g. Traefik ForwardAuth), you may also need to pass the `_forward_auth` cookie. Copy its value from the browser's developer tools (**Application → Cookies**) and set it as `METABASE_FORWARD_AUTH`. When set, this cookie is sent alongside `metabase.SESSION` on every request.

3.  **Username/Password (Fallback):**
    *   `METABASE_URL`: The URL of your Metabase instance.
    *   `METABASE_USERNAME`: Your Metabase username.
    *   `METABASE_PASSWORD`: Your Metabase password.

The server checks for credentials in the following order: `METABASE_API_KEY`, then `METABASE_SESSION_TOKEN`, then `METABASE_USERNAME`/`METABASE_PASSWORD`. You must provide credentials for at least one method.

**Example setup:**

Using API Key:
```bash
# Required environment variables
export METABASE_URL=https://your-metabase-instance.com
export METABASE_API_KEY=your_metabase_api_key
```

Using a session token (Google SSO / OAuth):
```bash
# Required environment variables
export METABASE_URL=https://your-metabase-instance.com
export METABASE_SESSION_TOKEN=your_session_token_from_browser_cookie
# Optional: required when Metabase is behind a forward-auth proxy
export METABASE_FORWARD_AUTH=your_forward_auth_cookie_value
```

Or, using Username/Password:
```bash
# Required environment variables
export METABASE_URL=https://your-metabase-instance.com
export METABASE_USERNAME=your_username
export METABASE_PASSWORD=your_password
```
You can set these environment variables in your shell profile or use a `.env` file with a package like `dotenv`.

## Development

Install dependencies:
```bash
npm install
```

Build the server:
```bash
npm run build
```

For development with auto-rebuild:
```bash
npm run watch
```

## Installation

### Option 1: Run directly from GitHub (no installation required)

You can run the server directly from this GitHub repository using `npx`. npm will
download the source, install dependencies, build it, and run it automatically:

```bash
npx -y github:martijnpieters/metabase-server
```

To use with Claude Desktop, add the server config:

On MacOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
On Windows: `%APPDATA%/Claude/claude_desktop_config.json`

```json
{
  "mcpServers": {
    "metabase-server": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "github:martijnpieters/metabase-server"],
      "env": {
        "METABASE_URL": "https://your-metabase-instance.com",
        "METABASE_API_KEY": "your_metabase_api_key"
      }
    }
  }
}
```

Note: On first run `npx` will build the TypeScript source, which takes a little
extra time. Subsequent runs reuse the cached build.

### Option 2: Install from GitHub Package Registry

The package is published to the [GitHub Package Registry](https://github.com/martijnpieters/metabase-server/pkgs/npm/metabase-server)
as `@martijnpieters/metabase-server`. To install it you need a GitHub personal
access token with at least `read:packages` scope.

Authenticate once:

```bash
npm login --registry=https://npm.pkg.github.com --scope=@martijnpieters
```

Then install globally:

```bash
npm install -g @martijnpieters/metabase-server --registry=https://npm.pkg.github.com
```

To use with Claude Desktop after a global install:

```json
{
  "mcpServers": {
    "metabase-server": {
      "command": "metabase-server",
      "env": {
        "METABASE_URL": "https://your-metabase-instance.com",
        "METABASE_API_KEY": "your_metabase_api_key"
      }
    }
  }
}
```

### Option 3: Build from source

```bash
# Clone, build, and link globally
git clone https://github.com/martijnpieters/metabase-server.git && cd metabase-server && npm i && npm run build && npm link
```

To use with Claude Desktop after linking:

```json
{
  "mcpServers": {
    "metabase-server": {
      "command": "metabase-server",
      "env": {
        "METABASE_URL": "https://your-metabase-instance.com",
        // Use API Key (preferred)
        "METABASE_API_KEY": "your_metabase_api_key"
        // Or Session Token from Google SSO
        // "METABASE_SESSION_TOKEN": "your_session_token_from_browser_cookie"
        // Or Username/Password (if neither API Key nor Session Token is set)
        // "METABASE_USERNAME": "your_username",
        // "METABASE_PASSWORD": "your_password"
      }
    }
  }
}
```

Note: You can also set these environment variables in your system instead of in the config file if you prefer.

### Installing via Claude Code CLI

To install metabase-server for Claude Code CLI (using GitHub directly):

```bash
claude mcp add metabase-server -s user \
  -e METABASE_URL="https://your-metabase-instance.com" \
  -e METABASE_USERNAME="your-email@example.com" \
  -e METABASE_PASSWORD="your-password" \
  -- npx -y github:martijnpieters/metabase-server
```

Or using API Key (preferred):

```bash
claude mcp add metabase-server -s user \
  -e METABASE_URL="https://your-metabase-instance.com" \
  -e METABASE_API_KEY="your-api-key" \
  -- npx -y github:martijnpieters/metabase-server
```

Configuration will be written to `~/.claude.json`:

```json
{
  "mcpServers": {
    "metabase-server": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "github:martijnpieters/metabase-server"],
      "env": {
        "METABASE_URL": "https://your-metabase-instance.com",
        "METABASE_USERNAME": "your-email@example.com",
        "METABASE_PASSWORD": "your-password"
      }
    }
  }
}
```

#### Troubleshooting

If MCP server fails to connect, check the logs:

```bash
cat ~/.claude/debug/latest | grep -i metabase
```

Common issues:
- Environment variables not configured (`"env": {}` is empty)
- Invalid URL format (must include `https://`)
- Incorrect credentials

### Debugging

Since MCP servers communicate over stdio, debugging can be challenging. We recommend using the [MCP Inspector](https://github.com/modelcontextprotocol/inspector), which is available as a package script:

```bash
npm run inspector
```

The Inspector will provide a URL to access debugging tools in your browser.

## Testing

After configuring the environment variables as described in the "Configuration" section, you can manually test the server's authentication. The MCP Inspector (`npm run inspector`) is a useful tool for sending requests to the server.

### 1. Testing with API Key Authentication

1.  Set the `METABASE_URL` and `METABASE_API_KEY` environment variables with your Metabase instance URL and a valid API key.
2.  Ensure `METABASE_USERNAME` and `METABASE_PASSWORD` are unset or leave them, as the API key should take precedence.
3.  Start the server: `npm run build && node build/index.js` (or use your chosen method for running the server, like via Claude Desktop config).
4.  Check the server logs. You should see a message indicating that it's using API key authentication (e.g., "Using Metabase API Key for authentication.").
5.  Using an MCP client or the MCP Inspector, try calling a tool, for example, `tools/call` with `{"name": "list_dashboards"}`.
6.  Verify that the tool call is successful and you receive the expected data.

### 2. Testing with Session Token Authentication (Google SSO / OAuth)

1.  Ensure the `METABASE_API_KEY` environment variable is unset.
2.  Complete the Google OAuth flow in your browser by navigating to your Metabase instance and signing in.
3.  Open the browser's developer tools (F12), go to **Application → Cookies**, select your Metabase instance URL, and copy the value of the `metabase.SESSION` cookie.
4.  Set `METABASE_URL` and `METABASE_SESSION_TOKEN` (the copied cookie value).
5.  If your Metabase instance is behind a forward-auth proxy, also copy the `_forward_auth` cookie and set `METABASE_FORWARD_AUTH` to that value.
6.  Start the server.
7.  Check the server logs. You should see "Using Metabase session token for authentication (e.g. obtained via Google SSO).".
8.  Using an MCP client or the MCP Inspector, try calling the `list_dashboards` tool.
9.  Verify that the tool call is successful.

### 3. Testing with Username/Password Authentication (Fallback)

1.  Ensure both `METABASE_API_KEY` and `METABASE_SESSION_TOKEN` environment variables are unset.
2.  Set `METABASE_URL`, `METABASE_USERNAME`, and `METABASE_PASSWORD` with valid credentials for your Metabase instance.
3.  Start the server.
4.  Check the server logs. You should see a message indicating that it's using username/password authentication (e.g., "Using Metabase username/password for authentication." followed by "Authenticating with Metabase using username/password...").
5.  Using an MCP client or the MCP Inspector, try calling the `list_dashboards` tool.
6.  Verify that the tool call is successful.

### 4. Testing Authentication Failures

*   **Invalid API Key:**
    1.  Set `METABASE_URL` and an invalid `METABASE_API_KEY`. Ensure `METABASE_USERNAME`, `METABASE_PASSWORD`, and `METABASE_SESSION_TOKEN` variables are unset.
    2.  Start the server.
    3.  Attempt to call a tool (e.g., `list_dashboards`). The tool call should fail, and the server logs might indicate an authentication error from Metabase (e.g., "Metabase API error: Invalid X-API-Key").
*   **Invalid Session Token:**
    1.  Ensure `METABASE_API_KEY` is unset. Set `METABASE_URL` and an invalid `METABASE_SESSION_TOKEN`.
    2.  Start the server.
    3.  Attempt to call a tool. The tool call should fail with an authentication error from Metabase.
*   **Invalid Username/Password:**
    1.  Ensure `METABASE_API_KEY` and `METABASE_SESSION_TOKEN` are unset. Set `METABASE_URL` and invalid `METABASE_USERNAME`/`METABASE_PASSWORD`.
    2.  Start the server.
    3.  Attempt to call a tool. The tool call should fail due to failed session authentication. The server logs might show "Authentication failed" or "Failed to authenticate with Metabase".
*   **Missing Credentials:**
    1.  Unset `METABASE_API_KEY`, `METABASE_SESSION_TOKEN`, `METABASE_USERNAME`, and `METABASE_PASSWORD`. Set only `METABASE_URL`.
    2.  Attempt to start the server.
    3.  The server should fail to start and log an error message stating that authentication credentials are required (e.g., "Either (METABASE_URL and METABASE_API_KEY) or (METABASE_URL and METABASE_SESSION_TOKEN) or (METABASE_URL, METABASE_USERNAME, and METABASE_PASSWORD) environment variables are required").
