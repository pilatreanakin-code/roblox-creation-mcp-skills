---
name: roblox-luau
description: Makes the AI write correct, clean, professional Luau code for Roblox games via the Roblox Studio MCP. Use this skill for ANY Luau scripting task — game systems, server scripts, client scripts, data storage, combat, shops, pets, NPCs, or any Roblox game logic. Triggers on any mention of scripting, coding, Luau, programming for Roblox, game systems, DataStore, RemoteEvent, or any Roblox code request.
---

# Roblox Luau — Professional Game Scripting Skill

---

## BEFORE WRITING ANY CODE

1. Read `references/common-mistakes.md` — know what NOT to do
2. Read `references/client-server.md` — know where code belongs
3. Read `references/mcp-patterns.md` — know how MCP execution works
4. If building a game system (shop, pets, combat, etc.) — read the matching file in `references/systems/`

---

## HARD RULES (every line of code must follow these)

1. **task.wait() not wait(). task.spawn() not spawn(). task.delay() not delay().** Old APIs are deprecated.
2. **Instance.new("ClassName") then set properties, set Parent LAST.** Never use the 2-argument constructor.
3. **pcall() on EVERY DataStore call.** No exceptions. Retry up to 3 times with increasing delay.
4. **NEVER access LocalPlayer from a Script.** LocalPlayer only exists in LocalScripts.
5. **NEVER access ServerStorage from a LocalScript.** Client cannot see it.
6. **NEVER trust client data.** Server validates EVERYTHING — prices, item IDs, distances, cooldowns.
7. **NEVER use while true do without task.wait() inside.** This freezes the thread.
8. **Always use WaitForChild() in startup code.** FindFirstChild can return nil if objects haven't loaded yet.
9. **Always check for nil.** After FindFirstChild, after GetAsync, after any lookup that might fail.
10. **Always disconnect events when done.** Store connections, disconnect on PlayerRemoving or Destroy.
11. **Always use Enum values, not strings.** `Enum.KeyCode.Space` not `"Space"`.
12. **Remotes go in ReplicatedStorage.** Create a Remotes folder. Both client and server reference it.
13. **Server scripts in ServerScriptService. Client scripts in StarterPlayerScripts or StarterGui.** Never reversed.
14. **Use Debris:AddItem() for temporary objects.** Auto-cleanup after N seconds.
15. **Debounce on Touched events and button clicks.** Prevent double-firing.

---

## WHEN BUILDING THROUGH MCP (run_code)

MCP run_code executes in SERVER context (like the command bar). Key rules:

- **To create server logic:** Create a Script instance, set its Source, parent to ServerScriptService
- **To create client logic:** Create a LocalScript instance, set its Source, parent to StarterPlayerScripts or StarterGui
- **Don't run client code directly** — it executes as server and will fail (no LocalPlayer, no UserInputService)
- **To create shared modules:** Create ModuleScript, set Source, parent to ReplicatedStorage
- **To create remotes:** Instance.new("RemoteEvent"), name it, parent to ReplicatedStorage.Remotes

```lua
-- Creating a server script via MCP
local script = Instance.new("Script")
script.Name = "ShopHandler"
script.Source = [[ 
    -- server code here
]]
script.Parent = game:GetService("ServerScriptService")

-- Creating a client script via MCP
local ls = Instance.new("LocalScript")
ls.Name = "ShopGUI"
ls.Source = [[ 
    -- client code here
]]
ls.Parent = game:GetService("StarterGui")
```

---

## CODE STRUCTURE FOR GAME SYSTEMS

Every game system has the same pattern:

```
ServerScriptService/
  [SystemName]Handler.Script        ← server logic
  
StarterPlayerScripts/ or StarterGui/
  [SystemName]Client.LocalScript    ← client logic (if needed)

ReplicatedStorage/
  Remotes/
    [SystemName]Event.RemoteEvent   ← communication
  Modules/
    [SystemName]Config.ModuleScript ← shared config (prices, items, etc.)
```

**Flow:** Client sends request → Remote → Server validates → Server executes → Server notifies client → Client updates GUI

---

## REFERENCE FILES

Read the relevant ones BEFORE writing code:

**Core rules:**
- `references/client-server.md` — What runs where, what can access what
- `references/remotes.md` — RemoteEvents, RemoteFunctions, validation patterns
- `references/datastore.md` — Data persistence, saving, loading, error handling

**API reference:**
- `references/services.md` — Every Roblox service with common methods
- `references/luau-syntax.md` — Types, tables, math, CFrame, Vector3, strings

**Game systems (one file per system):**
- `references/systems/currency.md`
- `references/systems/inventory.md`
- `references/systems/pets.md`
- `references/systems/rebirth.md`
- `references/systems/zones.md`
- `references/systems/shop.md`
- `references/systems/daily-rewards.md`
- `references/systems/codes.md`
- `references/systems/combat.md`
- `references/systems/npc-ai.md`
- `references/systems/leaderboard.md`

**Quality:**
- `references/performance.md` — Optimization, memory, pools, debounce
- `references/oop-modules.md` — ModuleScript patterns, OOP, singletons
- `references/anti-exploit.md` — Server validation, rate limiting, kick patterns
- `references/common-mistakes.md` — Top mistakes AIs make (READ THIS FIRST)
- `references/mcp-patterns.md` — How to create scripts via MCP run_code
