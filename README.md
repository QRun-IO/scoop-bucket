# QRun-IO Scoop bucket

[![Tests](https://github.com/QRun-IO/scoop-bucket/actions/workflows/ci.yml/badge.svg)](https://github.com/QRun-IO/scoop-bucket/actions/workflows/ci.yml) [![Excavator](https://github.com/QRun-IO/scoop-bucket/actions/workflows/excavator.yml/badge.svg)](https://github.com/QRun-IO/scoop-bucket/actions/workflows/excavator.yml)

[Scoop](https://scoop.sh) manifests for QRun-IO command-line tools on Windows.

| App | What | Architectures |
|---|---|---|
| `qqq` | the QQQ command-line client and terminal UI (`qqq ui`), one binary, from the `cli-v*` releases of [QRun-IO/qqq-cli](https://github.com/QRun-IO/qqq-cli) | 64-bit (x64) only |

## Install

```pwsh
scoop bucket add qrun https://github.com/QRun-IO/scoop-bucket
scoop install qqq                  # or: scoop install qrun/qqq
qqq --version                      # qqq 1.0.0 (... binary with ui ...)
qqq login --server https://your-qqq-server
qqq ui                             # needs Windows Terminal or another ConPTY terminal
```

Upgrade with `scoop update` then `scoop update qqq`.

## Unsigned binary and SmartScreen

`qqq-windows-x64.exe` is not Authenticode-signed yet. Installing through Scoop is not affected: Scoop
checks the download against the sha256 in the manifest and does not mark the file as downloaded from the
internet. If you download the `.exe` from the releases page with a browser instead, Microsoft Defender
SmartScreen warns the first time you run it ("Windows protected your PC"). Choose **More info**, then
**Run anyway**, after checking the file against the release's `SHA256SUMS`
(`Get-FileHash .\qqq-windows-x64.exe -Algorithm SHA256`).

The Windows build is cross-compiled; report problems on
[QRun-IO/qqq-cli](https://github.com/QRun-IO/qqq-cli/issues).

## How the manifest is updated

`bucket/qqq.json` is generated in qqq-cli at `docs/packaging/qqq.json`. After a `cli-v<version>` GitHub
release, in a qqq-cli checkout:

```sh
gh release download cli-v<version> --repo QRun-IO/qqq-cli -p SHA256SUMS -D /tmp/qqq-<version>
node scripts/update-packaging.mjs /tmp/qqq-<version>/SHA256SUMS
cp docs/packaging/qqq.json <this repo>/bucket/qqq.json
```

The manifest also has `checkver` (the latest GitHub release of qqq-cli, tag `cli-v<version>`) and
`autoupdate` (the `.exe` URL for that tag, its hash read from the release's `SHA256SUMS`), so the
Excavator workflow below can bump it on its own once a newer release is marked latest.

## CI

GitHub Actions, as in [ScoopInstaller/BucketTemplate](https://github.com/ScoopInstaller/BucketTemplate):

- **CI** (`ci.yml`): on pushes to `main` and on pull requests, runs Scoop's bucket tests (Pester, under
  Windows PowerShell and PowerShell 7): every manifest parses, matches Scoop's schema and has
  `checkver`/`autoupdate`.
- **Excavator** (`excavator.yml`): every 4 hours, runs `checkver` with `autoupdate` through
  `ScoopInstaller/GithubActions` and pushes updated manifests to `main`.
- **Issues** and **Pull Requests**: the standard `ScoopInstaller/GithubActions` handlers (hash checks on
  `/verify`, hash-error issues).

Helper scripts in `bin/` wrap Scoop's own (`checkver.ps1`, `checkhashes.ps1`, `checkurls.ps1`,
`formatjson.ps1`, `auto-pr.ps1`); they need `SCOOP_HOME` or an installed Scoop.

## License

Apache-2.0, see [LICENSE](LICENSE). The manifests are packaging only; qqq itself is licensed as stated in
qqq-cli.
