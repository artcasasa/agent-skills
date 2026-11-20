# agent-skills

Skills for AI coding agents (Claude Code, Cursor, Gemini CLI, GitHub Copilot, Cline), focused on code quality and software testing.

## Skills

| Skill | What it does |
| --- | --- |
| [code-review](skills/code-review/) | Evaluates every item in a fixed checklist against a set of code changes. Each item is reviewed by an agent that writes a result record to disk, and a verifier script confirms that no item was skipped before the report is produced. |
| [test-review](skills/test-review/) | Runs the project's test suite, then evaluates 17 test-quality items (run results, coverage, assertion quality, maintainability) against a change. Uses the same verified fan-out as `code-review`, so no item is skipped. |
| [understand-code](skills/understand-code/) | Explores an unfamiliar codebase and produces a structured overview of its architecture, data flow, conventions, key components and entry points. |

## Installing a skill

Skills are rendered into an agent's configuration with [robotter](https://github.com/davidbrownell/robotter):

```
uvx robotter render_skill skills/code-review claude-code
```

## Credits

The `code-review` and `understand-code` skills are copied from [David Brownell's dotfiles](https://github.com/davidbrownell/dotfiles) (MIT License). `test-review` is new, built on the `code-review` verification method and verifier. The source commit is recorded in [UPSTREAM.md](UPSTREAM.md).

## License

MIT. See [LICENSE](LICENSE).
