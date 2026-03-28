# tars-tools

Core formula repository for [**tars**](https://github.com/chenchunaidu/tars): JSON manifests under `Formulas/`, same role as Homebrew’s core tap for the tars CLI.

## Layout

- **`Formulas/<letter>/`** — Homebrew-style buckets: `a`–`z` by the first character of the tool name (ASCII; names starting with a digit use `0`; anything else uses `@`). Example: `Formulas/n/nxcurl.json`.
- **Flat `Formulas/*.json`** — still supported by tars for older taps; new formulas in this repo should use the letter subfolder.

Each formula is a JSON object with at least `name` and `version`, plus either a single `url` and `sha256` (64 hex chars), or a **`platforms`** map. Keys are Go-style `GOOS_GOARCH` in lowercase (for example `darwin_arm64`, `linux_amd64`, `windows_amd64`); each value is `{"url": "…", "sha256": "…"}`. Optional `default` is used when the host has no exact match. Optional fields include `description`, `homepage`, `license`, `bin`, `install`, `usage`, and `model` (agent-facing metadata). See the [tars README](https://github.com/chenchunaidu/tars/blob/main/README.md) and [`internal/formula/formula.go`](https://github.com/chenchunaidu/tars/blob/main/internal/formula/formula.go) for the full contract and `~/.tars` layout.

**Example:** [`Formulas/n/nxcurl.json`](Formulas/n/nxcurl.json) is a multi-platform manifest for [nxcurl](https://github.com/chenchunaidu/nxcurl) using per-OS GitHub release assets. Legacy single-`url` formulas still work for one archive that applies to every platform you care about.

## Use this repo as the core tap

After you publish this repository, point tars at it:

```bash
export TARS_CORE_URL='https://github.com/<you>/tars-tools.git'
```

Empty or unset `TARS_CORE_URL` falls back to the default upstream core URL defined in tars.

Then:

```bash
tars update              # refresh formula definitions
tars list --available    # list formula names from taps
tars install <name>
```

## Contributing a formula

1. Install [tars](https://github.com/chenchunaidu/tars) from a recent release or `main` (multi-platform formulas need `platforms` support).
2. Scaffold: `tars publish init mytool` — then move the file to `Formulas/m/mytool.json` (letter folder = first character of the name, per Homebrew) and fill either top-level `url` / `sha256` or a `platforms` map, plus agent-oriented fields (`usage`, `model`, etc.).
3. Hash each release artifact: `tars hash <downloaded-file>` for each `sha256` (per platform when using `platforms`).
4. Validate: `tars publish validate Formulas/m/mytool.json`.
5. Open a pull request.

## Security

- **Install time:** [tars](https://github.com/chenchunaidu/tars) refuses to install until the downloaded file’s SHA-256 matches the declared hash (top-level `sha256` or the entry for the current platform in `platforms`).
- **This repo’s CI:** Workflows run `tars publish validate` (schema + required `sha256` format) and **download each formula artifact and check the bytes hash**: for `platforms`, every listed `url` is verified; for legacy formulas, the top-level `url` is verified.
- **Still your responsibility:** Review what you ship—who publishes the binary, whether the URL is HTTPS and trustworthy, and whether the tool is safe to run. Formulas are not a sandbox; they are signed-off URLs plus hashes. For stricter policies you can add review rules (e.g. only `github.com/*/releases/*`), dependency review, or manual approval for new upstreams.

## CI

On push and pull request, workflows validate every `Formulas/**/*.json` and verify artifact checksums against the declared `sha256`.
