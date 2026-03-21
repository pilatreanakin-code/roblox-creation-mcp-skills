# Luau Syntax Quick Reference

## Type Annotations
```lua
local function add(a: number, b: number): number
    return a + b
end

type PlayerData = {
    coins: number,
    level: number,
    inventory: {[number]: boolean},
}
```

## String Interpolation (Luau 0.61+)
```lua
local name = "Player"
local msg = `Hello {name}, you have {coins} coins!`
```

## Tables
```lua
-- Array
local fruits = {"apple", "banana", "cherry"}
for i, fruit in ipairs(fruits) do end

-- Dictionary
local stats = {health = 100, speed = 16}
for key, value in pairs(stats) do end

-- Useful methods
table.insert(t, value)
table.remove(t, index)
table.find(t, value)          -- returns index or nil
table.clone(t)                -- shallow copy
table.clear(t)                -- empty table
table.sort(t, comparator)
```

## Math
```lua
math.clamp(x, min, max)
math.random(1, 100)           -- integer between 1-100
math.random()                 -- float 0-1
math.floor(x)  math.ceil(x)  math.abs(x)
math.huge                     -- infinity
math.rad(degrees)             -- degrees to radians
```

## CFrame
```lua
CFrame.new(x, y, z)
CFrame.new(position, lookAtPosition)
CFrame.lookAt(from, to)
CFrame.Angles(rx, ry, rz)     -- radians
cf1 * cf2                      -- combine (apply cf2 relative to cf1)
cf * Vector3.new(0, 0, -5)    -- point 5 studs in front
```

## Vector3
```lua
Vector3.new(x, y, z)
v.Magnitude                    -- length
v.Unit                         -- normalized (length 1)
v1:Lerp(v2, alpha)            -- interpolate
(v1 - v2).Magnitude           -- distance between two points
```

## Instance
```lua
-- Create (Parent LAST)
local p = Instance.new("Part")
p.Size = Vector3.new(4,1,4)
p.Parent = workspace

-- Find
workspace:FindFirstChild("Name")        -- nil if not found
workspace:WaitForChild("Name")          -- yields until found
workspace:WaitForChild("Name", 5)       -- timeout after 5s
workspace:FindFirstChildOfClass("Part")
workspace:GetChildren()
workspace:GetDescendants()

-- Clone
local copy = template:Clone()
copy.Parent = workspace

-- Destroy (always use this, not Parent = nil)
instance:Destroy()
```

## Control Flow
```lua
-- Ternary
local x = condition and valueIfTrue or valueIfFalse

-- Nil coalescing (Luau)
local value = maybeNil or defaultValue

-- Optional chaining pattern
local name = player and player.Character and player.Character.Name
```

## Enums (always use, never strings)
```lua
Enum.KeyCode.Space
Enum.EasingStyle.Quint
Enum.Material.Neon
Enum.HumanoidStateType.Running
```
