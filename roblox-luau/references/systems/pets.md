# Pet System

## Server — Data
```lua
playerData[player].pets = {
    owned = {},    -- {petId, petId, ...}
    equipped = {}, -- max 3 (or gamepass limit)
}
```

## Equip/Unequip (server validates max)
```lua
local MAX_EQUIPPED = 3

PetEvent.OnServerEvent:Connect(function(player, action, petIndex)
    local data = playerData[player].pets
    if action == "Equip" then
        if #data.equipped >= MAX_EQUIPPED then return end
        if not data.owned[petIndex] then return end
        table.insert(data.equipped, petIndex)
    elseif action == "Unequip" then
        local idx = table.find(data.equipped, petIndex)
        if idx then table.remove(data.equipped, idx) end
    elseif action == "EquipBest" then
        -- Sort owned by power descending, equip top N
        table.clear(data.equipped)
        local sorted = table.clone(data.owned)
        table.sort(sorted, function(a,b) return PetStats[a].power > PetStats[b].power end)
        for i = 1, math.min(MAX_EQUIPPED, #sorted) do
            table.insert(data.equipped, sorted[i])
        end
    end
    PetEvent:FireClient(player, "Updated", data)
end)
```

## Egg Hatching (weighted random)
```lua
local function hatchEgg(eggId)
    local pool = EggPools[eggId]  -- {{petId=1, weight=50}, {petId=2, weight=30}, ...}
    local totalWeight = 0
    for _, entry in pool do totalWeight += entry.weight end
    local roll = math.random() * totalWeight
    local cumulative = 0
    for _, entry in pool do
        cumulative += entry.weight
        if roll <= cumulative then return entry.petId end
    end
end
```
