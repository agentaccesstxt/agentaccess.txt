# Examples

Copy the `agentaccess.txt` from the case that matches yours into the directory you want to govern, and edit the identifiers and paths. Identifiers come from the [registry](../REGISTRY.md); matching semantics are in the [spec](../SPEC.md) (§5–§6).

- **[01-block-everything](01-block-everything/agentaccess.txt)** — no agentic AI tools at all. The most common case; two lines.
- **[02-single-tool-allowed](02-single-tool-allowed/agentaccess.txt)** — one tool is welcome, everything else stays out.
- **[03-restricted-to-paths](03-restricted-to-paths/agentaccess.txt)** — a tool may work in some subdirectories only.
- **[04-monorepo-sensitive-package](04-monorepo-sensitive-package/agentaccess.txt)** — agents may work in the repo, except one sensitive subtree.
- **[05-single-files](05-single-files/agentaccess.txt)** — rules for individual files and file types: `$` anchoring, wildcards, and a per-file carve-out.

Each file governs the directory that contains it and everything beneath it; the nearest file wins, with no merging from ancestors (§4).
