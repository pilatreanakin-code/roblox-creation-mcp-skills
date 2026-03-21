# Layouts — UIListLayout, UIGridLayout, UIPageLayout, Responsive Design

## UIListLayout (vertical/horizontal lists)
**Use for:** Menu buttons, item lists, stat displays, any sequential content.
```lua
local list = Instance.new("UIListLayout")
list.FillDirection = Enum.FillDirection.Vertical    -- or Horizontal
list.HorizontalAlignment = Enum.HorizontalAlignment.Center
list.VerticalAlignment = Enum.VerticalAlignment.Top
list.Padding = UDim.new(0, 8)                       -- gap between items
list.SortOrder = Enum.SortOrder.LayoutOrder         -- sort by LayoutOrder property
list.Parent = scrollingFrame
```

**Auto-size the parent to fit contents:**
```lua
scrollingFrame.AutomaticCanvasSize = Enum.AutomaticSize.Y
-- The ScrollingFrame canvas grows to fit all items
```

## UIGridLayout (grids for inventory, shops)
**Use for:** Item grids, pet inventory, achievement badges, emoji picker.
```lua
local grid = Instance.new("UIGridLayout")
grid.CellSize = UDim2.new(0, 80, 0, 80)             -- each cell 80x80
grid.CellPadding = UDim2.new(0, 8, 0, 8)            -- gap between cells
grid.FillDirection = Enum.FillDirection.Horizontal
grid.HorizontalAlignment = Enum.HorizontalAlignment.Center
grid.SortOrder = Enum.SortOrder.LayoutOrder
grid.FillDirectionMaxCells = 4                        -- max 4 per row (0 = unlimited)
grid.Parent = scrollingFrame
```

**Responsive grid (cells scale with parent):**
```lua
grid.CellSize = UDim2.new(0.23, 0, 0.23, 0)  -- 23% of parent = ~4 columns with padding
```

## UIPageLayout (swipeable pages/tabs)
**Use for:** Tab systems, onboarding, category pages.
```lua
local page = Instance.new("UIPageLayout")
page.Animated = true
page.TweenTime = 0.3
page.EasingStyle = Enum.EasingStyle.Quint
page.EasingDirection = Enum.EasingDirection.Out
page.Circular = false
page.Padding = UDim.new(0, 0)
page.GamepadInputEnabled = false
page.ScrollWheelInputEnabled = false
page.TouchInputEnabled = true
page.Parent = tabContainer

-- Navigate pages
page:JumpToIndex(0)   -- first page
page:Next()           -- next page
page:Previous()       -- previous page
```

## UITableLayout (data tables)
```lua
local table = Instance.new("UITableLayout")
table.FillDirection = Enum.FillDirection.Vertical
table.Padding = UDim2.new(0, 4, 0, 4)
table.FillEmptySpaceColumns = true
table.Parent = tableFrame
-- Children are rows (Frames), grandchildren are cells
```

## Common Layout Patterns

### Tab Bar + Content Area
```lua
-- Top: horizontal tab buttons
-- Bottom: content area that changes based on selected tab

local tabBar = Instance.new("Frame")
tabBar.Size = UDim2.new(1, 0, 0, 44)
tabBar.Position = UDim2.new(0, 0, 0, 0)
tabBar.Parent = mainFrame

local tabLayout = Instance.new("UIListLayout")
tabLayout.FillDirection = Enum.FillDirection.Horizontal
tabLayout.Padding = UDim.new(0, 4)
tabLayout.Parent = tabBar

local contentArea = Instance.new("Frame")
contentArea.Size = UDim2.new(1, 0, 1, -48)
contentArea.Position = UDim2.new(0, 0, 0, 48)
contentArea.Parent = mainFrame
-- Each tab shows/hides its content frame inside contentArea
```

### Sidebar + Main Content
```lua
local sidebar = Instance.new("Frame")
sidebar.Size = UDim2.new(0.25, 0, 1, 0)
sidebar.Position = UDim2.new(0, 0, 0, 0)
sidebar.Parent = mainFrame

local content = Instance.new("Frame")
content.Size = UDim2.new(0.75, 0, 1, 0)
content.Position = UDim2.new(0.25, 0, 0, 0)
content.Parent = mainFrame
```

### Bottom Navigation Bar (mobile-friendly)
```lua
local navBar = Instance.new("Frame")
navBar.Size = UDim2.new(1, 0, 0, 60)
navBar.Position = UDim2.new(0, 0, 1, -60)
navBar.AnchorPoint = Vector2.new(0, 0)
navBar.Parent = screenGui

local navLayout = Instance.new("UIListLayout")
navLayout.FillDirection = Enum.FillDirection.Horizontal
navLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
navLayout.Padding = UDim.new(0, 20)
navLayout.Parent = navBar
-- Add icon buttons as children
```

### Centered Popup with Dim Background
```lua
-- Dim overlay (full screen, semi-transparent black)
local dim = Instance.new("Frame")
dim.Size = UDim2.new(1, 0, 1, 0)
dim.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
dim.BackgroundTransparency = 0.5
dim.ZIndex = 10
dim.Parent = screenGui

-- Popup centered on top
local popup = Instance.new("Frame")
popup.Size = UDim2.new(0.4, 0, 0.35, 0)
popup.AnchorPoint = Vector2.new(0.5, 0.5)
popup.Position = UDim2.new(0.5, 0, 0.5, 0)
popup.ZIndex = 11
popup.Parent = screenGui
```

### Item Card (for shops/inventory)
```lua
local function createItemCard(parent, name, price, rarity)
    local card = Instance.new("Frame")
    card.Size = UDim2.new(1, 0, 0, 90)
    card.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
    card.BorderSizePixel = 0
    card.Parent = parent
    Instance.new("UICorner", card).CornerRadius = UDim.new(0, 8)
    Instance.new("UIPadding", card).PaddingLeft = UDim.new(0, 12)
    
    -- Rarity stripe on left edge
    local stripe = Instance.new("Frame")
    stripe.Size = UDim2.new(0, 4, 1, 0)
    stripe.BackgroundColor3 = RarityColors[rarity]
    stripe.BorderSizePixel = 0
    stripe.Parent = card
    
    -- Item name
    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(0.6, 0, 0.5, 0)
    nameLabel.Position = UDim2.new(0.05, 0, 0, 0)
    nameLabel.Text = name
    nameLabel.Font = Enum.Font.GothamBold
    nameLabel.TextSize = 16
    nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    nameLabel.TextXAlignment = Enum.TextXAlignment.Left
    nameLabel.BackgroundTransparency = 1
    nameLabel.Parent = card
    
    -- Price
    local priceLabel = Instance.new("TextLabel")
    priceLabel.Size = UDim2.new(0.6, 0, 0.5, 0)
    priceLabel.Position = UDim2.new(0.05, 0, 0.5, 0)
    priceLabel.Text = "$" .. price
    priceLabel.Font = Enum.Font.Gotham
    priceLabel.TextSize = 14
    priceLabel.TextColor3 = Color3.fromRGB(255, 200, 50)
    priceLabel.TextXAlignment = Enum.TextXAlignment.Left
    priceLabel.BackgroundTransparency = 1
    priceLabel.Parent = card
    
    -- Buy button
    local buyBtn = Instance.new("TextButton")
    buyBtn.Size = UDim2.new(0.25, 0, 0.5, 0)
    buyBtn.Position = UDim2.new(0.72, 0, 0.25, 0)
    buyBtn.Text = "BUY"
    buyBtn.Font = Enum.Font.GothamBold
    buyBtn.TextSize = 14
    buyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    buyBtn.BackgroundColor3 = Color3.fromRGB(80, 200, 80)
    buyBtn.BorderSizePixel = 0
    buyBtn.AutoButtonColor = false
    buyBtn.Parent = card
    Instance.new("UICorner", buyBtn).CornerRadius = UDim.new(0, 6)
    
    return card, buyBtn
end
```
