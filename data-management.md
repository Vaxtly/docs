## Data Management

Export and import your data from **Settings → Data**.

### Export

Five export options:

- **Everything** — collections (with nested folders and requests) + environments + MCP servers + configuration
- **Collections** — all collections with their full folder/request tree
- **Environments** — all environments with variables (vault-synced environments export with empty variables)
- **MCP Servers** — all MCP server configurations and transport settings
- **Config** — sync and vault settings (tokens are excluded for security)

You can also export individual items by right-clicking in the sidebar and choosing "Export" — this works for both collections and MCP servers.

Exported data is saved as a JSON file named `vaxtly-export-{type}-{date}.json`.

### Export — OpenAPI

Right-click a collection in the sidebar and choose **Export as OpenAPI** to generate an [OpenAPI 3.0.3](https://spec.openapis.org/oas/v3.0.3) specification file in YAML format.

Vaxtly maps your collection data to the OpenAPI spec:

- **Collection** name and description → `info.title` and `info.description`
- **Folders** → tags (requests are tagged by their top-level parent folder)
- **Request** method and URL → `paths` and operations
- **Query params** and **headers** → `parameters`
- **Body** (JSON, XML, form-data, URL-encoded) → `requestBody` with content type and examples
- **Auth** (Bearer, Basic, API Key, OAuth 2.0) → `components.securitySchemes`
- **Common base URL** → `servers` (auto-detected from request URLs, including <code v-pre>{{variable}}</code> prefixes)
- <code v-pre>{{variable}}</code> segments in paths → OpenAPI path parameters (`{variable}`)

Sensitive credentials (tokens, passwords) are **never included** in the export — only the security scheme definitions.

Since Vaxtly doesn't store response schemas, all operations include a default `200: Successful response`. You can enhance the exported spec in any OpenAPI editor afterward.

The file is saved as `openapi-{collection}-{date}.yaml`.

### Export — Postman

Right-click a collection in the sidebar and choose **Export as Postman Collection** to generate a [Postman Collection v2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json) JSON file you can import directly into Postman.

Vaxtly maps your collection data to the Postman schema:

- **Collection** name and description → `info.name` and `info.description`
- **Folders** → nested item groups (the full hierarchy is preserved)
- **Request** method, URL, and name → request items
- **Query params** and **headers** → `url.query` and `header` (disabled entries keep their `disabled` flag)
- **Body** (JSON, XML, raw, GraphQL, form-data, URL-encoded) → `body` with the matching mode
- **Auth** (Bearer, Basic, API Key, OAuth 2.0) at the request, folder, and collection level → Postman `auth` blocks (auth set to *Inherit* is left for Postman to inherit from the parent)
- **Collection variables** → `variable`

> [!WARNING]
> Unlike the OpenAPI export, the Postman export includes auth credentials (tokens, passwords, keys) in plaintext — the same as the native Vaxtly export — so the resulting file can be used immediately. Handle and share it accordingly.

Pre-request scripts and assertions are not translated to Postman events. The file is saved as `postman-collection-{collection}-{date}.json`.

### Import — OpenAPI

Import an [OpenAPI 3.x](https://spec.openapis.org/oas/v3.0.3) or Swagger 2.x specification file (JSON or YAML) to create a new collection.

Vaxtly maps the spec to collection data:

- `info.title` and `info.description` → **Collection** name and description
- `tags` → **Folders** (undeclared tags referenced by operations are created automatically)
- `paths` and operations → **Requests** with method, URL, and name (from `summary`, `operationId`, or `METHOD /path`)
- `servers[0].url` + path → full request URL
- `parameters` (query, header) → **Query params** and **Headers**
- `requestBody` → **Body** with detected type (JSON with example values, form-data, URL-encoded, XML, raw)
- `securitySchemes` + `security` → **Auth** (Bearer, Basic, API Key, OAuth 2.0)
- OpenAPI path parameters (`{variable}`) → <code v-pre>{{variable}}</code> in request URLs

If a collection with the same name already exists, the imported collection is renamed with a numeric suffix (e.g., "My API (2)").

### Import — Vaxtly Format

Import a previously exported Vaxtly JSON file. Collections, environments, MCP servers, and config are restored into the current workspace. All export types are supported — single items (one collection or one MCP server), category exports (all collections, all MCP servers, etc.), and full "Everything" exports.

### Import — Postman

Vaxtly auto-detects and supports three Postman formats:

- **Workspace dump** — full Postman export with multiple collections
- **Collection v2.1** — single Postman collection export (detected by `info._postman_id` or `info.schema`)
- **Environment** — Postman environment export (detected by `_postman_variable_scope` or `values` array)

### Import — Insomnia

Vaxtly auto-detects the Insomnia v4 JSON export format. The following resources are imported:

- **Workspaces** → Collections
- **Request groups** → Folders (with nested folder hierarchy preserved)
- **Requests** → Requests (method, URL, headers, query params, body, auth)
- **Environments** → Environments (variables as key-value pairs)

Supported body types: JSON, XML, form-data, URL-encoded, and GraphQL. Authentication types (Bearer, Basic, API Key, OAuth 2.0) are mapped automatically.

The import summary shows how many collections, environments, folders, and requests were imported, along with any errors.

### Import — cURL

Vaxtly detects cURL commands automatically in two ways:

**Paste into URL bar** — Copy a cURL command from browser DevTools, documentation, or Stack Overflow, then paste it into the URL bar. Vaxtly detects it and populates the current request tab with the parsed method, URL, headers, query params, body, and auth.

**Clipboard detection** — When you switch to Vaxtly with a cURL command in your clipboard, a modal appears asking if you'd like to import it. Click **Import Request** to create a new draft tab pre-filled with the parsed request. The same cURL won't prompt again after you import or dismiss it.

Supported cURL flags: `-X`/`--request`, `-H`/`--header`, `-d`/`--data`/`--data-raw`, `--data-urlencode`, `-F`/`--form`, `-u`/`--user` (basic auth), `-A`/`--user-agent`, `-b`/`--cookie`, and `Authorization: Bearer/Basic` header extraction. Line continuations, single/double/ANSI-C quoting, and combined short flags (`-sSL`) are all handled.

### Drag-and-Drop Import

You can drag and drop a JSON or YAML file directly onto the Data tab. A drop zone with a dashed border appears — drop the file and it will be auto-detected and imported.

> [!TIP]
> Use the browse button or drag-and-drop to import files. The file format is detected automatically — you don't need to specify whether it's a Vaxtly, Postman, Insomnia, or OpenAPI file.
