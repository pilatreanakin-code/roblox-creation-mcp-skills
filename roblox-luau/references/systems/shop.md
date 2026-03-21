# Shop System

## Architecture
- Server: holds item catalog (prices, requirements). Validates purchases.
- Client: shows GUI, sends item ID on buy click.
- Remote: ShopEvent

## Server — Item Catalog (NEVER on client)
```lua
local Catalog = {
    [1] = {name = "Speed Boost", price = 500, category = "Gear"},
    [2] = {name = "Storm Runner", price = 1000, category = "Vehicles"},
}
```

## Server — Purchase Handler
```lua
ShopEvent.OnServerEvent:Connect(function(player, action, itemId)
    if action ~= "Buy" then return end
    if type(itemId) ~= "number" then return end
    
    local item = Catalog[itemId]
    if not item then return end
    if playerData[player].inventory[itemId] then
        ShopEvent:FireClient(player, "AlreadyOwned")
        return
    end
    if player.leaderstats.Coins.Value < item.price then
        ShopEvent:FireClient(player, "NotEnough")
        return
    end
    
    player.leaderstats.Coins.Value -= item.price
    playerData[player].coins -= item.price
    playerData[player].inventory[itemId] = true
    ShopEvent:FireClient(player, "Success", itemId)
end)
```

## Client — Buy Button
```lua
buyBtn.MouseButton1Click:Connect(function()
    ShopEvent:FireServer("Buy", itemId)  -- ONLY the ID, never price
end)

ShopEvent.OnClientEvent:Connect(function(result, data)
    if result == "Success" then
        -- update GUI, show notification
    elseif result == "NotEnough" then
        -- shake button, show error
    end
end)
```
