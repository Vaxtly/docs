## Collections & Folders

Collections group related requests together. Within a collection you can create nested **folders** to organize requests further (e.g., by resource or feature). Folders can be nested up to 10 levels deep.

### Managing Collections

Right-click in the sidebar to access the context menu:

- **Add Request** / **Add Folder** — create items inside the collection
- **Rename** — inline editing of the name
- **Settings** — open the collection settings tab to configure auth, environments, and variables
- **Enable/Disable Sync** — toggle Git remote sync for this collection
- **Push to Remote** / **Pull from Remote** — manual sync operations (when sync is enabled)
- **Run Collection** — execute all requests in the collection sequentially (see below)
- **Export** / **Export as OpenAPI** / **Export as Postman Collection** — download the collection in the Vaxtly, OpenAPI, or Postman format (see [Data Management](./data-management.md))
- **Delete** — permanently remove the collection and all its contents

### Collection Runner

Right-click a collection and select **Run Collection** to execute all requests sequentially. A modal opens showing:

- **Progress bar** — fills as requests complete
- **Results table** — each request's method, name, status code, response time, and test results (pass/fail counts from assertions)
- **Summary** — total, passed, failed, skipped, and total duration

The runner executes requests in sidebar order: root-level requests first (by sort order), then folders recursively. Pre-request scripts, post-response scripts, and assertions all run for each request. WebSocket requests are skipped.

Variables flow between requests naturally — post-response scripts from one request set variables that subsequent requests can use.

Click **Cancel** at any time to stop the run (remaining requests are marked as skipped). The modal cannot be dismissed while a run is in progress.

> [!TIP]
> The runner uses the same request execution path as manual sends, so cookies, environment variables, and auth tokens all work exactly as expected.

### Drag-and-Drop

Requests can be moved between folders and collections by dragging them. Drop a request onto a collection or folder to move it there. A blue border highlights valid drop targets. If the collection has sync enabled, both the source and target collections are automatically pushed to the remote after a move.

> [!NOTE]
> **Note:** Folders and collections themselves cannot be reordered by drag-and-drop — only requests can be dragged.

### Sync Indicator

Collections with sync enabled show a colored dot next to their name: **green** when synced, **orange** when there are local changes that haven't been pushed yet.

### Collection & Folder Settings

Right-click a collection or folder and select **Settings** to open it as a tab. The settings editor has sub-tabs:

- **Auth** — set authentication for all requests in this collection/folder (see [Authentication > Auth Inheritance](./authentication.md#auth-inheritance))
- **Environments** — choose which environments are available and set a default
- **Scripts** — configure pre-request and post-response scripts that run for every request in this collection/folder (see [Scripts > Collection & Folder-Level Scripts](./scripts.md#collection-folder-level-scripts))
- **Variables** — (collection only) define collection-level variables

Changes are saved with <kbd>Cmd+S</kbd> (or the Save button). The tab shows an unsaved indicator dot when you have pending changes.

### Default Environments

Each collection or folder can have a **default environment**. When you open a request, Vaxtly walks up the folder chain to find the nearest default: the immediate folder is checked first, then its parent, then the collection. The first one found is activated automatically.

Set a default in the collection or folder **Settings** tab under the **Environments** sub-tab.

> [!TIP]
> Folder-level defaults take precedence over collection-level defaults, so you can have a collection-wide "staging" env but override it for specific folders.
