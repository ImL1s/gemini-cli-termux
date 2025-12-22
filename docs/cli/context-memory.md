# Context Memory (Termux fork)

The Termux build ships a deterministic, merge-safe context memory layer stored
at `~/.gemini/context_memory/`:

- `base.json` (read-mostly): stable facts. Writes are blocked unless you enable
  **Allow Base Memory Writes** in `/settings → Context Memory`.
- `user.json` (snapshot): compact state regenerated from the journal (key-based
  upsert, TTL/ephemeral guardrails, sensitivity=high excluded from autoload).
- `user.journal.jsonl` (append-only): every `/save_memory` appends here;
  compaction processes only new bytes.

Bootstrap: if `user.json` is missing and `GEMINI.md` exists, the CLI seeds a
bootstrap entry into `user.json` on first load.

Settings (all default ON unless noted):

- **Enable Context Memory**: master toggle.
- **Primary Memory**: ordering among `GEMINI.md`, `JSON Base`, `JSON User`.
- **Auto-load** flags: per-source autoload at session start.
- **Allow Base Memory Writes**: OFF by default. When ON,
  `/save_memory {"fact":"...", "target":"base"}` writes to `base.json`;
  otherwise it is blocked.

TTS policy:

- `/settings → Notifications → Enable TTS Notifications` (default ON). When OFF,
  any `termux-tts-speak` invocation is blocked at tool level.

Storage notes:

- Dir `~/.gemini/context_memory` is created 0700; files 0600.
- Snapshot limits: 50 entries, ~20KB; journal rotation at ~2MB.
- Keyless entries are accepted only if `expiresAt` is set or `tags` include
  `ephemeral`.
