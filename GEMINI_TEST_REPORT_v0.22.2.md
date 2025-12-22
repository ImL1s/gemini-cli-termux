# 🧪 Gemini CLI Termux Test Report (v0.22.2-termux)

**Date:** 2025-12-17  
**Environment:** Termux (Android/aarch64), Node v24.11.1, npm (per shell),
optional native deps omitted (node-pty/keytar/tree-sitter-bash).  
**Build:** `bundle/gemini.js --version` → 0.22.2-termux.

## Summary

Re-ran the manual checklist from `GEMINI_TEST_SUITE.md` with the new
base-write/TTS toggles. Most steps pass; DEP0169 warning persists (upstream).
Context memory bootstrap from GEMINI.md verified after wipe.

## Results

| #    | Test                                  | Status                                      | Notes                                               |
| ---- | ------------------------------------- | ------------------------------------------- | --------------------------------------------------- |
| 1.1  | `gemini --version`                    | ✅ Pass                                     | 0.22.2-termux                                       |
| 1.2  | Env info (node/uname/PREFIX)          | ✅ Pass                                     | Node v24.11.1, aarch64, Termux prefix OK            |
| 2.1  | `gemini --help`                       | ✅ Pass                                     | Help displayed                                      |
| 4.1  | `gemini extensions list`              | ✅ Pass                                     | “No extensions installed.”                          |
| 4.2  | `gemini extensions settings`          | ⚪ Not run                                  |                                                     |
| 5.1  | `gemini mcp list`                     | ⚠️ Warn                                     | Connects; DEP0169 warning                           |
| 5.2  | `gemini mcp add --help`               | ✅ Pass                                     | Prints help                                         |
| 6.1  | `gemini -o json "echo hello"`         | ✅ Pass (YOLO)                              | DEP0169 warning                                     |
| 6.2  | `GEMINI_JSONL=1 gemini -o json "pwd"` | ✅ Pass (YOLO, workspace GEMINI.md present) |
| 7.1  | Read file                             | ✅ Pass                                     | `read file.txt`                                     |
| 7.2  | List files                            | ✅ Pass                                     | `ls`                                                |
| 8.1  | `termux-info`                         | ✅ Pass                                     |                                                     |
| 8.2  | `which termux-open-url`               | ✅ Pass                                     |                                                     |
| 8.3  | `LD_LIBRARY_PATH` passthrough         | ✅ Pass                                     | Value preserved                                     |
| 9.1  | Global bundle path check              | ✅ Pass                                     | bundle/gemini.js exists under npm root -g           |
| 9.2  | `node bundle/gemini.js --version`     | ✅ Pass                                     | 0.22.2-termux                                       |
| 10.1 | Optional native deps graceful         | ✅ Pass                                     | `require('node-pty')` → MODULE_NOT_FOUND (expected) |
| 11.1 | Termux detect helper                  | ✅ Pass                                     | `isTermux()` -> true                                |
| 11.2 | termux-tools discovery                | ✅ Pass                                     | scripts/termux-tools/discovery.sh                   |
| 11.3 | termux_battery_status call            | ✅ Pass                                     | JSON output                                         |
| 11.4 | termux install helpers                | ⚪ Not run                                  | (heavy)                                             |
| 12.1 | Context memory bootstrap              | ✅ Pass                                     | user.json recreated from GEMINI.md after wipe       |
| 12.2 | Autoload toggles                      | ⚪ Not re-toggled                           | Defaults ON                                         |
| 12.3 | Journal mirror                        | ⚪ Not run                                  |                                                     |
| 12.4 | Primary ordering                      | ⚪ Not run                                  |                                                     |
| 12.5 | Base RO guard                         | ⚪ Not re-run                               | base writes now gated by setting                    |

## Notes

- New settings tested: base write toggle (available), TTS toggle (blocks
  `termux-tts-speak` when off).
- DEP0169 (`url.parse`) still emitted on CLI startup (upstream dependency).
