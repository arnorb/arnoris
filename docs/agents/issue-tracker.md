# Tracker: local files

Specs and tickets for this repo live as Markdown files under `.scratch/`, one directory per feature.

## Status and comments

- Each ticket file has a `Status:` line under its title: `ready-for-agent`, `claimed` or `done`.
- Comments and history append to the bottom of the file under `## Comments`.

## Publish a spec

Write `.scratch/<feature-slug>/spec.md`, creating the directory if needed.

## Publish tickets

One file per ticket at `.scratch/<feature-slug>/issues/<NN>-<slug>.md`, numbered from `01` in dependency order, blockers first. Never a single combined file. Each file starts:

```md
# <NN>: <Ticket title>

Status: ready-for-agent
```

The ticket body follows. Its `## Blocked by` section lists ticket numbers, such as `01, 03`, or `None`.

## Fetch a spec or ticket

Read the file at the path Arnór gives, or find it by number in the feature's `issues/` directory. A ticket is on the frontier when every ticket in its `## Blocked by` is `done`.

## Finish a ticket

Set `Status: done` and note the commit SHA under `## Comments`.

## Wayfinding operations

- **Map:** `.scratch/<effort>/map.md`.
- **Ticket:** `.scratch/<effort>/issues/NN-<slug>.md`, numbered from `01`. Three lines sit under the title: `Type:` (`research`, `prototype`, `grill`, `task`), `Status:` (`open`, `claimed`, `resolved`) and `Blocked by: NN, NN`.
- **Blocking:** a ticket is unblocked when every file it lists is `resolved`.
- **Frontier:** open, unblocked, unclaimed tickets; lowest number first.
- **Claim:** set `Status: claimed` and save before any other work.
- **Resolve:** append the answer under `## Answer`, set `Status: resolved`, then append its gist and link to the map's Decisions so far.
