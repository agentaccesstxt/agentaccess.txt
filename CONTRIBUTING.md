# Contributing

This is a community convention in **Draft 00**: the most valuable contribution right now is critical discussion, not code.

## Discussing and changing the spec

- Issues and pull requests are both welcome. For substantive changes — grammar, semantics, new directives — an issue making the argument in prose is usually the better starting point, so the design discussion isn't squeezed into a diff review.
- The spec's [§11](SPEC.md#11-open-questions-for-discussion) lists the questions we already know are open; picking one of those up is a great starting point.
- Arguments from precedent carry weight: `robots.txt` and `AGENTS.md` solved adjacent problems, and departures from their choices should be justified, not accidental.

## Registering an agent identifier

The [registry](REGISTRY.md) records one canonical `vendor/product` identifier per agent (spec [§6.1](SPEC.md#61-agent-identifiers)).

**If you represent the vendor:** open a pull request adding or updating your row with status `confirmed`, from an account whose affiliation is verifiable (organization membership, or a link from the vendor's official documentation to the PR). Confirming an identifier is a claim about the name — you are adopting it as the canonical token for your agent — not a claim about implementation status. When your tool does support the convention, link that documentation in the Reference column.

**If you do not represent the vendor:** open a pull request with status `proposed`, following the naming pattern of existing entries. Proposed entries are best-effort guesses that the vendor may later correct.

Identifier rules: lowercase, `vendor/product`, naming the agent product rather than the underlying model, stable across versions.

## Style

- Prose in this repository is **not hard-wrapped**: one paragraph per line, no line breaks at any column. Let your editor soft-wrap.
- `.editorconfig` covers the rest (UTF-8, LF, final newline).
- [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) keywords (MUST, SHOULD, MAY) are reserved for the spec; do not use them casually in other documents.

## License

By contributing, you agree that your contributions are licensed under [CC BY 4.0](LICENSE), like the rest of this repository.
