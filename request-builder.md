## Request Builder

The request builder is the main area for composing HTTP requests. It has six sub-tabs below the URL bar: **Params**, **Headers**, **Body**, **Auth**, **Scripts**, and **Tests**.

### URL Bar

At the top is the URL input with a **method selector** combobox. Preset methods: GET, POST, PUT, PATCH, DELETE, QUERY, HEAD, OPTIONS, and LIST — each color-coded for quick identification. QUERY ([RFC 10008](https://www.rfc-editor.org/rfc/rfc10008)) is the safe, idempotent method that carries the query in the request body — use the Body tab just like you would with POST. You can also type any custom method (e.g. PURGE, LOCK, PROPFIND) directly into the selector and press <kbd>Enter</kbd>. Custom methods are uppercased automatically and shown in a neutral color.

Environment variables (<code v-pre>{{variable}}</code>) in the URL are highlighted inline — green for resolved, red for unresolved. Hover over a highlighted variable to see its value and source.

**cURL paste** — Paste a cURL command into the URL bar and it will be automatically detected and parsed. The method, URL, headers, query params, body, and auth are all populated into the current tab.

### Draft Requests

Press <kbd>Cmd+N</kbd> or double-click empty space in the tab bar to create a **draft request**. Drafts are scratchpads — they live in memory, don't belong to a collection, and don't appear in the sidebar. You can edit and send them freely.

When you press <kbd>Cmd+S</kbd> on a draft, a **collection picker** appears where you choose (or create) a collection. The draft is then promoted to a saved request.

Drafts are transient — they are lost when you restart the app.

### Sending & Cancelling

Press the **Send** button or <kbd>Cmd+Enter</kbd> to execute the request. You can also press <kbd>Enter</kbd> in the URL bar. While a request is in flight, the Send button becomes a **Cancel** button. Cancellation also triggers automatically if the request exceeds the configured timeout.

Sending a saved request implicitly writes it to the database. If the collection has sync enabled, changes are pushed to the remote in the background. Draft requests can be sent without saving — the response appears normally.

### Layout

The editor area is split between the request builder and the response viewer. You can switch the split orientation between **rows** (top/bottom) and **columns** (side-by-side) in Settings → General. The divider is draggable to resize the split (15–85%).

### Request Settings

Configurable in **Settings → General**:

- **Timeout** — 1 to 300 seconds (default: 30)
- **Follow Redirects** — on by default
- **Verify SSL** — on by default; disable for self-signed certificates
- **Layout** — rows or columns

For proxy and certificate configuration, see [Network Settings](/network-settings).
