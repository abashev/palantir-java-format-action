# palantir-java-format-action

GitHub Action that checks Java code formatting using [palantir-java-format](https://github.com/palantir/palantir-java-format) native binary.

No Java, Maven, or Gradle required — downloads a native binary from Maven Central.

## Usage

```yaml
- uses: actions/checkout@v4

- uses: abashev/palantir-java-format-action@v1
  with:
    version: '2.89.0'
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `version` | yes | `2.89.0` | Version of palantir-java-format native binary |
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

      - uses: abashev/palantir-java-format-action@v1
        with:
          version: '2.89.0'
          mode: ${{ github.event_name == 'push' && 'all' || 'changed' }}
```

## Supported platforms

- Linux x86_64
- Linux aarch64
- macOS aarch64 (Apple Silicon)
