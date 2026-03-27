# tars-tools

Core formula repository for [**tars**](https://github.com/chenchunaidu/tars): JSON manifests under `Formulas/`, same role as Homebrew’s core tap for the tars CLI.

## Layout

- **`Formulas/<letter>/`** — Homebrew-style buckets: `a`–`z` by the first character of the tool name (ASCII; names starting with a digit use `0`; anything else uses `@`). Example: `Formulas/r/ripgrep.json`.
- **Flat `Formulas/*.json`** — still supported by tars for older taps; new formulas in this repo should use the letter subfolder.

Each formula is a JSON object with at least `name`, `version`, `url`, and `sha256` (64 hex chars). Optional fields include `description`, `homepage`, `license`, `bin`, `install`, `usage`, and `model` (agent-facing metadata). See the [tars README](https://github.com/chenchunaidu/tars/blob/main/README.md) for the full command list and `~/.tars` layout.

**Example:** [`Formulas/r/ripgrep.json`](Formulas/r/ripgrep.json) is a real, installable manifest for [ripgrep](https://github.com/BurntSushi/ripgrep) using the **Linux x86_64 musl** GitHub release asset. tars currently uses one `url` per formula (no per-OS matrix), so other platforms need a different asset URL and `sha256` (often a separate formula name or your own tap).

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

1. Install [tars](https://github.com/chenchunaidu/tars) and build the `tars` binary.
2. Scaffold: `tars publish init mytool` — then move the file to `Formulas/m/mytool.json` (letter folder = first character of the name, per Homebrew) and fill `url`, `sha256`, and agent-oriented fields (`usage`, `model`, etc.).
3. Hash a release artifact: `tars hash <downloaded-file>` for the `sha256` field.
4. Validate: `tars publish validate Formulas/m/mytool.json`.
5. Open a pull request.

## Security

- **Install time:** [tars](https://github.com/chenchunaidu/tars) refuses to install until the downloaded file’s SHA-256 matches the formula’s `sha256` field (integrity against tampering and download corruption).
- **This repo’s CI:** Workflows run `tars publish validate` (schema + required `sha256` format) and **download each formula’s `url` and check the bytes hash to `sha256`**, so bad URLs or mismatched checksums fail the build before merge.
- **Still your responsibility:** Review what you ship—who publishes the binary, whether the URL is HTTPS and trustworthy, and whether the tool is safe to run. Formulas are not a sandbox; they are signed-off URLs plus hashes. For stricter policies you can add review rules (e.g. only `github.com/*/releases/*`), dependency review, or manual approval for new upstreams.

## CI

On push and pull request, workflows validate every `Formulas/**/*.json` and verify artifact checksums against the declared `sha256`.
