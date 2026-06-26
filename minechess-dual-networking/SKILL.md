---
name: minechess-dual-networking
description: Build, adapt, or debug a lightweight multiplayer layer for browser games using both PeerJS and Photon with one shared protocol surface. Use when Codex needs to turn a local multiplayer or turn-based web game into a reusable Game Jam-ready online mode, preserve a dual transport setup, wire lobby/create/join flows, or carry forward the MineChess networking pattern into a new project.
---

# Minechess Dual Networking

Use this skill to reuse the MineChess multiplayer pattern instead of rebuilding networking from scratch.

## Quick Start

1. Identify whether the user wants LAN-first play, internet play, or both.
2. Keep one shared game protocol and one shared connection interface.
3. Put transport-specific logic behind separate providers:
   `PeerJS` for simple P2P or LAN flows.
   `Photon` for hosted room discovery, online rooms, and fallback internet play.
4. Keep the game code talking only to the shared provider API and packet schema.
5. Preserve a visible host/join/create flow in the UI and keep error messages mode-specific.

## Workflow

### 1. Decide the transport goal

Use `PeerJS` when:
- players are on the same LAN or can exchange peer IDs manually
- lower setup and direct P2P matters more than lobby discovery
- the project can tolerate peer server quirks or a public fallback

Use `Photon` when:
- players need hosted room discovery or internet-friendly rooms
- the project needs room metadata such as labels, status, and player counts
- reconnect behavior and cloud-hosted rooms matter more than raw simplicity

Use both when:
- the game should support a fast Game Jam LAN demo and a shareable online build
- the UI can expose a mode switch without duplicating game logic

### 2. Preserve the shared abstraction

Keep both transports implementing the same surface:
- connection state
- local and remote player identifiers
- room creation and join flow
- packet send helpers
- reconnect hooks
- lobby room list, even if one mode returns an empty list

Do not let game rules call transport-specific SDK methods directly. Route everything through the connection provider contract.

### 3. Keep the protocol event-driven

Send game actions, not full board snapshots, for the normal turn loop. Use state sync only for recovery, late correction, or debugging.

Mirror the MineChess packet style:
- action type
- match id
- turn number
- payload
- timestamp
- sequence number
- ACK support for reliable delivery

Read [references/minechess-architecture.md](references/minechess-architecture.md) before changing packet structure or transport responsibilities.

### 4. Build the UI around mode switching

Support these user-visible actions:
- switch between `peerjs` and `photon`
- create room
- join room
- view local id or room id
- see current connection status
- retry or reconnect after failure

If adapting a new game, keep UI copy transport-aware. MineChess changes room naming, lobby display, and validation rules depending on the active mode.

### 5. Adapt the pattern to a new Game Jam project

When porting this to a new game:
- map each existing game action to a packet type
- define what data is authoritative on the host or room
- keep transport-independent packet handlers near the game reducer or action pipeline
- keep setup and environment variables minimal enough for Jam teammates to run quickly

Read [references/game-jam-porting-checklist.md](references/game-jam-porting-checklist.md) for the porting sequence.

## Source Pattern

MineChess already demonstrates the important split:
- transport selector in `src/network/ConnectionProvider.tsx`
- PeerJS transport in `src/network/PeerJsConnectionProvider.tsx`
- Photon transport in `src/network/PhotonConnectionProvider.tsx`
- shared packets in `src/network/protocol.ts`
- lobby and mode-specific UI in `src/components/GameModals.tsx`
- environment toggles in `.env.example`

When working on the original project, read those files first before making network changes.

## Guardrails

- Keep transport selection outside the core game logic.
- Keep `PeerJS` and `Photon` using the same packet contract wherever possible.
- Keep reconnect behavior explicit and testable.
- Prefer a small compatibility layer over duplicating host/join logic in multiple components.
- Do not replace event packets with whole-state syncing unless debugging or recovery truly requires it.
- If only one mode is broken, fix that provider first instead of refactoring both.
