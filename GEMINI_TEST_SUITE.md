# 🧪 Gemini CLI Termux Test Suite (v0.21.1-termux)

**Goal**: Validate the Termux build without native deps (node-pty/keytar/tree-sitter). Run from a clean shell on Termux ARM64.

## 0. Prep
- Create workspace: `rm -rf ~/gemini-test && mkdir ~/gemini-test && cd ~/gemini-test`

## 1. Version & Env
1.1 `gemini --version` → shows `0.21.1-termux`
1.2 `node -v`, `uname -m`, `echo $PREFIX` (expect Termux paths / aarch64)

## 2. CLI Basics
2.1 `gemini --help` exits 0
2.2 `gemini --version --json` returns JSON with version field
2.3 `gemini --config-path` prints default config path (no crash)

## 3. Hooks (new in 0.21.x)
3.1 `gemini hooks list` shows no error (empty or items accepted)
3.2 `gemini hooks migrate` runs without crash in empty workspace
3.3 `gemini hooks events` shows available events (session_start/session_end)

## 4. Extensions
4.1 `gemini extensions list` succeeds (even if empty)
4.2 `gemini extensions settings` prints settings schema

## 5. MCP
5.1 `gemini mcp list` works (empty ok)
5.2 `gemini mcp add --help` exits 0

## 6. Non-interactive
6.1 `gemini --json --no-color "echo hello"` returns JSON stream
6.2 `GEMINI_JSONL=1 gemini --json "pwd"` produces JSONL lines

## 7. File ops (Termux safe)
7.1 `echo hi > file.txt` then `gemini --json "read file.txt"` contains "hi"
7.2 `gemini --json "list files"` lists current dir

## 8. Termux specifics
8.1 `termux-info` runs (or prints not available) without crash
8.2 `which termux-open-url` exists
8.3 `echo $LD_LIBRARY_PATH` preserved inside `node -e "console.log(process.env.LD_LIBRARY_PATH)"`

## 9. Package/binary
9.1 `ls $(npm root -g)/@mmmbuto/gemini-cli-termux/bundle/gemini.js` exists
9.2 `node bundle/gemini.js --version` (from repo) prints 0.21.1-termux

## 10. Known limits (assert graceful handling)
10.1 `node-pty` optional: running `require('node-pty')` should fail gracefully (not crash CLI)
10.2 `gemini hooks` when called without subcommand should print help (not LLM)

## Expected Outcome
- All steps run without crash; informational steps may show empty lists.
- No build commands required; native optional deps may fail to build but CLI must still work.
