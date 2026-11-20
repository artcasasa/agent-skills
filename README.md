# agent-skills

Skills for AI coding agents (Claude Code, Cursor, Gemini CLI, GitHub Copilot, Cline), focused on code quality and software testing.

## Skills

| Skill | What it does |
| --- | --- |
| [code-review](skills/code-review/) | Evaluates every item in a fixed checklist against a set of code changes. Each item is reviewed by an agent that writes a result record to disk, and a verifier script confirms that no item was skipped before the report is produced. |
| [understand-code](skills/understand-code/) | Explores an unfamiliar codebase and produces a structured overview of its architecture, data flow, conventions, key components and entry points. |

## Installing a skill

Skills are rendered into an agent's configuration with [robotter](https://github.com/davidbrownell/robotter):

```
uvx robotter render_skill skills/code-review claude-code
```

## Credits

The `code-review` and `understand-code` skills are adapted from [David Brownell's dotfiles](https://github.com/davidbrownell/dotfiles) (MIT License). The source commit is recorded in [UPSTREAM.md](UPSTREAM.md).

## License

MIT. See [LICENSE](LICENSE).
