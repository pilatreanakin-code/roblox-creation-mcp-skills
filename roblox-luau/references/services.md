# Roblox Services — Quick Reference

## Players (server + client)
```lua
local Players = game:GetService("Players")
Players.PlayerAdded:Connect(function(player) end)
Players.PlayerRemoving:Connect(function(player) end)
Players:GetPlayers()  -- all current players
Players:GetPlayerByUserId(userId)
-- CLIENT ONLY:
Players.LocalPlayer  -- NEVER use in Script
```

## Workspace (both)
```lua
workspace.Gravity = 196.2  -- default
workspace:Raycast(origin, direction, params)  -- modern raycasting
-- CLIENT: workspace.CurrentCamera
```

## ReplicatedStorage (both) — shared modules, remotes, assets
## ServerStorage (server only) — templates, private data
## ServerScriptService (server only) — server scripts

## TweenService (both)
```lua
local TweenService = game:GetService("TweenService")
local info = TweenInfo.new(1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
local tween = TweenService:Create(part, info, {Position = Vector3.new(0,10,0)})
tween:Play()
tween.Completed:Wait()
```

## RunService (both)
```lua
local RS = game:GetService("RunService")
RS.Heartbeat:Connect(function(dt) end)        -- every frame, after physics (server + client)
RS.RenderStepped:Connect(function(dt) end)     -- every frame, before render (CLIENT ONLY)
RS:IsClient()  RS:IsServer()
```

## UserInputService (CLIENT ONLY)
```lua
local UIS = game:GetService("UserInputService")
UIS.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.E then end
end)
UIS.TouchEnabled  -- check if mobile
```

## CollectionService (both)
```lua
local CS = game:GetService("CollectionService")
CS:AddTag(instance, "Enemy")
local enemies = CS:GetTagged("Enemy")
CS:GetInstanceAddedSignal("Enemy"):Connect(function(inst) end)
```

## MarketplaceService (server mostly)
```lua
local MPS = game:GetService("MarketplaceService")
MPS:PromptProductPurchase(player, productId)
MPS:PromptGamePassPurchase(player, passId)
MPS:UserOwnsGamePassAsync(player.UserId, passId)

MPS.ProcessReceipt = function(receiptInfo)
    -- Grant item based on receiptInfo.ProductId
    return Enum.ProductPurchaseDecision.PurchaseGranted
end
```

## TeleportService (server)
```lua
local TS = game:GetService("TeleportService")
TS:Teleport(placeId, player)
TS:TeleportToPlaceInstance(placeId, instanceId, player)
```

## BadgeService (server)
```lua
local BS = game:GetService("BadgeService")
if not BS:UserHasBadgeAsync(player.UserId, badgeId) then
    BS:AwardBadge(player.UserId, badgeId)
end
```

## Debris (both)
```lua
game:GetService("Debris"):AddItem(instance, 5)  -- auto-destroy after 5 seconds
```

## HttpService (server — JSON only in published games)
```lua
local HS = game:GetService("HttpService")
local json = HS:JSONEncode(table)
local tbl = HS:JSONDecode(jsonString)
local guid = HS:GenerateGUID()
```

## PathfindingService (server)
```lua
local PFS = game:GetService("PathfindingService")
local path = PFS:CreatePath({AgentRadius = 2, AgentHeight = 5})
path:ComputeAsync(startPos, endPos)
local waypoints = path:GetWaypoints()
```

## Lighting (both)
```lua
local Lighting = game:GetService("Lighting")
Lighting.TimeOfDay = "14:00:00"
Lighting.Ambient = Color3.fromRGB(100, 100, 100)
```

## SoundService (both)
```lua
game:GetService("SoundService"):PlayLocalSound(sound)  -- client
```
