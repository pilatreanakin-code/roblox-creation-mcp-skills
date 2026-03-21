# Combat / Damage System

## RULE: ALL damage calculated on server. Client only sends input direction.

## Server — Hit Detection
```lua
AttackEvent.OnServerEvent:Connect(function(player, targetId)
    -- Rate limit
    local now = tick()
    if lastAttack[player] and (now - lastAttack[player]) < 0.5 then return end
    lastAttack[player] = now
    
    -- Validate target exists
    local target = workspace:FindFirstChild(targetId)
    if not target then return end
    local targetHum = target:FindFirstChildOfClass("Humanoid")
    if not targetHum or targetHum.Health <= 0 then return end
    
    -- Distance check (prevent hit from across the map)
    local playerPos = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    if not playerPos then return end
    local distance = (playerPos.Position - target:GetPivot().Position).Magnitude
    if distance > 15 then return end  -- max attack range
    
    -- Apply damage
    local damage = playerData[player].attackPower or 10
    targetHum:TakeDamage(damage)
end)
```

## Server — Death Handling
```lua
humanoid.Died:Connect(function()
    -- Drop loot
    -- Grant XP to killer
    -- Respawn after delay
    task.delay(5, function()
        npc:Destroy()
        spawnNPC(spawnPoint)
    end)
end)
```
