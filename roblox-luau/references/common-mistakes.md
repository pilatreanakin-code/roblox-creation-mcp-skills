# Common Mistakes AIs Make — READ THIS FIRST

## Deprecated APIs
```lua
-- WRONG:              -- CORRECT:
wait(1)                task.wait(1)
spawn(fn)              task.spawn(fn)
delay(1, fn)           task.delay(1, fn)
Instance.new("P",ws)   local p = Instance.new("P") p.Parent = ws
```

## Wrong Context
```lua
-- WRONG: LocalPlayer in a Script (server)
local player = game.Players.LocalPlayer  -- ERROR: nil on server

-- WRONG: ServerStorage in a LocalScript (client)  
local ss = game.ServerStorage  -- ERROR: not accessible

-- WRONG: UserInputService in a Script
UIS.InputBegan:Connect(...)  -- ERROR: no input on server
```

## DataStore Without pcall
```lua
-- WRONG:
local data = DS:GetAsync(key)  -- server crashes on failure

-- CORRECT:
local ok, data = pcall(function() return DS:GetAsync(key) end)
```

## Trusting Client
```lua
-- WRONG: client sends price
remote.OnServerEvent:Connect(function(player, itemId, price)
    player.leaderstats.Coins.Value -= price  -- EXPLOITABLE
end)

-- CORRECT: server looks up price
remote.OnServerEvent:Connect(function(player, itemId)
    local price = Catalog[itemId].price  -- server's data
    if player.leaderstats.Coins.Value >= price then
        player.leaderstats.Coins.Value -= price
    end
end)
```

## Infinite Loop Without Wait
```lua
-- WRONG (freezes game):
while true do
    checkStuff()
end

-- CORRECT:
while true do
    checkStuff()
    task.wait(0.1)
end
```

## Not Checking nil
```lua
-- WRONG:
local part = workspace:FindFirstChild("MaybeExists")
part.Position = Vector3.new(0,5,0)  -- ERROR if nil

-- CORRECT:
local part = workspace:FindFirstChild("MaybeExists")
if part then
    part.Position = Vector3.new(0,5,0)
end
```

## Not Cleaning Up
```lua
-- WRONG: connection leaks forever
part.Touched:Connect(function() end)  -- never disconnected

-- CORRECT:
local conn = part.Touched:Connect(function() end)
-- later: conn:Disconnect()
```

## FireServer from Server / FireClient from Client
```lua
-- WRONG:
-- In a Script: remote:FireServer(...)     -- server can't fire to server
-- In a LocalScript: remote:FireClient(...)  -- client can't fire to client

-- CORRECT:
-- LocalScript: remote:FireServer(...)
-- Script: remote:FireClient(player, ...)
```

## Forgetting BindToClose
```lua
-- Without this, data loss on server shutdown:
game:BindToClose(function()
    for _, player in Players:GetPlayers() do
        savePlayer(player)
    end
end)
```

## Using Strings for Enums
```lua
-- WRONG:
part.Material = "Neon"

-- CORRECT:
part.Material = Enum.Material.Neon
```
