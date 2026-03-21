# DataStore — Data Persistence

## Basics
```lua
local DSS = game:GetService("DataStoreService")
local DS = DSS:GetDataStore("PlayerData")
local key = "Player_" .. player.UserId
```

## Always pcall — NEVER raw DataStore calls
```lua
-- WRONG:
local data = DS:GetAsync(key)  -- crashes server if DataStore fails

-- CORRECT:
local success, data = pcall(function()
    return DS:GetAsync(key)
end)
if not success then
    warn("DataStore error: " .. tostring(data))
end
```

## Retry Pattern
```lua
local function safeGet(ds, key)
    for i = 1, 3 do
        local ok, result = pcall(function() return ds:GetAsync(key) end)
        if ok then return result end
        warn("DS retry " .. i .. ": " .. tostring(result))
        task.wait(2 ^ i)
    end
    return nil
end
```

## UpdateAsync over SetAsync
```lua
-- PREFERRED (handles race conditions):
pcall(function()
    DS:UpdateAsync(key, function(old)
        old = old or {coins = 0, inventory = {}}
        old.coins += earned
        return old
    end)
end)
```

## Load on Join with Defaults
```lua
local DEFAULT_DATA = {coins = 0, gems = 0, inventory = {}, rebirths = 0, level = 1}

Players.PlayerAdded:Connect(function(player)
    local key = "Player_" .. player.UserId
    local data = safeGet(DS, key)
    if data then
        playerData[player] = data
    else
        playerData[player] = table.clone(DEFAULT_DATA)
    end
    -- Create leaderstats from data
end)
```

## Save on Leave + BindToClose
```lua
local function savePlayer(player)
    local key = "Player_" .. player.UserId
    local data = playerData[player]
    if not data then return end
    pcall(function() DS:SetAsync(key, data) end)
    playerData[player] = nil
end

Players.PlayerRemoving:Connect(savePlayer)

game:BindToClose(function()
    for _, player in Players:GetPlayers() do
        savePlayer(player)
    end
end)
```

## Auto-Save (every 5 minutes)
```lua
task.spawn(function()
    while true do
        task.wait(300)
        for _, player in Players:GetPlayers() do
            task.spawn(function()
                local key = "Player_" .. player.UserId
                pcall(function() DS:SetAsync(key, playerData[player]) end)
            end)
        end
    end
end)
```

## OrderedDataStore (leaderboards)
```lua
local ODS = DSS:GetOrderedDataStore("TopCoins")

-- Save score
pcall(function() ODS:SetAsync("Player_" .. userId, score) end)

-- Get top 10
local success, pages = pcall(function()
    return ODS:GetSortedAsync(false, 10)  -- false = descending
end)
if success then
    local top = pages:GetCurrentPage()
    for rank, entry in ipairs(top) do
        print(rank, entry.key, entry.value)
    end
end
```
