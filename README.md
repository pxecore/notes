# System Architecture

pxecore-notes is a client-side, zero-build publishing engine. It pulls Markdown files directly from this GitHub repository and renders them as an interactive research notebook with instant page loads, offline caching, and a connected topic graph.

Here is how the pipeline works:

### 1. Zero-Latency Loading & Smart Sync
The app avoids loading spinners by immediately rendering posts from `localStorage` (or bundled fallbacks). In the background, it checks for repo updates using a single recursive Git tree call:

GET https://api.github.com/repos/pxecore/notes/git/trees/main?recursive=1

To avoid burning the unauthenticated GitHub rate limit (60 req/hour), requests send an `If-None-Match` header with the last saved ETag. If nothing changed, GitHub returns `304 Not Modified`, which costs zero API quota.

### 2. Rate-Limit Immune Content Fetching
When updates exist, the engine does not fetch file contents through the REST API. Instead, it streams raw markdown concurrently from GitHub's raw CDN (`raw.githubusercontent.com`), which has no rate limits:

https://raw.githubusercontent.com/pxecore/notes/main/posts/...

### 3. Automatic Folder-Based Taxonomies
You can organize notes flat (`posts/note.md`) or inside subfolders (`posts/kernel/page-tables.md`):
- If a post's frontmatter doesn't declare a `category`, the engine derives it from the parent folder (e.g. `kernel/` -> `KERNEL`).
- Slugs are automatically prefixed with the folder name (`kernel--page-tables`) to prevent URL collisions.
- If a note omits a `date`, the system falls back to cached commit timestamps or repository history.

### 4. Zero-Dependency Parser & Defensive Sanitization
Instead of heavy third-party bundles, the engine runs a lightweight in-house parser. Before touching the DOM, content passes through defensive sanitization pipelines:
- Strips `<script>`, `<iframe>`, `<object>`, `<embed>`, `<form>`, and inline `on*` event handlers.
- Strictly validates link and image URIs, dropping dangerous schemes (`javascript:`, `data:`, `vbscript:`).
- Automatically parses frontmatter and extracts leading quotes as clean post summaries without body repetition.

### 5. Knowledge Graph & Tag Indexing
The engine extracts inline `#hashtags` (filtering out syntax keywords like `include`, `define`, `struct`) and frontmatter tags to build an in-memory index. This feeds real-time search and renders an interactive, physics-based SVG node graph showing how research topics interconnect.

### 6. Workflow
Publishing takes one step: commit a `.md` file to the `posts/` folder. The site auto-discovers it, parses it, and renders it live on the next visit—no build steps, static site generators, or server setups required.
