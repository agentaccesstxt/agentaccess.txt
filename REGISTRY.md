# Agent identifier registry

Canonical identifiers for agentic AI tools, as used in `Agent:` lines (spec [§6.1](SPEC.md#61-agent-identifiers)). Identifiers are lowercase `vendor/product` tokens, matched case-insensitively as whole tokens.

**Status meanings:**

- **proposed** — suggested by this project; the vendor has not confirmed it.
- **confirmed** — the vendor has adopted the identifier as the canonical name for their agent. This is a claim about the name, not about implementation: whether and how the tool honors the convention is documented, where it exists, via the Reference column.

No entry is currently vendor-confirmed. To register or confirm an identifier, see [CONTRIBUTING.md](CONTRIBUTING.md).

| Identifier | Vendor | Product | Status | Reference |
|---|---|---|---|---|
| `anthropic/claude-code` | Anthropic | Claude Code | proposed | https://claude.com/product/claude-code |
| `openai/codex` | OpenAI | Codex | proposed | https://openai.com/codex |
| `google/gemini-cli` | Google | Gemini CLI | proposed | https://github.com/google-gemini/gemini-cli |
| `github/copilot` | GitHub | Copilot | proposed | https://github.com/features/copilot |
| `cursor/cursor` | Cursor | Cursor | proposed | https://cursor.com |
| `jetbrains/junie` | JetBrains | Junie | proposed | https://www.jetbrains.com/junie/ |
| `amazon/q-developer` | Amazon | Q Developer | proposed | https://aws.amazon.com/q/developer/ |
| `windsurf/windsurf` | Cognition | Windsurf | proposed | https://devin.ai/desktop |

Agents match only their own identifier and `*` (spec §6.1). Identifiers name the *agent product*, not the underlying model: `anthropic/claude-code` refers to the Claude Code tool, regardless of which model powers it.
