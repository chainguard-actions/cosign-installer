# Hardening Report: sigstore--cosign-installer/v4.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

Action **sigstore--cosign-installer/v4.1.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Two steps write the attacker-controlled `inputs.install-dir` value (via env var `input_install_dir`) to `$GITHUB_PATH` without the required sanitization (`printf '%s' ... | tr -d '\n\r'`). Step 2 (bash, Linux/macOS): `run: envsubst <<<"${input_install_dir}" >> "$GITHUB_PATH"`. Step 3 (PowerShell, Windows): `echo "${install_dir}" | Out-File -FilePath $env:GITHUB_PATH`. A malicious value containing newlines in `inputs.install-dir` could inject arbitrary entries into GITHUB_PATH, enabling path-hijacking attacks.

Locations:

- `action.yml:271`
- `action.yml:278`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection vulnerabilities in action.yml:

1. Bash step (Linux/macOS, line ~271): Replaced single-line `envsubst <<< "${input_install_dir}" >> "$GITHUB_PATH"` with a multi-line script that sanitizes the value using `printf '%s' ... | tr -d '\n\r'` before writing to GITHUB_PATH.

2. PowerShell step (Windows, line ~278): Replaced `echo "${install_dir}" | Out-File -FilePath $env:GITHUB_PATH` with a script that strips newlines using `-replace "`r|`n", ""` before writing to GITHUB_PATH.

Both fixes prevent attackers from injecting arbitrary entries into GITHUB_PATH via newline characters in the `inputs.install-dir` value.

