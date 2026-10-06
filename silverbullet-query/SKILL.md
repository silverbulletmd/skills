---
name: silverbullet-query
description: Use when answering questions about indexed SilverBullet pages, tasks, tags, relations, custom attributes, or SLIQ queries, including optional SilverBullet+ semantic search.
---

# Query SilverBullet

Before querying a local desktop space, determine whether the command environment can reach the host's loopback runtime without invoking `sb` as a probe. A blocked sandbox or VM can make `sb` repeatedly launch the app even when it is already open. In Codex, request outside-sandbox execution for each command that contacts the local space (`exec_command` with `sandbox_permissions: "require_escalated"`) or use an explicitly configured loopback permission. `sb --help` and `sb space ls` do not contact that server. In Cowork, use an available host-side connector; `127.0.0.1` inside its VM refers to the VM. If neither is available, do not run or retry the live query there; report that the index is unavailable from this environment.

Inspect `sb --help` and `sb space ls`; select an explicit `--space` or user-provided `--url`. Do not infer the target from an unrelated cwd or the sole configured space. For local work, confirm the connection corresponds to the folder; for remote work, no local copy is required. Read the selected space's operating manual with file tools or `sb fs read`.

Queries require the runtime and its index. The "Explore your space" section of `sb --help` lists calls for the space's tags (`index.tags()`), a tag's schema (`index.tagSchema`), and the SLIQ reference. Inspect a small sample of indexed objects to discover available fields, including custom attributes:

```sh
sb --space notes query 'from t = index.tasks() limit 3 select t' --json
```

Fields absent from the sample may exist on other objects; narrow samples to the relevant tag or pages.

Read [SLIQ](references/sliq.md) before constructing a query. Pass just the query body to `sb query`:

```sh
sb --space notes query 'from t = index.tasks() where not t.done order by t.page limit 20 select {page=t.page, task=t.name, ref=t.ref}' --json
```

Start with bounded results and only relevant fields. Retain page/ref information so answers can point back to source. A limit is a partial answer unless the task explicitly requests a sample; paginate or compute a count when completeness matters. Ordinary queries should use read-only expressions; SLIQ can call mutation-capable Lua functions.

For textual context, read the actual local page or `sb fs read` rather than answering solely from an index snippet. Queries are not full-file search. A pending index, unavailable runtime, or error is not evidence that no matches exist. After edits, verify a narrow expected result before relying on the index; do not indiscriminately reboot a space to refresh it.

For conceptual or related-page search, read [semantic search](references/semantic-search.md). It is an optional SilverBullet+ capability, not a requirement for SLIQ.
