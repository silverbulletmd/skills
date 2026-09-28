# SilverBullet skills

Portable LLM skills for working with SilverBullet through ordinary file tools and the `sb` CLI. The seven skill folders live at the root of this repository.

| Skill | Use for |
| --- | --- |
| `silverbullet-markdown` | Shared Markdown conventions, links, metadata, tasks, and embedded code |
| `silverbullet-local` | A space folder accessible to the assistant, including synced folders |
| `silverbullet-remote` | A space without an accessible local file copy, including localhost-only access |
| `silverbullet-query` | Index schemas, SLIQ, tasks/tags/relations, optional Desktop semantic search |
| `silverbullet-lua` | API discovery, Space Lua authoring, runtime verification, source inspection |
| `silverbullet-knowledge` | Knowledge-space setup, ingestion, synthesis, and maintenance |
| `silverbullet-mentions` | Indexed mention queues and authored replies |

Choose local versus remote by file access, not hostname. Markdown conventions live once in `silverbullet-markdown`; the local, remote, Lua, and knowledge skills link to it when authoring pages. Query and mentions skills provide specialized workflows.

## Installation

Install the complete set with the cross-agent [skills CLI](https://github.com/vercel-labs/skills):

```sh
npx skills add silverbulletmd/skills --skill '*'
```

Choose your assistant and installation scope when prompted. Add `--global` for a user-level installation. To update installed skills, run `npx skills update`.

For manual installation, download or clone this repository and follow the directory layout described below.

## Prerequisites and manual installation

Local Markdown editing needs file tools only. Remote file operations need an `sb` build with `fs` commands and a reachable configured space. Queries, Lua, and mentions additionally need the Runtime API. Check `sb --help` and subcommand help; `sb` is distinct from the SilverBullet Server executable. Native folder registration/opening depends on CLI build support. Semantic search is optional and specific to SilverBullet Desktop's premium features.

Mentions require a runtime with the `identity.mentions` API. The skills need no Python runner or custom Lua scripts.

Local desktop runtime commands connect to a loopback server. If an assistant's shell runs in a sandbox or VM that cannot reach the host's loopback address, `sb` may mistake the running space for a stopped one and try to launch another app process. The skills instruct Codex to request outside-sandbox execution for local `sb` commands or use a permission profile that explicitly allows `127.0.0.1`; a skill cannot grant itself that permission. Cowork local sessions need an available host-side connector for such commands when their shell VM cannot reach the host. Cowork cloud sessions cannot use the host's loopback server directly. Remote HTTPS spaces remain usable when the assistant's network policy permits them. Do not repeatedly invoke `sb` from a blocked environment.

Copy all seven `silverbullet-*` directories, including their references, into the same assistant user- or project-level skills directory. Keep their directory names so the relative links to `silverbullet-markdown/SKILL.md` resolve. For a selective installation, include `silverbullet-markdown` alongside local, remote, Lua, or knowledge skills; skill dependencies are not installed automatically. No installer, marketplace packaging, or client configuration is included. This source tree is not automatically installed into your assistant.

Examples of requests: “Update these local project notes,” “Edit the roadmap in my remote space,” “Find open tasks by project,” “Inspect the editor API and add a command,” “Ingest this source into the knowledge space,” and “Review mentions addressed to helper.”

## Maintenance and checks

Validate each skill's frontmatter using your skill validator. Verify relative reference links and command recipes against the installed CLI. Mention API and Inbox behavior are tested in SilverBullet.md alongside the implementation. Perform live workflow trials only against disposable fixtures. Current CLI limits are documented in the skills: human-readable space listings/logs, runtime-dependent queries, conditional remote writes, and no remote full-text `fs search` command.
