### System Architecture: `pxecore-research-notes`

The system utilizes a hybrid architecture that combines zero-latency loading with live GitHub synchronization. The operational flow consists of five core phases:

**1. Dual-Layer Data Sourcing**
The application prevents loading states by instantly rendering a static fallback layer (`data/posts.ts`). Concurrently, a background process queries the GitHub API (`pxecore/notes/posts`) to retrieve the current directory state of `.md` files.

**2. Local Caching Mechanism**
To minimize API requests, data is cached in `localStorage` (`pxecore_blog_cache`). The system compares modification timestamps; if the repository version is newer or the file is un-cached, it downloads the raw `.md` file and updates the interface in real-time.

**3. Parsing and Sanitization**
The system extracts metadata (title, date, category, description) from the Markdown frontmatter. The body text is compiled into HTML via `marked.js` and strictly sanitized using DOMPurify (`sanitizeHTML`) to prevent XSS vulnerabilities before rendering.

**4. Data Indexing and Graph Mapping**
Inline hashtags (e.g., `#kernel`) are extracted programmatically. This indexing supports real-time search filtering across metadata and powers the `GraphView` component, which maps interconnected posts into an interactive node graph based on shared tags.

**5. Local Telemetry**
Read metrics are tracked via `localStorage` (`pxecore_blog_views`). The system increments view counts upon access, calculates the read-share percentage, and dynamically plots a sparkline trend graph based on this data.

**Workflow Summary**
Deploying content requires only committing a new `.md` file to the designated GitHub repository. The application automatically detects, fetches, processes, and displays the new post without requiring a rebuild or manual deployment.
