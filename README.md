# agentaccess.txt

A plain-text file that declares which agentic AI tools may operate in a directory tree — and on which paths. Modeled on `robots.txt`; complementary to `AGENTS.md`.

**`AGENTS.md` tells agents how to work in a project; `agentaccess.txt` tells them whether they may.**

## The two-line version

Put this in any directory you want AI tools to leave alone:

```
Agent: *
Disallow: /
```

Every conforming agent — CLI tool, IDE integration, background indexer — checks for this file *before* reading anything else in the tree, and stays out.

## Why

Developers run several agentic AI tools side by side, and not every tool is appropriate for every directory: a client repository under NDA, a sensitive internal project, a personal-notes folder. Today that intent can only be expressed per tool — `.cursorignore`, `.aiexclude`, `.aiignore`, per-workspace IDE settings — and the rules don't travel: every restriction must be repeated in every tool's own list, on every machine. **That multiplication is where restrictions quietly die.** Forget one path in one tool's list — or install a new tool after the restriction was decided, starting with an empty list — and nothing warns you: that tool simply reads what all the others were told to leave alone. Worse, existing ignore files answer the wrong question: they say *which paths* to exclude, uniformly, but none can say *which tools* are welcome.

`agentaccess.txt` puts the restriction in the directory itself, declared once by the directory's owner, honored by every conforming agent on any machine. Per-agent rules use the familiar `robots.txt` grammar:

```
# Default: no agentic AI tools.
Agent: *
Disallow: /

# Claude Code may work in docs and public source, nothing else.
Agent: anthropic/claude-code
Allow: /docs/
Allow: /src/public/
Disallow: /
```

Rules match individual files too, not only directories: `Disallow: /*.env$` blocks every `.env` file in the tree (see the [examples](examples/)).

## What it is not

**Not a security boundary.** Like `robots.txt`, this is a cooperative signal: it works because reputable vendors have strong incentives to honor clearly declared intent, not because it can stop a determined actor. It is also not an instruction file for agents (that's [`AGENTS.md`](https://agents.md)) and not a path-exclusion list for tools that are otherwise permitted (that's `.agentignore` and friends). See the [spec](SPEC.md) §2 and §8, and the [FAQ](FAQ.md).

What it does change, security-wise, is which *class* of attack can reach restricted paths: enforced in tool code rather than model judgment, the policy holds even when the model is steered — so prompt injection hidden in a README, a rules file, or a poisoned tool result no longer suffices, and **an attacker instead needs a hostile process on the machine, a far rarer and harder position. The threat is demoted, not eliminated;** [Appendix A](SPEC.md#appendix-a-threat-classes-and-degree-of-mitigation) of the spec surveys the documented attack classes and states honestly how much each one changes.

## Documentation

- **[Specification](SPEC.md)** — Draft 00, the normative document.
- **[Examples](examples/)** — commented `agentaccess.txt` files for common cases; copy-paste and edit.
- **[FAQ](FAQ.md)** — "it's voluntary, so what's the point?" and other fair objections, answered.
- **[Registry](REGISTRY.md)** — canonical agent identifiers (`anthropic/claude-code`, `google/gemini-cli`, …).
- **[Contributing](CONTRIBUTING.md)** — how to propose spec changes and how vendors register an identifier.

This repository carries its own [`agentaccess.txt`](agentaccess.txt), naturally.

## Status

**Draft 00 — open for community discussion.** No tool is known to implement the convention yet; the [registry](REGISTRY.md) tracks vendor adoption as it happens. The spec ends with a list of [open questions](SPEC.md#11-open-questions-for-discussion); issues and discussion are welcome. Nothing here is final, including the grammar.

## License

Text of this repository, including the specification: [CC BY 4.0](LICENSE).
