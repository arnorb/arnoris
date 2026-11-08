# Domain docs

How agents read this repo's domain language and decisions. The `domain-model` skill owns the formats and creates the files when a term or decision first resolves.

## Layout

- One context: `CONTEXT.md` and `docs/adr/` at the repo root.

## Before exploring, read

- `CONTEXT.md`.
- The ADRs that touch the area you are about to work in.

If a file does not exist yet, proceed silently. Do not flag its absence or suggest creating it.

## Use the glossary's terms

- Name domain concepts with the glossary's canonical English term in code, issues, test names and commits.
- Use the paired Icelandic term when quoting the client or writing Icelandic interface copy.
- Never drift to a word the glossary lists under `_Avoid_`.
- A concept missing from the glossary means you are inventing language or have found a gap. Reconsider, or note the gap for `domain-model`.

## Flag decision conflicts

If your output contradicts an ADR or a logged decision, say so rather than overriding it silently:

> _Contradicts ADR-0007 (prices copied at checkout), but worth reopening because…_
