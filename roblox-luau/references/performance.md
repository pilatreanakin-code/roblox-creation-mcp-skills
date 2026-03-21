# Performance Best Practices

## Use Modern APIs
```lua
task.wait(1)      -- NOT wait(1)
task.spawn(fn)    -- NOT spawn(fn)
task.delay(1, fn) -- NOT delay(1, fn)
```

## Never Loop Without Yielding
```lua
-- WRONG (freezes):
while true do doStuff() end

-- CORRECT:
while true do
    doStuff()
    task.wait()  -- or task.wait(0.1) for less frequent
end
```

## Disconnect Events
```lua
local connections = {}

Players.PlayerAdded:Connect(function(player)
    local conn = player.CharacterAdded:Connect(function(char) end)
    connections[player] = conn
end)

Players.PlayerRemoving:Connect(function(player)
    if connections[player] then
        connections[player]:Disconnect()
        connections[player] = nil
    end
end)
```

## Cache References
```lua
-- WRONG (searches every frame):
RunService.Heartbeat:Connect(function()
    local part = workspace:FindFirstChild("MyPart")
    if part then part.Position += Vector3.new(0,0.1,0) end
end)

-- CORRECT (cache once):
local part = workspace:WaitForChild("MyPart")
RunService.Heartbeat:Connect(function()
    part.Position += Vector3.new(0,0.1,0)
end)
```

## Set Parent Last
```lua
local part = Instance.new("Part")
part.Size = Vector3.new(2,2,2)
part.Color = Color3.fromRGB(255,0,0)
part.Anchored = true
part.Parent = workspace  -- LAST
```

## Debounce
```lua
local debounce = false
button.MouseButton1Click:Connect(function()
    if debounce then return end
    debounce = true
    -- do stuff
    task.wait(0.5)
    debounce = false
end)

-- Touched debounce
local touchDebounce = {}
part.Touched:Connect(function(hit)
    local player = Players:GetPlayerFromCharacter(hit.Parent)
    if not player then return end
    if touchDebounce[player] then return end
    touchDebounce[player] = true
    -- do stuff
    task.wait(1)
    touchDebounce[player] = nil
end)
```

## Debris for Temp Objects
```lua
local Debris = game:GetService("Debris")
local projectile = Instance.new("Part")
projectile.Parent = workspace
Debris:AddItem(projectile, 5)  -- auto-destroy after 5s
```

## Use CollectionService Tags
```lua
-- Instead of looping all workspace children:
for _, obj in CollectionService:GetTagged("Coin") do
    -- process only tagged objects
end
```

## Reduce Remote Traffic
```lua
-- WRONG: fire every frame
RunService.Heartbeat:Connect(function()
    remote:FireServer(position)  -- 60 calls/second
end)

-- CORRECT: fire at intervals
local lastSend = 0
RunService.Heartbeat:Connect(function()
    if tick() - lastSend > 0.1 then  -- 10 times/second max
        remote:FireServer(position)
        lastSend = tick()
    end
end)
```
