# 🧪 Gemini CLI Termux - Test Report v0.21.2-termux

**Platform**: Android 12 / Termux (aarch64) **Node**: 24.9.0 **Version Tested**:
0.21.2-termux **Package name**: @mmmbuto/gemini-cli-termux **Workspace**:
/data/data/com.termux/files/home/gemini-test

## Summary

- Install: `npm install -g --ignore-optional --ignore-scripts` ✅ (optional
  native deps skipped)
- Version: `gemini --version` → 0.21.2-termux ✅;
  `node bundle/gemini.js --version` → 0.21.2-termux ✅
- Basic CLI: `gemini --help` exits 0 ✅ (punycode deprecation warning only)
- MCP: `gemini mcp list` ok (empty) ✅; `gemini mcp add --help` ok ✅
- Extensions: `gemini extensions list` ok (empty) ✅; `extensions settings`
  requires subcommand → prints help ⚠️
- Hooks: `gemini hooks list` logs missing optional `node-pty` (MODULE_NOT_FOUND)
  then exits 0 ⚠️ (expected when native deps skipped)
- Termux env: `termux-info`, `which termux-open-url`, LD_LIBRARY_PATH inside
  node ✅
- Package/binary: global bundle present; path exists ✅
- LLM-dependent steps (non-interactive prompts, file tool) skipped due to no API
  key on this device ⚠️

## Detailed Results vs Suite

- 1.1/1.2 Version & env: ✅
- 2.1 help: ✅ | 2.2 `--version --json`: outputs plain version (no JSON) ⚠️ |
  2.3 `--config-path`: unsupported → help shown ⚠️
- 3.x Hooks: runs with node-pty missing warning ⚠️
- 4.x Extensions: list ok; settings needs subcommand ⚠️
- 5.x MCP: pass
- 6.x Non-interactive JSON exec: skipped (no API key)
- 7.x File ops via CLI: skipped (needs model)
- 8.x Termux specifics: pass
- 9.x Package/binary: pass
- 10.x Optional deps: `require('node-pty')` fails gracefully ✅; `gemini hooks`
  without subcommand shows help ✅

## Verdict

**PASS with warnings**

- Warnings: optional native dep messages (expected on Termux),
  `--version --json` not JSON, `--config-path` flag unsupported, extensions
  settings needs subcommand, LLM-dependent tests skipped.

## Recommendations

- Run full `npm run preflight` + non-interactive tests on x86_64 CI with API key
  to cover automation paths.
- Consider suppressing `node-pty` MODULE_NOT_FOUND log on Termux build to reduce
  noise.
