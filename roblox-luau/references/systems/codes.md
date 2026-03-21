# Codes System

## Server
```lua
local ValidCodes = {
    LAUNCH = {coins = 500, gems = 10},
    STORM2024 = {coins = 1000},
}

CodeEvent.OnServerEvent:Connect(function(player, code)
    if type(code) ~= "string" then return end
    code = string.upper(code)
    
    local reward = ValidCodes[code]
    if not reward then
        CodeEvent:FireClient(player, "Invalid")
        return
    end
    
    local data = playerData[player]
    data.redeemedCodes = data.redeemedCodes or {}
    if data.redeemedCodes[code] then
        CodeEvent:FireClient(player, "AlreadyRedeemed")
        return
    end
    
    -- Grant rewards
    data.redeemedCodes[code] = true
    if reward.coins then addCoins(player, reward.coins) end
    if reward.gems then addGems(player, reward.gems) end
    
    CodeEvent:FireClient(player, "Success", reward)
end)
```
