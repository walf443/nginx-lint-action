# nginx-lint-action

GitHub Action for [nginx-lint](https://github.com/walf443/nginx-lint) — Lint nginx configuration files in your CI pipeline.

## Usage

### Basic

```yaml
- uses: walf443/nginx-lint-action@v1
  with:
    files: nginx.conf
```

### Multiple files

```yaml
- uses: walf443/nginx-lint-action@v1
  with:
    files: nginx.conf conf.d/default.conf
```

### With configuration file

```yaml
- uses: walf443/nginx-lint-action@v1
  with:
    files: nginx.conf
    config: .nginx-lint.toml
```

### Partial configuration with context

Lint a partial config file (e.g., a server block snippet):

```yaml
- uses: walf443/nginx-lint-action@v1
  with:
    files: conf.d/mysite.conf
    context: http,server
```

### JSON output

```yaml
- uses: walf443/nginx-lint-action@v1
  with:
    files: nginx.conf
    format: json
```

### Pin to a specific version

```yaml
- uses: walf443/nginx-lint-action@v1
  with:
    files: nginx.conf
    version: "0.3.0"
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `files` | Yes | — | Path to nginx configuration file(s), space-separated |
| `version` | No | `latest` | nginx-lint Docker image tag |
| `format` | No | `text` | Output format (`text` or `json`) |
| `config` | No | — | Path to `.nginx-lint.toml` configuration file |
| `context` | No | — | Parent context for partial config files (e.g., `http,server`) |
| `args` | No | — | Additional CLI arguments passed to nginx-lint |

## Full workflow examples

### Lint specific files

```yaml
name: Lint nginx config

on:
  push:
    paths:
      - "**.conf"
      - ".nginx-lint.toml"
  pull_request:
    paths:
      - "**.conf"
      - ".nginx-lint.toml"

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: walf443/nginx-lint-action@v1
        with:
          files: nginx.conf
```

### Lint only changed files

Only run nginx-lint on `.conf` files that were changed in a push or pull request:

```yaml
name: Lint nginx config

on:
  pull_request:
    paths:
      - "**.conf"
  push:
    paths:
      - "**.conf"

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Get changed conf files
        id: changed
        run: |
          if [ "${{ github.event_name }}" = "pull_request" ]; then
            FILES=$(git diff --name-only --diff-filter=ACMR \
              ${{ github.event.pull_request.base.sha }} \
              ${{ github.sha }} -- '*.conf' | tr '\n' ' ')
          else
            FILES=$(git diff --name-only --diff-filter=ACMR \
              ${{ github.event.before }} \
              ${{ github.sha }} -- '*.conf' | tr '\n' ' ')
          fi
          echo "files=$FILES" >> "$GITHUB_OUTPUT"

      - uses: walf443/nginx-lint-action@v1
        if: steps.changed.outputs.files != ''
        with:
          files: ${{ steps.changed.outputs.files }}
```

## License

[MIT](LICENSE)
