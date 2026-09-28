---
name: silverbullet-knowledge
description: Use when creating or maintaining a SilverBullet knowledge space, ingesting sources, synthesizing notes, organizing links and metadata, or reviewing stale and contradictory information.
---

# Maintain a knowledge space

For live `sb` operations against a local desktop space, determine whether the command environment can reach the host's loopback server without invoking `sb` as a probe. A blocked sandbox or VM can make `sb` repeatedly launch the app even when it is open. In Codex, request outside-sandbox execution for each command that contacts the local space (`exec_command` with `sandbox_permissions: "require_escalated"`) or use an explicitly configured loopback permission. `sb --help` and `sb space ls` do not contact that server. In Cowork, use an available host-side connector; `127.0.0.1` inside its VM refers to the VM. If neither is available, work on accessible local files where appropriate and report that live operations are unavailable. Do not run or retry runtime commands in the blocked environment.

Read the selected space's operating manual and the shared [silverbullet-markdown skill](../silverbullet-markdown/SKILL.md). For an accessible folder use file tools; for remote-only files inspect `sb space ls`, select explicit `--space` or user-provided `--url`, and use `sb fs read/edit/write` with revision-aware changes. Do not assume the working directory or sole saved connection is the intended space.

## Establish conventions

For a new space, establish purpose, location/connection, and useful page types from the request; ask only for missing decisions. Propose a small structure before imposing an ontology on existing notes. Create an entry page, a short operating manual, and a source/activity log when they serve the workflow. Match the assistant's supported manual convention; avoid two diverging manuals.

Use readable page names, wiki links, a small stable tag vocabulary, and source provenance. Preserve existing directory layout and formatting. Read the shared [silverbullet-markdown skill](../silverbullet-markdown/SKILL.md) for frontmatter and tag scope. Keep facts and explanations in durable prose; dynamic dashboards are an optional enhancement.

For a requested new local SilverBullet+ space, inspect `sb space add --help` and `sb open --help` and use the available folder registration/open commands. A newly registered folder may need opening before it has a usable server port. Check existing registrations before adding; current registration can reject duplicates. Plain local notes need no registration. Remote connection setup can be interactive and require user sign-in; do not create a local substitute when the user requested a remote space.

## Maintain the space

* Ingest: read the source, identify existing pages it affects, update them with provenance, and link related concepts. Preserve source dates and separate quoted claims from synthesis.
* Answer: inspect relevant pages and, if available, use `sb query` for structured questions. Cite page names and source material; save substantive synthesis when requested or authorized by the space's manual.
* Review: look for unsupported/stale claims, contradictions, missing links, and unnecessary duplication. Present uncertain conflicts rather than choosing a winner without evidence.

Remote `fs ls --glob` searches filenames, not content. Read relevant pages or use the index when available. A limited query is not an exhaustive audit. For remote edits use inspected revisions and reassess conflicts; for local edits re-read current content before changing it. Verify saved pages and links. Avoid broad renames or restructuring beyond the requested scope.
