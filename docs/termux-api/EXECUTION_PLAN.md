# Execution Plan - Sonnet 4.5

**Project**: gemini-cli-termux **Author**: DioNanos **Date**: 2025-12-17
**Status**: AWAITING APPROVAL

---

## Overview

Piano di esecuzione per implementare le patch e miglioramenti descritti nella
documentazione. Da eseguire con Sonnet 4.5 SOLO dopo approvazione.

**IMPORTANTE**: Solo modifiche locali, NESSUN PUSH.

---

## Pre-Requisiti

Prima di iniziare:

```bash
cd ~/Dev/gemini-cli-termux
git status  # Verificare stato pulito
git branch  # Verificare branch corrente
```

---

## Task List

### FASE 1: Miglioramenti Installazione

#### Task 1.1: Creare postinstall script

**File**: `scripts/postinstall.js` **Azione**: CREATE **Priorità**: ALTA

```javascript
// scripts/postinstall.js
const os = require('os');

// Only show message on Termux/Android
if (
  os.platform() === 'android' ||
  process.env.TERMUX_VERSION ||
  (process.env.PREFIX && process.env.PREFIX.includes('com.termux'))
) {
  console.log('');
  console.log(
    '╔══════════════════════════════════════════════════════════════╗',
  );
  console.log(
    '║  gemini-cli-termux installed successfully on Termux!         ║',
  );
  console.log(
    '║                                                              ║',
  );
  console.log(
    '║  Note: Native module warnings above are EXPECTED.            ║',
  );
  console.log(
    '║  The CLI works with reduced PTY functionality.               ║',
  );
  console.log(
    '║                                                              ║',
  );
  console.log(
    '║  Quick start: gemini --version                               ║',
  );
  console.log(
    '║  First run:   gemini                                         ║',
  );
  console.log(
    '╚══════════════════════════════════════════════════════════════╝',
  );
  console.log('');
}
```

---

#### Task 1.2: Aggiornare package.json con postinstall

**File**: `package.json` **Azione**: EDIT **Priorità**: ALTA

Aggiungere in `scripts`:

```json
"postinstall": "node scripts/postinstall.js || true"
```

---

#### Task 1.3: Creare termux-setup.sh helper

**File**: `scripts/termux-setup.sh` **Azione**: CREATE **Priorità**: MEDIA

```bash
#!/data/data/com.termux/files/usr/bin/bash

set -e

echo "=== Gemini CLI Termux Setup ==="
echo ""

# Check Node version
NODE_VERSION=$(node -v 2>/dev/null | cut -d'v' -f2 | cut -d'.' -f1)
if [ -z "$NODE_VERSION" ] || [ "$NODE_VERSION" -lt 20 ]; then
    echo "❌ Error: Node.js 20+ required"
    echo "   Install with: pkg install nodejs-lts"
    exit 1
fi
echo "✓ Node.js version: $(node -v)"

# Check if in project directory
if [ ! -f "package.json" ]; then
    echo "❌ Error: Run this script from gemini-cli-termux root directory"
    exit 1
fi

# Install dependencies (suppress optional warnings)
echo ""
echo "Installing dependencies..."
npm install --ignore-optional --ignore-scripts 2>&1 | grep -v "npm warn" || true

# Build
echo ""
echo "Building bundle..."
npm run build 2>&1
npm run bundle 2>&1

echo ""
echo "=== Setup Complete ==="
echo ""
echo "Run with: node bundle/gemini.js"
echo "Or link globally: npm link"
echo ""
```

---

#### Task 1.4: Aggiornare Makefile

**File**: `Makefile` **Azione**: EDIT **Priorità**: MEDIA

Aggiungere target:

```makefile
# Termux-specific install and build
termux-install:
	@echo "=== Installing for Termux ==="
	npm install --ignore-optional --ignore-scripts
	npm run build
	npm run bundle
	@echo ""
	@echo "Done! Run: node bundle/gemini.js"

termux-clean:
	rm -rf node_modules
	rm -f bundle/gemini.js

.PHONY: termux-install termux-clean
```

---

### FASE 2: Termux Detection Utility

#### Task 2.1: Creare termux-detect.ts

**File**: `packages/core/src/utils/termux-detect.ts` **Azione**: CREATE
**Priorità**: MEDIA

```typescript
/**
 * @license
 * Copyright 2025 Google LLC
 * SPDX-License-Identifier: Apache-2.0
 */

import { execSync } from 'node:child_process';

export interface TermuxEnvironment {
  isTermux: boolean;
  hasTermuxApi: boolean;
  apiVersion?: string;
  prefix: string;
  availableCommands: string[];
}

/**
 * Detect if running in Termux environment
 */
export function isTermux(): boolean {
  return (
    process.platform === 'android' ||
    !!process.env.TERMUX_VERSION ||
    !!(process.env.PREFIX && process.env.PREFIX.includes('com.termux'))
  );
}

/**
 * Detect full Termux environment including API availability
 */
export function detectTermuxEnvironment(): TermuxEnvironment {
  if (!isTermux()) {
    return {
      isTermux: false,
      hasTermuxApi: false,
      prefix: '',
      availableCommands: [],
    };
  }

  let hasTermuxApi = false;
  let apiVersion: string | undefined;
  const availableCommands: string[] = [];

  try {
    // Check if termux-api is installed
    execSync('which termux-battery-status', { stdio: 'ignore' });
    hasTermuxApi = true;

    // Try to get version
    try {
      const result = execSync(
        'pkg show termux-api 2>/dev/null | grep Version || echo ""',
        { encoding: 'utf-8' },
      );
      const match = result.match(/Version:\s*(.+)/);
      if (match) {
        apiVersion = match[1].trim();
      }
    } catch {
      // Version detection failed, continue
    }

    // Detect available commands
    const commands = [
      'termux-battery-status',
      'termux-clipboard-get',
      'termux-clipboard-set',
      'termux-toast',
      'termux-notification',
      'termux-tts-speak',
      'termux-vibrate',
      'termux-torch',
      'termux-location',
      'termux-wifi-connectioninfo',
      'termux-camera-info',
      'termux-sensor',
      'termux-dialog',
    ];

    for (const cmd of commands) {
      try {
        execSync(`which ${cmd}`, { stdio: 'ignore' });
        availableCommands.push(cmd);
      } catch {
        // Command not available
      }
    }
  } catch {
    // termux-api not installed
  }

  return {
    isTermux: true,
    hasTermuxApi,
    apiVersion,
    prefix: process.env.PREFIX || '/data/data/com.termux/files/usr',
    availableCommands,
  };
}
```

---

#### Task 2.2: Export termux-detect da core index

**File**: `packages/core/src/index.ts` **Azione**: EDIT **Priorità**: MEDIA

Aggiungere export:

```typescript
export * from './utils/termux-detect.js';
```

---

### FASE 3: Punycode Warning Suppression

#### Task 3.1: Aggiornare esbuild banner

**File**: `esbuild.config.js` **Azione**: EDIT **Priorità**: BASSA

Modificare il banner JS per includere:

```javascript
banner: {
  js: `import { createRequire } from 'module'; const require = createRequire(import.meta.url); globalThis.__filename = require('url').fileURLToPath(import.meta.url); globalThis.__dirname = require('path').dirname(globalThis.__filename);
// Termux compatibility: clipboardy expects TERMUX__PREFIX but Termux sets PREFIX
if (process.platform === 'android' && process.env.PREFIX && !process.env.TERMUX__PREFIX) { process.env.TERMUX__PREFIX = process.env.PREFIX; }
// Suppress punycode deprecation warning on Termux
if (process.platform === 'android') {
  const origEmit = process.emit;
  process.emit = function(name, data) {
    if (name === 'warning' && data && data.name === 'DeprecationWarning' && data.message && data.message.includes('punycode')) return false;
    return origEmit.apply(process, arguments);
  };
}`,
},
```

---

### FASE 4: Tool Discovery Scripts (User-space)

#### Task 4.1: Creare discovery.sh in scripts/termux-tools/

**File**: `scripts/termux-tools/discovery.sh` **Azione**: CREATE **Priorità**:
MEDIA

(Contenuto già definito in DISCOVERY_SETUP.md)

---

#### Task 4.2: Creare call.sh in scripts/termux-tools/

**File**: `scripts/termux-tools/call.sh` **Azione**: CREATE **Priorità**: MEDIA

(Contenuto già definito in DISCOVERY_SETUP.md)

---

### FASE 5: Documentazione

#### Task 5.1: Aggiornare README.md principale

**File**: `README.md` **Azione**: EDIT **Priorità**: ALTA

Aggiungere sezione Termux-API:

```markdown
## Termux-API Integration (New!)

This fork supports integration with Termux-API commands for Android device
access.

See [docs/termux-api/](./docs/termux-api/) for:

- Integration plan and architecture
- Complete command reference
- Tool discovery setup guide
- Patch documentation
```

---

#### Task 5.2: Aggiornare docs/TERMUX.md

**File**: `docs/TERMUX.md` **Azione**: EDIT **Priorità**: MEDIA

Aggiungere sezione Termux-API:

````markdown
## Termux-API Support (Optional)

Enable access to Android hardware and APIs:

1. Install Termux-API package:
   ```bash
   pkg install termux-api jq
   ```
````

2. Install Termux:API app from F-Droid

3. Setup tool discovery (optional):

   ```bash
   # Copy scripts to config
   mkdir -p ~/.config/gemini/termux-tools
   cp scripts/termux-tools/*.sh ~/.config/gemini/termux-tools/
   chmod +x ~/.config/gemini/termux-tools/*.sh

   # Configure in settings.json
   echo '{"tool_discovery_command": "bash ~/.config/gemini/termux-tools/discovery.sh", "tool_call_command": "bash ~/.config/gemini/termux-tools/call.sh"}' > ~/.config/gemini/settings.json
   ```

4. Test:
   ```bash
   gemini "What's my battery status?"
   ```

See [docs/termux-api/](./docs/termux-api/) for complete documentation.

````

---

#### Task 5.3: Aggiornare docs/patches/README.md
**File**: `docs/patches/README.md`
**Azione**: EDIT
**Priorità**: BASSA

Aggiornare versione e aggiungere nuove patch:
```markdown
# Termux Patches (0.22.0-termux)

Minimal changes to run `gemini-cli` on Android/Termux ARM64 without native deps.

## Patch List

1. **Clipboard (TERMUX__PREFIX)** – On Termux set `TERMUX__PREFIX` from
   `$PREFIX` so clipboardy detects Termux correctly.
2. **Optional native modules** – Leave `node-pty`, `keytar`, `tree-sitter-bash`
   in `optionalDependencies`; build failures are tolerated.
3. **Core exports** – `packages/core/src/index.ts` re-exports stdio utilities.
4. **Bundle** – Prebuilt `bundle/gemini.js` shipped in npm package.
5. **is-in-ci override** – Prevents ink from detecting Termux as CI.
6. **Punycode warning** – Suppresses deprecation warning on Termux.
7. **Termux detection** – `packages/core/src/utils/termux-detect.ts` utility.
8. **Postinstall message** – Clear success message on Termux install.

## New in 0.22.0

- Termux-API tool discovery support
- Improved installation experience
- Helper scripts for build from source

**Version**: 0.22.0-termux
````

---

## Execution Order

```
FASE 1 (Installazione) - Alta priorità
├── 1.1 scripts/postinstall.js [CREATE]
├── 1.2 package.json [EDIT - postinstall]
├── 1.3 scripts/termux-setup.sh [CREATE]
└── 1.4 Makefile [EDIT - termux targets]

FASE 2 (Detection) - Media priorità
├── 2.1 packages/core/src/utils/termux-detect.ts [CREATE]
└── 2.2 packages/core/src/index.ts [EDIT - export]

FASE 3 (Warning) - Bassa priorità
└── 3.1 esbuild.config.js [EDIT - banner]

FASE 4 (Discovery Scripts) - Media priorità
├── 4.1 scripts/termux-tools/discovery.sh [CREATE]
└── 4.2 scripts/termux-tools/call.sh [CREATE]

FASE 5 (Docs) - Alta priorità
├── 5.1 README.md [EDIT]
├── 5.2 docs/TERMUX.md [EDIT]
└── 5.3 docs/patches/README.md [EDIT]
```

---

## Verifica Post-Esecuzione

```bash
# 1. Build test
npm install --ignore-optional --ignore-scripts
npm run build
npm run bundle

# 2. Version check
node bundle/gemini.js --version

# 3. Termux detection (se su Termux)
node -e "const {isTermux} = require('./packages/core/dist/utils/termux-detect.js'); console.log('isTermux:', isTermux())"

# 4. Discovery test (se configurato)
bash scripts/termux-tools/discovery.sh | jq '.[] | .name' | head -5

# 5. Git status (NESSUN PUSH)
git status
git diff --stat
```

---

## Commit Message (quando approvato)

```
feat(termux): improve installation and add Termux-API support

- Add postinstall script with clear success message for Termux
- Add termux-setup.sh helper for build from source
- Add Makefile targets for Termux install
- Add termux-detect.ts utility for environment detection
- Add punycode warning suppression on Android
- Add Tool Discovery scripts for Termux-API integration
- Update documentation

Fixes #XX
```

---

### FASE 6: Merge Automation

#### Task 6.1: Creare check-termux-patches.sh

**File**: `scripts/check-termux-patches.sh` **Azione**: CREATE **Priorità**:
ALTA

Script per verificare che tutte le patch siano intatte dopo merge upstream.
(Vedi MERGE_STRATEGY.md per contenuto completo)

---

## Note per Sonnet 4.5

1. Eseguire task in ordine di FASE
2. Verificare build dopo ogni FASE
3. NON eseguire git push
4. Segnalare eventuali errori di compilazione
5. Tutti i file TypeScript devono passare typecheck
6. Tutte le patch devono essere facilmente riaplicabili dopo merge upstream
7. Usare commenti `// TERMUX PATCH:` per identificare modifiche

---

**STATUS**: In attesa approvazione DAG

_Author: DioNanos_
