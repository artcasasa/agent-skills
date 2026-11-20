---
name: test-review
description: Run a project's test suite, then evaluate every item in a fixed test-quality checklist (run results, coverage, assertion quality, maintainability) against a set of changes, with mechanical verification that no item was skipped. Use when the user asks to review tests, check test quality, verify test coverage of a change, or run the test checklist.
metadata:
    version: 0.1.0
---
# Test Review

Runs the test suite, then evaluates a fixed checklist of test-quality items against a set of changes.
The governing requirement is that every item is actually evaluated and none is silently skipped.

A model asked to evaluate N items will emit N rows whether or not it evaluated N, so a self-reported
coverage table cannot detect the failure it exists to catch. Instead: the item list is data
(`checklist_items.json`), agents write per-item records to disk, and `verify_results.py` reconciles
the two. The parent resolves scope, runs the suite, launches agents, runs the verifier, and reports.
It never evaluates an item itself.

## Workflow

**1. Resolve the scope.** Default is the current branch: uncommitted work plus everything on the
branch not on mainline. Use a user-named scope (commit range, paths, PR) if given. If the user asks
to review the whole test suite, the scope is every test file and the production code it covers.

```shell
git status --short
git merge-base HEAD main                 # or master / the repo's mainline
git diff --stat <merge-base>
```

Record the exact diff command that reproduces the change; every agent re-runs it. Do not paste diff
contents into the prompts. If the scope is empty, stop and say so.

**2. Run the test suite.** Detect the runner from the project files and run it once from the
repository root. Use the first that applies:

| Project file | Command |
| --- | --- |
| `pyproject.toml` with `uv.lock` | `uv run pytest --durations=20 -rs` |
| `pyproject.toml`, `setup.cfg`, or `pytest.ini` | `python -m pytest --durations=20 -rs` |
| `package.json` with a `test` script | `npm test` |
| `go.mod` | `go test ./... -v` |
| `Cargo.toml` | `cargo test` |

If the project documents its own test command (README, CONTRIBUTING, justfile, Makefile), use that
instead. Add coverage only if the project already has it configured (for example `--cov` when
`pytest-cov` is installed). Do not install packages to add coverage.

Save the full output to `<scratchpad>/test-review/suite_output.txt`. If the suite cannot start
(missing runner, import error, build failure), stop and report the error. Failing tests do not stop
the review; they are evaluated by item `R-pass`.

**3. Read `checklist_items.json`.** It defines batches. Each item carries a fixed `severity` of
`critical`, `high`, `medium`, or `low`, and a `population` that the agent must enumerate. Use its
batching as-is. Do not regroup, split, or drop batches. Create `<scratchpad>/test-review/results/`.

**4. Fan out, one agent per batch.** Fill the placeholders in `agent_prompt.md` and launch one
`Agent` call per batch, all in a single message. Each agent gets only its own items (with their
populations), the scope, the diff command, the suite output path, and an absolute
`{{RESULTS_PATH}}` of `results/<BATCH_ID>.json`. Do not reduce the batch set to save tokens. An item
not sent is an item the verifier reports as unevaluated.

**5. Verify.** This gates the report.

```shell
uv run --no-project "<skill-dir>/verify_results.py" "<scratchpad>/test-review/results" --items "<skill-dir>/checklist_items.json"
```

Exit 0 only when every item has exactly one well-formed record; otherwise it names the IDs that are
missing, duplicated, unknown, or malformed. Do not report while it is failing.

On failure, `SendMessage` the owning agent naming only the affected IDs and asking it to append or
correct those records. Never write or edit a record yourself to make the verifier pass. Re-run the
verifier after each round. If an agent still cannot produce a valid record after a second attempt,
report the run as FAILED under "Unevaluated items". If Python is unavailable, stop rather than
reconciling by hand.

**6. Report.** Only after exit 0, built from the record files joined to `checklist_items.json`, not
the agents' reply text. Each failing item goes under the heading matching its `severity`. Omit a
severity heading that has no failures.

## Output Format

```
# Test Review Results

**Scope:** <what was evaluated>
**Diff command:** `<command>`
**Test command:** `<command>`
**Suite result:** <passed / failed / skipped counts, and coverage if produced>
**Items:** <N> across <M> batches

## Verification
<verbatim verify_results.py output>

## All Results
| ID | Category | Item | Severity | Population | Verdict | Evidence |
| --- | --- | --- | --- | --- | --- | --- |

## Failures
### Critical
| ID | Category | Item | Detail | Evidence |
| --- | --- | --- | --- | --- |

### High
...

### Medium
...

### Low
...

## Unevaluated items
<any 'n/a' item with the reason from its evidence, or "none". If the run failed verification, name
the IDs here and label the run FAILED.>
```

Every verdict's evidence must state what was examined. An empty population is a legitimate `n/a`.
Do not widen it to produce a finding. Do not add items, categories, or judgments beyond
`checklist_items.json`. Do not change production or test code during the review; this skill reports
only.

## Maintaining the verifier

`verify_results.py` is shared with the `code-review` skill and has its own tests. Run them after
changing it:

```shell
uv run --no-project --with typer --with pytest pytest "<skill-dir>/verify_results_test.py"
```

## Credits

The verification method, `verify_results.py`, its tests, and the agent prompt structure come from
the `code-review` skill by David Brownell (MIT). See `UPSTREAM.md` at the repository root.
