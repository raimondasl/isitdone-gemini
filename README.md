# isitdone for Gemini CLI

**Gemini CLI can't say "done" until the tests actually pass.**

This extension registers [isitdone](https://github.com/raimondasl/isitdone) as a Gemini CLI hook. When Gemini tries to end its turn claiming the work is complete, the hook runs the repository's *real* test, typecheck and lint commands on the *exact* working tree and blocks the turn with the actual failure output until they pass (three attempts, then it lets Gemini stop and says why). A second hook warns right after an edit that weakened a test (`.skip`, deleted tests, downgraded assertions, `|| true` in the test script).

No API keys, no network, no telemetry, no dependencies: it is exit codes. MIT.

## Install

```
gemini extensions install https://github.com/raimondasl/isitdone-gemini
```

Restart Gemini CLI. The hooks call `npx -y @aivolution/isitdone`, so Node.js 20+ and npm are required; the first run downloads the package. Check the plumbing in any repository with:

```
npx isitdone doctor
```

It pipes a synthetic "all tests pass" stop through the hook and proves it blocks. `npx isitdone` runs the same checks by hand and prints a signed receipt bound to the working-tree hash.

## What runs

| Event | Matcher | Command | Timeout |
|---|---|---|---|
| `AfterAgent` | `*` | `npx -y @aivolution/isitdone hook --host gemini` | 600 s |
| `AfterTool` | `write_file\|replace` | `npx -y @aivolution/isitdone hook --host gemini --event edit` | 30 s |

The Stop check is claim-gated: fast checks (typecheck, lint) on every stop; the full test suite only when Gemini's message claims completion. A pass is cached per working-tree hash, so an unchanged tree never re-runs. Checks are detected from `package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, .NET, Gradle/Maven and Makefiles, and can be set in `.isitdone.json`.

The extension also ships the `isitdone` skill, which teaches the agent to run `npx isitdone` itself before claiming completion.

## Without the extension

The same registration can be written into a project or user `settings.json` with:

```
npx isitdone init --agent gemini
```

## Everything else

Documentation, the other eleven supported agents, the GitHub Action and `isitdone history` (which grades your past "done" claims from local Gemini CLI, Claude Code, Codex, Qwen Code and Cursor transcripts) live in the main repository: https://github.com/raimondasl/isitdone.

Built and maintained by Claude under human direction, as an experiment in agent-driven open source.
