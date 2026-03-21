# GUI Elements — Complete Reference

Every Roblox GUI instance with all key properties. Created via `Instance.new("ClassName")`.

## Containers

### ScreenGui
**Use:** Root container for all 2D UI. Goes in StarterGui or PlayerGui.
```lua
local sg = Instance.new("ScreenGui")
sg.Name = "ShopGui"
sg.ResetOnSpawn = false          -- ALWAYS false unless you want it to reset on death
sg.IgnoreGuiInset = false        -- true = ignore top bar (full screen)
sg.ZIndexBehavior = Enum.ZIndexBehavior.Sibling  -- recommended
sg.Enabled = false               -- start hidden, show via script
sg.Parent = game.Players.LocalPlayer.PlayerGui
```

### Frame
**Use:** Container for grouping elements. The building block of every GUI.
```lua
local frame = Instance.new("Frame")
frame.Name = "MainPanel"
frame.Size = UDim2.new(0.4, 0, 0.6, 0)           -- 40% width, 60% height
frame.Position = UDim2.new(0.3, 0, 0.2, 0)        -- centered
frame.AnchorPoint = Vector2.new(0.5, 0.5)          -- anchor center
frame.Position = UDim2.new(0.5, 0, 0.5, 0)        -- true center with anchor
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
frame.BackgroundTransparency = 0                    -- 0=opaque, 1=invisible
frame.BorderSizePixel = 0                           -- ALWAYS 0, use UIStroke instead
frame.ClipsDescendants = true                       -- clip children to frame bounds
frame.Parent = screenGui
```

### ScrollingFrame
**Use:** Scrollable container for long lists, inventories, chat.
```lua
local scroll = Instance.new("ScrollingFrame")
scroll.Name = "ItemList"
scroll.Size = UDim2.new(1, 0, 0.8, 0)
scroll.CanvasSize = UDim2.new(0, 0, 0, 0)           -- set to (0,0,0,0) and use AutomaticCanvasSize
scroll.AutomaticCanvasSize = Enum.AutomaticSize.Y    -- auto-expand vertically
scroll.ScrollBarThickness = 6
scroll.ScrollBarImageColor3 = Color3.fromRGB(100, 100, 120)
scroll.BackgroundTransparency = 1
scroll.BorderSizePixel = 0
scroll.Parent = frame
```

## Interactive Elements

### TextButton
**Use:** Clickable button with text. Most common interactive element.
```lua
local btn = Instance.new("TextButton")
btn.Name = "BuyButton"
btn.Size = UDim2.new(0.3, 0, 0, 40)
btn.Position = UDim2.new(0.35, 0, 0.9, 0)
btn.BackgroundColor3 = Color3.fromRGB(80, 200, 80)  -- green
btn.Text = "Buy"
btn.TextColor3 = Color3.fromRGB(255, 255, 255)
btn.Font = Enum.Font.GothamBold
btn.TextSize = 18
btn.BorderSizePixel = 0
btn.AutoButtonColor = false                           -- disable default hover, use custom
btn.Parent = frame

btn.MouseButton1Click:Connect(function()
    print("Clicked!")
end)
```

### ImageButton
**Use:** Clickable button with an image (icons, item slots).
```lua
local imgBtn = Instance.new("ImageButton")
imgBtn.Name = "ItemSlot"
imgBtn.Size = UDim2.new(0, 64, 0, 64)
imgBtn.Image = "rbxassetid://123456789"               -- Roblox asset ID
imgBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
imgBtn.BorderSizePixel = 0
imgBtn.AutoButtonColor = false
imgBtn.Parent = frame
```

### TextBox
**Use:** Text input field (search bars, code entry, chat).
```lua
local input = Instance.new("TextBox")
input.Name = "SearchBox"
input.Size = UDim2.new(0.8, 0, 0, 36)
input.PlaceholderText = "Search items..."
input.PlaceholderColor3 = Color3.fromRGB(150, 150, 150)
input.Text = ""
input.TextColor3 = Color3.fromRGB(255, 255, 255)
input.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
input.Font = Enum.Font.Gotham
input.TextSize = 14
input.ClearTextOnFocus = false
input.BorderSizePixel = 0
input.Parent = frame

input.FocusLost:Connect(function(enterPressed)
    if enterPressed then
        print("Searched: " .. input.Text)
    end
end)
```

## Display Elements

### TextLabel
**Use:** Non-interactive text. Titles, stats, descriptions.
```lua
local label = Instance.new("TextLabel")
label.Name = "Title"
label.Size = UDim2.new(1, 0, 0, 40)
label.Text = "ITEM SHOP"
label.TextColor3 = Color3.fromRGB(255, 255, 255)
label.Font = Enum.Font.GothamBold
label.TextSize = 24
label.BackgroundTransparency = 1
label.TextXAlignment = Enum.TextXAlignment.Center
label.TextYAlignment = Enum.TextYAlignment.Center
label.Parent = frame
```

### ImageLabel
**Use:** Display an image (icons, backgrounds, decorations).
```lua
local img = Instance.new("ImageLabel")
img.Name = "CoinIcon"
img.Size = UDim2.new(0, 24, 0, 24)
img.Image = "rbxassetid://123456789"
img.BackgroundTransparency = 1
img.ScaleType = Enum.ScaleType.Fit
img.Parent = frame
```

### ViewportFrame
**Use:** 3D model preview inside GUI (item previews, character display).
```lua
local vp = Instance.new("ViewportFrame")
vp.Name = "ItemPreview"
vp.Size = UDim2.new(0, 150, 0, 150)
vp.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
vp.Parent = frame

-- Add a camera
local cam = Instance.new("Camera")
cam.CFrame = CFrame.new(Vector3.new(0, 1, 3), Vector3.new(0, 0.5, 0))
cam.Parent = vp
vp.CurrentCamera = cam

-- Clone a model into the viewport
local model = game.ReplicatedStorage.Items.Sword:Clone()
model.Parent = vp
```

## Sizing Cheat Sheet

```lua
-- ALWAYS prefer Scale (proportional) over Offset (pixels)
UDim2.new(0.5, 0, 0.5, 0)     -- 50% of parent width and height (GOOD)
UDim2.new(0, 200, 0, 100)     -- fixed 200x100 pixels (only for icons/small elements)

-- AnchorPoint for easy centering
frame.AnchorPoint = Vector2.new(0.5, 0.5)  -- anchor at center
frame.Position = UDim2.new(0.5, 0, 0.5, 0) -- now truly centered

-- Common positions
-- Top center: Position = UDim2.new(0.5, 0, 0, 10), AnchorPoint = (0.5, 0)
-- Bottom center: Position = UDim2.new(0.5, 0, 1, -10), AnchorPoint = (0.5, 1)
-- Left center: Position = UDim2.new(0, 10, 0.5, 0), AnchorPoint = (0, 0.5)
-- Full screen: Size = UDim2.new(1, 0, 1, 0), Position = UDim2.new(0, 0, 0, 0)
```
