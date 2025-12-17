# Termux-API Integration Plan

**Project**: gemini-cli-termux **Version**: 0.22.0-termux **Author**: DioNanos
**Date**: 2025-12-17 **Status**: Planning Phase

---

## Executive Summary

Questo documento descrive il piano per integrare i comandi nativi Termux-API nel
fork `gemini-cli-termux`, permettendo a Gemini CLI di sfruttare le API hardware
e software di Android attraverso Termux.

## Table of Contents

1. [Architettura Attuale](#architettura-attuale)
2. [Comandi Termux-API](#comandi-termux-api)
3. [Approcci di Integrazione](#approcci-di-integrazione)
4. [Raccomandazione](#raccomandazione)
5. [Roadmap Implementazione](#roadmap-implementazione)
6. [File di Riferimento](#file-di-riferimento)

---

## Architettura Attuale

### Struttura Monorepo

```
gemini-cli-termux/
├── packages/
│   ├── core/           # Logic core, tools, config
│   │   └── src/
│   │       ├── tools/  # Tool implementations
│   │       ├── mcp/    # MCP support
│   │       └── config/ # Configuration
│   ├── cli/            # CLI interface
│   ├── a2a-server/     # Agent-to-Agent server
│   └── vscode-ide-companion/
└── bundle/             # Built executable
```

### Sistema Tool

I tools in Gemini CLI seguono un pattern ben definito:

1. **BaseDeclarativeTool**: Classe base per definire tools
2. **BaseToolInvocation**: Classe per l'esecuzione del tool
3. **ToolRegistry**: Registra e gestisce tutti i tools

**File chiave**:

- `packages/core/src/tools/tools.ts` - Interfacce e classi base
- `packages/core/src/tools/tool-registry.ts` - Registry dei tools
- `packages/core/src/tools/shell.ts` - Esempio: ShellTool

### Flusso Tool Call

```
LLM → ToolRegistry.getTool() → DeclarativeTool.build() → ToolInvocation.execute()
```

### Meccanismi di Discovery

1. **Built-in Tools**: Registrati manualmente nel registry
2. **Discovered Tools**: Via `tool_discovery_command` in config
3. **MCP Tools**: Via Model Context Protocol servers

---

## Comandi Termux-API

### Categorizzazione per Funzionalità

| Categoria          | Comandi                                                                                                  | Complessità     |
| ------------------ | -------------------------------------------------------------------------------------------------------- | --------------- |
| **System Info**    | battery-status, audio-info, wifi-connectioninfo, wifi-scaninfo, telephony-deviceinfo, telephony-cellinfo | Bassa           |
| **Notifications**  | notification, notification-remove, notification-list, toast                                              | Bassa           |
| **Clipboard**      | clipboard-get, clipboard-set                                                                             | Bassa           |
| **Media**          | camera-photo, camera-info, microphone-record, media-player, media-scan, tts-speak, speech-to-text        | Media           |
| **Location**       | location                                                                                                 | Media           |
| **Sensors**        | sensor, infrared-frequencies, infrared-transmit, torch, vibrate, brightness                              | Media           |
| **Communication**  | sms-send, sms-inbox, sms-list, telephony-call, call-log, contact-list                                    | Alta (privacy)  |
| **Storage**        | storage-get, download, share, open, open-url, saf-\*                                                     | Media           |
| **Security**       | fingerprint, keystore                                                                                    | Alta (security) |
| **System Control** | volume, wake-lock, wake-unlock, wallpaper, wifi-enable                                                   | Media           |
| **Dialogs**        | dialog                                                                                                   | Media           |
| **NFC**            | nfc                                                                                                      | Alta            |
| **USB**            | usb                                                                                                      | Alta            |
| **Job Scheduler**  | job-scheduler                                                                                            | Media           |

### Comandi Prioritari (Fase 1)

1. **termux-battery-status** - Info batteria (JSON output)
2. **termux-clipboard-get/set** - Clipboard operations
3. **termux-toast** - Notifiche toast
4. **termux-notification** - Notifiche persistenti
5. **termux-tts-speak** - Text-to-Speech
6. **termux-vibrate** - Feedback tattile
7. **termux-torch** - Flashlight control
8. **termux-location** - GPS location
9. **termux-wifi-connectioninfo** - Network info
10. **termux-audio-info** - Audio info

---

## Approcci di Integrazione

### Approccio A: Tools Nativi Dedicati

**Descrizione**: Creare classi TypeScript dedicate per ogni categoria di comandi
Termux.

**Struttura proposta**:

```
packages/core/src/tools/termux/
├── index.ts
├── termux-base.ts
├── termux-system.ts      # battery, wifi, audio, telephony
├── termux-notification.ts # toast, notification
├── termux-clipboard.ts   # clipboard operations
├── termux-media.ts       # camera, microphone, tts, speech
├── termux-location.ts    # GPS
├── termux-sensors.ts     # sensor, torch, vibrate
└── termux-storage.ts     # download, share, open
```

**Pro**:

- Integrazione profonda con Gemini
- Validazione parametri type-safe
- Descrizioni ottimizzate per LLM
- Gestione errori specifica
- Conferma utente granulare

**Contro**:

- Molto codice da scrivere (~50 tools)
- Manutenzione ongoing
- Accoppiamento forte

**Effort stimato**: Alto (2-3 settimane)

---

### Approccio B: MCP Server per Termux-API

**Descrizione**: Creare un MCP server standalone che espone tutti i comandi
Termux come tools MCP.

**Struttura proposta**:

```
termux-mcp-server/
├── package.json
├── src/
│   ├── index.ts          # MCP server entry
│   ├── tools/            # Tool definitions
│   └── utils/            # Helper functions
└── README.md
```

**Configurazione**:

```json
// settings.json
{
  "mcpServers": {
    "termux": {
      "command": "npx",
      "args": ["@mmmbuto/termux-mcp-server"]
    }
  }
}
```

**Pro**:

- Riutilizzabile con altri client MCP
- Separazione delle responsabilità
- Facile da aggiornare indipendentemente
- Standard MCP ampiamente supportato
- Pubblicabile su npm separatamente

**Contro**:

- Overhead di comunicazione
- Dipendenza processo separato
- Debugging più complesso

**Effort stimato**: Medio (1-2 settimane)

---

### Approccio C: Tool Discovery Script

**Descrizione**: Creare uno script che genera FunctionDeclarations per i comandi
Termux, sfruttando il meccanismo di tool discovery esistente.

**Implementazione**:

```bash
# termux-tool-discovery.sh
#!/bin/bash
cat << 'EOF'
[
  {
    "name": "termux_battery_status",
    "description": "Get battery status including percentage, health, and charging state",
    "parametersJsonSchema": {
      "type": "object",
      "properties": {}
    }
  },
  ...
]
EOF
```

**Configurazione**:

```json
// settings.json
{
  "tool_discovery_command": "bash ~/.config/gemini/termux-tool-discovery.sh",
  "tool_call_command": "bash ~/.config/gemini/termux-tool-call.sh"
}
```

**Pro**:

- Sfrutta infrastruttura esistente
- Zero modifiche al core
- Configurabile per utente
- Facile da estendere

**Contro**:

- Meno controllo sulla validazione
- Dipende da script esterni
- Gestione errori limitata

**Effort stimato**: Basso (3-5 giorni)

---

### Approccio D: Shell Allowlist Extension

**Descrizione**: Estendere le shell permissions per auto-approvare comandi
`termux-*`.

**Implementazione**:

```typescript
// packages/core/src/utils/shell-permissions.ts
const TERMUX_COMMANDS = [
  'termux-battery-status',
  'termux-clipboard-get',
  'termux-clipboard-set',
  // ...
];

export function isTermuxCommand(command: string): boolean {
  return TERMUX_COMMANDS.some((tc) => command.startsWith(tc));
}
```

**Pro**:

- Impatto minimo sul codice
- Usa ShellTool esistente
- Quick win

**Contro**:

- Nessuna semantica aggiuntiva
- LLM deve conoscere la sintassi
- Nessuna validazione parametri
- Nessuna descrizione per LLM

**Effort stimato**: Minimo (1-2 giorni)

---

### Approccio E: Ibrido (Raccomandato)

**Descrizione**: Combinare gli approcci B e C per massima flessibilità.

**Fase 1**: Tool Discovery Script (quick win)

- Genera dichiarazioni per tutti i comandi
- Permette a Gemini di usare Termux immediatamente

**Fase 2**: MCP Server (produzione)

- Implementa server MCP completo
- Validazione robusta
- Pubblicabile su npm

**Fase 3**: Tools Nativi (opzionale)

- Solo per comandi critici/frequenti
- Integrazione ottimizzata

---

## Raccomandazione

**Approccio consigliato: E (Ibrido)**

### Motivazioni

1. **Quick Win**: Tool Discovery permette di iniziare subito
2. **Scalabilità**: MCP Server è lo standard per estensioni
3. **Flessibilità**: Tools nativi solo dove serve
4. **Manutenibilità**: Ogni fase può essere sviluppata indipendentemente

### Priorità Implementazione

| Fase | Approccio        | Comandi                         | Priorità |
| ---- | ---------------- | ------------------------------- | -------- |
| 1    | Discovery Script | Tutti                           | Alta     |
| 2    | MCP Server       | System, Clipboard, Notification | Media    |
| 3    | Native Tools     | TTS, Location                   | Bassa    |

---

## Roadmap Implementazione

### Fase 1: Tool Discovery (Quick Win)

**Files da creare**:

- `scripts/termux-tool-discovery.sh`
- `scripts/termux-tool-call.sh`
- `docs/termux-api/DISCOVERY_SETUP.md`

**Tasks**:

1. [ ] Creare script discovery con tutte le FunctionDeclarations
2. [ ] Creare script call con dispatch dei comandi
3. [ ] Documentare configurazione utente
4. [ ] Test su Termux
5. [ ] Aggiornare README

### Fase 2: MCP Server

**Files da creare**:

- Nuovo package `packages/termux-mcp/`
- O repository separato `termux-mcp-server`

**Tasks**:

1. [ ] Scaffold MCP server
2. [ ] Implementare tools System (battery, wifi, audio)
3. [ ] Implementare tools Clipboard
4. [ ] Implementare tools Notification
5. [ ] Implementare tools Media
6. [ ] Test integration
7. [ ] Pubblicare su npm

### Fase 3: Native Tools (Opzionale)

**Files da modificare**:

- `packages/core/src/tools/` - Nuovi tool files
- `packages/core/src/index.ts` - Export tools

**Tasks**:

1. [ ] Implementare TermuxTTSTool
2. [ ] Implementare TermuxLocationTool
3. [ ] Implementare TermuxClipboardTool
4. [ ] Registrare tools in registry
5. [ ] Test e documentazione

---

## File di Riferimento

### Architettura Core

| File                                       | Descrizione           |
| ------------------------------------------ | --------------------- |
| `packages/core/src/tools/tools.ts`         | Interfacce base tools |
| `packages/core/src/tools/tool-registry.ts` | Registry e discovery  |
| `packages/core/src/tools/shell.ts`         | Esempio ShellTool     |
| `packages/core/src/tools/mcp-tool.ts`      | MCP tool wrapper      |
| `packages/core/src/tools/mcp-client.ts`    | MCP client            |

### Configurazione

| File                                   | Descrizione     |
| -------------------------------------- | --------------- |
| `packages/core/src/config/config.ts`   | Config loader   |
| `packages/core/src/config/settings.ts` | Settings schema |

### Documentazione Esistente

| File             | Descrizione       |
| ---------------- | ----------------- |
| `docs/TERMUX.md` | Setup Termux      |
| `README.md`      | Overview progetto |

---

## Appendici

- [COMMANDS.md](./COMMANDS.md) - Dettaglio tutti i comandi Termux-API
- [DISCOVERY_SETUP.md](./DISCOVERY_SETUP.md) - Guida setup Tool Discovery
- [MCP_SERVER.md](./MCP_SERVER.md) - Specifiche MCP Server

---

_Author: DioNanos_
