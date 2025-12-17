# 🤖 Gemini CLI – Termux Edition

Android/Termux optimized fork of Google Gemini CLI. Installs cleanly on Termux
by skipping native modules and adding clipboard detection for Termux.

[![npm](https://img.shields.io/npm/v/@mmmbuto/gemini-cli-termux?style=flat-square&logo=npm)](https://www.npmjs.com/package/@mmmbuto/gemini-cli-termux)
[![downloads](https://img.shields.io/npm/dt/@mmmbuto/gemini-cli-termux?style=flat-square)](https://www.npmjs.com/package/@mmmbuto/gemini-cli-termux)
[![ko-fi](https://img.shields.io/badge/☕_Support-Ko--fi-FF5E5B?style=flat-square&logo=ko-fi)](https://ko-fi.com/dionanos)

---

## What This Is

**Optimized Termux edition** of `google-gemini/gemini-cli`.

This project focuses on maintaining a first-class experience for Gemini on
Android/Termux. It provides critical adaptations for the mobile environment
while tracking upstream development closely.

- **Termux-First:** Pre-configured for Android filesystem and clipboard.
- **Lightweight:** Native dependencies managed for ARM64 without complex
  compilation.
- **Up-to-Date:** Synchronized with the latest Google Gemini CLI features.

## Installation (Termux)

```bash
pkg update && pkg upgrade -y
pkg install nodejs-lts -y
npm install -g @mmmbuto/gemini-cli-termux

gemini --version  # expected: 0.22.0-termux (latest)
```

Build from source:

```bash
git clone https://github.com/DioNanos/gemini-cli-termux.git
cd gemini-cli-termux
npm install --ignore-optional --ignore-scripts
npm run build && npm run bundle
node bundle/gemini.js --version
```

## Termux Optimizations

- **Smart Clipboard:** Auto-detects Android environment to enable seamless
  clipboard operations (fixes `TERMUX__PREFIX`).
- **Streamlined Install:** Native modules (`node-pty`, `keytar`,
  `tree-sitter-bash`) are handled gracefully to ensure instant installation
  without compilation errors.
- **Clean UX:** Suppresses desktop-centric warnings (like home directory checks)
  to optimize the experience for mobile terminal usage.
- **ARM64 Native:** Bundled specifically for Android architecture.

## Environment Specifics

- **Shell Integration:** Uses robust `child_process` fallback instead of
  `node-pty` for maximum stability on Android.
- **Credentials:** Keys are stored in standard config files for portability (no
  dependency on system keychains).
- **Parser:** Simplified Bash parsing to reduce heavy binary dependencies.

## Documentation & Fixes

### 📚 Complete Documentation

- **[Test Results](./GEMINI_TEST_REPORT_v0.22.0.md)** - Comprehensive test
  report with analysis
- **[Test Suite](./GEMINI_TEST_SUITE.md)** - Test methodology and checklist
- **[Patches & Fixes](./docs/patches/)** - Known issues and workarounds

### 🔧 Common Issues & Solutions

| Issue                 | Quick Fix                     | Documentation                                       |
| --------------------- | ----------------------------- | --------------------------------------------------- |
| node-pty warning      | `export NODE_NO_WARNINGS=1`   | [Details](./docs/patches/node-pty-warning.md)       |
| CLI syntax (`--json`) | Use `-o json` instead         | [Details](./docs/patches/cli-syntax-differences.md) |
| Hooks commands        | Use interactive mode `/hooks` | [Details](./docs/patches/hooks-interactive-only.md) |

### 📝 Quick Reference

```bash
# Correct usage examples
gemini -o json "your prompt"              # ✅ JSON output
gemini --output-format json "prompt"      # ✅ Also works
gemini --json "prompt"                    # ❌ Wrong syntax

# Quiet mode (suppress warnings)
export NODE_NO_WARNINGS=1
gemini "your prompt"

# Hooks management (interactive only)
gemini           # Start interactive mode
/hooks           # Manage hooks
```

See [docs/patches/README.md](./docs/patches/README.md) for complete solutions.

## Updating

```bash
npm install -g @mmmbuto/gemini-cli-termux@latest
```

### Versions

- **latest**: 0.22.0-termux (this build)
- **stable**: 0.22.0-termux

## Tests

- Suite: [`GEMINI_TEST_SUITE.md`](./GEMINI_TEST_SUITE.md)
- Latest report:
  [`GEMINI_TEST_REPORT_v0.22.0.md`](./GEMINI_TEST_REPORT_v0.22.0.md)
  - PASS with warnings (node-pty optional missing log; `--version --json`
    outputs plain string; config-path flag unsupported; extensions settings
    needs subcommand).
  - Non-interactive/file tests executed via agent; Termux checks pass;
    package/bundle verified.
  - Optional native modules (node-pty, keytar, tree-sitter-bash) not built on
    Termux → warnings expected; CLI remains functional.

## Changelog (Termux)

- **0.22.0-termux**: Sync with upstream (0.21.0-nightly); added hide banner
  patch; restored ARM64 dependency.
- **0.21.4-termux**: (Previous)

## Upstream Tracking

- Upstream: https://github.com/google-gemini/gemini-cli
- Divergent files: `esbuild.config.js`, `docs/TERMUX.md`, `package.json`,
  `README.md`, `test-gemini/*`

## License

Apache 2.0 (same as upstream). See LICENSE.
