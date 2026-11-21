# ShellGuard Audit — GitHub Action

Fail the build on shell-injection call sites before they ship.

`shellguard audit` statically scans Python source for the patterns that turn
user input into shell commands:

- **SG001** — `shell=True` hands untrusted text to a shell parser
- **SG002** — `os.system` / `os.popen` builds a command string for a shell
- **SG003** — `subprocess` call passes a string, not an argv list
- **SG004** — `asyncio.create_subprocess_shell` runs a command string through a shell

Import aliases (`import subprocess as sp`) and `from ... import ...` forms are
resolved. Safe list-form argv (`subprocess.run(["echo", x])`) is never flagged.

## Usage

```yaml
name: security
on: [push, pull_request]
jobs:
  shellguard:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: derbaroser-eng/shellguard-action@v1
        with:
          path: src
```

Exit codes: `0` clean, `1` findings (build fails; JSON report written to
`shellguard-audit.json`), `2` bad input.

## Inputs

| input | default | meaning |
|---|---|---|
| `path` | `.` | single directory or file to audit |
| `version` | `1.4.0` | shellguard release version to install |
| `wheel-sha256` | pinned to the v1.4.0 wheel | install is refused on mismatch |

Security properties: the wheel is downloaded from the official GitHub release
and verified with `sha256sum -c` before install; paths starting with `-` are
rejected (option-injection guard); inputs reach the shell only through
environment variables, never interpolated into a run block.

## The library behind the action

shellguard is also a typed Python library + CLI (`sg run`, `sg quote`,
`sg audit`, offline HMAC license keys) with 126 green tests:
https://derbaroser-eng.github.io/shellguard/

MIT licensed. A commercial license for proprietary redistribution is
available on the landing page.
