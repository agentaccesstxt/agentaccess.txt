# FAQ

Fair objections, answered. Sections of the [spec](SPEC.md) are referenced as §N.

## It's voluntary. What's the point?

The same point as `robots.txt`, which has worked for thirty years on the same trust model. The file cannot stop a determined actor — the spec says so in its second section, in bold (§2). What it does is make intent *declarable once* and *machine-readable everywhere*, which is the missing precondition for reputable vendors to honor it. Vendors have strong incentives to comply: enterprise customers demand it, and "our tool ignored a clearly declared restriction" is not a headline any of them wants. Environments that need enforcement should use OS access controls, sandboxing, or managed policy — which always take precedence.

## Why not `.gitignore` syntax? Everyone knows it

Because the central feature of this convention is *per-agent* rules, and `.gitignore` grammar has no concept of an addressee — it can only say "these paths," never "these paths, for this tool." The `robots.txt` grammar provides agent groups natively, transfers existing parser code, and — usefully — cannot be confused with the path-exclusion lists that already exist (§5, §8). That settles the *group structure*; whether the *path-matching* rules themselves should be gitignore-like — component matching instead of prefix matching — is genuinely open, and the first question the spec asks for feedback on (§11).

## Why isn't this just a section in `AGENTS.md`?

Because the two files answer different questions for different audiences, at different times. `AGENTS.md` is onboarding content for agents that are *already permitted*: it is read into model context as instructions. `agentaccess.txt` is an access policy evaluated *before* anything is read — including `AGENTS.md` itself — and it must never enter model context (§9). Merging them would put the access decision inside the very content whose reading it is supposed to gate.

## Why isn't this called `agents.txt`?

It would be the obvious name — the symmetry with `robots.txt` and `AGENTS.md` is attractive. But `agents.txt` is already in use and contested on the web, most visibly by a web-scope capability spec at [agents-txt.com](https://agents-txt.com), and a repository deployed as a static site would serve its local policy at the URL where web agents expect that other grammar. `agentaccess.txt` says what the file is at the cost of one extra word (§8, naming note).

## Does this protect against prompt injection?

For restricted paths: yes, provided the agent enforces the policy in tool code rather than model judgment — which the spec tells implementers to do (§9). The policy is parsed deterministically by the harness, and disallowed operations are rejected before their results can reach the model. A model steered by injected instructions — whether hidden in a [README](https://tracebit.com/blog/code-exec-deception-gemini-ai-cli-hijack), a [rules file](https://www.pillar.security/blog/new-vulnerability-in-github-copilot-and-cursor-how-hackers-can-weaponize-code-agents), or fragmented across MCP tool descriptions and results — can *attempt* a disallowed read but cannot execute it. This matters because model-level refusal is empirically unreliable: fragmented-instruction attacks have raised model compliance [from 42% to 82% in testing](https://thehackernews.com/2026/08/malicious-mcp-servers-can-split.html) (GhostSplice, ASSET Research Group, 2026). [Appendix A](SPEC.md#appendix-a-threat-classes-and-degree-of-mitigation) of the spec surveys documented attack classes and states the honest degree of protection for each.

## So it stops data exfiltration?

Only of content in restricted trees. The convention governs what may *enter* model context, not what may leave it: an injected instruction can still make an agent exfiltrate content the policy allows it to read (§9). Egress control is a real and separate problem, out of scope here. Do not deploy this file believing otherwise.

## What about a malicious MCP server or extension that reads my disk directly?

Out of reach, honestly. A malicious local process with your user's privileges does not ask the harness for permission, and no cooperative convention restrains it (§2, §9, [Appendix A.5](SPEC.md#a5-compromised-agent-distribution)). That boundary belongs to OS sandboxing and supply-chain hygiene. What the convention does cover is the far more common case where such a server merely *injects instructions* and the manipulated model then uses its own tools — those tool calls hit the policy.

## Why a visible file and not a dotfile?

A restriction marker should be noticeable to the humans browsing the directory, not only to the tools it addresses (§4). Precedent: `robots.txt`, `AGENTS.md` — declarations of intent are traditionally visible.

## Why fail-closed when no group matches?

If a directory owner wrote an `agentaccess.txt` naming specific agents and no `*` group, the file's presence itself signals intent to restrict; treating unnamed agents as unrestricted would reward tools for not being listed (§6). Whether this default is too surprising is an open question the draft explicitly asks for feedback on (§11).

## Can the user override it?

Yes, deliberately. The file expresses the *directory owner's* intent to the *user's* tool, and those can legitimately differ — a repository owner's preference should not permanently lock out the person who owns the machine. But an override must be an explicit, per-invocation user action that names the restriction — never a silent default or a remembered setting (§7).

## My tool already asked my permission — why is it still blocked?

Because the two consents answer different questions. When a tool asks "may I access this path?", you authorize the *tool*; an `agentaccess.txt` rule is the *directory owner's* declaration, and only a deliberate override that names the restriction can lift it. The two layers compose restrictively — access requires both — so a session approval or an allowlist entry never silently outranks the file, and a conforming agent tells you when a permission you granted was narrowed this way (§7).

## Isn't the file itself a prompt-injection surface?

Only if implemented wrongly, which is why the spec forbids it: the contents are policy to be parsed, never instructions, and must not be injected into model context as natural language (§9). A correct implementation gives an attacker nothing to hide instructions in — unlike agent rules files, where exactly that attack [has been demonstrated](https://www.pillar.security/blog/new-vulnerability-in-github-copilot-and-cursor-how-hackers-can-weaponize-code-agents) ([Appendix A.2](SPEC.md#a2-poisoned-agent-instruction-files)).
