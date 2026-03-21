# Currency System

## Architecture
- Server: holds authoritative coin values in playerData table + leaderstats
- Client: displays values, sends purchase requests
- Remote: ShopEvent (client → server: buy request, server → client: result)

## Server
```lua
-- On player join: create leaderstats
local ls = Instance.new("Folder")
ls.Name = "leaderstats"
ls.Parent = player

local coins = Instance.new("IntValue")
coins.Name = "Coins"
coins.Value = playerData[player].coins or 0
coins.Parent = ls
```

## Adding/Removing Currency (SERVER ONLY)
```lua
local function addCoins(player, amount)
    playerData[player].coins += amount
    player.leaderstats.Coins.Value = playerData[player].coins
end

local function removeCoins(player, amount)
    if playerData[player].coins < amount then return false end
    playerData[player].coins -= amount
    player.leaderstats.Coins.Value = playerData[player].coins
    return true
end
```

## Client reads from leaderstats (auto-replicated)
```lua
local coins = player.leaderstats:WaitForChild("Coins")
coinLabel.Text = coins.Value
coins.Changed:Connect(function(newVal)
    coinLabel.Text = newVal
end)
```
