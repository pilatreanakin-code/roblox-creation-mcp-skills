# RemoteEvents and RemoteFunctions

## When to Use Which
- **RemoteEvent** — fire and forget. Client tells server something happened. Server tells client something happened. No return value. Use for 90% of cases.
- **RemoteFunction** — request + response. Client asks server for data, waits for answer. Use only for short data lookups. NEVER use InvokeClient (server calling client function — exploitable).

## Setup
```lua
-- Server: create remotes on startup
local remotes = Instance.new("Folder")
remotes.Name = "Remotes"
remotes.Parent = game:GetService("ReplicatedStorage")

local shopEvent = Instance.new("RemoteEvent")
shopEvent.Name = "ShopEvent"
shopEvent.Parent = remotes
```

## Client → Server
```lua
-- Client
local remote = game.ReplicatedStorage:WaitForChild("Remotes"):WaitForChild("ShopEvent")
remote:FireServer("BuyItem", itemId)  -- send action + item ID only, NEVER price

-- Server
remote.OnServerEvent:Connect(function(player, action, itemId)
    -- player is auto-injected, always first argument
    -- VALIDATE everything here
end)
```

## Server → Client
```lua
-- Server: notify one player
remote:FireClient(player, "PurchaseSuccess", itemId)

-- Server: notify all players
remote:FireAllClients("Announcement", "New event started!")

-- Client: listen
remote.OnClientEvent:Connect(function(action, data)
    -- update GUI based on action
end)
```

## Server Validation Pattern
```lua
remote.OnServerEvent:Connect(function(player, action, itemId)
    -- Type check
    if type(action) ~= "string" then return end
    if type(itemId) ~= "number" then return end
    
    -- Existence check
    local item = ItemCatalog[itemId]
    if not item then return end
    
    -- Permission check
    if item.requiredLevel and player.leaderstats.Level.Value < item.requiredLevel then return end
    
    -- Affordability check
    if player.leaderstats.Coins.Value < item.price then return end
    
    -- Already owned check
    if playerData[player].inventory[itemId] then return end
    
    -- All checks passed — execute
    player.leaderstats.Coins.Value -= item.price
    playerData[player].inventory[itemId] = true
    remote:FireClient(player, "PurchaseSuccess", itemId)
end)
```

## Rate Limiting
```lua
local lastAction = {}
remote.OnServerEvent:Connect(function(player, ...)
    local now = tick()
    local key = player.UserId
    if lastAction[key] and (now - lastAction[key]) < 0.5 then return end
    lastAction[key] = now
    -- process normally
end)
```

## Common Mistakes
- Sending price from client (exploitable — server must look up price itself)
- Using RemoteFunction for simple actions (can hang if client doesn't respond)
- Not type-checking arguments (exploiter can send any type)
- Calling FireServer from server or FireClient from client (wrong direction)
