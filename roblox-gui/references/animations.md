# Animations — TweenService, Hover Effects, Transitions

## TweenService Basics
```lua
local TweenService = game:GetService("TweenService")

-- Create a tween
local tweenInfo = TweenInfo.new(
    0.3,                           -- duration (seconds)
    Enum.EasingStyle.Quint,        -- easing style
    Enum.EasingDirection.Out,      -- easing direction
    0,                             -- repeat count (0 = no repeat)
    false,                         -- reverses
    0                              -- delay before start
)

local tween = TweenService:Create(frame, tweenInfo, {
    BackgroundTransparency = 0,    -- target properties
    Position = UDim2.new(0.5, 0, 0.5, 0),
})
tween:Play()
```

## Best Easing Styles for GUI
```lua
-- Smooth open/close (RECOMMENDED DEFAULT)
Enum.EasingStyle.Quint, Enum.EasingDirection.Out

-- Snappy/responsive buttons
Enum.EasingStyle.Back, Enum.EasingDirection.Out    -- slight overshoot, feels alive

-- Bounce (playful games)
Enum.EasingStyle.Bounce, Enum.EasingDirection.Out

-- Linear (progress bars, loading)
Enum.EasingStyle.Linear, Enum.EasingDirection.InOut

-- Elastic (notifications popping in)
Enum.EasingStyle.Elastic, Enum.EasingDirection.Out
```

## GUI Open/Close Animations

### Fade In/Out
```lua
local function fadeIn(gui)
    gui.Visible = true
    gui.BackgroundTransparency = 1
    local tween = TweenService:Create(gui, TweenInfo.new(0.3, Enum.EasingStyle.Quint), {
        BackgroundTransparency = 0
    })
    tween:Play()
    -- Also fade all children
    for _, child in gui:GetDescendants() do
        if child:IsA("GuiObject") then
            local origTransparency = child.BackgroundTransparency
            child.BackgroundTransparency = 1
            TweenService:Create(child, TweenInfo.new(0.3, Enum.EasingStyle.Quint), {
                BackgroundTransparency = origTransparency
            }):Play()
        end
    end
end

local function fadeOut(gui)
    local tween = TweenService:Create(gui, TweenInfo.new(0.2, Enum.EasingStyle.Quint), {
        BackgroundTransparency = 1
    })
    tween:Play()
    tween.Completed:Wait()
    gui.Visible = false
end
```

### Scale Pop (recommended for popups/shops)
```lua
local function popIn(gui)
    gui.Visible = true
    gui.Size = UDim2.new(0, 0, 0, 0)
    gui.Position = UDim2.new(0.5, 0, 0.5, 0)
    gui.AnchorPoint = Vector2.new(0.5, 0.5)
    
    local tween = TweenService:Create(gui, TweenInfo.new(0.35, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
        Size = UDim2.new(0.5, 0, 0.6, 0)  -- target size
    })
    tween:Play()
end

local function popOut(gui)
    local tween = TweenService:Create(gui, TweenInfo.new(0.2, Enum.EasingStyle.Quint, Enum.EasingDirection.In), {
        Size = UDim2.new(0, 0, 0, 0)
    })
    tween:Play()
    tween.Completed:Wait()
    gui.Visible = false
end
```

### Slide In/Out (good for side panels, notifications)
```lua
local function slideInFromRight(gui, targetX)
    gui.Visible = true
    gui.Position = UDim2.new(1.5, 0, gui.Position.Y.Scale, 0)  -- start off-screen right
    
    TweenService:Create(gui, TweenInfo.new(0.4, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), {
        Position = UDim2.new(targetX, 0, gui.Position.Y.Scale, 0)
    }):Play()
end

local function slideOutToRight(gui)
    local tween = TweenService:Create(gui, TweenInfo.new(0.3, Enum.EasingStyle.Quint, Enum.EasingDirection.In), {
        Position = UDim2.new(1.5, 0, gui.Position.Y.Scale, 0)
    })
    tween:Play()
    tween.Completed:Wait()
    gui.Visible = false
end

-- Variants: slideInFromLeft (start at -0.5), slideInFromTop, slideInFromBottom
```

## Button Effects

### Hover Color Change
```lua
local function setupHover(btn, normalColor, hoverColor)
    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15), {
            BackgroundColor3 = hoverColor
        }):Play()
    end)
    btn.MouseLeave:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15), {
            BackgroundColor3 = normalColor
        }):Play()
    end)
end
-- Usage: setupHover(buyBtn, Color3.fromRGB(80,200,80), Color3.fromRGB(100,220,100))
```

### Hover Scale (subtle zoom)
```lua
local function setupScaleHover(btn)
    local originalSize = btn.Size
    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15, Enum.EasingStyle.Quint), {
            Size = UDim2.new(
                originalSize.X.Scale * 1.05, originalSize.X.Offset,
                originalSize.Y.Scale * 1.05, originalSize.Y.Offset
            )
        }):Play()
    end)
    btn.MouseLeave:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15, Enum.EasingStyle.Quint), {
            Size = originalSize
        }):Play()
    end)
end
```

### Click Press Animation
```lua
local function setupClickAnim(btn)
    btn.MouseButton1Down:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.08), {
            Size = UDim2.new(
                btn.Size.X.Scale * 0.95, btn.Size.X.Offset,
                btn.Size.Y.Scale * 0.95, btn.Size.Y.Offset
            )
        }):Play()
    end)
    btn.MouseButton1Up:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.1, Enum.EasingStyle.Back), {
            Size = UDim2.new(
                btn.Size.X.Scale / 0.95, btn.Size.X.Offset,
                btn.Size.Y.Scale / 0.95, btn.Size.Y.Offset
            )
        }):Play()
    end)
end
```

### Button Glow on Hover (UIStroke)
```lua
local function setupGlowHover(btn, glowColor)
    local stroke = Instance.new("UIStroke")
    stroke.Color = glowColor
    stroke.Thickness = 0
    stroke.Transparency = 0.5
    stroke.Parent = btn
    
    btn.MouseEnter:Connect(function()
        TweenService:Create(stroke, TweenInfo.new(0.2), {Thickness = 2, Transparency = 0}):Play()
    end)
    btn.MouseLeave:Connect(function()
        TweenService:Create(stroke, TweenInfo.new(0.2), {Thickness = 0, Transparency = 0.5}):Play()
    end)
end
```

## Notification Pop-in
```lua
local function showNotification(text, duration)
    duration = duration or 3
    
    local notif = Instance.new("Frame")
    notif.Size = UDim2.new(0.3, 0, 0, 50)
    notif.Position = UDim2.new(0.5, 0, 0, -60)  -- above screen
    notif.AnchorPoint = Vector2.new(0.5, 0)
    notif.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
    notif.BorderSizePixel = 0
    notif.Parent = screenGui
    
    Instance.new("UICorner", notif).CornerRadius = UDim.new(0, 8)
    
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, 0, 1, 0)
    label.Text = text
    label.TextColor3 = Color3.fromRGB(255, 255, 255)
    label.Font = Enum.Font.GothamBold
    label.TextSize = 16
    label.BackgroundTransparency = 1
    label.Parent = notif
    
    -- Slide in
    TweenService:Create(notif, TweenInfo.new(0.4, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
        Position = UDim2.new(0.5, 0, 0, 20)
    }):Play()
    
    -- Auto dismiss
    task.delay(duration, function()
        local out = TweenService:Create(notif, TweenInfo.new(0.3, Enum.EasingStyle.Quint), {
            Position = UDim2.new(0.5, 0, 0, -60)
        })
        out:Play()
        out.Completed:Wait()
        notif:Destroy()
    end)
end
```

## Screen Dim (behind popups)
```lua
local function createDim(parent)
    local dim = Instance.new("Frame")
    dim.Name = "ScreenDim"
    dim.Size = UDim2.new(1, 0, 1, 0)
    dim.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    dim.BackgroundTransparency = 1
    dim.ZIndex = 5
    dim.Parent = parent
    
    TweenService:Create(dim, TweenInfo.new(0.3), {BackgroundTransparency = 0.5}):Play()
    return dim
end

local function removeDim(dim)
    local tween = TweenService:Create(dim, TweenInfo.new(0.2), {BackgroundTransparency = 1})
    tween:Play()
    tween.Completed:Wait()
    dim:Destroy()
end
```
