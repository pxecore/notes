System Architecture: pxecore-research-notes
The system utilizes a hybrid architecture that combines zero-latency loading with live GitHub synchronization. The operational flow consists of five core phases:

1. Dual-Layer & Recursive Data Sourcing
The application prevents loading states by instantly rendering a static fallback layer (data/posts.ts). Concurrently, a background process queries the GitHub API (pxecore/notes/posts) using a two-pass traversal mechanism that retrieves both flat .md files directly in the root folder and nested files within categorized subdirectories (e.g., posts/ai/, posts/cyber/).

2. Local Caching & Rate-Limit Protection
To minimize API requests and prevent GitHub rate-limit exhaustion, data is stored in localStorage (pxecore_cached_posts_v5). The engine maintains a secondary persistent registry (pxecore_commit_dates_cache_v5) to cache commit modification dates per file path. Cache structures are versioned to automatically bust and refresh outdated data schemas when architecture updates occur.

3. Parsing, Categorization, and Sanitization
The system extracts metadata (title, date, category, description) from the Markdown frontmatter. If no explicit category is declared in the frontmatter, the engine dynamically derives the category from the parent subdirectory's name (e.g., ai/ $\rightarrow$ AI). Slugs are automatically prefixed with their folder name (folder--filename) to guarantee uniqueness across subdirectories. The body text is compiled into HTML via marked.js and strictly sanitized using DOMPurify (sanitizeHTML) to prevent XSS vulnerabilities before rendering.

4. Data Indexing and Graph Mapping
Inline hashtags (e.g., #kernel) are extracted programmatically. This indexing supports real-time search filtering across metadata and powers the GraphView component, which maps interconnected posts into an interactive node graph based on shared tags.

5. Local Telemetry
Read metrics are tracked via localStorage (pxecore_blog_views). The system increments view counts upon access, calculates the read-share percentage, and dynamically plots a sparkline trend graph based on this data.

Workflow Summary
Deploying content requires simply committing a new .md file to the designated GitHub repository—either directly inside posts/ or neatly organized inside any subdirectory like posts/<topic>/. The application automatically detects subfolders, derives categories, fetches, processes, and displays the new posts in real time without requiring a rebuild or manual deployment.
