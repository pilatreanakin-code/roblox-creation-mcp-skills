# Code Patterns — Complete Working Templates

The AI MUST follow these patterns when building GUIs. These are NOT optional — they are the exact code structures to use. Adapt names and values to match the user's request, but the structure stays the same.

---

## PATTERN 1: Button + Frame + LocalScript (MOST COMMON)

Whenever the user asks for a button that opens a frame/screen, build ALL THREE: the button, the frame, AND the LocalScript that wires them. Never build just the visuals without the script.

### Complete ScreenGui Structure
```
ScreenGui (ResetOnSpawn=false, IgnoreGuiInset=true)
  Dim (TextButton, full screen, black, starts invisible)
  [Name]ButtonFrame (Frame, the sidebar/nav button)
    [Name]Btn (TextButton inside, transparent, catches clicks)
    Icon (ImageLabel, placeholder)
    Label (TextLabel below frame, name of the button)
  [Name]Frame (Frame, the main panel, starts Visible=false)
    Title (TextLabel, above or inside the frame)
    CloseBtn (TextButton, X, overlapping top-right)
    Content area (tabs, grids, lists, etc.)
  GUIController (LocalScript, wires EVERYTHING)
```

### The LocalScript Template (ALWAYS CREATE THIS)
Every GUI gets a LocalScript. This is the exact template — adapt variable names to match:

```lua
-- GUIController.LocalScript (inside the ScreenGui)
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local player = game.Players.LocalPlayer
local gui = script.Parent

-- References (adapt names to match what was built)
local openBtn = gui:WaitForChild("[Name]ButtonFrame"):WaitForChild("[Name]Btn")
local mainFrame = gui:WaitForChild("[Name]Frame")
local closeBtn = mainFrame:WaitForChild("CloseBtn")
local dim = gui:WaitForChild("Dim")

-- Collect ALL elements that should appear/disappear with the GUI
-- This includes: title, close button, decorative frames, anything outside mainFrame
local companionElements = {}
for _, child in gui:GetChildren() do
    if child.Name:find("Title") or child.Name:find("Decor") then
        table.insert(companionElements, child)
    end
end

-- SAVE ORIGINAL PROPERTIES ONCE at startup (prevents transparency bug on reopen)
local companionOriginals = {}
for _, el in companionElements do
    if el:IsA("GuiObject") then
        companionOriginals[el] = {
            BackgroundTransparency = el.BackgroundTransparency,
            TextTransparency = el:IsA("TextLabel") and el.TextTransparency or nil,
            TextStrokeTransparency = el:IsA("TextLabel") and el.TextStrokeTransparency or nil,
        }
    end
end

local targetSize = mainFrame.Size
local isOpen = false

-------------------------------------------------
-- OPEN / CLOSE ANIMATIONS
-- CRITICAL: animate ALL elements together
-- CRITICAL: use saved originals, not current values
-------------------------------------------------
local function openGUI()
    if isOpen then return end
    isOpen = true
    
    -- Show ALL elements
    mainFrame.Visible = true
    dim.Visible = true
    for _, el in companionElements do el.Visible = true end
    
    -- Animate dim
    TweenService:Create(dim, TweenInfo.new(0.25), {
        BackgroundTransparency = 0.5
    }):Play()
    
    -- Animate frame (scale pop from center)
    mainFrame.AnchorPoint = Vector2.new(0.5, 0.5)
    mainFrame.Position = UDim2.new(0.5, 0, 0.5, 0)
    mainFrame.Size = UDim2.new(0, 0, 0, 0)
    TweenService:Create(mainFrame, TweenInfo.new(0.35, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
        Size = targetSize
    }):Play()
    
    -- Fade in companion elements using SAVED originals
    for _, el in companionElements do
        local orig = companionOriginals[el]
        if orig then
            el.BackgroundTransparency = 1
            if el:IsA("TextLabel") then
                el.TextTransparency = 1
                el.TextStrokeTransparency = 1
            end
            local tweenProps = {BackgroundTransparency = orig.BackgroundTransparency}
            if orig.TextTransparency then tweenProps.TextTransparency = orig.TextTransparency end
            if orig.TextStrokeTransparency then tweenProps.TextStrokeTransparency = orig.TextStrokeTransparency end
            TweenService:Create(el, TweenInfo.new(0.3), tweenProps):Play()
        end
    end
end

local function closeGUI()
    if not isOpen then return end
    isOpen = false
    
    -- Animate dim
    TweenService:Create(dim, TweenInfo.new(0.2), {
        BackgroundTransparency = 1
    }):Play()
    
    -- Fade out companion elements
    for _, el in companionElements do
        if el:IsA("GuiObject") then
            local tweenProps = {BackgroundTransparency = 1}
            if el:IsA("TextLabel") then
                tweenProps.TextTransparency = 1
                tweenProps.TextStrokeTransparency = 1
            end
            TweenService:Create(el, TweenInfo.new(0.15), tweenProps):Play()
        end
    end
    
    -- Animate frame
    local tween = TweenService:Create(mainFrame, TweenInfo.new(0.2, Enum.EasingStyle.Quint, Enum.EasingDirection.In), {
        Size = UDim2.new(0, 0, 0, 0)
    })
    tween:Play()
    tween.Completed:Connect(function()
        mainFrame.Visible = false
        dim.Visible = false
        for _, el in companionElements do el.Visible = false end
        
        -- RESTORE original values so next open works correctly
        for _, el in companionElements do
            local orig = companionOriginals[el]
            if orig then
                el.BackgroundTransparency = orig.BackgroundTransparency
                if orig.TextTransparency then el.TextTransparency = orig.TextTransparency end
                if orig.TextStrokeTransparency then el.TextStrokeTransparency = orig.TextStrokeTransparency end
            end
        end
    end)
end

-------------------------------------------------
-- WIRING (button, close, dim, escape)
-------------------------------------------------
openBtn.MouseButton1Click:Connect(function()
    if isOpen then closeGUI() else openGUI() end
end)

closeBtn.MouseButton1Click:Connect(closeGUI)
dim.MouseButton1Click:Connect(closeGUI)

UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == Enum.KeyCode.Escape and isOpen then
        closeGUI()
    end
end)

-------------------------------------------------
-- HOVER + CLICK EFFECTS (apply to ALL buttons)
-------------------------------------------------
local function setupButtonEffects(btn)
    if not btn:IsA("TextButton") and not btn:IsA("ImageButton") then return end
    if btn.Name == "Dim" then return end -- skip dim
    if btn.BackgroundTransparency >= 0.9 then return end -- skip invisible buttons
    
    local origSize = btn.Size
    local origColor = btn.BackgroundColor3
    local hoverColor = Color3.new(
        math.min(origColor.R * 1.15, 1),
        math.min(origColor.G * 1.15, 1),
        math.min(origColor.B * 1.15, 1)
    )
    
    -- Hover: lighten color
    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15), {
            BackgroundColor3 = hoverColor
        }):Play()
    end)
    btn.MouseLeave:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15), {
            BackgroundColor3 = origColor
        }):Play()
    end)
    
    -- Click: press down then bounce
    btn.MouseButton1Down:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.08), {
            Size = UDim2.new(
                origSize.X.Scale * 0.93, origSize.X.Offset,
                origSize.Y.Scale * 0.93, origSize.Y.Offset
            )
        }):Play()
    end)
    btn.MouseButton1Up:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.12, Enum.EasingStyle.Back), {
            Size = origSize
        }):Play()
    end)
end

-- Apply to every button in the GUI
for _, descendant in gui:GetDescendants() do
    if descendant:IsA("TextButton") or descendant:IsA("ImageButton") then
        setupButtonEffects(descendant)
    end
end

-------------------------------------------------
-- TAB SWITCHING (MUST swap actual content, not just colors)
-------------------------------------------------
local tabBar = mainFrame:FindFirstChild("TabBar")
if tabBar then
    local activeColor = Color3.fromRGB(50, 180, 255) -- adapt to style
    local inactiveColor = Color3.fromRGB(230, 230, 230)
    local activeTextColor = Color3.fromRGB(255, 255, 255)
    local inactiveTextColor = Color3.fromRGB(80, 80, 80)
    
    -- Collect all content frames (one per tab)
    -- Convention: each tab has a matching ScrollingFrame named "[TabName]Content"
    -- e.g., tab "Vehicles" -> frame "VehiclesContent"
    local contentFrames = {}
    for _, tab in tabBar:GetChildren() do
        if tab:IsA("TextButton") then
            local frameName = tab.Text:gsub("%s", "") .. "Content"
            local frame = mainFrame:FindFirstChild(frameName)
            if frame then
                contentFrames[tab.Text] = frame
            end
        end
    end
    
    -- If no separate content frames exist, look for a single ItemGrid
    local singleGrid = mainFrame:FindFirstChild("ItemGrid")
    
    local function switchTab(activeTab)
        -- Update tab visuals
        for _, t in tabBar:GetChildren() do
            if t:IsA("TextButton") then
                t.BackgroundColor3 = inactiveColor
                t.TextColor3 = inactiveTextColor
            end
        end
        activeTab.BackgroundColor3 = activeColor
        activeTab.TextColor3 = activeTextColor
        
        -- Swap content frames
        if next(contentFrames) then
            for name, frame in contentFrames do
                frame.Visible = (name == activeTab.Text)
            end
        end
        
        print("Tab selected: " .. activeTab.Text)
    end
    
    for _, tab in tabBar:GetChildren() do
        if tab:IsA("TextButton") then
            tab.MouseButton1Click:Connect(function()
                switchTab(tab)
            end)
        end
    end
end

-------------------------------------------------
-- BUY BUTTON CLICKS (if item cards exist)
-------------------------------------------------
local itemGrid = mainFrame:FindFirstChild("ItemGrid")
if itemGrid then
    for _, card in itemGrid:GetChildren() do
        if card:IsA("Frame") then
            local buyBtn = card:FindFirstChild("BuyBtn")
            local nameLabel = card:FindFirstChild("ItemName")
            if buyBtn then
                buyBtn.MouseButton1Click:Connect(function()
                    local itemName = nameLabel and nameLabel.Text or "unknown"
                    print("Buying: " .. itemName)
                    -- TODO: fire RemoteEvent to server
                end)
            end
        end
    end
end

print("[GUI] " .. gui.Name .. " fully wired")
```

---

## PATTERN 2: Dim Background
Always create this in every ScreenGui that has popups or major screens:

```lua
local dim = Instance.new("TextButton")
dim.Name = "Dim"
dim.Size = UDim2.new(1, 0, 1, 0)
dim.Position = UDim2.new(0, 0, 0, 0)
dim.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
dim.BackgroundTransparency = 1 -- starts invisible
dim.Text = ""
dim.AutoButtonColor = false
dim.BorderSizePixel = 0
dim.ZIndex = 5
dim.Visible = false
dim.Parent = screenGui
```

---

## PATTERN 3: Multiple Screens (Shop, Pets, Inventory, Settings)
When a game has multiple screens, use a screen manager:

```lua
local screens = {
    Shop = {frame = shopFrame, title = shopTitle, targetSize = UDim2.new(0.55, 0, 0.65, 0)},
    Pets = {frame = petsFrame, title = petsTitle, targetSize = UDim2.new(0.6, 0, 0.7, 0)},
    -- add more...
}
local currentScreen = nil

local function openScreen(name)
    -- Close current if any
    if currentScreen then
        local cur = screens[currentScreen]
        cur.frame.Visible = false
        if cur.title then cur.title.Visible = false end
    end
    
    -- Open new
    currentScreen = name
    local scr = screens[name]
    scr.frame.Visible = true
    if scr.title then scr.title.Visible = true end
    dim.Visible = true
    
    TweenService:Create(dim, TweenInfo.new(0.25), {BackgroundTransparency = 0.5}):Play()
    scr.frame.Size = UDim2.new(0, 0, 0, 0)
    TweenService:Create(scr.frame, TweenInfo.new(0.35, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
        Size = scr.targetSize
    }):Play()
end

local function closeAll()
    if not currentScreen then return end
    local scr = screens[currentScreen]
    TweenService:Create(dim, TweenInfo.new(0.2), {BackgroundTransparency = 1}):Play()
    local tween = TweenService:Create(scr.frame, TweenInfo.new(0.2, Enum.EasingStyle.Quint, Enum.EasingDirection.In), {
        Size = UDim2.new(0, 0, 0, 0)
    })
    tween:Play()
    tween.Completed:Connect(function()
        scr.frame.Visible = false
        if scr.title then scr.title.Visible = false end
        dim.Visible = false
    end)
    currentScreen = nil
end

-- Wire each button to its screen
shopBtn.MouseButton1Click:Connect(function() openScreen("Shop") end)
petsBtn.MouseButton1Click:Connect(function() openScreen("Pets") end)
closeBtn.MouseButton1Click:Connect(closeAll)
dim.MouseButton1Click:Connect(closeAll)
```

---

## PATTERN 4: Notification Toast
```lua
local function notify(screenGui, text, color, duration)
    duration = duration or 3
    color = color or Color3.fromRGB(80, 220, 50)
    
    local notif = Instance.new("Frame")
    notif.Size = UDim2.new(0.25, 0, 0.05, 0)
    notif.Position = UDim2.new(0.5, 0, -0.06, 0)
    notif.AnchorPoint = Vector2.new(0.5, 0)
    notif.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    notif.BorderSizePixel = 0
    notif.ZIndex = 50
    notif.Parent = screenGui
    Instance.new("UICorner", notif).CornerRadius = UDim.new(0, 12)
    local stroke = Instance.new("UIStroke", notif)
    stroke.Color = color
    stroke.Thickness = 3
    
    local label = Instance.new("TextLabel", notif)
    label.Size = UDim2.new(1, 0, 1, 0)
    label.Text = text
    label.Font = Enum.Font.GothamBold
    label.TextScaled = true
    label.TextColor3 = Color3.fromRGB(40, 40, 40)
    label.BackgroundTransparency = 1
    local pad = Instance.new("UIPadding", label)
    pad.PaddingLeft = UDim.new(0, 12)
    pad.PaddingRight = UDim.new(0, 12)
    
    TweenService:Create(notif, TweenInfo.new(0.4, Enum.EasingStyle.Back), {
        Position = UDim2.new(0.5, 0, 0.02, 0)
    }):Play()
    
    task.delay(duration, function()
        local out = TweenService:Create(notif, TweenInfo.new(0.25, Enum.EasingStyle.Quint), {
            Position = UDim2.new(0.5, 0, -0.06, 0)
        })
        out:Play()
        out.Completed:Wait()
        notif:Destroy()
    end)
end
```

---

## PATTERN 5: Slider (for settings)
```lua
local function createSlider(parent, name, yPos, defaultValue, onChange)
    local row = Instance.new("Frame", parent)
    row.Size = UDim2.new(0.9, 0, 0.12, 0)
    row.Position = UDim2.new(0.05, 0, yPos, 0)
    row.BackgroundTransparency = 1
    
    local label = Instance.new("TextLabel", row)
    label.Size = UDim2.new(0.3, 0, 1, 0)
    label.Text = name
    label.Font = Enum.Font.GothamBold
    label.TextScaled = true
    label.TextColor3 = Color3.fromRGB(40, 40, 40)
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.BackgroundTransparency = 1
    
    local track = Instance.new("Frame", row)
    track.Size = UDim2.new(0.55, 0, 0.3, 0)
    track.Position = UDim2.new(0.35, 0, 0.35, 0)
    track.BackgroundColor3 = Color3.fromRGB(200, 200, 200)
    track.BorderSizePixel = 0
    Instance.new("UICorner", track).CornerRadius = UDim.new(1, 0)
    
    local fill = Instance.new("Frame", track)
    fill.Size = UDim2.new(defaultValue, 0, 1, 0)
    fill.BackgroundColor3 = Color3.fromRGB(80, 220, 50)
    fill.BorderSizePixel = 0
    Instance.new("UICorner", fill).CornerRadius = UDim.new(1, 0)
    
    local knob = Instance.new("Frame", track)
    knob.Size = UDim2.new(0, 20, 0, 20)
    knob.AnchorPoint = Vector2.new(0.5, 0.5)
    knob.Position = UDim2.new(defaultValue, 0, 0.5, 0)
    knob.BackgroundColor3 = Color3.fromRGB(80, 220, 50)
    knob.BorderSizePixel = 0
    Instance.new("UICorner", knob).CornerRadius = UDim.new(1, 0)
    local knobStroke = Instance.new("UIStroke", knob)
    knobStroke.Color = Color3.fromRGB(40, 140, 20)
    knobStroke.Thickness = 2
    
    local valLabel = Instance.new("TextLabel", row)
    valLabel.Size = UDim2.new(0.08, 0, 1, 0)
    valLabel.Position = UDim2.new(0.92, 0, 0, 0)
    valLabel.Text = tostring(math.floor(defaultValue * 100))
    valLabel.Font = Enum.Font.GothamBold
    valLabel.TextScaled = true
    valLabel.TextColor3 = Color3.fromRGB(40, 40, 40)
    valLabel.BackgroundTransparency = 1
    
    -- Drag logic
    local dragging = false
    knob.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local trackAbsPos = track.AbsolutePosition.X
            local trackAbsSize = track.AbsoluteSize.X
            local mouseX = input.Position.X
            local pct = math.clamp((mouseX - trackAbsPos) / trackAbsSize, 0, 1)
            fill.Size = UDim2.new(pct, 0, 1, 0)
            knob.Position = UDim2.new(pct, 0, 0.5, 0)
            valLabel.Text = tostring(math.floor(pct * 100))
            if onChange then onChange(pct) end
        end
    end)
end
```

---

## PATTERN 6: Toggle Switch (for settings)
```lua
local function createToggle(parent, name, yPos, defaultOn, onToggle)
    local row = Instance.new("Frame", parent)
    row.Size = UDim2.new(0.9, 0, 0.12, 0)
    row.Position = UDim2.new(0.05, 0, yPos, 0)
    row.BackgroundTransparency = 1
    
    local label = Instance.new("TextLabel", row)
    label.Size = UDim2.new(0.6, 0, 1, 0)
    label.Text = name
    label.Font = Enum.Font.GothamBold
    label.TextScaled = true
    label.TextColor3 = Color3.fromRGB(40, 40, 40)
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.BackgroundTransparency = 1
    
    local toggleBg = Instance.new("TextButton", row)
    toggleBg.Size = UDim2.new(0.15, 0, 0.6, 0)
    toggleBg.Position = UDim2.new(0.8, 0, 0.2, 0)
    toggleBg.BackgroundColor3 = defaultOn and Color3.fromRGB(80, 220, 50) or Color3.fromRGB(180, 180, 180)
    toggleBg.Text = ""
    toggleBg.AutoButtonColor = false
    toggleBg.BorderSizePixel = 0
    Instance.new("UICorner", toggleBg).CornerRadius = UDim.new(1, 0)
    
    local knob = Instance.new("Frame", toggleBg)
    knob.Size = UDim2.new(0.45, 0, 0.85, 0)
    knob.AnchorPoint = Vector2.new(0, 0.5)
    knob.Position = defaultOn and UDim2.new(0.5, 0, 0.5, 0) or UDim2.new(0.05, 0, 0.5, 0)
    knob.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    knob.BorderSizePixel = 0
    Instance.new("UICorner", knob).CornerRadius = UDim.new(1, 0)
    
    local isOn = defaultOn
    toggleBg.MouseButton1Click:Connect(function()
        isOn = not isOn
        TweenService:Create(knob, TweenInfo.new(0.2, Enum.EasingStyle.Quint), {
            Position = isOn and UDim2.new(0.5, 0, 0.5, 0) or UDim2.new(0.05, 0, 0.5, 0)
        }):Play()
        TweenService:Create(toggleBg, TweenInfo.new(0.2), {
            BackgroundColor3 = isOn and Color3.fromRGB(80, 220, 50) or Color3.fromRGB(180, 180, 180)
        }):Play()
        if onToggle then onToggle(isOn) end
    end)
end
```

---

## WHEN TO USE EACH PATTERN

| User asks for | Use pattern |
|---|---|
| Any button + screen/frame | Pattern 1 (Button + Frame + LocalScript) |
| Multiple screens (shop + pets + inventory) | Pattern 1 + Pattern 3 (screen manager) |
| A popup (confirmation, reward, deal) | Pattern 1 (simplified, smaller frame) |
| Notifications | Pattern 4 |
| Settings with sliders | Pattern 5 |
| Settings with toggles | Pattern 6 |

**NEVER build GUI elements without also creating the LocalScript that controls them.** The visuals AND the logic are one package.
