# Daily Rewards

## Server — timestamp-based
```lua
DailyEvent.OnServerEvent:Connect(function(player)
    local data = playerData[player]
    local now = os.time()
    local lastClaim = data.lastDailyClaim or 0
    local elapsed = now - lastClaim
    
    if elapsed < 86400 then  -- 24 hours in seconds
        local remaining = 86400 - elapsed
        DailyEvent:FireClient(player, "TooEarly", remaining)
        return
    end
    
    -- Check streak (if more than 48h, reset streak)
    if elapsed > 172800 then
        data.dailyStreak = 0
    end
    data.dailyStreak = (data.dailyStreak or 0) + 1
    data.lastDailyClaim = now
    
    -- Grant reward based on streak day
    local reward = DailyRewards[math.min(data.dailyStreak, 7)]
    addCoins(player, reward.coins)
    
    DailyEvent:FireClient(player, "Claimed", data.dailyStreak, reward)
end)
```
