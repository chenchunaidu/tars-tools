# tars-tools

Core formula repository for [**tars**](https://github.com/chenchunaidu/tars): JSON manifests under `Formulas/`, same role as Homebrew’s core tap for the tars CLI.

## Layout

- **`Formulas/*.json`** — one file per tool. The basename should match the formula `name` (tars resolves `toolname` or `toolname.json` case-insensitively per file).

Each formula is a JSON object with at least `name`, `version`, `url`, and `sha256` (64 hex chars). Optional fields include `description`, `homepage`, `license`, `bin`, `install`, `usage`, and `model` (agent-facing metadata). See the [tars README](https://github.com/chenchunaidu/tars/blob/main/README.md) for the full command list and `~/.tars` layout.

## Use this repo as the core tap

After you publish this repository, point tars at it:

```bash
export NXTOOLS_CORE_URL='https://github.com/<you>/tars-tools.git'
```

Empty or unset `NXTOOLS_CORE_URL` falls back to the default upstream core URL defined in tars.

Then:

```bash
tars update              # refresh formula definitions
tars list --available    # list formula names from taps
tars install <name>
```

## Contributing a formula

1. Install [tars](https://github.com/chenchunaidu/tars) and build the `tars` binary.
2. Scaffold: `tars publish init mytool` — then move the file to `Formulas/mytool.json` and fill `url`, `sha256`, and agent-oriented fields (`usage`, `model`, etc.).
3. Hash a release artifact: `tars hash <downloaded-file>` for the `sha256` field.
4. Validate: `tars publish validate Formulas/mytool.json`.
5. Open a pull request.

CI on this repo runs `tars publish validate` on every `Formulas/*.json` when present.
