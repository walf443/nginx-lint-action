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
| `version` | No | `0.14.0@sha256:...` | nginx-lint Docker image tag (pinned with digest) |
| `format` | No | `github-actions` | Output format (`github-actions`, `errorformat`, or `json`) |
| `config` | No | — | Path to `.nginx-lint.toml` configuration file |
| `context` | No | — | Parent context for partial config files (e.g., `http,server`) |
| `args` | No | — | Additional CLI arguments passed to nginx-lint |
| `cache` | No | `true` | Cache compiled WASM plugins across CI runs using `actions/cache` |

## WASM plugin cache

nginx-lint compiles WASM plugins on startup and caches the compiled artifacts
on disk. This action persists that cache across CI runs with `actions/cache`,
so workflows using WASM plugins skip recompilation on warm runs.

Caching is enabled by default. Cache entries are keyed internally by plugin
bytes and compiler configuration, so a restored cache is always safe: updated
plugins recompile automatically and stale entries are just ignored.

The cache is saved even when linting fails, so the compiled plugins survive
the fix-and-rerun loop. When there is nothing to cache (no WASM plugins, or
an nginx-lint version without cache support), no cache entry is uploaded at
all, so the default costs nothing for workflows that don't use plugins.

To disable it:

```yaml
- uses: walf443/nginx-lint-action@v1
  with:
    files: nginx.conf
    cache: false
```

To manage the cache yourself (e.g., with custom keys), set `cache: false` and
cache `${{ runner.temp }}/nginx-lint-cache` with your own `actions/cache`
step — the action always mounts that directory as the nginx-lint cache root.

Note: the plugin cache requires an nginx-lint version with cache support
(newer than 0.15.0). Older versions simply ignore the cache directory, so
enabling it is harmless. If your `.nginx-lint.toml` sets `cache_dir`, it takes
precedence over the directory this action mounts and the cache will not be
persisted across runs.

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
