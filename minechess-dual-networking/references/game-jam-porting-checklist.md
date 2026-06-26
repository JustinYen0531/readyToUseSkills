# Game Jam porting checklist

Use this when applying the MineChess multiplayer model to a new browser game.

## 1. Extract the game actions

List the actions that must cross the network:

- room creation and room join
- ready or setup confirmation
- start game
- turn action packets
- reconnect or leave
- optional resync packet

If the new game has too many mutable fields, reduce them into intent packets before touching the transport.

## 2. Freeze a single connection API

Define one React context or service layer that the rest of the game will call.

Minimum surface:

- open host room
- join room
- send action packet
- watch latest packet
- show connection state
- reconnect
- disconnect

Make both transports match this API, even if one implementation is thinner.

## 3. Add transport selection early

Support one switchable mode field such as:

- local storage override
- `.env` default
- in-UI toggle for testers

Do this before wiring the actual game actions, or the UI and state flow will sprawl.

## 4. Port the transport-specific features deliberately

For `PeerJS`:

- decide whether to use your own peer server or public fallback
- keep peer IDs easy to share during a Jam
- test reconnect after closing one client tab

For `Photon`:

- confirm valid App ID and region
- decide whether room names are free text or curated
- decide which room properties appear in the lobby
- treat authentication and plugin mismatch errors as first-class UX cases

## 5. Keep recovery small but real

Include:

- ACK or equivalent packet confirmation
- reconnect attempt tracking
- one recovery path for re-open or re-join
- one explicit full-state sync packet for correction

Do not build a huge rollback system unless the Jam scope truly needs it.

## 6. Test in the order that saves time

1. open room and join room
2. send hello packet
3. send one gameplay action
4. verify turn number and payload integrity
5. test disconnect and reconnect
6. test mode switch

## 7. Use MineChess as the reference implementation

When you need a working example rather than theory, inspect:

- `src/network/ConnectionProvider.tsx`
- `src/network/PeerJsConnectionProvider.tsx`
- `src/network/PhotonConnectionProvider.tsx`
- `src/network/protocol.ts`
- `src/components/GameModals.tsx`
- `.env.example`

If you are starting from the MineChess codebase itself, preserve those boundaries and adapt payloads before changing architecture.
