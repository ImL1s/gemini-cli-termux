# Patch Analysis & Improvement Proposals

**Project**: gemini-cli-termux **Version**: 0.22.0-termux **Author**: DioNanos
**Date**: 2025-12-17

---

## Executive Summary

Questo documento analizza le patch attuali del fork Termux e propone
miglioramenti per l'installazione e l'esperienza utente.

---

## Patch Attuali

### 1. Clipboard TERMUX\_\_PREFIX

**File**: `esbuild.config.js` (riga 80-81)

```javascript
// Termux compatibility: clipboardy expects TERMUX__PREFIX but Termux sets PREFIX
if (
  process.platform === 'android' &&
  process.env.PREFIX &&
  !process.env.TERMUX__PREFIX
) {
  process.env.TERMUX__PREFIX = process.env.PREFIX;
}
```

**Scopo**: clipboardy usa `TERMUX__PREFIX` per rilevare Termux, ma Termux setta
`PREFIX`

**Status**: ✅ Funzionante

**Miglioramenti proposti**:

- Aggiungere anche `TERMUX_VERSION` check per maggiore robustezza
- Considerare contributo a clipboardy upstream

---

### 2. is-in-ci Override

**File**: `packages/cli/src/patches/is-in-ci.ts`

```typescript
// This is a replacement for the `is-in-ci` package that always returns false.
// We are doing this to avoid the issue where `ink` does not render the UI
// when it detects that it is running in a CI environment.

const isInCi = false;
export default isInCi;
```

**Scopo**: Termux può essere rilevato erroneamente come CI, disabilitando UI ink

**Status**: ✅ Funzionante

**Miglioramenti proposti**:

- Aggiungere log debug quando override è attivo
- Documentare meglio il comportamento

---

### 3. Native Modules Optional

**File**: `package.json`

```json
{
  "optionalDependencies": {
    "@lydell/node-pty": "1.1.0",
    "@lydell/node-pty-darwin-arm64": "1.1.0",
    "@lydell/node-pty-darwin-x64": "1.1.0",
    "@lydell/node-pty-linux-x64": "1.1.0",
    "@lydell/node-pty-win32-arm64": "1.1.0",
    "@lydell/node-pty-win32-x64": "1.1.0",
    "node-pty": "^1.0.0"
  }
}
```

**Scopo**: Permette installazione senza compilazione native modules

**Status**: ✅ Funzionante, ma con warning

**Miglioramenti proposti**: Vedi sezione
[Miglioramenti Installazione](#miglioramenti-installazione)

---

### 4. esbuild External Modules

**File**: `esbuild.config.js`

```javascript
const external = [
  '@lydell/node-pty',
  'node-pty',
  '@lydell/node-pty-darwin-arm64',
  '@lydell/node-pty-darwin-x64',
  '@lydell/node-pty-linux-x64',
  '@lydell/node-pty-win32-arm64',
  '@lydell/node-pty-win32-x64',
];
```

**Scopo**: Esclude moduli nativi dal bundle

**Status**: ✅ Funzionante

---

## Miglioramenti Installazione

### Problema 1: Warning durante npm install

**Sintomo**:

```
npm warn optional SKIPPING OPTIONAL DEPENDENCY: @lydell/node-pty@1.1.0
npm warn notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for @lydell/node-pty-linux-x64
```

**Soluzione proposta**:

#### A. Script postinstall personalizzato

```javascript
// scripts/postinstall.js
const os = require('os');

if (os.platform() === 'android' || process.env.TERMUX_VERSION) {
  console.log('');
  console.log(
    '╔══════════════════════════════════════════════════════════════╗',
  );
  console.log(
    '║  gemini-cli-termux installed successfully!                   ║',
  );
  console.log(
    '║                                                              ║',
  );
  console.log(
    '║  Note: Native module warnings above are EXPECTED on Termux.  ║',
  );
  console.log(
    '║  The CLI will work with reduced PTY functionality.           ║',
  );
  console.log(
    '║                                                              ║',
  );
  console.log(
    '║  Run: gemini --version                                       ║',
  );
  console.log(
    '╚══════════════════════════════════════════════════════════════╝',
  );
  console.log('');
}
```

**Modifica package.json**:

```json
{
  "scripts": {
    "postinstall": "node scripts/postinstall.js || true"
  }
}
```

#### B. Documentazione .npmrc

Creare `.npmrc` con:

```ini
# Suppress optional dependency warnings
loglevel=error
optional=false
```

**Nota**: Non raccomandato globalmente, ma utile per utenti avanzati.

---

### Problema 2: Build from source richiede flag manuali

**Sintomo**: Utenti devono ricordare `--ignore-optional --ignore-scripts`

**Soluzione proposta**:

#### A. Makefile migliorato

```makefile
# Makefile

.PHONY: install build clean termux-install

# Standard install (desktop)
install:
	npm install

# Termux-specific install
termux-install:
	@echo "Installing for Termux..."
	npm install --ignore-optional --ignore-scripts
	npm run build
	npm run bundle
	@echo ""
	@echo "Installation complete! Run: node bundle/gemini.js"

# Build only
build:
	npm run build
	npm run bundle

# Clean
clean:
	rm -rf node_modules bundle/gemini.js
```

#### B. Script helper

**File**: `scripts/termux-setup.sh`

```bash
#!/data/data/com.termux/files/usr/bin/bash

echo "=== Gemini CLI Termux Setup ==="

# Check Node version
NODE_VERSION=$(node -v | cut -d'v' -f2 | cut -d'.' -f1)
if [ "$NODE_VERSION" -lt 20 ]; then
    echo "Error: Node.js 20+ required. Install with: pkg install nodejs-lts"
    exit 1
fi

# Install dependencies
echo "Installing dependencies..."
npm install --ignore-optional --ignore-scripts 2>&1 | grep -v "npm warn"

# Build
echo "Building..."
npm run build
npm run bundle

echo ""
echo "=== Setup Complete ==="
echo "Run: node bundle/gemini.js"
echo "Or link globally: npm link"
```

---

### Problema 3: Prima esecuzione lenta

**Sintomo**: Prima esecuzione di `gemini` ha delay per auth/setup

**Soluzione proposta**:

#### A. Configurazione Termux-aware

**File**: `packages/core/src/config/termux-defaults.ts`

```typescript
export const TERMUX_DEFAULTS = {
  // Disable PTY by default on Termux
  enableInteractiveShell: false,

  // Use file-based credential storage
  credentialStorage: 'file',

  // Reduce telemetry overhead
  telemetryEnabled: false,

  // Termux-specific paths
  configDir: process.env.HOME + '/.config/gemini',
};

export function isTermux(): boolean {
  return (
    process.platform === 'android' ||
    !!process.env.TERMUX_VERSION ||
    !!process.env.PREFIX?.includes('com.termux')
  );
}
```

---

### Problema 4: Aggiornamento manuale

**Sintomo**: Utenti devono controllare manualmente le nuove versioni

**Soluzione proposta**:

#### A. Update checker opzionale

```typescript
// packages/cli/src/utils/update-check.ts
import { getLatestVersion } from 'latest-version';

export async function checkForUpdates(currentVersion: string): Promise<void> {
  if (process.env.GEMINI_SKIP_UPDATE_CHECK) return;

  try {
    const latest = await getLatestVersion('@mmmbuto/gemini-cli-termux');
    if (latest !== currentVersion) {
      console.log(`\n📦 Update available: ${currentVersion} → ${latest}`);
      console.log('   Run: npm install -g @mmmbuto/gemini-cli-termux@latest\n');
    }
  } catch {
    // Silently fail - network may be unavailable
  }
}
```

---

## Nuove Patch Proposte

### Patch 5: Termux-API Detection

**Scopo**: Rilevare automaticamente se Termux-API è installato

**Implementazione**:

```typescript
// packages/core/src/utils/termux-detect.ts
import { execSync } from 'child_process';

export interface TermuxEnvironment {
  isTermux: boolean;
  hasTermuxApi: boolean;
  apiVersion?: string;
  prefix: string;
}

export function detectTermuxEnvironment(): TermuxEnvironment {
  const isTermux =
    process.platform === 'android' ||
    !!process.env.TERMUX_VERSION ||
    !!process.env.PREFIX?.includes('com.termux');

  if (!isTermux) {
    return { isTermux: false, hasTermuxApi: false, prefix: '' };
  }

  let hasTermuxApi = false;
  let apiVersion: string | undefined;

  try {
    execSync('which termux-battery-status', { stdio: 'ignore' });
    hasTermuxApi = true;

    // Try to get version from package
    const result = execSync('pkg show termux-api 2>/dev/null | grep Version', {
      encoding: 'utf-8',
    });
    apiVersion = result.split(':')[1]?.trim();
  } catch {
    // termux-api not installed
  }

  return {
    isTermux: true,
    hasTermuxApi,
    apiVersion,
    prefix: process.env.PREFIX || '/data/data/com.termux/files/usr',
  };
}
```

---

### Patch 6: Suppress Harmless Warnings

**Scopo**: Nascondere warning non rilevanti su Termux

**Implementazione**:

```javascript
// In esbuild banner
if (process.platform === 'android') {
  // Suppress punycode deprecation warning
  const originalEmit = process.emit;
  process.emit = function (name, data, ...args) {
    if (
      name === 'warning' &&
      typeof data === 'object' &&
      data.name === 'DeprecationWarning' &&
      data.message?.includes('punycode')
    ) {
      return false;
    }
    return originalEmit.apply(process, arguments);
  };
}
```

---

### Patch 7: Graceful Fallback Messages

**Scopo**: Messaggi user-friendly quando feature non disponibili

**Implementazione**:

```typescript
// packages/core/src/utils/termux-fallbacks.ts
export const TERMUX_FALLBACK_MESSAGES = {
  'node-pty': 'PTY support disabled on Termux. Using basic shell mode.',
  keytar: 'Secure keychain unavailable. Credentials stored in config file.',
  'tree-sitter-bash':
    'Bash parsing simplified. Some syntax highlighting limited.',
};

export function logFallbackOnce(module: string): void {
  const key = `GEMINI_LOGGED_${module.toUpperCase()}`;
  if (process.env[key]) return;

  const message = TERMUX_FALLBACK_MESSAGES[module];
  if (message) {
    console.log(`ℹ️  ${message}`);
    process.env[key] = '1';
  }
}
```

---

## Roadmap Patch

### Fase 1: Quick Wins (v0.22.1)

- [ ] Aggiungere postinstall script con messaggio chiaro
- [ ] Creare termux-setup.sh helper
- [ ] Migliorare Makefile

### Fase 2: Polish (v0.23.0)

- [ ] Implementare Termux detection
- [ ] Aggiungere update checker
- [ ] Suppress warning punycode

### Fase 3: Integration (v0.24.0)

- [ ] Termux-API detection automatica
- [ ] Pre-configurazione Termux-aware
- [ ] Graceful fallback messages

---

## File da Modificare

| File                                          | Modifica                      |
| --------------------------------------------- | ----------------------------- |
| `package.json`                                | Aggiungere postinstall script |
| `scripts/postinstall.js`                      | Nuovo file                    |
| `scripts/termux-setup.sh`                     | Nuovo file                    |
| `Makefile`                                    | Target termux-install         |
| `esbuild.config.js`                           | Punycode warning suppress     |
| `packages/core/src/utils/termux-detect.ts`    | Nuovo file                    |
| `packages/core/src/config/termux-defaults.ts` | Nuovo file                    |

---

## Compatibilità Upstream

Le patch proposte sono progettate per:

1. **Non interferire** con funzionalità desktop
2. **Essere condizionali** (attive solo su Termux)
3. **Essere facilmente rimovibili** se upstream aggiunge supporto nativo

---

_Author: DioNanos_
