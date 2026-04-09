# awesome-claude-skills

> The missing skill store for Claude.

A curated collection of `.skill` files that give Claude new superpowers — install in seconds, no code required.

---

## What are skills?

Skills are instruction files that teach Claude how to do specific things: interact with APIs, automate workflows, generate files, and more. Drop a `.skill` file into Claude and it instantly knows how to do something new.

---

## Skills

| Skill | What it does | Install |
|---|---|---|
| [github](./github/) | Create repos, push code, open PRs, manage issues, trigger CI — all from Claude | [Download](./github/github.skill) |

> More skills coming. PRs welcome.

---

## How to install a skill

1. Download the `.skill` file
2. In Claude, go to **Settings → Skills**
3. Click **Install skill** and select the file
4. Done — Claude now has that capability

---

## Contributing

Have a skill that should be here? Open a PR.

**Requirements:**
- Must have a `SKILL.md` with name + description frontmatter
- Must work reliably and do something genuinely useful
- One skill per PR
- Include a packaged `.skill` file alongside the folder

See [CONTRIBUTING.md](./CONTRIBUTING.md) for the full guide.

---

## License

MIT
