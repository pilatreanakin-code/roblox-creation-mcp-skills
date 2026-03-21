# Zone / Area Gating

## Server — validate zone access
```lua
ZonePortal.Touched:Connect(function(hit)
    local player = Players:GetPlayerFromCharacter(hit.Parent)
    if not player then return end
    
    local requiredLevel = ZoneRequirements[zoneName]
    if player.leaderstats.Level.Value < requiredLevel then
        -- Bounce player back
        local hrp = hit.Parent:FindFirstChild("HumanoidRootPart")
        if hrp then
            hrp.CFrame = spawnPoint.CFrame
        end
        ZoneEvent:FireClient(player, "Locked", zoneName, requiredLevel)
    end
end)
```

## Never trust client to decide zone access
The server checks requirements. Client only shows "zone locked" UI.
