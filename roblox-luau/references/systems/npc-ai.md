# NPC / Enemy AI

## Server-Side Only
```lua
local PFS = game:GetService("PathfindingService")

local function npcBehavior(npc)
    local hum = npc:FindFirstChildOfClass("Humanoid")
    local hrp = npc:FindFirstChild("HumanoidRootPart")
    if not hum or not hrp then return end
    
    while hum.Health > 0 do
        -- Find nearest player
        local closest, closestDist = nil, math.huge
        for _, player in Players:GetPlayers() do
            local char = player.Character
            if char and char:FindFirstChild("HumanoidRootPart") then
                local dist = (hrp.Position - char.HumanoidRootPart.Position).Magnitude
                if dist < closestDist then
                    closest = char
                    closestDist = dist
                end
            end
        end
        
        if closest and closestDist < 50 then  -- aggro range
            if closestDist < 5 then
                -- Attack
                closest.Humanoid:TakeDamage(10)
                task.wait(1)
            else
                -- Pathfind to player
                local path = PFS:CreatePath()
                pcall(function()
                    path:ComputeAsync(hrp.Position, closest.HumanoidRootPart.Position)
                end)
                local waypoints = path:GetWaypoints()
                for _, wp in waypoints do
                    hum:MoveTo(wp.Position)
                    hum.MoveToFinished:Wait()
                end
            end
        else
            -- Idle / wander
            task.wait(2)
        end
        task.wait(0.3)
    end
end
```

## Spawning and Respawning
```lua
local function spawnNPC(template, spawnPart)
    local npc = template:Clone()
    npc:PivotTo(spawnPart.CFrame)
    npc.Parent = workspace.NPCs
    
    task.spawn(function() npcBehavior(npc) end)
    
    npc.Humanoid.Died:Connect(function()
        task.wait(10)
        npc:Destroy()
        spawnNPC(template, spawnPart)  -- respawn
    end)
end
```
