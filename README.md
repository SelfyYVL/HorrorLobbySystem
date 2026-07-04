# MireBorn — Lobby System

Lobby system for **MireBorn**, a horror game on Roblox. Handles player waiting rooms before a match starts, with real-time state synchronization between server and client.

![MireBorn Menu]((ssets/menu.png)

## Overview

- Groups players into lobbies, each identified by a unique `LobbyId`
- Automatic countdown to match start once a lobby reaches the minimum player threshold
- Leave-lobby GUI driven by `RemoteEvent`, with instant updates across all clients
- Lobby state synced through instance `Attributes` instead of global variables, avoiding client/server mismatches

## Architecture

```
src/
├── server/
│   └── LobbyClientHandlerInit.lua     -- creates/manages lobbies, countdown, matchmaking
├── client/
│   └── GuiMainMenuHandler.lua         -- main menu (Play / Credits), user input
└── shared/
    └── LobbyModule.lua                -- Lobby class (OOP with metatables), shared API
```

### Client/server flow

1. The client presses **Play** → `GuiMainMenuHandler` sends a request to the server via `RemoteEvent`
2. The server creates or assigns the player to a `Lobby` (an OOP object built with metatables) and sets `LobbyId` as an attribute on the player
3. The server updates a `LobbyId` attribute on the player instead of a boolean `InGame` flag — this avoids a bug where a player was still marked as "in game" after leaving the lobby, since presence is now checked by verifying the existence of `LobbyId` rather than reading a binary state
4. Once a lobby hits the minimum player count, a countdown starts and is replicated to all clients via `RemoteEvent`
5. If a player leaves, the leave GUI fires an event that removes the player from the lobby and propagates the update to the other clients

## Technical details

- **OOP with metatables**: each lobby is an instance of the `Lobby` class, exposing methods like `AddPlayer`, `RemovePlayer`, `StartCountdown`
- **Attribute-based sync**: lobby state (`LobbyId`, player count, countdown active) is exposed via `Instance:SetAttribute()` so the client can read it directly without polling the server
- **RemoteEvent-driven UI**: the leave GUI reacts only to received events, no polling involved

## Notes

Solo-developed as part of MireBorn, a horror game currently in development on Roblox. The code here was extracted directly from Roblox Studio (no Rojo), organized manually to reflect the actual client/server/shared architecture of the game.
