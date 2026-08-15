# The `agentaccess.txt` Convention

**Status:** Draft 00 — for community discussion\
**Home:** <https://github.com/agentaccesstxt/agentaccess.txt>\
**Author:** Peter Seprus\
**License:** [CC BY 4.0](LICENSE)\
**Date:** 2026-08-12\
**Updated:** 2026-08-15

---

## Abstract

This document proposes `agentaccess.txt`, a plain-text file convention that lets a directory declare which agentic AI tools may operate on its contents, and on which paths. It is modeled on `robots.txt` ([RFC 9309](https://www.rfc-editor.org/rfc/rfc9309)): a per-agent, allow/disallow grammar that conforming tools consult *before* reading anything else in the directory tree. Like `robots.txt`, it is a cooperative signal, not an enforcement mechanism.

The name complements the established `AGENTS.md` convention: **`AGENTS.md` tells agents how to work in a project; `agentaccess.txt` tells them whether they may.**

## 1. Motivation

Developers increasingly run several agentic AI tools side by side, and not every tool is appropriate for every project. A contractor may be permitted to use one assistant on a client's repository but not others; a company may allow AI tooling in most repositories but not in a sensitive one; an individual may simply want a personal-notes directory left alone.

Today this intent can only be expressed per tool, each with its own mechanism: `.cursorignore`, `.aiexclude` / `.geminiignore`, `.aiignore`, `.codeiumignore`, JetBrains' `.noai`, tool-specific permission settings, or server-side configuration. Two problems follow:

1. **The rules don't travel.** Each tool must be configured separately, and it is easy to forget one — especially when a new tool is installed after the restriction was decided.
2. **Existing ignore files answer the wrong question.** They express *which paths* to exclude, uniformly for whichever tool reads them. None can express *which tools* are welcome — "tool X may work here, tool Y may not."

These mechanisms are also limited in ways their vendors document plainly:

- Cursor's [documentation](https://cursor.com/docs/reference/ignore-file) states that terminal commands and MCP tools "run outside of Cursor's file access controls" and that `.cursorignore` "is not a security boundary."
- GitHub documents that [Copilot content exclusion](https://docs.github.com/en/copilot/concepts/context/content-exclusion) is "currently not supported in Edit and Agent modes."
- JetBrains [records](https://www.jetbrains.com/help/ai-assistant/disable-ai-assistant.html) that `.noai` "affects only the JetBrains AI Assistant plugin" and that ignored files "may still be processed."

A path excluded from indexing can therefore still be read by the same tool through a shell command. This convention places its requirement in the harness rather than the model (§9) and counts shell commands among the covered operations (§7); it remains, like the mechanisms it joins, a cooperative signal and not a sandbox (§2).

The problem is sharpest for **IDE-embedded agents**. A CLI agent is invoked deliberately, in a repository the user chose; an AI feature built into an editor activates simply by opening a folder, and may begin indexing before the user has expressed any intent at all. Editors do offer per-workspace controls (for example VS Code's `chat.disableAIFeatures` setting, which users can set globally and re-enable per workspace), but these are again tool-specific, and the burden of remembering falls on every user, for every tool, on every machine. A committed workspace setting is in effect a directory-borne restriction marker for one editor — evidence of the need this convention generalizes.

The problem is not confined to developer tooling. AI features increasingly ship in general-purpose applications that organize folders, summarize documents, and use files as reference material. Users of such applications cannot be assumed to know any per-tool exclusion mechanism, and the directories affected are personal and business documents rather than repositories. For this audience, a plain-text file in the directory itself is a more accessible form of restriction than any per-tool setting: one visible file in a folder they already manage, rather than a separate control to find and set in every application, one by one.

A single well-known file, placed once in the restricted directory, lets the restriction follow the directory rather than the tool: declared by the directory's owner, once, for every conforming agent on any machine.

Nor is the concern hypothetical. 2025–2026 produced a steady stream of documented attacks in which agentic tools were steered into reading, exfiltrating, or destroying files their users never intended to expose. Appendix A surveys these threat classes and states, for each, how much this convention would change.

## 2. Non-goals

**This is not a security boundary.** A tool that does not implement the convention, or a user who deletes the file, is unrestricted. `agentaccess.txt` occupies the same trust position as `robots.txt`: it works because reputable vendors have strong incentives to honor clearly declared intent, not because it can stop a determined actor. Environments that require enforcement should use tool-native managed policy, OS access controls, or sandboxing; those mechanisms are out of scope here and always take precedence in the restrictive direction.

This document also does not define instructions or context for agents (see `AGENTS.md`), nor within-repository content exclusion for tools that are otherwise permitted (see the emerging `.agentignore` effort, §8), nor web-origin capability declaration (see the separate web-scope `agents.txt` effort, §8).

## 3. Terminology

The key words MUST, MUST NOT, SHOULD, SHOULD NOT, and MAY are to be interpreted as described in BCP 14 ([RFC 2119](https://www.rfc-editor.org/rfc/rfc2119), [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174)) when, and only when, they appear in all capitals, as shown here.

**Agentic AI tool (agent):** software that autonomously or semi-autonomously reads, indexes, modifies, or transmits files on behalf of an AI model — coding assistants, CLI agents, IDE integrations, background indexers.

**Harness:** the deterministic software surrounding the model — the code that registers and invokes tools, applies permission checks, and assembles what the model sees (the industry's [agent harness](https://en.wikipedia.org/wiki/Agent_harness)). Where this document says a policy is enforced in the harness, it means enforced in that code, never entrusted to the model's judgment.

**Model context:** the material the harness supplies to the model as input — instructions, files, tool results. Content that never enters model context is invisible to the model.

**Covered operation:** a file or directory operation this convention governs; enumerated in §7.

**Traversable:** the narrow grant §7 defines for directories leading to allowed content — listable and nameable in model context, with covered operations on the directory itself still disallowed.

## 4. File name and discovery

The file is named **`agentaccess.txt`** (lowercase) and applies to the directory that contains it and, recursively, to everything beneath it. It is a visible file, not a dotfile: a restriction marker should be noticeable to humans browsing the directory, not only to the tools it addresses.

Before performing any covered operation (§7) under a directory, a conforming agent MUST look for `agentaccess.txt` in that directory and each ancestor directory. The **nearest** `agentaccess.txt` file governs; files in higher ancestors are not merged. If no file is found, the agent is unrestricted by this convention (status quo).

This check is a **pre-flight requirement**: the agent MUST evaluate the file before reading, indexing, or listing any other content in the governed tree. For IDE-embedded agents, opening a folder or workspace is itself a covered trigger — the check happens at open time, before any background indexing, not at the first user interaction with the AI feature. Reading `agentaccess.txt` itself is always permitted. The evaluated policy MAY be cached within a session, but the agent MUST re-verify the governing files — including checking for the appearance of a nearer `agentaccess.txt` — before each covered operation, so that a change takes effect no later than the next covered operation after it occurs; an implementation that evaluates once per session and never looks again does not conform. A modification-time check per ancestor directory suffices.

## 5. Syntax

The grammar deliberately follows `robots.txt`: the group structure, rule lines, and pattern matching defined in [RFC 9309 §2.2](https://www.rfc-editor.org/rfc/rfc9309#section-2.2) transfer unchanged, and existing parser logic for them carries over. The agent line is this convention's own: `Agent:` takes the place of `User-agent:`, and its value is an identifier as defined in §6.1 — a form the RFC's `product-token` grammar does not admit (it allows neither `/` nor digits), so a reused parser needs its agent-line value production replaced, not merely renamed.

- The file is UTF-8 ([RFC 3629](https://www.rfc-editor.org/rfc/rfc3629)), line-based. `#` begins a comment; comments and blank lines are ignored and carry no structural meaning — grouping comes from the directives alone.
- A **group** is one or more consecutive `Agent:` lines followed by zero or more rule lines, and applies to every agent it names. A group ends where the next begins: at the first `Agent:` line that follows a rule line. Blank lines do not separate groups, and consecutive `Agent:` lines with no rules between them head the same group — so a rule-less group (§6) is syntactically possible only as the last group in the file. To permit an agent everything unambiguously, write `Allow: /`.
- Rule lines are `Allow: <pattern>` or `Disallow: <pattern>`.
- Patterns are matched against the path of the target file or directory, relative to the directory containing `agentaccess.txt`, beginning with `/`. Matching follows robots.txt rules ([RFC 9309](https://www.rfc-editor.org/rfc/rfc9309)): prefix match — a pattern matches any path that begins with it — with `*` matching any sequence of characters including `/`, and `$` anchoring the end of the path. Matching operates on canonical paths — `.` and `..` segments are resolved before comparison, as are symlinks (§9) — so a pattern containing `..` can never match, and no rule can reach outside the governed tree (§4). Canonical form also includes the platform's case folding and Unicode normalization: on filesystems that treat paths case-insensitively or normalize them, pattern and path MUST both be folded and normalized under the platform's rules before comparison — folding the path alone is not enough, since on a case-preserving filesystem the on-disk form `/Secrets/` still differs byte-wise from the pattern `/secrets/` — so that a restriction cannot be bypassed by respelling the path.
- Authoring errors fail in opposite directions. An over-broad `Disallow` errs restrictive — the safe failure for a restriction file; an over-broad `Allow` errs permissive: `Allow: /docs` also matches `/docs-secret/`, and under longest-match evaluation (§6) it overrides the shorter `Disallow: /`. Authors SHOULD end `Allow` patterns with a trailing `/`, a `$` anchor, or a complete file name — and, as with any syntax, a linter is the dependable guard against this class of mistake.
- A pattern is valid only in the two forms the [RFC 9309](https://www.rfc-editor.org/rfc/rfc9309) grammar admits: beginning with `/`, or empty — an empty pattern matches nothing, so `Disallow:` with no value keeps its robots.txt meaning of restricting nothing. Any other pattern is **invalid** — the likely symptom of `.gitignore` habits, as in `Disallow: *.pem`. Here this convention deliberately departs from [RFC 9309](https://www.rfc-editor.org/rfc/rfc9309), which has parsers skip lines they cannot parse: a silently dropped `Disallow` in `robots.txt` merely costs extra crawling, but dropping one here would grant the very access the author meant to deny. A group containing an invalid rule line MUST be treated as if its rules were exactly `Disallow: /`, and the agent SHOULD surface the parse error to the user. This is distinct from the unknown-directive rule below: an unrecognized directive name is a future feature to skip; a malformed pattern on a known directive is an author error to fail closed on.
- A file that exists but cannot be read and parsed as specified — unreadable, or not valid UTF-8 — MUST be treated as if it read exactly `Disallow: /`, for every agent: presence alone declares the intent to restrict (§6), and a policy that cannot be evaluated fails closed like a malformed one.
- Unknown directives MUST be ignored (forward compatibility).

### Example

```
# agentaccess.txt — this repository is restricted.

# Default: no agentic AI tools.
Agent: *
Disallow: /

# Claude Code may work in docs and public source, nothing else.
Agent: anthropic/claude-code
Allow: /docs/
Allow: /src/public/
Disallow: /

# Gemini CLI is fully permitted.
Agent: google/gemini-cli
Allow: /
```

The two-line minimal form covers the most common case:

```
Agent: *
Disallow: /
```

## 6. Evaluation semantics

1. **Group selection.** An agent uses the group that names its identifier (§6.1). If more than one group matches it — as when one group names the full identifier and another the bare product token — the matching groups' rules MUST be combined and evaluated as a single group, as [RFC 9309 §2.2.1](https://www.rfc-editor.org/rfc/rfc9309#section-2.2.1) combines matching groups; rule selection below is order-independent, so the combination is unambiguous. If no group matches, the agent uses the `*` group. If there is no `*` group either, the agent MUST treat the entire tree as disallowed — the file's presence signals intent to restrict, so unmatched agents fail closed. This includes the degenerate case of a file with no groups at all: an empty `agentaccess.txt`, or one containing only comments, disallows the entire tree for every agent, for the same reason — presence alone declares the intent to restrict.
2. **Rule selection.** Within the selected group, the rule with the longest matching pattern — most octets, the measure of specificity [RFC 9309 §2.2.2](https://www.rfc-editor.org/rfc/rfc9309#section-2.2.2) uses — wins; the order of rules within a group is not significant. If an `Allow` and a `Disallow` match with equal specificity, `Allow` wins. If no rule matches, the path is allowed.
3. A group containing only `Agent:` lines and no rules allows everything for the named agents. Per §5, such a group can exist only at the end of the file; prefer an explicit `Allow: /`.

### 6.1 Agent identifiers

Identifiers are lowercase tokens of the form `vendor/product`: two non-empty parts joined by exactly one `/`, each part consisting of lowercase letters, digits, `-`, and `.`. They are matched case-insensitively as whole tokens — never as executable names or paths, which are platform-specific. Examples: `anthropic/claude-code`, `openai/codex`, `google/gemini-cli`, `cursor/cursor`, `jetbrains/junie`.

Each agent has exactly one canonical identifier, published by its vendor and recorded in the public [registry file](REGISTRY.md) maintained in this repository. An agent MUST match only its own identifier — in full or bare-product form (below) — and `*`.

The namespace is open in the manner of HTTP `User-Agent` product tokens ([RFC 9110 §10.1.5](https://www.rfc-editor.org/rfc/rfc9110#section-10.1.5)): any vendor can mint its own identifier without permission, and the registry records canonical spellings. Open to mint is not open to claim: under the rule above, matching another vendor's identifier is non-conformance. That is the lesson of the precedent — `User-Agent` strings decayed into compatibility masquerade because claiming others' names was tolerated — and, as everywhere in this convention (§2), the rule cannot stop a tool from lying about its identity, but it makes the lie a checkable violation of a published rule rather than accepted practice.

Authors MAY write the product part alone (`Agent: claude-code`); agents SHOULD match this against their product token when the vendor prefix is absent.

### 6.2 Worked example

Applying these rules to the example file of §5:

| Agent | Path | Winning rule | Result |
|---|---|---|---|
| `anthropic/claude-code` | `/docs/guide.md` | `Allow: /docs/` | allowed |
| `anthropic/claude-code` | `/src/public/api.ts` | `Allow: /src/public/` | allowed |
| `anthropic/claude-code` | `/docs` (the directory entry itself) | `Disallow: /` | disallowed |
| `anthropic/claude-code` | `/src/internal/keys.pem` | `Disallow: /` | disallowed |
| `google/gemini-cli` | `/src/internal/keys.pem` | `Allow: /` | allowed |
| `openai/codex` (not named in the file) | `/docs/guide.md` | `*` group: `Disallow: /` | disallowed |

The third row is the subtle one: `Allow: /docs/` is not a prefix of `/docs`, so the directory entry itself falls to `Disallow: /` while the contents beneath it are allowed. The directory nonetheless remains traversable under §7 — `/docs` with `/` appended is a prefix of the `Allow: /docs/` pattern — so its name may appear in listings even though covered operations on the entry itself are disallowed. Had the file named agents but no `*` group, the last row would be disallowed all the same — unmatched agents fail closed — and an empty file disallows every path for every agent.

Traversal (§7) shows the same policy as a listing walk. For `anthropic/claude-code`: listing `/` — disallowed as a path, yet traversable — emits `docs` and `src`, both likewise disallowed as entries yet traversable, since `/docs/` and `/src/` are prefixes of the group's `Allow` patterns; listing `/src` emits `public` but not `internal`, whose path prefixes no `Allow` pattern; listing `/docs` emits `guide.md`. The names `internal` and `keys.pem` never enter model context. These rows and listings are intended as seed cases for a shared conformance test suite (Appendix A.6).

## 7. Conformance

The **covered operations** are reading, listing, indexing, embedding, summarizing, creating, modifying, renaming, deleting, and transmitting a file or directory — by any means under the agent's control, including shell commands it executes. A conforming agent, having selected its rules per §6:

- MUST NOT perform a covered operation on a disallowed path, and MUST NOT include the file names or directory listings of disallowed paths in model context, except as the traversal rule below provides;
- MUST treat a directory as **traversable** — it may be listed, and its name may appear in listings and in model context — when its own path is allowed, or when its path, with `/` appended unless it already ends in `/`, is a prefix of an `Allow` pattern in the selected group; the root `/` is therefore traversable whenever the group contains a non-empty `Allow` pattern. Note the direction of this test: the path is the prefix and the pattern the longer string — the reverse of §6 matching — because the question is not whether a rule covers the path but whether allowed content lies beneath it. Each entry the listing emits is governed by that entry's own path. This is the only exception to the preceding rule, and it discloses nothing beyond the names of directories on the way to content the policy already allows — without it, `Allow: /docs/` under `Disallow: /` would grant access to a directory the agent could never find;
- MUST apply the pre-flight check of §4 before any of the above;
- MUST compose this policy restrictively with its own permission model: access to a path requires both to permit it. A tool-native grant — a session approval, an allowlist entry, a remembered workspace permission — does not lift a `Disallow` rule, because it authorizes the *tool* without naming the *restriction* and is therefore not the deliberate override defined below. When such a grant conflicts with this policy, the restriction prevails, and the agent SHOULD inform the user that its granted permission was narrowed by `agentaccess.txt`;
- SHOULD inform the user, once per session, that `agentaccess.txt` restricted its access, without revealing disallowed content;
- MAY offer an explicit, per-invocation user override, since the file expresses the *directory owner's* intent to the *user's* tool; an override MUST require a deliberate user action naming the restriction (never a silent default or remembered setting), and the agent SHOULD state clearly that a restriction is being overridden.

Vendors SHOULD document their identifier and their support for this convention.

## 8. Relationship to existing conventions

| Mechanism | Question it answers | Relationship |
|---|---|---|
| `.gitignore` | what is not part of the repository | unrelated; syntax inspiration only |
| `.agentignore`, `.aiignore`, `.cursorignore`, `.aiexclude`, … | *which paths* an otherwise-permitted tool must skip | complementary; applied after `agentaccess.txt` grants access |
| `.noai` (JetBrains) | whether one vendor's AI features may operate in a project at all | closest prior art: filesystem-scope and binary, but single-vendor — this convention generalizes it per-agent and per-path |
| `AGENTS.md` | how permitted agents should behave | complementary; only consulted where access is allowed |
| web-scope `agents.txt` (agents-txt.com) | what agent protocols and capabilities a *website* announces at its origin | unrelated; different domain (web origin vs. local filesystem), different grammar |
| Tool-native managed policy, OS ACLs, sandboxes | enforcement | out of scope; always take precedence in the restrictive direction |
| `robots.txt` / IETF AIPREF / `ai.txt` (Spawning) | web crawling, AI-usage, and AI-training preferences for published content | same trust model, different domain (web vs. local filesystem) |

`agentaccess.txt` deliberately reuses the `robots.txt` grammar rather than the `.gitignore` grammar so that the per-agent group structure comes for free and the file cannot be confused with a path-exclusion list. The name is deliberately distinct from both `AGENTS.md` (a different role: onboarding document vs. access policy) and the web-scope `agents.txt` (a different domain: web origins vs. local filesystems) — a repository deployed as a static site would otherwise serve its local policy at the URL where web agents expect the other spec's grammar.

> **Naming note.** `agents.txt` was this proposal's original working name, chosen for symmetry with `robots.txt` and `AGENTS.md`. It was abandoned because the name is already in use and contested on the web — most visibly the [agents-txt.com](https://agents-txt.com) specification, which despite the shared name does nearly the opposite job: it is a *discovery* file through which a website advertises what agents *can* do at its origin (payment protocols, authorization schemes, integration endpoints such as MCP servers), whereas this convention is an *access policy* through which a directory owner declares what agents *may* do in a filesystem tree. Reusing the name would have created exactly the static-site collision described above and invited perpetual confusion between two files of opposite posture — one opens doors, the other closes them. `agentaccess.txt` says what the file is at the cost of one extra word.

## 9. Security considerations

Compliance is voluntary (§2). Further notes:

- **Information disclosure:** The file's *presence and contents* are observable to any tool; do not place sensitive information in comments or patterns.
- **Sensitive-path enumeration:** Patterns can act as a map of what the owner considers sensitive — the classic `robots.txt` failure, where a disallow list doubles as a target list. The filesystem version is weaker than it first appears: unlike a public URL, this file is readable only by someone who already has access to the directory, and such a reader can already enumerate subdirectory names — the marginal disclosure is prioritization, not existence. Still, authors who want to name nothing at all SHOULD place the file *inside* the sensitive directory with a bare `Disallow: /`, rather than naming the subtree from a parent; nearest-file discovery (§4) makes the two equivalent for content access. They are not equivalent for enumeration: a parent-file pattern also governs whether the subtree's name appears in listings (§7), while the in-place file leaves the parent tree ungoverned and the directory's name visible there.
- **Policy tampering:** The file confers no protection against anyone who can modify it: write access to the directory is write access to the policy. Changes to `agentaccess.txt` are security-relevant and SHOULD receive the same review scrutiny as changes to `CODEOWNERS` or CI configuration — an attacker may loosen rules to expose restricted content (including by adding a new descendant file that, under nearest-file discovery, replaces a stricter ancestor for its subtree — §4, §11), or tighten them to hide malicious code from AI-assisted review (§11).
- **Prompt injection:** Agents MUST treat `agentaccess.txt` purely as access policy. Its contents are not instructions to the model, and MUST NOT be injected into model context as natural language, which would create a prompt-injection surface. This includes parse-error reporting (§5): the error goes to the user in the tool's own interface, and the offending line never enters model context — a deliberately malformed pattern whose text is in fact instructions would otherwise turn the error report into an injection vector.

A conforming agent SHOULD evaluate this file in deterministic tool code, outside the model: the harness parses the policy and rejects disallowed operations before their results can enter model context. Enforced this way, restrictions hold even when the model itself is manipulated — including prompt injection delivered through tool results or tool descriptions, and instructions fragmented across several such channels, which empirically defeat model-level refusals. A manipulated model can attempt a disallowed operation but cannot execute it. This robustness does not extend to compromised or non-conforming processes (a malicious local plugin or MCP server reading the filesystem directly); that boundary requires OS-level enforcement (§2). Denial messages returned to the model SHOULD be fixed strings that reveal neither content nor policy text, and paths SHOULD be canonicalized — symlinks resolved — before the policy check.

This convention governs what may enter model context, not what may leave it. It cannot prevent exfiltration of content the agent is permitted to read; egress control is a separate problem and out of scope.

## 10. Future extensions

Kept out of version 1 intentionally: operation classes (read-only vs. write vs. network), expiry dates, cryptographic attestation of compliance, and an include mechanism. The unknown-directive rule (§5) leaves room for these.

## 11. Open questions for discussion

1. Are robots.txt pattern semantics (§5) right for a *filesystem* audience? Prefix matching inverts developer intuition — under it, `/.env` matches `/.env.local` while gitignore readers expect it to name exactly one file, and `Disallow: /notes` silently covers `/notes-public/`. Gitignore-style component matching would fit developer muscle memory, but its native evaluation is order-dependent (last match wins), which conflicts with this draft's order-independent longest-match rule — and a hybrid (robots.txt groups, gitignore patterns, longest-match precedence) must first define specificity for patterns that don't compare by length.
2. Is fail-closed for unmatched agents (§6) the right default, or too surprising?
3. Under nearest-file discovery (§4), a descendant `agentaccess.txt` replaces its ancestors outright — so a new deep file containing `Allow: /` silently voids a root `Disallow` for its subtree, and a *new file* deep in a tree draws far less review attention than a diff to a root policy file (§9). Should ancestor files instead compose restrictively — by intersection, as §7 already composes this policy with tool-native permissions — so that a descendant can tighten but never loosen? And if so, does the legitimate loosening case (a deliberately shared subtree inside a restricted project) need an explicit delegation mechanism in the ancestor file?
4. Should discovery stop at a boundary (home directory, filesystem root, VCS root), or always walk to `/`?
5. Bare product tokens without vendor prefix: convenience worth the collision risk?
6. Should conforming agents detect material policy changes between sessions and re-surface the restriction notice, so that a policy the user once saw cannot be silently swapped for another (cf. the approve-then-swap pattern of Appendix A.3)?
7. May a tool explicitly tasked with security auditing disregard or override the file, given that restrictions can be abused to cloak malicious code from AI-assisted review (§9) — and if so, under what conditions?
8. Should this effort and its identifier registry live in this repository, or attach to an existing community (Agent Client Protocol, the AGENTS.md community), given the complementary roles of `AGENTS.md` and this file?

## Appendix A: Threat classes and degree of mitigation

This appendix is motivational, not normative. It surveys documented classes of attack, overreach, and error involving agentic AI tools and states, for each, how much a harness-enforced `agentaccess.txt` (§9) changes. All degrees assume a conforming, uncompromised agent; nothing here overrides §2.

### A.1 Indirect prompt injection via content the agent reads

Malicious instructions arrive inside material the agent legitimately processes — a README, an issue, a web page, a tool result — and steer it into reading and exfiltrating sensitive files. Documented examples:

- The [Gemini CLI hijack](https://tracebit.com/blog/code-exec-deception-gemini-ai-cli-hijack) (Tracebit, 2025): instructions hidden in a README combined with an allow-list parsing flaw yielded silent data exfiltration.
- The [Google Antigravity exfiltration](https://www.promptarmor.com/resources/google-antigravity-exfiltrates-data) (PromptArmor, 2025): injection hidden in a web page drove the agent to read a project's `.env` with a shell command that bypassed the tool's own `.gitignore`-based protection for that file, then exfiltrate the credentials through an allowlisted domain.
- [GhostSplice](https://thehackernews.com/2026/08/malicious-mcp-servers-can-split.html) (ASSET Research Group, 2026): instructions fragmented across MCP tool descriptions and tool results raised model compliance from 42% to 82% across eleven tested models, no fragment looking malicious on its own.

**Degree: strong, for restricted trees.** Enforcement never consults the model's judgment (§9), so it holds no matter how thoroughly the model is steered: disallowed content can be neither read nor exfiltrated, because it never enters context. Content the policy allows remains exfiltratable — egress control is out of scope (§9).

### A.2 Poisoned agent-instruction files

Instruction files the agent trusts are themselves the injection vector. Documented example: the [Rules File Backdoor](https://www.pillar.security/blog/new-vulnerability-in-github-copilot-and-cursor-how-hackers-can-weaponize-code-agents) (Pillar Security, 2025) — malicious instructions hidden in Cursor and GitHub Copilot rules files behind zero-width and bidirectional Unicode characters, invisible in review; both vendors declined to treat it as a vulnerability, placing the burden on users.

**Degree: partial, plus structural immunity.** Once the agent is steered the situation reduces to A.1: restricted trees stay unreachable. Structurally, `agentaccess.txt` cannot join this attack class itself — it is parsed as policy and never enters model context as language (§9), so there is nothing to hide instructions in.

### A.3 Malicious or compromised MCP servers and tool integrations

The tool channel itself is hostile: instructions ride in tool descriptions or results, or the agent is steered into configuration writes that escalate to code execution. Documented examples:

- [CurXecute](https://www.tenable.com/blog/faq-cve-2025-54135-cve-2025-54136-vulnerabilities-in-cursor-curxecute-mcpoison) (CVE-2025-54135, Aim Security, 2025): prompt injection made Cursor write `.cursor/mcp.json` and auto-execute the attacker's server.
- MCPoison (CVE-2025-54136, Check Point, 2025): an approved MCP configuration could be silently swapped afterward.
- GhostSplice (A.1).

**Degree: partial.** Covered operations include writes, so a policy disallowing the configuration path would have blocked a conforming agent from the CurXecute write. Model-steering variants are covered as in A.1. A malicious *local process* reading or writing disk on its own authority is beyond any cooperative convention — that boundary requires OS-level enforcement (§2, §9).

### A.4 Ambient activation and over-broad workspace access

No attacker at all: an IDE-embedded AI feature activates and begins indexing simply because a folder was opened, before the user expresses any intent (§1).

**Degree: full, for conforming editors.** The pre-flight check binds at open time, before any background indexing (§4). This is the convention's home turf.

### A.5 Compromised agent distribution

The agent itself ships hostile. Documented example: the [Amazon Q Developer VS Code extension](https://aws.amazon.com/security/security-bulletins/AWS-2025-015/) v1.84.0 (July 2025), where an attacker with an inappropriately scoped repository token committed a prompt instructing the agent to "clean a system to a near-factory state" — deleting home directories and cloud resources — into a release of an extension installed nearly a million times. A syntax error prevented execution.

**Degree: none to marginal.** A compromised tool is non-conforming by definition; no cooperative signal restrains hostile code. (In the narrow variant where honest tool code executes a poisoned *prompt*, harness enforcement would still refuse restricted trees — but the general defense here is supply-chain hygiene and OS enforcement, §2.)

### A.6 Enforcement bugs in otherwise-honest tools

The tool intends confinement but implements it wrong. Documented examples:

- The [Claude Code path-restriction bypass](https://cymulate.com/blog/cve-2025-547954-54795-claude-inverseprompt/) (CVE-2025-54794, Cymulate, 2025): prefix matching instead of canonical path comparison let `/home/user/project-secrets` pass as inside `/home/user/project`.
- The [Anthropic Filesystem MCP server "EscapeRoute" flaws](https://cymulate.com/blog/cve-2025-53109-53110-escaperoute-anthropic/) (CVE-2025-53109 and CVE-2025-53110, Cymulate, 2025): the same prefix confusion together with a symlink-resolution error.
- [Sandbox-escape flaws in the Cursor IDE](https://www.csoonline.com/article/4191923/sandbox-bypass-flaws-in-cursor-ide-highlight-prompt-injection-as-an-rce-vector.html) (CVE-2026-50548 and CVE-2026-50549, Cato Networks, 2026): reaching files outside the project directory through a working-directory override and a path-canonicalization fallback.

**Degree: none directly.** The convention inherits the implementation quality of the tools that honor it. This class is why §9 tells implementations to resolve symlinks and canonicalize paths before the policy check, and why a shared conformance test corpus is worth more to this convention than additional prose.

### A.7 Accidental destructive action by a conforming agent

No attacker and no injection: a conforming agent, acting on a benign instruction, deletes or overwrites data through its own error. Documented examples:

- The [Gemini CLI file-overwrite cascade](https://arstechnica.com/information-technology/2025/07/ai-coding-assistants-chase-phantoms-destroy-real-user-data/) (Ars Technica, 2025): a failed `mkdir` went undetected, and the subsequent `move` commands renamed each file onto the same nonexistent destination in turn, overwriting the directory's contents.
- The [Claude Code home-directory wipe](https://www.docker.com/blog/coding-agent-horror-stories-the-rm-rf-incident/) (Docker, 2025): a repository-cleanup task produced an `rm -rf` whose trailing `~/` expanded to the user's entire home directory.
- The [Claude file-cleanup deletion](https://futurism.com/artificial-intelligence/claude-wife-photos) (Futurism, 2026): a desktop-tidying task scoped to temporary files ran `rm -rf` against a directory of years of personal photos.

**Degree: partial, on covered paths only.** Evaluated in the harness and never entering model context (§9), a `Disallow` is honored however the model reasons — so the file can serve as a declarative do-not-touch marker against an agent's own errors, its plainest value for the personal directories §1 describes. Two honest limits: it protects only paths a governing file can cover, so data destroyed over a network socket (a remote production database) is out of scope (§2); and where a tool disregards a constraint it already acknowledged — a "plan-only" mode that acts anyway — the convention is no more reliable than that tool's own guardrail, which is an enforcement bug (A.6), not this class.

---

The convention is at its strongest precisely where the model is the weak link (A.1–A.4) — because it removes the model from the enforcement path — and it is honestly worth nothing where the tool itself is hostile or broken (A.5–A.6). Against an agent's own accidents (A.7) it helps partially, and only on paths a policy file can cover. That is the same asymmetry `robots.txt` has lived with for thirty years.
