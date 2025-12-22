# Termux-API Integration Plan

**Project**: gemini-cli-termux **Version**: 0.22.0-termux **Author**: DioNanos
**Date**: 2025-12-17 **Status**: Planning Phase

---

## Executive Summary

This document describes the plan to integrate native Termux-API commands into
the `gemini-cli-termux` fork, allowing Gemini CLI to leverage Android hardware
and software APIs through Termux.

## Table of Contents

1. [Current Architecture](#current-architecture)
2. [Termux-API Commands](#termux-api-commands)
3. [Integration Approaches](#integration-approaches)
4. [Recommendation](#recommendation)
5. [Implementation Roadmap](#implementation-roadmap)
6. [Reference Files](#reference-files)

---

## Current Architecture

### Monorepo Structure

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

### Tool System

Tools in Gemini CLI follow a well-defined pattern:

1. **BaseDeclarativeTool**: Base class for defining tools
2. **BaseToolInvocation**: Class for tool execution
3. **ToolRegistry**: Registers and manages all tools

**Key files**:

- `packages/core/src/tools/tools.ts` - Interfaces and base classes
- `packages/core/src/tools/tool-registry.ts` - Tool registry
- `packages/core/src/tools/shell.ts` - Example: ShellTool

### Tool Call Flow

```
LLM → ToolRegistry.getTool() → DeclarativeTool.build() → ToolInvocation.execute()
```

### Discovery Mechanisms

1. **Built-in Tools**: Manually registered in the registry
2. **Discovered Tools**: Via `tool_discovery_command` in config
3. **MCP Tools**: Via Model Context Protocol servers

---

## Termux-API Commands

### Categorization by Functionality

| Category           | Commands                                                                                                 | Complexity      |
| ------------------ | -------------------------------------------------------------------------------------------------------- | --------------- |
| **System Info**    | battery-status, audio-info, wifi-connectioninfo, wifi-scaninfo, telephony-deviceinfo, telephony-cellinfo | Low             |
| **Notifications**  | notification, notification-remove, notification-list, toast                                              | Low             |
| **Clipboard**      | clipboard-get, clipboard-set                                                                             | Low             |
| **Media**          | camera-photo, camera-info, microphone-record, media-player, media-scan, tts-speak, speech-to-text        | Medium          |
| **Location**       | location                                                                                                 | Medium          |
| **Sensors**        | sensor, infrared-frequencies, infrared-transmit, torch, vibrate, brightness                              | Medium          |
| **Communication**  | sms-send, sms-inbox, sms-list, telephony-call, call-log, contact-list                                    | High (privacy)  |
| **Storage**        | storage-get, download, share, open, open-url, saf-\*                                                     | Medium          |
| **Security**       | fingerprint, keystore                                                                                    | High (security) |
| **System Control** | volume, wake-lock, wake-unlock, wallpaper, wifi-enable                                                   | Medium          |
| **Dialogs**        | dialog                                                                                                   | Medium          |
| **NFC**            | nfc                                                                                                      | High            |
| **USB**            | usb                                                                                                      | High            |
| **Job Scheduler**  | job-scheduler                                                                                            | Medium          |

### Priority Commands (Phase 1)

1. **termux-battery-status** - Battery info (JSON output)
2. **termux-clipboard-get/set** - Clipboard operations
3. **termux-toast** - Toast notifications
4. **termux-notification** - Persistent notifications
5. **termux-tts-speak** - Text-to-Speech
6. **termux-vibrate** - Haptic feedback
7. **termux-torch** - Flashlight control
8. **termux-location** - GPS location
9. **termux-wifi-connectioninfo** - Network info
10. **termux-audio-info** - Audio info

---

## Integration Approaches

### Approach A: Native Dedicated Tools

**Description**: Create dedicated TypeScript classes for each category of Termux
commands.

**Proposed Structure**:

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

**Pros**:

- Deep integration with Gemini
- Type-safe parameter validation
- LLM-optimized descriptions
- Specific error handling
- Granular user confirmation

**Cons**:

- Much code to write (~50 tools)
- Ongoing maintenance
- Strong coupling

**Estimated Effort**: High (2-3 weeks)

---

### Approach B: MCP Server for Termux-API

**Description**: Create a standalone MCP server that exposes all Termux commands
as MCP tools.

**Proposed Structure**:

```
termux-mcp-server/
├── package.json
├── src/
│   ├── index.ts          # MCP server entry
│   ├── tools/            # Tool definitions
│   └── utils/            # Helper functions
└── README.md
```

**Configuration**:

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

**Pros**:

- Reusable with other MCP clients
- Separation of concerns
- Easy to update independently
- Widely supported MCP standard
- Publishable to npm separately

**Cons**:

- Communication overhead
- Separate process dependency
- More complex debugging

**Estimated Effort**: Medium (1-2 weeks)

---

### Approach C: Tool Discovery Script

**Description**: Create a script that generates FunctionDeclarations for Termux
commands, leveraging the existing tool discovery mechanism.

**Implementation**:

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

**Configuration**:

```json
// settings.json
{
  "tool_discovery_command": "bash ~/.config/gemini/termux-tool-discovery.sh",
  "tool_call_command": "bash ~/.config/gemini/termux-tool-call.sh"
}
```

**Pros**:

- Leverages existing infrastructure
- Zero core modifications
- User configurable
- Easy to extend

**Cons**:

- Less control over validation
- Depends on external scripts
- Limited error handling

**Estimated Effort**: Low (3-5 days)

---

### Approach D: Shell Allowlist Extension

**Description**: Extend shell permissions to auto-approve `termux-*` commands.

**Implementation**:

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

**Pros**:

- Minimal code impact
- Uses existing ShellTool
- Quick win

**Cons**:

- No additional semantics
- LLM must know the syntax
- No parameter validation
- No description for LLM

**Estimated Effort**: Minimal (1-2 days)

---

### Approach E: Hybrid (Recommended)

**Description**: Combine approaches B and C for maximum flexibility.

**Phase 1**: Tool Discovery Script (quick win)

- Generates declarations for all commands
- Allows Gemini to use Termux immediately

**Phase 2**: MCP Server (production)

- Implements complete MCP server
- Robust validation
- Publishable to npm

**Phase 3**: Native Tools (optional)

- Only for critical/frequent commands
- Optimized integration

---

## Recommendation

**Recommended Approach: E (Hybrid)**

### Rationale

1. **Quick Win**: Tool Discovery allows starting immediately
2. **Scalability**: MCP Server is the standard for extensions
3. **Flexibility**: Native tools only where needed
4. **Maintainability**: Each phase can be developed independently

### Implementation Priority

| Phase | Approach         | Commands                        | Priority |
| ----- | ---------------- | ------------------------------- | -------- |
| 1     | Discovery Script | All                             | High     |
| 2     | MCP Server       | System, Clipboard, Notification | Medium   |
| 3     | Native Tools     | TTS, Location                   | Low      |

---

## Implementation Roadmap

### Phase 1: Tool Discovery (Quick Win)

**Files to create**:

- `scripts/termux-tool-discovery.sh`
- `scripts/termux-tool-call.sh`
- `docs/termux-api/DISCOVERY_SETUP.md`

**Tasks**:

1. [ ] Create discovery script with all FunctionDeclarations
2. [ ] Create call script with command dispatch
3. [ ] Document user configuration
4. [ ] Test on Termux
5. [ ] Update README

### Phase 2: MCP Server

**Files to create**:

- New package `packages/termux-mcp/`
- Or separate repository `termux-mcp-server`

**Tasks**:

1. [ ] Scaffold MCP server
2. [ ] Implement System tools (battery, wifi, audio)
3. [ ] Implement Clipboard tools
4. [ ] Implement Notification tools
5. [ ] Implement Media tools
6. [ ] Test integration
7. [ ] Publish to npm

### Phase 3: Native Tools (Optional)

**Files to modify**:

- `packages/core/src/tools/` - New tool files
- `packages/core/src/index.ts` - Export tools

**Tasks**:

1. [ ] Implement TermuxTTSTool
2. [ ] Implement TermuxLocationTool
3. [ ] Implement TermuxClipboardTool
4. [ ] Register tools in registry
5. [ ] Test and documentation

---

## Reference Files

### Core Architecture

| File                                       | Description            |
| ------------------------------------------ | ---------------------- |
| `packages/core/src/tools/tools.ts`         | Base tool interfaces   |
| `packages/core/src/tools/tool-registry.ts` | Registry and discovery |
| `packages/core/src/tools/shell.ts`         | Example ShellTool      |
| `packages/core/src/tools/mcp-tool.ts`      | MCP tool wrapper       |
| `packages/core/src/tools/mcp-client.ts`    | MCP client             |

### Configuration

| File                                   | Description     |
| -------------------------------------- | --------------- |
| `packages/core/src/config/config.ts`   | Config loader   |
| `packages/core/src/config/settings.ts` | Settings schema |

### Existing Documentation

| File             | Description      |
| ---------------- | ---------------- |
| `docs/TERMUX.md` | Setup Termux     |
| `README.md`      | Project Overview |

---

## Appendices

- [COMMANDS.md](./COMMANDS.md) - Termux-API commands detail
- [DISCOVERY_SETUP.md](./DISCOVERY_SETUP.md) - Tool Discovery setup guide
- [MCP_SERVER.md](./MCP_SERVER.md) - MCP Server specifications

---

_Author: DioNanos_
