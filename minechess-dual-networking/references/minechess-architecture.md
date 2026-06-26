# MineChess dual-networking architecture

## What exists already

MineChess is not a single-network prototype anymore. It already has a dual transport design:

- `src/network/ConnectionProvider.tsx`
  Chooses `peerjs` or `photon` from local storage or `VITE_NETWORK_MODE`.
- `src/network/PeerJsConnectionProvider.tsx`
  Implements direct peer connections with reconnect logic, pending packet retry, and ACK handling.
- `src/network/PhotonConnectionProvider.tsx`
  Implements Photon room connections, lobby room snapshots, built-in App ID fallback, and reconnect behavior.
- `src/network/protocol.ts`
  Defines the shared packet schema and action packet types.
- `src/components/GameModals.tsx`
  Exposes the mode switch, room create/join flow, Photon-specific room naming, and network-aware error messages.

## Shared interface contract

Both providers are shaped to satisfy the same `ConnectionContextValue` contract. The important fields are:

- `status`
- `isConnected`
- `localPeerId`
- `remotePeerId`
- `lobbyRooms`
- `openPeer`
- `connectToPeer`
- `reconnect`
- `sendPacket`
- `sendActionPacket`
- `sendJson`
- `sendJsonString`
- `setRoomMatchStatus`
- `disconnect`
- `destroyPeer`

This is the main reuse point. If you port this pattern, preserve the contract and swap the game-specific payloads.

## PeerJS behavior

PeerJS mode is the lighter transport:

- creates a peer from env-configured host settings when available
- falls back to the public PeerJS server if the local server is unavailable
- uses direct `peer.connect(...)`
- stores the last opened peer id and target peer id for reconnect
- retries packet delivery with `ACK`, timeout, and max retry tracking
- returns an empty `lobbyRooms` list because PeerJS has no hosted lobby

Relevant environment variables:

- `VITE_PEER_HOST`
- `VITE_PEER_PORT`
- `VITE_PEER_PATH`
- `VITE_PEER_SECURE`

## Photon behavior

Photon mode is the hosted-room transport:

- dynamically imports `photon-realtime`
- connects to region master using `VITE_PHOTON_REGION`
- uses one selected App ID plus backup fallback candidates
- stores preferred App ID in local storage
- creates or joins rooms and publishes room metadata to the lobby
- keeps `roomLabel` and `matchStatus` in custom room properties
- reports room snapshots through `lobbyRooms`
- retries reconnects unless the error is marked fatal

Relevant environment variables:

- `VITE_PHOTON_APP_ID`
- `VITE_PHOTON_APP_ID_BACKUP`
- `VITE_PHOTON_REGION`
- `VITE_PHOTON_APP_VERSION`
- `VITE_PHOTON_PROTOCOL`

MineChess also has built-in fallback App IDs inside the Photon provider, so the app can still function when env values are missing.

## Packet design

The shared packet schema contains:

- `type`
- `matchId`
- `turn`
- `payload`
- `ts`
- `seq`

Important packet types already present:

- handshake and presence: `HELLO`, `AUTH_RESULT`, `PLAYER_READY`
- game start and exit: `START_GAME`, `LEAVE_MATCH`
- turn actions: `MOVE`, `ATTACK`, `SCAN`, `SENSOR_SCAN`, `PLACE_MINE`, `PICKUP_FLAG`, `DROP_FLAG`, `EVOLVE`, `END_TURN`, `SKIP_TURN`
- reliability and recovery: `STATE_SYNC`, `PING`, `PONG`, `ACK`

This is a good Jam pattern: transmit the player's intent and keep `STATE_SYNC` for recovery or authoritative correction.

## UI implications

MineChess treats transport differences as product-level UX, not just implementation detail:

- `peerjs` expects 4-digit peer-style room IDs
- `photon` allows hosted online rooms with predefined room-name choices
- the join/create modal text changes with the selected mode
- Photon shows current App ID context and hosted room information
- error messages are normalized into player-friendly text

If you reuse this pattern, keep the UX split visible. Hiding the mode differences entirely usually creates confusing failure states.
