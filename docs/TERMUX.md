# Gemini CLI – Termux Guide

How to install and run the Termux edition `@mmmbuto/gemini-cli-termux` on
Android.

## Prerequisites

- Termux installed
- Node.js 20+ (`pkg install nodejs-lts -y`)
- Git (only if building from source)

## Install via npm (recommended)

```bash
npm install -g @mmmbuto/gemini-cli-termux

gemini --version
# expected: 0.22.0-termux (latest)
```

Features of the npm build

- ARM64/Android bundle included
- Termux clipboard patch (`PREFIX` -> `TERMUX__PREFIX`)
- Native modules left optional; no NDK required

## Build from source (Termux fork)

```bash
git clone https://github.com/DioNanos/gemini-cli-termux.git
cd gemini-cli-termux
npm install --ignore-optional --ignore-scripts
npm run build && npm run bundle
node bundle/gemini.js --version
```

## Known issues

1. Native modules (keytar, node-pty, tree-sitter-bash) fail on Termux → ignored
   with the install flags above.
2. Clipboardy TERMUX\_\_PREFIX is patched in the bundle.
3. Node punycode warning is harmless; optional:
   `node --no-deprecation bundle/gemini.js`.

## Limitations

- No full PTY support → some interactive shell features limited
- No secure keychain → credentials stored in plain config files
- Bash parsing simplified (no tree-sitter)

## Update

- npm: `npm install -g @mmmbuto/gemini-cli-termux@latest`
- source:
  `git pull && npm install --ignore-optional --ignore-scripts && npm run build && npm run bundle`

## Termux-API Support (Optional)

Enable access to Android hardware and APIs:

1. Install Termux-API package:

   ```bash
   pkg install termux-api jq
   ```

2. Install Termux:API app from F-Droid

3. Setup tool discovery:

   ```bash
   # Copy scripts to config
   mkdir -p ~/.config/gemini/termux-tools
   cp scripts/termux-tools/*.sh ~/.config/gemini/termux-tools/
   chmod +x ~/.config/gemini/termux-tools/*.sh

   # Configure in settings.json
   cat > ~/.config/gemini/settings.json << 'EOF'
   {
     "tool_discovery_command": "bash ~/.config/gemini/termux-tools/discovery.sh",
     "tool_call_command": "bash ~/.config/gemini/termux-tools/call.sh"
   }
   EOF
   ```

4. Test:
   ```bash
   gemini "What's my battery status?"
   ```

See [docs/termux-api/](./docs/termux-api/) for complete documentation.

## Report Termux issues

Use the fork issues: https://github.com/DioNanos/gemini-cli-termux/issues

Sunset: will deprecate when upstream adds native Termux support.
