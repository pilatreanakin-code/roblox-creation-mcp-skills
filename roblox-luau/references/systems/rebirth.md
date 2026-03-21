# Rebirth / Prestige System

## Server
```lua
RebirthEvent.OnServerEvent:Connect(function(player)
    local data = playerData[player]
    local cost = RebirthCost(data.rebirths)  -- escalating cost
    
    if data.coins < cost then
        RebirthEvent:FireClient(player, "NotEnough")
        return
    end
    
    -- Reset progress
    data.coins = 0
    data.level = 1
    data.xp = 0
    
    -- Grant multiplier
    data.rebirths += 1
    data.multiplier = 1 + (data.rebirths * 0.1)  -- +10% per rebirth
    
    -- Update leaderstats
    player.leaderstats.Coins.Value = 0
    player.leaderstats.Rebirths.Value = data.rebirths
    
    RebirthEvent:FireClient(player, "Success", data.rebirths)
end)
```

## Client — always show confirmation popup before rebirth
