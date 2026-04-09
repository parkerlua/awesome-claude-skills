# ⚡ Awesome Claude Skills

> A community library of SKILL.md files for Claude Code — reusable, drop-in skills that make Claude dramatically better at specific tasks.

[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## What is a SKILL.md?

Claude Code supports **skills** — markdown files that give Claude deep, task-specific instructions. Instead of re-explaining your standards every session, you drop a `SKILL.md` into your project and Claude follows it automatically.

Think of skills like plugins: each one makes Claude an expert at one specific thing.

```
your-project/
├── .claude/
│   └── skills/
│       ├── testing/SKILL.md      ← Claude writes tests your way
│       ├── code-review/SKILL.md  ← Claude reviews like a senior dev
│       └── documentation/SKILL.md
├── src/
└── ...
```

---

## Skills Library

### 🧪 Testing
| Skill | Description |
|-------|-------------|
| [testing](skills/testing/SKILL.md) | Write comprehensive tests with proper structure, coverage, and edge cases |

### 📝 Documentation
| Skill | Description |
|-------|-------------|
| [documentation](skills/documentation/SKILL.md) | Generate clear, useful docs — JSDoc, README, API references |

### 🔍 Code Review
| Skill | Description |
|-------|-------------|
| [code-review](skills/code-review/SKILL.md) | Review code like a senior engineer — bugs, performance, security, style |

### 🔧 Refactoring
| Skill | Description |
|-------|-------------|
| [refactoring](skills/refactoring/SKILL.md) | Refactor safely with clear reasoning, minimal diff, no behavior changes |

### 🌿 Git Workflow
| Skill | Description |
|-------|-------------|
| [git-workflow](skills/git-workflow/SKILL.md) | Conventional commits, clean branch names, good PR descriptions |

### 🔌 API Design
| Skill | Description |
|-------|-------------|
| [api-design](skills/api-design/SKILL.md) | Design RESTful or GraphQL APIs with consistency and best practices |

### 🐛 Debugging
| Skill | Description |
|-------|-------------|
| [debugging](skills/debugging/SKILL.md) | Systematic debugging — hypothesis-driven, minimal reproduction, root cause |

### 🗄️ Database
| Skill | Description |
|-------|-------------|
| [database](skills/database/SKILL.md) | Safe migrations, query optimization, schema design |

### ⚙️ Automation
| Skill | Description |
|-------|-------------|
| [github](skills/github/SKILL.md) | Create repos, manage PRs/issues, trigger CI, automate GitHub workflows via gh CLI or REST API |

---

## Quick Start

**1. Pick a skill from the library above**

**2. Copy it into your project:**
```bash
mkdir -p .claude/skills/<skill-name>
cp path/to/SKILL.md .claude/skills/<skill-name>/SKILL.md
```

**3. Tell Claude to use it:**
```
Read .claude/skills/testing/SKILL.md then write tests for auth.ts
```

That's it. Claude now follows the skill's guidelines for that task.

---

## Contributing

Have a skill that's made Claude noticeably better at something? Please share it.

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines. The bar is simple: **does this skill produce meaningfully better output than asking Claude without it?**

---

## Community

- 💬 [Discussions](../../discussions) — share results, ask questions, request skills
- 🐛 [Issues](../../issues) — report problems or suggest improvements
- 🌟 Star this repo if it's useful — it helps others find it

---

## License

MIT — use freely in personal and commercial projects.
