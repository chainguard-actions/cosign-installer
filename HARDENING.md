# Hardening Report: sigstore--cosign-installer/v4.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **sigstore--cosign-installer/v4.1.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The Linux/macOS step writes the attacker-controlled `inputs.install-dir` value (via env var `input_install_dir`) directly to `$GITHUB_PATH` using `envsubst <<< "${input_install_dir}" >> "$GITHUB_PATH"` without the required sanitization (`printf '%s' ... | tr -d '\n\r'`). A malicious caller could supply a value containing embedded newlines to inject arbitrary entries into `$GITHUB_PATH`, potentially hijacking the PATH used by subsequent steps.

Locations:

- `action.yml:196`

### github-env-injection (severity: high)

The Windows step writes the attacker-controlled `inputs.install-dir` value (via env var `input_install_dir`) directly to `$env:GITHUB_PATH` using `echo "${install_dir}" | Out-File -FilePath $env:GITHUB_PATH -Encoding utf8 -Append` without any newline sanitization. A malicious caller could supply a value containing embedded newlines to inject arbitrary entries into `$GITHUB_PATH`, potentially hijacking the PATH used by subsequent steps.

Locations:

- `action.yml:204`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:
1. Linux/macOS step (line ~196): Replaced single-line `envsubst <<<"${input_install_dir}" >> "$GITHUB_PATH"` with a multi-line run block that: (a) expands the variable with envsubst, (b) strips newlines using `printf '%s' "$expanded" | tr -d '\n\r'`, then (c) writes the sanitized value to $GITHUB_PATH.
2. Windows step (line ~204): Replaced `echo "${install_dir}" | Out-File ...` with PowerShell that strips newlines using `-replace '[\r\n]', ''` before writing to $env:GITHUB_PATH via Add-Content. Both fixes prevent newline injection attacks where a malicious caller could supply embedded newlines in the install-dir input to inject arbitrary entries into $GITHUB_PATH.

