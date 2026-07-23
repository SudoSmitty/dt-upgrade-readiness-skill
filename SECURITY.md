# Security

## What this skill is

`dt-upgrade-readiness` is an [Agent Skills](https://agentskills.io) package:
Markdown instructions (`SKILL.md`), a bundled data file of read-only checks
(`references/readiness-checks.json`), an HTML report template, and one Python
script (`scripts/collect.py`). Installing it (via `npx skills add` or a manual
copy) only **copies these files** into your agent's skills directory — nothing
executes at install time.

## Capabilities & data handling

- **Read-only.** The skill never creates, updates, or deletes anything in your
  Dynatrace tenant. It only runs read queries.
- **No dependencies.** There is no `package.json`, lockfile, or third-party
  library. `collect.py` uses only the Python standard library.
- **How it runs.** `collect.py` invokes **your own, already-authenticated
  `dtctl`** as a subprocess to run the bundled readiness checks against the
  tenant you select:
  - DQL queries via `dtctl query` (read-only fetches from Grail/entities).
  - Read-only check functions via `dtctl exec function` (JavaScript that only
    *reads* settings/config/platform state).
  The check definitions are static and bundled in the repo — **no code is
  downloaded or fetched at runtime.**
- **Network.** The only network access is `dtctl` talking to the Dynatrace
  environment you authenticated it to. The skill makes no other outbound calls,
  sends no telemetry, and exfiltrates nothing.
- **Local writes.** It writes a temp file per code-check (deleted immediately
  after use) and, at the end, one HTML report in your working directory. The
  report is static HTML with no scripts.
- **Credentials.** The skill reads no tokens itself; authentication is handled
  entirely by `dtctl` (OS keyring or `dtctl`'s configured token storage).

## Notes for security scanners

- **Socket "shell/network capability"** — expected: `collect.py` shells out to
  the `dtctl` CLI over the network to your tenant. This is the skill's core,
  intended function, scoped to read-only calls against an environment you chose.
- **Pipe-to-shell installs** — this repo deliberately avoids `curl … | sh` and
  `irm … | iex`. dtctl is installed via Homebrew, or on Windows by downloading
  the official installer, reviewing it, and running it.
- The long lines and `decodeBase64ToString(...)` in
  `references/readiness-checks.json` are **Dynatrace DQL**, not obfuscated code
  or embedded binaries.

## Reporting a vulnerability

Please open a GitHub issue describing the concern (avoid including secrets or
tenant data). Security-relevant reports will be prioritized.
