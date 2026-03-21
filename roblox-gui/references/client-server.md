# Client-Server Communication for GUIs

## RemoteEvents (fire and forget, no return value)

### Setup
```lua
-- Server: create the remote
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local remotes = ReplicatedStorage:FindFirstChild("Remotes")
if not remotes then
    remotes = Instance.new("Folder")
    remotes.Name = "Remotes"
    remotes.Parent = ReplicatedStorage
end

local buyItemEvent = Instance.new("RemoteEvent")
buyItemEvent.Name = "BuyItem"
buyItemEvent.Parent = remotes
```

### Client → Server (player action)
```lua
-- Client: fire when player clicks buy
local remote = game.ReplicatedStorage.Remotes.BuyItem
remote:FireServer("SwordOfFlames", 500)

-- Server: listen
remote.OnServerEvent:Connect(function(player, itemName, price)
    -- ALWAYS validate on server, never trust client
    print(player.Name .. " wants to buy " .. itemName)
end)
```

### Server → Client (update UI)
```lua
-- Server: notify client of data change
local updateUI = remotes.UpdateUI
updateUI:FireClient(player, {coins = newBalance, item = itemName})

-- Client: listen and update GUI
updateUI.OnClientEvent:Connect(function(data)
    coinLabel.Text = data.coins
end)
```

### Server → All Clients (broadcast)
```lua
-- Server: announce to everyone
remote:FireAllClients("PlayerX bought a Legendary Sword!")

-- Client: show notification
remote.OnClientEvent:Connect(function(message)
    showNotification(message)
end)
```

## RemoteFunctions (request + response)

### Client asks server for data
```lua
-- Server: create and handle
local getShopItems = Instance.new("RemoteFunction")
getShopItems.Name = "GetShopItems"
getShopItems.Parent = remotes

getShopItems.OnServerInvoke = function(player)
    -- Return data to client
    return {
        {name = "Sword", price = 500, owned = false},
        {name = "Shield", price = 300, owned = true},
    }
end

-- Client: request and use
local items = game.ReplicatedStorage.Remotes.GetShopItems:InvokeServer()
for _, item in ipairs(items) do
    createItemCard(item.name, item.price, item.owned)
end
```

## Anti-Exploit for GUI Actions

### RULE: Never trust the client

**BAD (exploitable):**
```lua
-- Client tells server the price (client can change it to 0)
remote:FireServer("Sword", 0)  -- hacker sends price = 0
```

**GOOD (server validates):**
```lua
-- Client only sends what they want to buy
remote:FireServer("Sword")  -- just the item ID

-- Server looks up the REAL price from its own data
remote.OnServerEvent:Connect(function(player, itemId)
    local itemData = ServerItemDatabase[itemId]
    if not itemData then return end  -- invalid item
    
    local playerData = GetPlayerData(player)
    if playerData.coins < itemData.price then return end  -- can't afford
    if playerData.ownedItems[itemId] then return end  -- already owns
    
    -- All checks passed, process purchase
    playerData.coins -= itemData.price
    playerData.ownedItems[itemId] = true
    
    -- Notify client of success
    updateRemote:FireClient(player, {
        success = true,
        newBalance = playerData.coins,
        item = itemId
    })
end)
```

### Rate Limiting
```lua
-- Server: prevent spam-clicking
local lastAction = {}

remote.OnServerEvent:Connect(function(player, ...)
    local now = tick()
    local userId = player.UserId
    if lastAction[userId] and (now - lastAction[userId]) < 0.5 then
        return  -- too fast, ignore
    end
    lastAction[userId] = now
    
    -- Process normally...
end)
```

## Data Binding (update GUI from data)

### Leaderstats → GUI
```lua
-- Client: watch leaderstats and update HUD
local player = game.Players.LocalPlayer
local leaderstats = player:WaitForChild("leaderstats")
local coins = leaderstats:WaitForChild("Coins")

-- Initial
coinLabel.Text = coins.Value

-- Auto-update when value changes
coins.Changed:Connect(function(newValue)
    coinLabel.Text = newValue
    -- Optional: flash green on increase, red on decrease
end)
```

### RemoteEvent → GUI refresh
```lua
-- When server changes data, fire update to client
-- Client rebuilds the relevant GUI section

local dataUpdate = remotes.DataUpdated
dataUpdate.OnClientEvent:Connect(function(data)
    -- Refresh all GUI elements that depend on this data
    updateCoinDisplay(data.coins)
    updateInventoryGrid(data.inventory)
    updateEquippedItems(data.equipped)
end)
```

## Open/Close GUI Architecture

### Toggle Pattern (recommended)
```lua
-- Client LocalScript
local Players = game:GetService("Players")
local player = Players.LocalPlayer
local gui = player.PlayerGui:WaitForChild("ShopGui")
local mainFrame = gui:WaitForChild("MainFrame")
local isOpen = false

local function toggleShop()
    isOpen = not isOpen
    if isOpen then
        -- Load fresh data from server
        local items = remotes.GetShopItems:InvokeServer()
        populateShop(items)
        popIn(mainFrame)  -- animation
    else
        popOut(mainFrame)  -- animation
    end
end

-- Open via button
shopButton.MouseButton1Click:Connect(toggleShop)

-- Close via X button
closeButton.MouseButton1Click:Connect(function()
    if isOpen then toggleShop() end
end)

-- Close via Escape key
game:GetService("UserInputService").InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.Escape and isOpen then
        toggleShop()
    end
end)

-- Close via clicking dim background
dimFrame.MouseButton1Click:Connect(function()
    if isOpen then toggleShop() end
end)
```

## Sound Effects for GUI
```lua
-- Create sounds in ReplicatedStorage or attach to ScreenGui
local clickSound = Instance.new("Sound")
clickSound.SoundId = "rbxassetid://6895079853"  -- soft click
clickSound.Volume = 0.3
clickSound.Parent = gui

local purchaseSound = Instance.new("Sound")
purchaseSound.SoundId = "rbxassetid://6026984224"  -- coin/cash register
purchaseSound.Volume = 0.5
purchaseSound.Parent = gui

-- Play on events
button.MouseButton1Click:Connect(function()
    clickSound:Play()
end)
```
