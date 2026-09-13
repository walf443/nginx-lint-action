# Test plugin

`server-tokens-enabled.wasm` is the builtin `server-tokens-enabled` rule of
[nginx-lint](https://github.com/walf443/nginx-lint), built as a WASM component
from `plugins/builtin/security/server_tokens_enabled` at v0.21.0 with
`make build-plugins`. It is committed so the `test-plugins` job in CI does not
need a Rust and wasm-tools toolchain.

`../fixtures` holds that plugin's `tests/fixtures` cases, in the layout
`nginx-lint test-plugins --fixtures` expects (`<case>/error/nginx.conf` and
`<case>/expected/nginx.conf`).

Rebuild it when the plugin ABI changes:

```bash
cd nginx-lint && make build-plugins
cp plugins/builtin/security/server_tokens_enabled/target/wasm32-unknown-unknown/release/server_tokens_enabled_plugin.wasm.component.wasm \
   nginx-lint-action/test/plugins/server-tokens-enabled.wasm
```
