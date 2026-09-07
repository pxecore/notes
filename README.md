markdown
# System Architecture: pxecore Research Notes
A lightweight, serverless publishing engine that combines zero-latency client rendering with live GitHub synchronization.
---
### Architecture Overview
[GitHub Repo: posts/] │ ├── (1. Single ETag Tree Query) ──> 304 Not Modified (0 API Quota Used) │ OR └── (2. Raw CDN Stream) ──────────> [Markdown Parser & DOMPurify] │ ┌────────┴────────┐ ▼ ▼ [Local Cache v6] [Interactive GraphView]

### Core Phases
#### 1. Instant Paint & ETag Smart Sync
- **Zero-Latency Loading:** Renders instantly from versioned local storage (`pxecore_cached_posts_v6`) or bundled fallbacks—eliminating layout shifts and loading spinners.
- **Quota-Free Polling:** Background sync queries the repository tree via a single recursive call (`git/trees/main?recursive=1`) using HTTP `If-None-Match` (ETag). Unmodified repos return `304 Not Modified`, consuming **zero GitHub API rate limits**.
#### 2. Rate-Limit Immune CDN Pipeline
- Content is streamed concurrently via GitHub's raw edge CDN (`raw.githubusercontent.com`), completely bypassing the restrictive 60 req/hr unauthenticated REST API ceiling.
#### 3. Automatic Taxonomy & Namespacing
- **Folder-to-Category:** Supports flat (`posts/*.md`) and nested structures (`posts/<category>/*.md`). If frontmatter lacks a category, the engine dynamically derives it from the parent folder name.
- **Collision-Proof Slugs:** Subdirectory files are automatically namespaced (`category--slug`) to prevent URL collisions.
- **Commit Date Resolution:** Automatically inherits commit timestamps if explicit publication dates are omitted.
#### 4. Defensive Parsing & Sanitization
- Frontmatter metadata is parsed in-memory; Markdown compiles to semantic HTML.
- Every node passes through strict `DOMPurify` sanitization and URI validation to guarantee immunity against stored XSS attacks.
#### 5. Dynamic Knowledge Graph
- Automatically indexes inline `#hashtags` and metadata tags to render a real-time, interactive node-link graph (`GraphView`) visualizing cross-disciplinary research connections.
#### 6. Zero-Build Workflow
- **Publishing:** Simply commit a `.md` file to the `posts/` folder on GitHub.
- **Reflection:** The client engine auto-discovers, parses, and publishes the new note in real time—no static rebuilds, webhooks, or CI/CD pipelines required.

