# palantir-java-format-action

GitHub Action that checks Java code formatting using [palantir-java-format](https://github.com/palantir/palantir-java-format) native binary.

No Java, Maven, or Gradle required — downloads a native binary from Maven Central.

## Usage

```yaml
- uses: actions/checkout@v4

- uses: abashev/palantir-java-format-action@v2
  with:
    version: '2.96.0'
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `version` | yes | `2.96.0` | Version of palantir-java-format native binary |
| `mode` | no | `changed` | `changed` — only files from PR or push; `all` — every `.java` file in repo |

## How `mode: changed` works

- **Pull request** — gets the list of changed files from the PR via `gh pr diff`
- **Push** — gets files changed between `before` and `after` commits in the push
- **Other events** — falls back to checking all `.java` files

## Behaviour

- If all files are formatted correctly → passes ✅
- If any file has formatting issues → lists unformatted files and fails ❌

When the check fails, fix locally with:

```bash
palantir-java-format --palantir --replace <files>
```

## Excluding files

Both the action and the [pre-commit hook](#git-pre-commit-hook) read an optional
`.palantir-java-format-exclude` file from the repository root. Every non-empty, non-comment
line is a [git pathspec](https://git-scm.com/docs/gitglossary#Documentation/gitglossary.txt-aiddefpathspecapathspec)
(relative to the repo root); any Java file matching a pattern is skipped. If the file is
absent, nothing is excluded.

```gitignore
# .palantir-java-format-exclude

# Standalone jbang scripts — the formatter would rewrite their //DEPS directives into comments
samples/**

# Generated sources
**/build/generated/**
```

Notes:

- Patterns use git pathspec syntax, so both `samples/**` and `samples` match everything under `samples/`.
- Lines starting with `#` are comments; blank lines are ignored.
- This file is the **single source of exclusions** — it is shared by the action and the hook, so there is no separate action input to configure.

## Recommended setup

```yaml
on:
  pull_request:
  push:
    branches: [main]

jobs:
  format:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: abashev/palantir-java-format-action@v2
        with:
          version: '2.96.0'
          mode: ${{ github.event_name == 'push' && 'all' || 'changed' }}
```

## Git pre-commit hook

You can also use the same formatter as a local git hook to catch formatting issues before they reach CI.

Copy [`pre-commit`](pre-commit) to your project:

```bash
cp pre-commit .git/hooks/pre-commit
# or with a custom hooks directory
cp pre-commit .githooks/pre-commit
git config core.hooksPath .githooks
```

The hook automatically downloads and caches the native binary on first run, then checks only staged `.java` files on each commit. It also honours the same [`.palantir-java-format-exclude`](#excluding-files) file as the action.

## Supported platforms

- Linux x86_64
- Linux aarch64
- macOS aarch64 (Apple Silicon)
