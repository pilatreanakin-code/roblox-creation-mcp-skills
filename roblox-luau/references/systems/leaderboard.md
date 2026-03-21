# Leaderboard System

## Server — Save Scores to OrderedDataStore
```lua
local ODS = DSS:GetOrderedDataStore("TopCoins")

local function updateLeaderboard(player)
    pcall(function()
        ODS:SetAsync("Player_" .. player.UserId, playerData[player].coins)
    end)
end
```

## Server — Fetch Top 10
```lua
local function getTopPlayers(limit)
    local success, pages = pcall(function()
        return ODS:GetSortedAsync(false, limit or 10)
    end)
    if not success then return {} end
    
    local top = {}
    for rank, entry in ipairs(pages:GetCurrentPage()) do
        local userId = tonumber(entry.key:match("%d+"))
        local name = "Unknown"
        pcall(function()
            name = Players:GetNameFromUserIdAsync(userId)
        end)
        table.insert(top, {rank = rank, name = name, score = entry.value})
    end
    return top
end
```

## Client — Request and Display
```lua
LeaderboardEvent.OnClientEvent:Connect(function(topList)
    for _, entry in topList do
        -- create row in leaderboard GUI
    end
end)
```
