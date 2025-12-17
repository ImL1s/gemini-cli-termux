# Merge Strategy - Upstream Sync

**Project**: gemini-cli-termux **Author**: DioNanos **Date**: 2025-12-17

---

## Overview

Strategia per mantenere le patch Termux facilmente applicabili dopo ogni merge
con l'upstream `google-gemini/gemini-cli`.

---

## Principi di Design

### 1. Isolamento delle Patch

Tutte le modifiche Termux devono essere:

- **Isolate** in file separati quando possibile
- **Condizionali** (`if (isTermux())`)
- **Additive** piuttosto che modificative
- **Documentate** con commenti `// TERMUX PATCH:`

### 2. File Strategy

| Tipo            | Approccio   | Conflitti Merge |
| --------------- | ----------- | --------------- |
| **Nuovi file**  | Preferito   | Nessuno         |
| **Edit minime** | Accettabile | Rari            |
| **Refactor**    | Evitare     | Frequenti       |

### 3. Struttura Patch-Friendly

```
gemini-cli-termux/
├── packages/core/src/
│   ├── utils/
│   │   └── termux-detect.ts     # NUOVO - no conflitti
│   └── index.ts                  # EDIT minima - 1 riga export
├── scripts/
│   ├── postinstall.js           # NUOVO - no conflitti
│   ├── termux-setup.sh          # NUOVO - no conflitti
│   └── termux-tools/            # NUOVO - no conflitti
│       ├── discovery.sh
│       └── call.sh
├── esbuild.config.js            # EDIT - banner (isolato)
├── package.json                 # EDIT - scripts.postinstall
└── Makefile                     # EDIT - target aggiuntivi
```

---

## Procedura Merge Upstream

### Step 1: Fetch upstream

```bash
cd ~/Dev/gemini-cli-termux
git remote add upstream https://github.com/google-gemini/gemini-cli.git 2>/dev/null || true
git fetch upstream
```

### Step 2: Crea branch di merge

```bash
git checkout -b merge-upstream-vX.Y.Z
git merge upstream/main --no-commit
```

### Step 3: Risolvi conflitti (se presenti)

File con conflitti probabili:

- `package.json` - Risolvere mantenendo nostri scripts
- `esbuild.config.js` - Risolvere mantenendo nostro banner
- `packages/core/src/index.ts` - Risolvere mantenendo nostri export

### Step 4: Verifica patch intatte

```bash
# Check che i nostri file esistano ancora
ls -la packages/core/src/utils/termux-detect.ts
ls -la scripts/postinstall.js
ls -la scripts/termux-setup.sh
ls -la scripts/termux-tools/

# Check che le modifiche siano presenti
grep "TERMUX__PREFIX" esbuild.config.js
grep "postinstall" package.json
grep "termux-detect" packages/core/src/index.ts
```

### Step 5: Build test

```bash
npm install --ignore-optional --ignore-scripts
npm run build
npm run bundle
node bundle/gemini.js --version
```

### Step 6: Commit merge

```bash
git add .
git commit -m "merge: upstream vX.Y.Z + Termux patches"
```

---

## Conflitti Comuni e Soluzioni

### package.json

**Conflitto tipico**: `scripts` section modificata upstream

**Soluzione**:

```json
{
  "scripts": {
    // ... upstream scripts ...
    "postinstall": "node scripts/postinstall.js || true" // TERMUX PATCH
  }
}
```

### esbuild.config.js

**Conflitto tipico**: `banner` modificato upstream

**Soluzione**: Mantenere il nostro banner con commento

```javascript
banner: {
  js: `/* UPSTREAM BANNER */
// TERMUX PATCH START
if (process.platform === 'android' && process.env.PREFIX && !process.env.TERMUX__PREFIX) {
  process.env.TERMUX__PREFIX = process.env.PREFIX;
}
// TERMUX PATCH END
`,
},
```

### packages/core/src/index.ts

**Conflitto tipico**: Export aggiunti/modificati upstream

**Soluzione**: Aggiungere il nostro export alla fine

```typescript
// ... upstream exports ...

// TERMUX PATCH
export * from './utils/termux-detect.js';
```

---

## Automazione Merge Check

### Script di verifica post-merge

**File**: `scripts/check-termux-patches.sh`

```bash
#!/bin/bash
# Verifica che le patch Termux siano intatte dopo un merge

set -e

echo "=== Checking Termux Patches ==="

ERRORS=0

# Check file esistenza
FILES=(
  "packages/core/src/utils/termux-detect.ts"
  "scripts/postinstall.js"
  "scripts/termux-setup.sh"
  "scripts/termux-tools/discovery.sh"
  "scripts/termux-tools/call.sh"
)

for f in "${FILES[@]}"; do
  if [ -f "$f" ]; then
    echo "✓ $f exists"
  else
    echo "✗ $f MISSING"
    ERRORS=$((ERRORS + 1))
  fi
done

# Check contenuti chiave
if grep -q "TERMUX__PREFIX" esbuild.config.js; then
  echo "✓ esbuild.config.js has TERMUX patch"
else
  echo "✗ esbuild.config.js MISSING TERMUX patch"
  ERRORS=$((ERRORS + 1))
fi

if grep -q "postinstall" package.json; then
  echo "✓ package.json has postinstall"
else
  echo "✗ package.json MISSING postinstall"
  ERRORS=$((ERRORS + 1))
fi

if grep -q "termux-detect" packages/core/src/index.ts; then
  echo "✓ core/index.ts has termux-detect export"
else
  echo "✗ core/index.ts MISSING termux-detect export"
  ERRORS=$((ERRORS + 1))
fi

echo ""
if [ $ERRORS -eq 0 ]; then
  echo "=== All patches intact ==="
  exit 0
else
  echo "=== $ERRORS patches missing/broken ==="
  exit 1
fi
```

### Git Hook (opzionale)

**File**: `.husky/post-merge`

```bash
#!/bin/bash
bash scripts/check-termux-patches.sh || echo "WARNING: Termux patches need attention!"
```

---

## Tracking Upstream Changes

### File da monitorare

| File Upstream                | Impatto | Azione             |
| ---------------------------- | ------- | ------------------ |
| `package.json`               | Alto    | Verificare scripts |
| `esbuild.config.js`          | Alto    | Verificare banner  |
| `packages/core/src/index.ts` | Medio   | Verificare export  |
| `packages/core/src/tools/*`  | Basso   | Nessuna azione     |
| `packages/cli/*`             | Basso   | Nessuna azione     |

### Changelog Tracking

Mantenere nota delle versioni upstream integrate:

```
docs/termux-api/UPSTREAM_TRACKING.md
```

```markdown
# Upstream Tracking

| Version        | Date       | Notes              |
| -------------- | ---------- | ------------------ |
| 0.21.0-nightly | 2025-12-12 | Initial fork base  |
| 0.22.0-nightly | 2025-12-17 | Synced, patches ok |
```

---

## Minimizzare Conflitti Futuri

### DO

- ✅ Creare nuovi file invece di modificare esistenti
- ✅ Usare funzioni wrapper invece di modificare funzioni esistenti
- ✅ Aggiungere alla fine dei file invece che nel mezzo
- ✅ Usare commenti `// TERMUX PATCH` per identificare modifiche
- ✅ Mantenere le modifiche atomiche e isolate

### DON'T

- ❌ Rinominare variabili/funzioni upstream
- ❌ Ristrutturare codice upstream
- ❌ Modificare logica core esistente
- ❌ Aggiungere dipendenze che richiedono build native
- ❌ Rimuovere codice upstream (solo aggiungere condizionali)

---

## Recovery da Merge Fallito

Se un merge crea troppi conflitti:

```bash
# Abort merge
git merge --abort

# Cherry-pick nostri commit invece
git log --oneline | head -20  # Trova commit Termux
git checkout -b manual-merge upstream/main
git cherry-pick <commit1> <commit2> ...

# Oppure re-apply patch manualmente
git diff main~5..main > termux-patches.diff
git checkout upstream/main
git apply termux-patches.diff
```

---

_Author: DioNanos_
