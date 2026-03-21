# MCP Execution Patterns

## How MCP run_code Works
Code executed through the Roblox Studio MCP runs in SERVER context (like the Studio command bar). This means:
- `game.Players.LocalPlayer` = nil (no local player in server context)
- Code can access ServerStorage, ServerScriptService
- Code does NOT persist after stopping play-test
- To make persistent code, create Script/LocalScript instances with Source

## Creating a Server Script via MCP
```lua
local script = Instance.new("Script")
script.Name = "ShopHandler"
script.Source = [[
    local Players = game:GetService("Players")
    local RS = game:GetService("ReplicatedStorage")
    -- server code here
]]
script.Parent = game:GetService("ServerScriptService")
```

## Creating a Client Script via MCP
```lua
local ls = Instance.new("LocalScript")
ls.Name = "ShopClient"
ls.Source = [[
    local player = game.Players.LocalPlayer
    local RS = game:GetService("ReplicatedStorage")
    -- client code here
]]
ls.Parent = game:GetService("StarterGui")  -- or StarterPlayerScripts
```

## Creating a ModuleScript via MCP
```lua
local mod = Instance.new("ModuleScript")
mod.Name = "GameConfig"
mod.Source = [[
    local Config = {}
    Config.MaxPets = 3
    Config.StartingCoins = 100
    return Config
]]
mod.Parent = game:GetService("ReplicatedStorage")
```

## Creating Remotes via MCP
```lua
local RS = game:GetService("ReplicatedStorage")
local remotes = RS:FindFirstChild("Remotes")
if not remotes then
    remotes = Instance.new("Folder")
    remotes.Name = "Remotes"
    remotes.Parent = RS
end

local re = Instance.new("RemoteEvent")
re.Name = "ShopEvent"
re.Parent = remotes
```

## Full System via MCP (create everything at once)
When building a game system through MCP, create in this order:
1. Remotes folder + RemoteEvents
2. ModuleScripts (shared config)
3. Server Script (with full Source)
4. Client Script (with full Source)

This ensures everything exists before scripts try to reference each other.

## Testing After Creating
After creating scripts via MCP:
- Server scripts start immediately
- Client scripts need a play-test restart (or rejoin) to activate
- To test client code: stop and restart the play-test in Studio
