## Environments & Variables

Environments let you define sets of variables (like `base_url`, `api_key`) and switch between them — e.g., development vs. staging vs. production. Each environment belongs to a workspace.

### Creating & Activating

Create environments from the sidebar's **Environments** panel. Activate one by clicking its name in the sidebar list, or using the **environment selector** dropdown in the tab bar. Only one environment can be active at a time per workspace. Clicking an already-active environment deactivates it.

Each variable row has a checkbox to enable or disable it individually without deleting. You can also use the **Bulk Edit** button to switch to a plain-text editor where each line is `key:value` — handy for pasting variables from other sources. Prefix a line with `#` to disable it.

### Importing .env Files

Click the **import button** (download icon) next to the **+** button in the Environments sidebar header to import a `.env` file. Vaxtly opens a file picker, parses the file, and creates a new environment with all the variables pre-populated.

The environment name is derived from the filename — `.env.production` becomes "production", `.env.staging` becomes "staging", and a plain `.env` becomes "dotenv".

Supported `.env` syntax:

- `KEY=VALUE` — basic key-value pairs
- `KEY="value with spaces"` — double-quoted values (supports `\n`, `\t`, `\\`, `\"` escape sequences)
- `KEY='literal value'` — single-quoted values (no escape processing)
- `export KEY=VALUE` — optional `export` prefix is stripped
- `# comments` and inline comments (`VALUE # comment`) are ignored
- Empty lines are skipped

### Variable Syntax

Use <code v-pre>{{variableName}}</code> anywhere in your request — URL, headers, query params, body, and auth fields. Variable names can contain letters, numbers, underscores, hyphens, and dots. Variables are resolved at send time from the active environment.

Variables are highlighted inline wherever they appear: **green** for resolved variables, **red** for unresolved ones. Hover to see the resolved value and its source (e.g., "Env: Production" or "Collection").

### Parent / Child Environments

Environments can inherit from a parent. A common shape is one root env that holds shared values, with child envs (e.g. `local`, `prod`) that override only what's specific to them.

```
Myapp           ← root: holds shared values
├── local      ← inherits from Myapp, overrides apiBase
└── prod       ← inherits from Myapp, overrides apiBase + token
```

Pick a parent in the env editor's **"Inherits from"** dropdown. The sidebar shows children indented under their parent. Both root and child envs are independently selectable as the active environment.

**How values resolve.** When a child is active, Vaxtly merges the parent's enabled variables first, then the child's — so child entries override the parent on a per-key basis, while parent-only keys still resolve. Disabled child entries are ignored (the parent value applies); to suppress an inherited key, override it with an empty string instead.

**Override action.** A child's editor shows an **"Inherited from {Parent}"** table listing every parent variable, its source, and an **Override** button that copies the row into the child's own variables for editing.

**Used by.** Root envs show their children as **"Used by"** pills at the top of the editor; click one to jump straight into that child's tab.

**Limits.** Inheritance is one level deep — a child cannot itself become a parent. Parent and child must live in the same workspace. Deleting a parent **orphans its children** (they remain as their own root envs with their own values intact).

**Vault-synced parents.** Activating a child whose parent is vault-synced fetches both the parent's and the child's secrets so inherited values resolve correctly. Per-env failures are surfaced individually so you can tell which side of the chain failed to load.

### Resolution Order

When the same variable name exists in multiple places, the highest-priority source wins:

1. **Collection variables** (set by post-response scripts or manually) — highest priority
2. **Child environment variables** — when a child is active
3. **Parent environment variables** — inherited base layer

### Nested References

Variables can reference other variables: if `base_url` is <code v-pre>{{protocol}}://{{host}}</code>, Vaxtly resolves the chain automatically. The maximum nesting depth is 10 iterations to prevent infinite loops. Unresolved variables are left as literal <code v-pre>{{varName}}</code> in the output.

### Encryption

Environment variable values are encrypted at rest using AES-256-CBC. The `enc:` prefix in the database indicates encrypted values — this is handled transparently. You never see the prefix in the UI.

> [!TIP]
> **Tip:** For vault-synced environments, variable values are never stored in the local database at all — they're held in memory only and fetched from the vault provider on each app launch.
