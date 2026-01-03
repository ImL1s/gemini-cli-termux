# Merge Strategy - Upstream Sync

**Project**: gemini-cli-termux **Author**: DioNanos **Date**: 2025-12-17

---

## Overview

Strategy for maintaining Termux patches easily applicable after each merge with
the upstream `google-gemini/gemini-cli`.

---

## Design Principles

### 1. Patch Isolation

All Termux modifications must be:

- **Isolated** in separate files when possible
- **Conditional** (`if (isTermux())`)
- **Additive** rather than modifying existing code
- **Documented** with `// TERMUX PATCH:` comments

### 2. File Strategy

| Type             | Approach   | Merge Conflicts |
| ---------------- | ---------- | --------------- |
| **New files**    | Preferred  | None            |
| **Minimal edit** | Acceptable | Rare            |
| **Refactor**     | Avoid      | Frequent        |

### 3. Patch-Friendly Structure

```
gemini-cli-termux/
├── packages/core/src/
│   ├── utils/
│   │   └── termux-detect.ts     # NEW - no conflicts
│   │   └── contextMemory.ts     # NEW - Termux context memory logic
│   └── index.ts                  # Minimal EDIT - 2 export lines
├── scripts/
│   ├── postinstall.js           # NEW - no conflicts
│   ├── termux-setup.sh          # NEW - no conflicts
│   └── termux-tools/            # NEW - no conflicts
│       ├── discovery.sh
│       └── call.sh
├── esbuild.config.js            # EDIT - banner (isolated)
├── package.json                 # EDIT - scripts.postinstall, dist-tags
└── Makefile                     # EDIT - additional targets
```

---

## Upstream Merge Procedure

### Step 1: Branch Management

We use a "Sync Branch" strategy to isolate upstream merges from the stable
`main` branch.

1.  **Fetch Upstream**:

    ```bash
    git fetch upstream
    ```

2.  **Create Sync Branch**: Name format: `sync/v<version>` (e.g.,
    `sync/v0.24.0`) or `merge-upstream-v<version>`.
    ```bash
    git checkout -b sync/v0.24.0 upstream/main
    ```

### Step 2: Apply Termux Patches

Since we start fresh from upstream, we must restore our specific configuration.

1.  **Merge `main` into Sync Branch** (Optional but recommended if `main` has
    unreleased fixes):

    ```bash
    # Only if there are specific hotfixes in main not present in upstream
    git merge main
    ```

    _Alternatively, manually re-apply patches if starting clean from upstream is
    preferred._

2.  **Resolve Conflicts**: When merging, prioritize Termux-specific files:
    - **`package.json`**: Restore `name`, `private: false`, `repository`, and
      `scripts.postinstall`.
    - **`esbuild.config.js`**: Ensure the `TERMUX PATCH` banner is present.
    - **`packages/core/src/index.ts`**: Ensure `termux-detect` is exported.

3.  **Restore Termux Files**: If starting from a clean upstream checkout,
    restore these files from `main`:
    ```bash
    git checkout main -- scripts/postinstall.cjs \
                        scripts/termux-setup.sh \
                        scripts/termux-tools/ \
                        scripts/check-termux-patches.sh \
                        packages/core/src/utils/termux-detect.ts
    ```

### Step 3: Verification

1.  **Run Patch Check**:

    ```bash
    bash scripts/check-termux-patches.sh
    ```

    _Must report "All Termux patches intact"._

2.  **Build**:

    ```bash
    npm install
    npm run build
    npm run bundle
    ```

3.  **Verify Version**:
    ```bash
    node bundle/gemini.js --version
    # Output should match target version (e.g., 0.24.0-termux)
    ```

### Step 4: Finalize Release

1.  **Merge to Main**:

    ```bash
    git checkout main
    git merge sync/v0.24.0 --no-ff
    ```

2.  **Tag Release**:

    ```bash
    git tag v0.24.0-termux
    ```

3.  **Push**:
    ```bash
    git push origin main --tags
    ```

---

## Common Conflicts and Solutions

### package.json

**Typical conflict**: `scripts` section modified upstream

**Solution**:

```json
{
  "scripts": {
    // ... upstream scripts ...
    "postinstall": "node scripts/postinstall.js || true" // TERMUX PATCH
  }
}
```

### esbuild.config.js

**Typical conflict**: `banner` modified upstream

**Solution**: Keep our banner with comment

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

**Typical conflict**: Exports added/modified upstream

**Solution**: Add our export at the end

```typescript
// ... upstream exports ...

// TERMUX PATCH
export * from './utils/termux-detect.js';
export * from './utils/contextMemory.js';

### Context memory safety

- Keep `contextMemory.ts` additive; if upstream adds similar logic, retain
  Termux fields `allowBaseWrite`, runtime setters, key/TTL guards, sensitivity
  filter, and incremental journal processing.

### Shell / TTS guard

- `packages/core/src/tools/shell.ts` contains a guard to block
  `termux-tts-speak` when `notifications.ttsEnabled` is false. Re-apply after
  merges if the shell tool changes.
```

---

## Merge Check Automation

### Post-merge verification script

**File**: `scripts/check-termux-patches.sh`

```bash
#!/bin/bash
# Verify that Termux patches are intact after a merge

set -e

echo "=== Checking Termux Patches ==="

ERRORS=0

# Check file existence
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

# Check key contents
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

### Git Hook (optional)

**File**: `.husky/post-merge`

```bash
#!/bin/bash
bash scripts/check-termux-patches.sh || echo "WARNING: Termux patches need attention!"
```

---

## Tracking Upstream Changes

### Files to monitor

| Upstream File                | Impact | Action         |
| ---------------------------- | ------ | -------------- |
| `package.json`               | High   | Verify scripts |
| `esbuild.config.js`          | High   | Verify banner  |
| `packages/core/src/index.ts` | Medium | Verify exports |
| `packages/core/src/tools/*`  | Low    | No action      |
| `packages/cli/*`             | Low    | No action      |

### Changelog Tracking

Keep note of integrated upstream versions:

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

## Minimizing Future Conflicts

### DO

- ✅ Create new files instead of modifying existing ones
- ✅ Use wrapper functions instead of modifying existing ones
- ✅ Add to the end of files instead of the middle
- ✅ Use `// TERMUX PATCH` comments to identify modifications
- ✅ Keep changes atomic and isolated

### DON'T

- ❌ Rename upstream variables/functions
- ❌ Restructure upstream code
- ❌ Modify existing core logic
- ❌ Add dependencies that require native build
- ❌ Remove upstream code (only add conditionals)

---

## Recovery from Failed Merge

If a merge creates too many conflicts:

```bash
# Abort merge
git merge --abort

# Cherry-pick our commits instead
git log --oneline | head -20  # Find Termux commits
git checkout -b manual-merge upstream/main
git cherry-pick <commit1> <commit2> ...

# Or re-apply patch manually
git diff main~5..main > termux-patches.diff
git checkout upstream/main
git apply termux-patches.diff
```

---

_Author: DioNanos_
