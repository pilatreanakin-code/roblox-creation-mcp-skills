# Styling — UICorner, UIStroke, UIPadding, UIGradient, Fonts, Colors

## UICorner (rounded corners)
**Use on EVERY frame and button.** Default Roblox sharp corners look amateur.
```lua
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 8)  -- 8px radius (good default)
corner.Parent = frame
-- Soft: 4-6px | Standard: 8-10px | Very round: 12-16px | Pill: 999px
```

## UIStroke (borders/outlines)
**Use instead of BorderSizePixel** — much better looking.
```lua
local stroke = Instance.new("UIStroke")
stroke.Color = Color3.fromRGB(60, 60, 80)
stroke.Thickness = 1.5
stroke.Transparency = 0
stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border  -- Border or Contextual
stroke.Parent = frame
-- For glowing borders: use thicker stroke (2-3px) with bright color + slight transparency
```

## UIPadding (inner spacing)
**ALWAYS add to containers.** Text/elements touching frame edges = amateur.
```lua
local pad = Instance.new("UIPadding")
pad.PaddingTop = UDim.new(0, 12)
pad.PaddingBottom = UDim.new(0, 12)
pad.PaddingLeft = UDim.new(0, 16)
pad.PaddingRight = UDim.new(0, 16)
pad.Parent = frame
```

## UIGradient (color gradients)
```lua
local grad = Instance.new("UIGradient")
grad.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0, Color3.fromRGB(60, 60, 80)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(30, 30, 45)),
}
grad.Rotation = 90  -- top to bottom
grad.Parent = frame
-- For buttons: slight gradient makes them look 3D
-- For backgrounds: dark gradient adds depth
```

## UIAspectRatioConstraint (maintain proportions)
```lua
local aspect = Instance.new("UIAspectRatioConstraint")
aspect.AspectRatio = 1  -- 1:1 square (for item slots, icons)
aspect.Parent = frame
```

## UISizeConstraint (min/max size)
```lua
local sizeC = Instance.new("UISizeConstraint")
sizeC.MinSize = Vector2.new(200, 100)
sizeC.MaxSize = Vector2.new(600, 400)
sizeC.Parent = frame
```

## Fonts

Best fonts for Roblox GUIs:
```lua
-- Modern/Clean
Enum.Font.GothamBold      -- titles, buttons (MOST USED)
Enum.Font.GothamMedium    -- subtitles
Enum.Font.Gotham           -- body text

-- Gaming
Enum.Font.FredokaOne       -- playful, rounded
Enum.Font.Bangers          -- bold, impact

-- Fantasy/RPG
Enum.Font.SpecialElite     -- old paper feel
Enum.Font.Merriweather     -- serif, traditional

-- Sci-Fi
Enum.Font.Ubuntu           -- clean, technical
Enum.Font.RobotoMono       -- monospace, code-like

-- Text size guide:
-- Title: 24-32
-- Subtitle: 18-22
-- Body: 14-16
-- Small/caption: 10-12
-- Button text: 16-20
```

## Color Palettes by Style

### Clean/Modern (Adopt Me, Blox Fruits style)
```lua
local Colors = {
    bg_primary = Color3.fromRGB(25, 25, 35),        -- dark navy
    bg_secondary = Color3.fromRGB(35, 35, 50),      -- slightly lighter
    bg_card = Color3.fromRGB(45, 45, 60),            -- card/panel
    accent = Color3.fromRGB(80, 160, 255),           -- blue accent
    accent_hover = Color3.fromRGB(100, 180, 255),    -- lighter on hover
    success = Color3.fromRGB(80, 200, 100),          -- green (buy, confirm)
    danger = Color3.fromRGB(220, 70, 70),            -- red (delete, close)
    warning = Color3.fromRGB(255, 180, 50),          -- yellow/gold
    text_primary = Color3.fromRGB(255, 255, 255),    -- white
    text_secondary = Color3.fromRGB(170, 170, 190),  -- grey
    text_disabled = Color3.fromRGB(100, 100, 120),   -- dark grey
}
```

### Fantasy/RPG
```lua
local Colors = {
    bg_primary = Color3.fromRGB(20, 15, 10),
    bg_secondary = Color3.fromRGB(40, 30, 20),
    bg_card = Color3.fromRGB(55, 40, 25),
    accent = Color3.fromRGB(200, 160, 60),           -- gold
    success = Color3.fromRGB(60, 160, 60),
    danger = Color3.fromRGB(180, 40, 40),
    text_primary = Color3.fromRGB(240, 220, 180),    -- parchment white
    text_secondary = Color3.fromRGB(180, 160, 120),
    border = Color3.fromRGB(120, 90, 40),            -- gold border
}
```

### Sci-Fi/Tech
```lua
local Colors = {
    bg_primary = Color3.fromRGB(10, 10, 20),
    bg_secondary = Color3.fromRGB(15, 20, 35),
    bg_card = Color3.fromRGB(20, 30, 50),
    accent = Color3.fromRGB(0, 220, 255),            -- cyan
    accent_glow = Color3.fromRGB(0, 180, 220),
    success = Color3.fromRGB(0, 255, 130),
    danger = Color3.fromRGB(255, 50, 50),
    text_primary = Color3.fromRGB(200, 230, 255),
    text_secondary = Color3.fromRGB(120, 150, 180),
    border = Color3.fromRGB(0, 150, 200),
}
```

### Cartoon/Playful
```lua
local Colors = {
    bg_primary = Color3.fromRGB(255, 245, 230),      -- warm cream
    bg_secondary = Color3.fromRGB(255, 235, 210),
    bg_card = Color3.fromRGB(255, 255, 255),
    accent = Color3.fromRGB(255, 120, 50),            -- orange
    success = Color3.fromRGB(100, 210, 80),
    danger = Color3.fromRGB(240, 80, 80),
    text_primary = Color3.fromRGB(50, 40, 30),        -- dark brown
    text_secondary = Color3.fromRGB(120, 100, 80),
    border = Color3.fromRGB(200, 180, 150),
}
```

## Drop Shadow Technique
Roblox doesn't have native shadows. Fake it with an offset frame:
```lua
-- Shadow behind a panel
local shadow = Instance.new("Frame")
shadow.Name = "Shadow"
shadow.Size = UDim2.new(1, 6, 1, 6)
shadow.Position = UDim2.new(0, 3, 0, 3)
shadow.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
shadow.BackgroundTransparency = 0.7
shadow.BorderSizePixel = 0
shadow.ZIndex = -1  -- behind parent
shadow.Parent = frame

local shadowCorner = Instance.new("UICorner")
shadowCorner.CornerRadius = UDim.new(0, 10)
shadowCorner.Parent = shadow
```

## Rarity Color System (for items, pets, etc.)
```lua
local RarityColors = {
    Common = Color3.fromRGB(180, 180, 180),     -- grey
    Uncommon = Color3.fromRGB(80, 200, 80),     -- green
    Rare = Color3.fromRGB(60, 140, 255),        -- blue
    Epic = Color3.fromRGB(180, 60, 255),        -- purple
    Legendary = Color3.fromRGB(255, 180, 30),   -- gold
    Mythic = Color3.fromRGB(255, 50, 50),       -- red
}
```

---

## BUTTON SHAPES — HARD RULES

**Buttons are ALWAYS rounded squares or rounded rectangles. NEVER circles.**

### Corner Radius by Size
- **Large frames** (shop panel, inventory): UICorner 16-20px
- **Medium elements** (item cards, tab buttons): UICorner 10-12px
- **Small buttons** (close X, nav icons, action buttons): UICorner 6-8px
- **Tiny elements** (badges, indicators): UICorner 4-6px

**Rule: The smaller the element, the less rounded the corners.** A big shop frame = very round (16px). A small close button = slightly round (8px). A tiny badge = barely round (4px).

### Close Button
- ROUNDED SQUARE, never circle
- UICorner 8-10px (NOT UDim.new(1,0) which makes a circle)
- Thick border matching the style
- White bold "X" text

### Nav/Sidebar Buttons
- Square with UIAspectRatioConstraint
- UICorner 10-12px
- Thick border
- Icon inside (55-60% size, centered)
- Label BELOW the button: Position UDim2.new(0.5, 0, 1, 2) with AnchorPoint (0.5, 0). Gap must be 2-4 pixels MAX. Never more.

---

## TITLE TEXT — HARD RULES

- Minimum TextSize 56. Ideally 60-70. Must be the BIGGEST text on screen.
- Font: FredokaOne
- WHITE text, BLACK stroke (TextStrokeTransparency = 0)
- Position: ABOVE the main frame, touching or nearly touching the top edge

```lua
title.Font = Enum.Font.FredokaOne
title.TextSize = 62
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
title.TextStrokeTransparency = 0
```

---

## NAVIGATION BUTTON SIZE

- Minimum: UDim2.new(0.06, 0, 0, 0) with UIAspectRatioConstraint
- Recommended: 0.065 to 0.08
- Icon inside: 55-60% of button
- Label: FredokaOne size 15-16, 2-4px below button, no further

---

## CREATIVE DESIGN TECHNIQUES

**The AI must NOT make every screen look identical.** The STYLE stays consistent, the LAYOUT varies.

### PICK ONE TECHNIQUE PER SCREEN (not both, not neither)
For each screen, randomly choose ONE of these. Do NOT combine them. If the user specifies which one, use that.

**OPTION A — Colored Inner Frame:**
A white outer frame with a colored inner frame for depth.
```lua
local inner = Instance.new("Frame")
inner.Size = UDim2.new(0.95, 0, 0.93, 0)
inner.AnchorPoint = Vector2.new(0.5, 1)
inner.Position = UDim2.new(0.5, 0, 1, 0)  -- flush with bottom, no gap
inner.BackgroundColor3 = Color3.fromRGB(220, 240, 255)  -- pick a color
inner.Parent = outerFrame
-- Content (tabs, grid) goes INSIDE the inner frame
-- The inner frame MUST reach the bottom of the outer frame — no spacing
```

**OPTION B — Rotated Background Frame:**
A tilted frame behind the main one.
```lua
local decorFrame = Instance.new("Frame")
decorFrame.Name = "DecorFrame"
decorFrame.Size = UDim2.new(0.55, 0, 0.65, 0)
decorFrame.AnchorPoint = Vector2.new(0.5, 0.5)
decorFrame.Position = UDim2.new(0.5, 0, 0.5, 0)
decorFrame.Rotation = math.random(2, 4) * (math.random() > 0.5 and 1 or -1)
decorFrame.BackgroundColor3 = Color3.fromRGB(200, 220, 255)
decorFrame.ZIndex = mainFrame.ZIndex - 1
decorFrame.Parent = screenGui
-- Must animate with main frame
```

Color ideas: light blue (220,240,255), light purple (235,225,255), light green (225,255,230), light yellow (255,250,220), light pink (255,225,235)

---

## FRAME ALIGNMENT — HARD RULES

**ScrollingFrame (or content area) must reach the BOTTOM of its parent frame with NO gap.**
```lua
-- CORRECT: anchored to bottom, flush
scrollFrame.AnchorPoint = Vector2.new(0.5, 1)
scrollFrame.Position = UDim2.new(0.5, 0, 1, 0)  -- sits at bottom edge
scrollFrame.Size = UDim2.new(0.95, 0, 0.82, 0)  -- takes up remaining space

-- WRONG: floating above the bottom
scrollFrame.Position = UDim2.new(0.5, 0, 0.98, 0)  -- 2% gap at bottom
```

**If using a colored inner frame:** the inner frame ALSO reaches the bottom of the outer frame with no gap. The ScrollingFrame inside the inner frame reaches the bottom of the inner frame with no gap.

---

## ITEM CARD SPACING

**Cards inside grids need visible breathing room.** Use CellPadding with BOTH X and Y:
```lua
grid.CellPadding = UDim2.new(0.02, 0, 0.02, 0)  -- 2% padding between cards
-- For tighter grids: 0.015
-- For more spacious: 0.025
-- NEVER 0 or 0.005 — cards touching looks bad
```
Also add UIPadding on the ScrollingFrame itself: 10-12px all sides.

---

## TAB CONTENT SWITCHING — HARD RULES

**When a GUI has category tabs (Vehicles, Gear, etc.), each tab MUST show different content.** Not just change color — actually swap what's visible.

Two approaches:

**Approach A — Multiple ScrollingFrames (simple):**
Create a separate ScrollingFrame for each tab. Show the active one, hide the rest.
```lua
-- Create a frame for each category
local vehicleGrid = createGrid(items.vehicles)
local gearGrid = createGrid(items.gear)
-- etc.

-- Tab click swaps visibility
vehicleTab.MouseButton1Click:Connect(function()
    vehicleGrid.Visible = true
    gearGrid.Visible = false
    -- update tab colors
end)
```

**Approach B — Repopulate one ScrollingFrame (dynamic):**
One ScrollingFrame, clear and refill when tab changes.
```lua
local function populateGrid(grid, items)
    -- Clear existing cards
    for _, child in grid:GetChildren() do
        if child:IsA("Frame") then child:Destroy() end
    end
    -- Create new cards for this category
    for _, item in items do
        createItemCard(grid, item)
    end
end

vehicleTab.MouseButton1Click:Connect(function()
    populateGrid(itemGrid, vehicleItems)
end)
```

**If the user doesn't specify items per tab:** create placeholder cards for each tab with example names. The important thing is the tab WORKS and swaps content.

---

### Other tab/card layout variations
- **Pill tabs**: rounded pill shapes in a row
- **Underline tabs**: text only, active has colored bar underneath
- **Vertical sidebar tabs**: stacked on the left side
- **Icon tabs**: just icons, bigger, tooltips on hover
- **Banner cards**: image full top 60%, text and buy below
- **Minimal cards**: just icon and price, name on hover
- **Featured card**: first item 2x size spanning two columns

### IMPORTANT: Every GUI should use different creative choices. Variety makes it feel designed.
