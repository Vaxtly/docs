## Getting Started

Vaxtly is a modern API client for crafting, testing, and managing HTTP requests. Everything is organized around **workspaces** — each workspace holds its own collections, environments, and settings. Switch between workspaces using the dropdown at the top of the sidebar.

### UI Layout

The interface is split into three main areas:

- **Sidebar** (left) — three modes: **Collections** (browse your request tree), **Environments** (manage variable sets), and **MCP** (connect to and inspect MCP servers). Toggle between them using the icons in the footer toolbar. A search bar filters the list in real time.
- **Tab bar** (top) — open requests and environments as tabs. Each request tab shows the HTTP method badge and name. An orange dot indicates unsaved changes. Drag tabs left or right to reorder them — neighboring tabs slide apart to preview the new position. Double-click empty space in the tab bar to create a new draft request. Right-click a tab for options: Pin, Close, Close Others, Close All. Middle-click to close a tab. Pinned tabs survive "Close All."
- **Editor area** (center) — the request builder on top and response viewer on the bottom (or side-by-side, depending on layout setting). A draggable divider lets you resize the split.
- **Log panel** (bottom) — a collapsible session log that shows HTTP requests, sync operations, vault events, and script activity. Click the **Logs** button to expand. HTTP log entries have a chevron — click to expand and inspect the full request and response details (headers, body, query params, timing, cookies).

### Creating Your First Request

Press <kbd>Cmd+N</kbd> (or <kbd>Ctrl+N</kbd> on Linux/Windows) to create a new **draft request**. A draft is a scratchpad — it lives in memory, doesn't belong to any collection, and doesn't appear in the sidebar. You can edit the URL, headers, body, and send it right away.

You can also double-click empty space in the tab bar to create a draft.

### Saving

Press <kbd>Cmd+S</kbd> (or <kbd>Ctrl+S</kbd> on Linux/Windows) to save. If the request is a draft, a **collection picker** will appear — choose an existing collection or create a new one. The draft is then promoted to a persisted request and appears in the sidebar.

For already-saved requests, <kbd>Cmd+S</kbd> writes changes to the database. If the collection has remote sync enabled, saving also pushes changes to Git in the background.

> [!NOTE]
> Draft requests are transient — they are lost when you close the app. Save to a collection to keep them.

> [!TIP]
> **Tip:** Use <kbd>Cmd+B</kbd> to toggle the sidebar for more screen space while working.

### Appearance

Open Settings (gear icon, top-right) → **General** → **Appearance** to adjust how Vaxtly looks:

- **Theme** — choose **Light**, **Dark**, or **System** (follows your OS preference). Theme changes apply instantly across the app, including code editors.
- **Text size** — choose **Small**, **Medium** (default), or **Large**. Each step shifts the interface font scale by one point and affects everything: sidebar, tabs, request panes, modals, settings, and the code editor.
