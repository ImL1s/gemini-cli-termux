# Termux Patches (0.22.1-termux)

Minimal changes to run `gemini-cli` on Android/Termux ARM64 without native deps.

## Patch List

1. **Clipboard (TERMUX\_\_PREFIX)** – On Termux set `TERMUX__PREFIX` from
   `$PREFIX` so clipboardy detects Termux correctly.
2. **Optional native modules** – Leave `node-pty`, `keytar`, `tree-sitter-bash`
   in `optionalDependencies`; build failures are tolerated.
3. **Core exports** – `packages/core/src/index.ts` re-exports stdio utilities
   and hook/telemetry APIs so CLI bundling succeeds on Termux.
4. **Bundle** – Prebuilt `bundle/gemini.js` shipped in npm package
   (ARM64/Android) with policy files under `bundle/policies/`.
5. **is-in-ci override** – Prevents ink from detecting Termux as CI.
6. **Punycode warning** – Suppresses deprecation warning on Android.
7. **Termux detection** – `packages/core/src/utils/termux-detect.ts` utility.
8. **Postinstall message** – Clear success message on Termux install.

## New in 0.22.1

- Termux-API tool discovery support (`scripts/termux-tools/`)
- Improved installation experience (`scripts/postinstall.js`)
- Helper scripts for build from source (`scripts/termux-setup.sh`)
- Makefile targets: `termux-install`, `termux-clean`
- Merge verification script (`scripts/check-termux-patches.sh`)

## Expected Warnings

- Missing native modules may log warnings on Termux; functionality remains
  (non-PTY shell, plain token storage).

## Scope

No functional changes to upstream features; only compatibility/export fixes.

## Merge Strategy

All patches are designed to be merge-safe. See
[docs/termux-api/MERGE_STRATEGY.md](../termux-api/MERGE_STRATEGY.md) for details
on maintaining patches after upstream sync.

**Version**: 0.22.1-termux
