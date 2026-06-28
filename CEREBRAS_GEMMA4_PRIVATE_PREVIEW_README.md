# Cerebras Gemma 4 31B Private Preview

This branch adds a checked-in OpenCode configuration for testing Gemma 4 31B
against the Cerebras production API.

## Repository

- Fork: `https://github.com/ryanl-cerebras/opencode.git`
- Branch: `cerebras-gemma4-dogfood`
- Base branch: `codex/cerebras-reasoning-history-replay`

The base branch includes the Cerebras reasoning history replay fixes. This
branch adds a test configuration without changing OpenCode provider code.

## Model mapping

- OpenCode model: `cerebras-private-preview/gemma-4-31b`
- API model ID: `gemma-4-31b`
- Base URL: `https://api.cerebras.ai/v1`
- Model context limit: `131072`
- Configured input limit: `100000`
- Configured output limit: `8192`
- Private preview rate limits: `100 RPM`, `100000 TPM`
- Provider SDK: `@ai-sdk/cerebras`

The friendly OpenCode model name is intentionally separate from the API model
ID. Update only the `id` field in
`opencode.gemma4.cerebras.json` when the API model name changes.

The configured input and output limits keep a single request within the
private preview's 100000 TPM envelope. OpenCode does not proactively enforce
aggregate RPM or TPM limits across requests. If the shared quota is exhausted,
the Cerebras API returns a rate-limit response and OpenCode retries it.

## Build

```bash
cd /Users/Ryan.Loney/code/opencode-cerebras-gemma4
bun install
OPENCODE_VERSION=1.0.0-gemma4-test \
  /Users/Ryan.Loney/.cache/codex/bun-1.3.13/bun-darwin-aarch64/bun \
  ./packages/opencode/script/build.ts --single
```

The Apple Silicon binary is expected at:

```text
packages/opencode/dist/opencode-darwin-arm64/bin/opencode
```

The custom version prevents the local branch name and generated `0.0.0`
preview version from appearing in the OpenCode interface.

## Configure

Set the API key for the Cerebras production endpoint:

```bash
export CEREBRAS_API_KEY="<cerebras-api-key>"
export OPENCODE_CONFIG="$PWD/opencode.gemma4.cerebras.json"
export OPENCODE_DB=/tmp/opencode-gemma4-private-preview.db
```

The checked-in configuration reads `CEREBRAS_API_KEY`.

Use a fresh `OPENCODE_DB` path when testing changes that depend on conversation
history.

## Verify the model

```bash
./packages/opencode/dist/opencode-darwin-arm64/bin/opencode \
  models cerebras-private-preview
```

Expected model:

```text
cerebras-private-preview/gemma-4-31b
```

## Text smoke test

```bash
./packages/opencode/dist/opencode-darwin-arm64/bin/opencode run \
  --pure \
  -m cerebras-private-preview/gemma-4-31b \
  "Reply with exactly GEMMA4_OK."
```

## Tool smoke test

```bash
./packages/opencode/dist/opencode-darwin-arm64/bin/opencode run \
  --pure \
  --dangerously-skip-permissions \
  -m cerebras-private-preview/gemma-4-31b \
  "Do not modify files. Run pwd once via bash, then reply with exactly GEMMA4_TOOL_OK."
```

## Image smoke test

```bash
./packages/opencode/dist/opencode-darwin-arm64/bin/opencode run \
  --pure \
  --file /absolute/path/to/image.png \
  -m cerebras-private-preview/gemma-4-31b \
  "Describe the attached image in one sentence."
```

The `--file` option sends the image as a multimodal attachment. Mentioning a
path in the prompt alone does not attach the image.
