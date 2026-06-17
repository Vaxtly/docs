# AI Agent Integration

Vaxtly ships with a small CLI (and matching MCP server) that lets AI coding agents — Cursor, Claude Code, Codex, Claude Desktop, or anything else that can shell out or speak MCP — mirror API endpoints into Vaxtly while you build them.

The typical use: you've just written or edited a route handler and tell your agent *"add this to Vaxtly"*. Instead of clicking around to create a collection, folder, and request by hand, the agent does it for you. Re-running on the same endpoint updates the existing entry rather than duplicating it.

## How it works

The Vaxtly app exposes a **local-only socket** (Unix domain socket on macOS/Linux, named pipe on Windows) for the CLI and MCP server to talk to. There is **no TCP listener** and nothing reachable from the network, your browser, or any other user account on your machine. Authentication is a per-launch random token written to `~/.vaxtly/cli.json` with mode `0600`.

The CLI is bundled with the app. After installation, you symlink it onto your `$PATH` once.

> [!NOTE]
> Changes your agent makes through the CLI or MCP appear in the running app right away — the affected collection, folder, request, or environment refreshes live. No restart or manual reload needed.

## Quick start

> [!NOTE]
> Vaxtly must be running for the CLI to work. If it isn't, every command exits with code `4` and a clear "Vaxtly is not running" message.

### 1. Install the CLI

After installing Vaxtly, open Vaxtly and run this **inside Vaxtly's terminal** (or anywhere the bundled binary is reachable):

```bash
vaxtly install-cli
```

This symlinks the bundled binary into `~/.local/bin/vaxtly`. Make sure that directory is on your `$PATH`. If not, the command prints the line to add to your shell's rc file.

> [!NOTE]
> On Windows, the `install-cli` step is not yet supported. You can invoke the bundled binary directly from the app's `resources/cli/index.js`.

### 2. Verify it works

```bash
vaxtly ping
```

You should see `{"pong":true,"app_version":"…"}`.

### 3. Tell your agent about it

The fastest way for an agent to learn the tool is to run:

```bash
vaxtly guide
```

That prints a long-form, agent-optimized explanation of when and how to use each command. Pipe it into your agent's context (or paste it into the chat) once, and the agent will know:

- What `external_key` is and why every call needs one
- The inspect → modify pattern (`list` → `get` → `upsert`)
- The redaction policy (secrets are always masked on read; no opt-in)
- Common patterns with worked examples
- The decision rule between `upsert env` and `upsert env-var`

For shorter help, `vaxtly --help` lists the grammar. `vaxtly <verb> --help` and `vaxtly upsert <subcommand> --help` show per-command help with concrete examples.

## CLI reference (short form)

```
vaxtly upsert collection  --external-key <k> --name <n>
vaxtly upsert folder      --collection-external-key <c> --external-key <k> --name <n>
vaxtly upsert request     --collection-external-key <c> --external-key <k> --name <n>
                          [--method GET|POST|...] [--url <u>]
                          [--header 'X-Foo: bar' ...] [--query 'k=v' ...]
                          [--body '<json>' | --body @file]
                          [--auth-type bearer --bearer-token <t> | --auth-type basic --basic-username <u> --basic-password <p> | ...]
vaxtly upsert env         --external-key <k> --name <n> [--var 'KEY=value' ...]
vaxtly upsert env-var     --env-external-key <ek> --key <k> --value <v>

vaxtly list  workspaces|collections|folders|requests|envs   [filter flags]
vaxtly get   collection|folder|request|env  --external-key <k> [parent flags]

vaxtly ping
vaxtly guide
vaxtly mcp
```

**Idempotency.** Every upsert is keyed on `--external-key`. Re-running with the same key updates the existing entity; fields you don't pass are preserved. To modify an existing entry, `get` it first to see current state, then upsert only what changes.

**Updating something you created in the UI.** Entities made by hand in Vaxtly have no `external_key` yet, so upserting with a fresh key would create a *duplicate*. Instead, adopt the existing one: run the matching `list` command to get its `id` (a UUID), then pass `--id <uuid>` alongside the `--external-key` you want to assign it. This claims the existing entity under that key — no duplicate. The `--id` flag works on `upsert collection`, `folder`, `request`, and `env`; after adopting once, just use the external_key from then on.

```sh
# "Update the collection I made in the UI" — adopt it under a stable key
vaxtly list collections                               # find its id
vaxtly upsert collection --id <uuid> --external-key acme --name "Acme API"
```

**Exit codes.** `0` ok, `1` generic, `2` validation/not-found, `3` auth failed, `4` Vaxtly not running, `5` conflict. Stable surface — you can branch on these in scripts.

## MCP setup (Claude Desktop, Cursor, etc.)

If your agent host supports MCP, point it at `vaxtly mcp`. This is generally the smoother path because the tool descriptions, JSON schemas, and `readOnlyHint` annotations come baked in — the agent picks up everything from `tools/list` without needing to read CLI help.

**Claude Desktop** (`~/Library/Application Support/Claude/claude_desktop_config.json` on macOS):

```json
{
  "mcpServers": {
    "vaxtly": {
      "command": "vaxtly",
      "args": ["mcp"]
    }
  }
}
```

Restart Claude Desktop. The 15 Vaxtly tools (`vaxtly_ping`, `vaxtly_list_*`, `vaxtly_get_*`, `vaxtly_upsert_*`) will appear in the next conversation.

**Claude Code** — register it once at user scope (available in every project) with a single command:

```sh
claude mcp add --scope user vaxtly -- vaxtly mcp
```

**Cursor** and other MCP-capable clients use the same config shape — usually under an `mcpServers` key in their config file.

## Security model

> [!WARNING]
> Secrets you write through the CLI (bearer tokens, basic auth passwords, OAuth secrets, environment variable values) are encrypted at rest with AES-256-GCM. **Reads always return them as `"<redacted>"`** — there is no flag, no opt-in, no MCP parameter to view plaintext. If your agent needs to know an existing secret, you have to paste it manually.

This is deliberate. Any value an agent holds in its context can leak via prompt injection from another tool the agent uses. Defaulting to redacted closes that hole and makes the rare "agent needs to see this secret" case a deliberate, auditable choice.

**What's redacted:**

| Field | Behavior |
|-------|----------|
| `auth.bearer_token`, `auth.basic_password`, `auth.basic_username` | Replaced with `"<redacted>"` if non-empty; empty values preserved |
| `auth.api_key_value` | Same |
| `auth.oauth2_client_secret`, `oauth2_password`, `oauth2_access_token`, `oauth2_refresh_token` | Same |
| Every `value` field on environment variables | Replaced with `"<redacted>"`; keys (`API_KEY`, `BASE_URL`, etc.) remain visible so the agent can use <code v-pre>{{var}}</code> references |

`list.*` shapes are slimmed so they never carry these fields at all — `list.envs` returns no variables, `list.requests` returns no auth.

**Transport security:**

- Unix socket / named pipe, not TCP. Nothing reachable from the network or a browser.
- Per-launch 32-byte token, file mode `0600`. Rotated on every Vaxtly start; stale tokens are rejected with JSON-RPC error `-32001`.
- The token lives in the request envelope, not in `params` — so a method whose params happen to include an `auth` field can't shadow the transport credential.

## The two ways to update environment variables

This is the sharpest edge in the tool, so it's worth calling out:

**`vaxtly upsert env --var KEY=value`** — pass a complete set of variables. The variables array is **replaced**, not merged. Use this when creating a new env or populating from a known source (like a `.env` file).

**`vaxtly upsert env-var --env-external-key <ek> --key K --value V`** — add or update a **single** variable inside an existing env. The other variables are untouched.

The reason `upsert env-var` exists: because reads always redact values, your agent can't see the existing values in an env. If it tried to "preserve" them by re-passing them on `upsert env`, it'd write `<redacted>` as the literal value. `upsert env-var` sidesteps that — the agent passes only the one variable it's changing.

**Decision rule:** "set up a new env" or "import N vars from a file" → `upsert env --var ...`. "Add, change, or rotate ONE var" → `upsert env-var`.

## When something goes wrong

- **Exit 4 / "Vaxtly is not running"** — start the app and try again.
- **Exit 3 / "Authentication failed"** — Vaxtly restarted since the last call. The CLI reads the fresh token from `~/.vaxtly/cli.json` automatically; nothing to do but retry.
- **Exit 2 / "...not found"** — the external_key you passed doesn't exist in the parent scope. `vaxtly list collections` (or `list folders`, `list envs`) shows what's available.

For deeper troubleshooting, the dotfile at `~/.vaxtly/cli.json` shows the current socket path, token, PID, and app version. Inspect it (it's plain JSON) when debugging connectivity issues.
