# fastedge-actions

POC stand-in for `gcore-github-actions` (moves there once proven). One public
reusable workflow, **`fastedge-release.yaml`**, that turns a repo into versions
for the Gcore portal's **Deployments** tab. Builds run in the repo's own GitHub
Actions; the portal fetches the asset and deploys it — no Gcore credentials in
GitHub.

## Use it

Copy [`callers/fastedge.yaml`](callers/fastedge.yaml) to
`.github/workflows/fastedge.yaml` (the portal pre-fills this for you), keep the
triggers you want, open a PR.

| Mode | Caller trigger | Produces |
|---|---|---|
| A — Create release in GitHub *(default)* | `release: published` | assets on that Release |
| B — version tags | `push: tags: ["v*"]` | a Release for the tag + assets |
| C — every merge to `main` | `push: branches: [main]` | pre-release `main-<sha7>` + assets |
| D — PR previews *(default)* | `pull_request` | artifact `fastedge-preview-pr-<n>` (14 days); none for fork PRs |

## Contract (what the portal relies on)

- Each version is a GitHub Release with **exactly one `<name>.wasm`** and
  **`<name>.wasm.sha256`** (`sha256sum` format). Assets are never replaced.
- Previews are workflow artifacts with the same two files.
- Detection: the portal treats a repo as set up when a workflow under
  `.github/workflows/` contains
  `uses: godronus-org/fastedge-actions/.github/workflows/fastedge-release.yaml@`.

## Defaults (override with inputs)

| | Rust (`Cargo.toml`) | JS / TS (`package.json`) |
|---|---|---|
| Build | `cargo build --release --target <target>` (target from `.cargo/config.toml`, else `wasm32-wasip1`) | `build` script via pnpm / yarn / npm (from the lockfile) |
| Tests | `fastedge-test/` suite when present | `test` script when present |
| Output | the one `.wasm` the build produced (`output` input otherwise) | same |

Inputs: `build-command`, `output`, `test-command`, `run-tests`.
