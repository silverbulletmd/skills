# SilverBullet skills

One portable LLM skill for working with a SilverBullet space through ordinary file tools and the `sb` CLI.

| Skill | Use for |
| --- | --- |
| `silverbullet` | Installing and connecting `sb`, choosing local files versus remote access, safety rules, and handing over to `sb help` |

The skill is a bootstrap. Topic guidance (querying, Space Lua, Markdown conventions, mentions, building interfaces) is not duplicated here. It lives in the space's own skill pages and in the built-in copies embedded in `sb`, and `sb help` serves it, version-matched to the space you are working in.

## Installation

Install with the cross-agent [skills CLI](https://github.com/vercel-labs/skills):

```sh
npx skills add silverbulletmd/skills --skill silverbullet
```

Choose your assistant and installation scope when prompted. Add `--global` for a user-level installation. To update, run `npx skills update`.

For manual installation, download or clone this repository and copy the `silverbullet` directory into your assistant's user- or project-level skills directory. Nothing else is needed. This source tree is not installed into your assistant automatically.

## Prerequisites

The skill installs `sb` when it is missing:

```sh
curl -fsSL https://silverbullet.md/install-sb.sh | sh -s -- --edge
```

Follow any PATH instructions printed by the installer, then verify with `sb --help`. Local file-only work does not need `sb`. Remote file operations need a reachable configured space. Queries, Lua and mentions need the space's Runtime API.

If an assistant's shell runs in a sandbox or VM that cannot reach the host's loopback address, `sb` may mistake a running local space for a stopped one and launch another app process. The skill tells Codex to request outside-sandbox execution for such commands, and Cowork to use a host-side connector. A skill cannot grant itself that permission.

## How it works

1. The assistant loads the `silverbullet` skill and gets `sb` connected.
2. It runs `sb help` for the skills and page outline of the selected space, then `sb help <topic>` for the guidance it needs.

Example requests: "Update these local project notes", "Edit the roadmap in my remote space", "Find open tasks by project", "Add a command to the editor".

## Maintenance

Validate the skill's frontmatter with your skill validator, and check its command recipes against the installed CLI. Perform live workflow trials only against disposable fixtures.
