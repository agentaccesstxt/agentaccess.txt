# FAQ

Fair objections, answered. Sections of the [spec](SPEC.md) are referenced as §N.

## It's voluntary. What's the point?

Because right now the restriction has nowhere to live. "No AI tools in this directory" exists today only as scattered per-tool configuration — one format per tool, one copy per machine, all rotting quietly as tools update and new ones arrive. This file gives the intent one home, in the directory it protects, in a grammar any tool can read. **A restriction you declare once, where it applies, can no longer fail because you forgot a checkbox — it can only fail because a specific tool doesn't honor it, and that failure has a name attached.**

It does not shrink your job to a checkbox. You still write the policy and keep it truthful, still review changes to it the way you'd review `CODEOWNERS` (§9), and still care what runs on your machine — the [registry](REGISTRY.md) tells you which tools *claim* conformance. What the convention removes is the replication: the copy-per-tool-per-machine part of the job, which is exactly the part that was failing silently.

Will vendors actually honor it? They have better reasons than web crawlers have for honoring `robots.txt` — a precedent that is [visibly eroding](https://blog.cloudflare.com/perplexity-is-using-stealth-undeclared-crawlers-to-evade-website-no-crawl-directives/). A crawler's paying customer is not the site it scrapes, but a coding agent's paying customer **is** the person who wrote the restriction file. **Honoring `agentaccess.txt` is not a courtesy to a stranger; it is a product feature for the customer.** The alignment is strongest where the file's author and the tool's user coincide — your own notes, your organization's repositories. Where the two diverge, as with a contractor's tool inside a client's restricted repository, the vendor's paying customer may even want the restriction lifted; there the file offers declared intent and an audit trail rather than aligned incentives — which is exactly why an override must be a deliberate user action that names the restriction (§7).

The hard limit stays hard: this is a **cooperative signal** — it communicates intent to tools and people alike, and it binds only those who cooperate. A tool may not conform, a person may change its content, and no text file outranks a hostile process. During partial adoption this file gives you leverage over the conforming majority, not assurance over everything on your machine. If your threat model needs enforcement, use OS access controls, sandboxing, or managed policy — they always take precedence (§2), and see the containers question below.

## Every tool I use already has an ignore mechanism. Isn't this solved?

Per tool, partially. As a whole, no — and the whole is where restrictions die. Three structural facts hold across today's tools. None of the mechanisms travels: each speaks to exactly one tool, so every restriction must be restated in every tool's own format. None is per-agent: no existing mechanism can say "tool X may work here, tool Y may not." And coverage is weakest exactly where autonomy is highest — by the vendors' own documentation:

| Mechanism | Travels with the directory? | Per-agent? | Vendor-documented gaps |
|---|---|---|---|
| [`.cursorignore`](https://cursor.com/docs/reference/ignore-file) | with the repo, Cursor only | no | agent terminal and MCP tools "cannot block access" |
| [Copilot content exclusion](https://docs.github.com/en/copilot/how-tos/configure-content-exclusion/exclude-content-from-copilot) | no — org/repo settings | no | "do not support content exclusion": agent mode, coding agent, CLI |
| [`.geminiignore`](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/gemini-ignore.md) / [`.aiexclude`](https://docs.cloud.google.com/gemini/docs/codeassist/create-aiexclude-file) | with the repo, Gemini only | no | shell commands read ignored files ([documented workaround](https://github.com/google-gemini/gemini-cli/issues/13775)) |
| [`.aiignore` / `.noai`](https://youtrack.jetbrains.com/articles/SUPPORT-A-3359) | with the repo, JetBrains only | no | allowlisted commands skip checks; "may still be processed due to unforeseen issues" |
| [Claude Code](https://code.claude.com/docs/en/permissions) / [Amazon Q](https://github.com/aws/amazon-q-developer-cli/blob/main/docs/built-in-tools.md) deny rules | per-machine settings | no | recognized file commands only / bash not path-checked |
| `agentaccess.txt` | any directory, any conforming tool | **yes** | draft — no implementations yet |

Two further facts frame the proposal. The industry's strongest enforcement investments protect the filesystem from the agent's *writes* — kernel sandboxes confine edits and network, while what may *enter* model context is precisely the direction current tooling covers worst; **that read direction is this convention's entire concern.** And the encouraging inverse: each of these tools already enforces its own restrictions in deterministic code ([Claude Code states it outright](https://code.claude.com/docs/en/permissions): "Permission rules are enforced by Claude Code, not by the model"), so **a vendor implementing this convention wires one more policy source into a filter engine it already operates** — nothing foreign has to be built. To be clear about the price: the convention's conformance bar covers every operation including the shell (the `cat` question below), which most of these engines do not yet check even for their own ignore files. **That gap is the real implementation cost — the convention did not create it, it makes it visible and checkable — and a vendor that closes it has hardened its own mechanism in the same stroke.**

## Why not just use containers, a sandbox, or a separate OS user?

**If your threat model demands enforcement — use them.** The spec says so itself: OS access controls, sandboxes, and managed policy always take precedence (§2), and a devcontainer's isolation is kernel-enforced and fail-closed, which this file will never be. But **"just use Docker" answers a different question** than the one this file asks. **Isolation encodes a restriction in *infrastructure***: it lives in what you didn't mount and which user lacks a permission bit — invisible, per-machine, all-or-nothing (nothing per-agent), ending at the container boundary while the host IDE's ambient AI features keep reading the host filesystem. **This file is the opposite trade: *declared intent*** — visible, portable, per-agent, one file of cost — and no enforcement. The two compose rather than compete (§7): the file declares, the sandbox enforces. The precedent is exact: in 1994, server ACLs and authentication already existed and were strictly stronger than `robots.txt` — and `robots.txt` became universal anyway, because cheap, declared, portable intent turned out to be a *different product* than enforcement, not a worse version of it.

## Is this only for developers and repositories?

No — nothing in the convention assumes a repository, version control, or a developer. Discovery (§4) is plain filesystem: the file governs whatever directory contains it, whether that's a monorepo or a folder of scanned contracts. This matters because AI is no longer arriving only through coding tools: desktop apps now organize folders, summarize documents, and generate images from reference material, and their users have no ignore-file muscle memory, no dotfile conventions, and no per-tool exclusion vocabulary at all. For them the two-line disallow form (see the [examples](examples/)) is the whole convention — and because an agent matched by no group — not even `*` — gets no access (§6), even an *empty* `agentaccess.txt` already communicates the intent to restrict; the two lines just make it explicit. This is also why the file is visible plain text rather than a dotfile (§4): the person who needs to write it may never have seen a hidden file (pun intended).

## Does this protect against prompt injection?

For restricted paths: yes, provided the agent enforces the policy in tool code rather than model judgment — which the spec tells implementers to do (§9). The policy is parsed deterministically by the harness, and disallowed operations are rejected before their results can reach the model. A model steered by injected instructions — whether hidden in a [README](https://tracebit.com/blog/code-exec-deception-gemini-ai-cli-hijack), a [rules file](https://www.pillar.security/blog/new-vulnerability-in-github-copilot-and-cursor-how-hackers-can-weaponize-code-agents), or fragmented across MCP tool descriptions and results — can *attempt* a disallowed read but cannot execute it. That holds for every route the agent controls, including its shell tool: injected instructions that say to `cat` the file meet the same policy check, because conformance covers operations by any means, not just the file-reading tool (§7; see the next question). This matters because model-level refusal is empirically unreliable: fragmented-instruction attacks have raised model compliance [from 42% to 82% in testing](https://thehackernews.com/2026/08/malicious-mcp-servers-can-split.html) (GhostSplice, ASSET Research Group, 2026). [Appendix A](SPEC.md#appendix-a-threat-classes-and-degree-of-mitigation) of the spec surveys documented attack classes and states the honest degree of protection for each.

## The agent can run shell commands. Can't it just `cat` a disallowed file?

Not in a conforming agent — and this question is where conformance earns its keep. The spec restricts *operations*, not tools (§7): a conforming agent must not read disallowed content by any means under its control, and a shell command it executes is a means under its control. The question is worth asking because this is exactly where today's per-tool mechanisms are weakest, by the vendors' own accounts: Cursor documents that its [terminal and MCP tools cannot block access](https://cursor.com/docs/reference/ignore-file) to ignored files, Gemini CLI's issue tracker [recommends `cat` as the workaround](https://github.com/google-gemini/gemini-cli/issues/13775) for its own ignore file, and JetBrains documents that [allowlisted commands skip `.aiignore` checks](https://youtrack.jetbrains.com/articles/SUPPORT-A-3359). Implementations have two honest routes to conformance here: filter the file-reading commands the harness can recognize, accepting and disclosing the limits of recognition, or run shell commands under an OS sandbox that enforces the policy's paths at the kernel. What a conforming agent may not do is treat its shell as a separate authority the policy doesn't reach.

## So it stops data exfiltration?

Only of content in restricted trees. **The convention governs what may *enter* model context, not what may leave it**: an injected instruction can still make an agent exfiltrate content the policy allows it to read (§9). **Egress control is a real and separate problem, out of scope here. Do not deploy this file believing otherwise.**

## What about a malicious MCP server or extension that reads my disk directly?

A *malicious* one — out of reach, honestly. A hostile local process with your user's privileges does not ask the harness for permission, and no cooperative convention restrains it (§2, §9, [Appendix A.5](SPEC.md#a5-compromised-agent-distribution)). That boundary belongs to OS sandboxing and supply-chain hygiene. But note where the line runs: at malice, not at the kind of component. A *legitimate* MCP server or extension that reads files on behalf of a model is itself an agent under the spec's definition (§3) and can conform in its own right — valuable precisely because a harness's policy checks often stop at the MCP boundary ([Cursor documents exactly this gap](https://cursor.com/docs/reference/ignore-file)). And the far more common hostile case is covered either way: a server that merely *injects instructions* into the model still needs the agent's own tools to act, and those tool calls hit the policy.

## Isn't the file itself a prompt-injection surface?

Only if implemented wrongly, which is why the spec forbids it: the contents are policy to be parsed, never instructions, and must not be injected into model context as natural language (§9). A correct implementation gives an attacker nothing to hide instructions in — unlike agent rules files, where exactly that attack [has been demonstrated](https://www.pillar.security/blog/new-vulnerability-in-github-copilot-and-cursor-how-hackers-can-weaponize-code-agents) ([Appendix A.2](SPEC.md#a2-poisoned-agent-instruction-files)).

## Why fail-closed when no group matches?

If a directory owner wrote an `agentaccess.txt` naming specific agents and no `*` group, **the file's presence itself signals intent to restrict;** treating unnamed agents as unrestricted would reward tools for not being listed (§6). Whether this default is too surprising is an open question the draft explicitly asks for feedback on (§11).

## What happens if I write a pattern wrong?

The group fails closed, and the tool reports the error. A pattern that doesn't begin with `/` — the most likely mistake, since gitignore habits suggest writing `Disallow: *.pem` — is invalid, and a group containing an invalid rule is treated as if it were exactly `Disallow: /` (§5). This is a deliberate departure from `robots.txt`, whose parsers skip lines they cannot parse: silently dropping a rule from a crawler-politeness file costs some extra crawling, but silently dropping a rule from an access-restriction file would grant exactly the access the author meant to deny. The empty pattern remains valid and keeps its robots.txt meaning — `Disallow:` with no value restricts nothing.

The error report is itself constrained: it goes to the user, in the tool's own interface, and the malformed line never enters model context (§9). An implementation that echoed the offending line to the model would create an injection vector — a deliberately invalid "pattern" whose text is in fact instructions.

## My tool already asked my permission — why is it still blocked?

Because the two consents answer different questions. When a tool asks "may I access this path?", you authorize the *tool*; an `agentaccess.txt` rule is the *directory owner's* declaration, and only a deliberate override that names the restriction can lift it. The two layers compose restrictively — access requires both — so **a session approval or an allowlist entry never silently outranks the file**, and a conforming agent tells you when a permission you granted was narrowed this way (§7).

## Why not `.gitignore`-style globs for the paths? They're what filesystems use

Fair — and this is the openest question in the draft, so first the part that is settled. The grammar has two layers. The *group structure* (`Agent:` lines) comes from `robots.txt` ([RFC 9309](https://www.rfc-editor.org/rfc/rfc9309)) and stays: the central feature of this convention is per-agent rules, and `.gitignore` grammar has no concept of an addressee — it can say "these paths," never "these paths, for this tool." Groups also mean the file cannot be mistaken for yet another path-exclusion list (§5, §8).

The *path-matching* layer is where your instinct bites. The draft currently uses robots.txt semantics — prefix match, `*` spans `/`, `$` anchors the end — so that existing parsers transfer whole. But prefix matching inverts filesystem intuition: `Disallow: /.env` also covers `/.env.local`, and `Disallow: /notes` silently covers `/notes-public/`, where a glob reader expects each to name one thing. The catch with simply adopting gitignore globs is that their native evaluation is order-dependent — last match wins — which conflicts with this draft's order-independent longest-match rule, and a hybrid (robots.txt groups, glob patterns, longest-match precedence) first needs a definition of *specificity* for patterns that don't compare by length. That trade-off is exactly the first question the spec asks for feedback on (§11) — if you have an opinion, that discussion is where it counts.

## Why isn't this just a section in `AGENTS.md`?

Because the two files answer different questions for different audiences, at different times. `AGENTS.md` is onboarding content for agents that are *already permitted*: it is read into model context as instructions. `agentaccess.txt` is an access policy evaluated *before* anything is read — including `AGENTS.md` itself — and it must never enter model context (§9). Merging them would put the access decision inside the very content whose reading it is supposed to gate.

## Why isn't this called `agents.txt`?

It would be the obvious name — the symmetry with `robots.txt` and `AGENTS.md` is attractive. But `agents.txt` is already in use and contested on the web, most visibly by a web-scope capability spec at [agents-txt.com](https://agents-txt.com), and a repository deployed as a static site would serve its local policy at the URL where web agents expect that other grammar. `agentaccess.txt` says what the file is at the cost of one extra word (§8, naming note).

## Why a visible file and not a dotfile?

A restriction marker should be noticeable to the humans browsing the directory, not only to the tools it addresses (§4). Precedent: `robots.txt`, `security.txt` ([RFC 9116](https://www.rfc-editor.org/rfc/rfc9116)), `AGENTS.md` — declarations of intent are traditionally visible.
