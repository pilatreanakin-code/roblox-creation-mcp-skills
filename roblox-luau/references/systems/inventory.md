# Inventory System

## Server — Data Structure
```lua
playerData[player].inventory = {}  -- dictionary: itemId = true/count
```

## Server — Add/Remove/Check
```lua
local function giveItem(player, itemId)
    playerData[player].inventory[itemId] = true
    InventoryEvent:FireClient(player, "Added", itemId)
end

local function removeItem(player, itemId)
    playerData[player].inventory[itemId] = nil
    InventoryEvent:FireClient(player, "Removed", itemId)
end

local function hasItem(player, itemId)
    return playerData[player].inventory[itemId] ~= nil
end
```

## Server — Equip (with validation)
```lua
EquipEvent.OnServerEvent:Connect(function(player, itemId)
    if not hasItem(player, itemId) then return end
    playerData[player].equipped = itemId
    -- Apply item effects to character
    EquipEvent:FireClient(player, "Equipped", itemId)
end)
```

## Client — Sync on change
```lua
InventoryEvent.OnClientEvent:Connect(function(action, itemId)
    if action == "Added" then
        -- add card to inventory GUI
    elseif action == "Removed" then
        -- remove card from inventory GUI
    end
end)
```
