# Client vs Server — Rules

## Script Types and Locations
| Script Type | Runs On | Valid Locations |
|---|---|---|
| Script | Server | ServerScriptService, ServerStorage, Workspace |
| LocalScript | Client | StarterPlayerScripts, StarterCharacterScripts, StarterGui, PlayerGui |
| ModuleScript | Wherever required from | ReplicatedStorage (shared), ServerStorage (server only), StarterPlayerScripts (client only) |

## What Each Side Can Access
**Server CAN:** Everything. Workspace, all players, DataStores, ServerStorage, ServerScriptService, create/destroy anything.

**Server CANNOT:** LocalPlayer, UserInputService, Camera, ContextActionService, any client-only API.

**Client CAN:** LocalPlayer, PlayerGui, UserInputService, Camera, ReplicatedStorage, Workspace (read mostly).

**Client CANNOT:** ServerStorage, ServerScriptService, DataStores, other players' private data, modify server-authoritative state.

## Which Side Handles What
| System | Runs On | Why |
|---|---|---|
| GUI display/interaction | Client | Server has no screen |
| Player input (keys, mouse, touch) | Client | Server has no input |
| Data saving (DataStore) | Server | Client can't access DataStores |
| Currency/stats (authoritative) | Server | Client can't be trusted |
| Damage/health | Server | Prevent damage exploits |
| Inventory (authoritative) | Server | Client can't be trusted |
| Shop purchases | Both | Client requests, server validates and executes |
| Teleportation | Server | Prevent teleport exploits |
| NPC AI / pathfinding | Server | Authoritative game state |
| Leaderboards | Server | DataStore access |
| Admin commands | Server | Security |
| Visual effects / particles | Client | Performance, only local player needs to see |
| Camera control | Client | Each player has their own camera |
| Sound (local) | Client | Each player hears their own audio |

## The Golden Rule
The server is the ONLY source of truth. The client is just a display and input device. If the client says "I have 999999 coins" — ignore it. Check the server's data.
