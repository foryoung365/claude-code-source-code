# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the **Claude Code v2.1.88** source code, extracted from the npm package `@anthropic-ai/claude-code`. It is a TypeScript/React-based CLI application that provides an AI-powered programming assistant.

**Important**: This source is for technical research and educational purposes. Commercial use is strictly prohibited. 108 feature-gated internal modules are missing from this decompiled source.

## Technology Stack

- **Language**: TypeScript (target: ES2022, module: ESNext)
- **UI Framework**: React + Ink (terminal UI)
- **Build Target**: Bun (with esbuild fallback)
- **Runtime**: Node.js >= 18
- **Package Manager**: npm

## Build Commands

```bash
# Prepare source (patch Bun-specific imports, replace MACROs)
npm run prepare-src

# Full build (prepare + bundle with esbuild)
npm run build

# Type check only (no emit)
npm run check

# Run the built CLI
npm start
node dist/cli.js
```

## Architecture Overview

### Entry Points
- `src/entrypoints/cli.tsx` - CLI bootstrap, handles --version, --daemon-worker flags
- `src/entrypoints/mcp.ts` - MCP server entry
- `src/main.tsx` - Main application initialization (808KB, 4,683 lines)

### Core Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  ENTRY LAYER                                                │
│  cli.tsx ──> main.tsx ──> REPL.tsx (interactive)           │
│                    └──> QueryEngine.ts (headless/SDK)       │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  QUERY ENGINE                                               │
│  - query.ts (785KB) - Main agent loop                       │
│  - QueryEngine.ts - Headless query lifecycle                │
│  - StreamingToolExecutor.ts - Parallel tool execution       │
└─────────────────────────────────────────────────────────────┘
                              │
          ┌───────────────────┼───────────────────┐
          ▼                   ▼                   ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│  TOOL SYSTEM    │ │  SERVICE LAYER  │ │   STATE LAYER   │
│                 │ │                 │ │                 │
│ Tool Interface  │ │ api/claude.ts   │ │ AppState Store  │
│  ├─ call()      │ │  API client     │ │  ├─ permissions │
│  ├─ validate()  │ │ compact/        │ │  ├─ fileHistory │
│  ├─ checkPerms()│ │  auto-compact   │ │  ├─ agents      │
│  └─ prompt()    │ │ mcp/            │ │  └─ fastMode    │
│                 │ │  MCP protocol   │ │                 │
│ 40+ Tools:      │ │ analytics/      │ │ React Context   │
│  ├─ AgentTool   │ │  telemetry      │ │  ├─ useAppState │
│  ├─ BashTool    │ │ tools/          │ │  └─ useSetState │
│  ├─ FileRead    │ │  executor       │ │                 │
│  ├─ FileEdit    │ │ plugins/        │ └─────────────────┘
│  ├─ Glob/Grep   │ │  loader         │
│  └─ MCPTool     │ │ settingsSync/   │
└─────────────────┘ └─────────────────┘
```

### Key Directories

| Directory | Purpose |
|-----------|---------|
| `src/entrypoints/` | Application entry points (CLI, MCP, SDK) |
| `src/commands/` | 80+ slash command implementations |
| `src/tools/` | 40+ tool implementations (Bash, File, Agent, etc.) |
| `src/components/` | React/Ink terminal UI components |
| `src/services/` | Business logic (API, MCP, analytics, plugins) |
| `src/state/` | Application state management (AppState, ConversationState) |
| `src/tasks/` | Task execution framework (LocalShell, LocalAgent, RemoteAgent) |
| `src/utils/` | Utility functions (git, permissions, sandbox, telemetry) |
| `src/bridge/` | Claude Desktop / IDE bridge integration |
| `src/skills/` | Skill system implementation |

### Tool System

Tools are defined in `src/Tool.ts` and registered in `src/tools.ts`. Each tool implements:
- `call()` - Execute the tool
- `validate()` - Validate input parameters
- `checkPerms()` - Check permissions
- `render()` - Render UI progress
- `prompt()` - Add to system prompt

### Feature Flags

The codebase uses Bun's `feature()` compile-time intrinsic for dead code elimination:

```typescript
if (feature('KAIROS')) {
  // This code is eliminated from npm builds (returns false)
  // Only exists in Anthropic's internal monorepo
}
```

The `scripts/build.mjs` replaces `feature('X')` with `false` and `MACRO.X` with string literals during build.

### 12 Progressive Harness Mechanisms

The codebase implements a layered harness system:
1. **Tool Permissions** - 4 permission modes (auto, suggest, confirm, auto-edit)
2. **Bash Safety** - Command semantic analysis, read-only verification
3. **Auto Mode** - Progressive auto-approval
4. **Advisor System** - Model suggestions with human verification
5. **File State Cache** - Read/write tracking
6. **Denial Tracking** - Permission rejection recording
7. **Subagent Tool** - Child agent spawning
8. **Task System** - Local/remote task queues
9. **Compact Mode** - Context compression
10. **Tool Search** - Dynamic tool discovery
11. **Agent Swarms** - Multi-agent coordination
12. **Plan Mode** - Structured plan execution

## Important Implementation Details

### Build Limitations

- **108 modules are missing** - Feature-gated internal modules (DAEMON, KAIROS, PROACTIVE, etc.) are not in the npm package
- **Bun-specific features** - `feature()`, `MACRO.X`, `bun:bundle` are replaced at build time
- **Best-effort build** - Uses esbuild instead of Bun; some features may not work

### Key Files to Understand

| File | Purpose |
|------|---------|
| `src/main.tsx` | Application bootstrap and REPL initialization |
| `src/query.ts` | Main agent loop (largest file at 785KB) |
| `src/Tool.ts` | Tool interface definition |
| `src/tools.ts` | Tool registry and configuration |
| `src/commands.ts` | Command routing |
| `src/QueryEngine.ts` | Headless query engine for SDK |

### Testing

There are **no test files** in this repository. This is decompiled source code from the npm package, which excludes test files.

### Stubs

The `stubs/` directory provides replacements for Bun-specific APIs:
- `stubs/bun-bundle.ts` - `feature()` returns false
- `stubs/macros.ts` - MACRO constants
- `stubs/global.d.ts` - Global type declarations

## Development Workflow

1. **Make changes** to source files in `src/`
2. **Build**: `npm run build` (prepare-src + esbuild bundle)
3. **Test manually**: `node dist/cli.js --version`
4. **Type check**: `npm run check`

Note: The build process creates a `build-src/` directory with transformed source (feature flags replaced, imports patched) and outputs the bundle to `dist/cli.js`.
