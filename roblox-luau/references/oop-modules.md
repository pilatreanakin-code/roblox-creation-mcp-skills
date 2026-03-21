# OOP and Module Patterns

## Basic ModuleScript
```lua
-- ReplicatedStorage/Modules/MathUtils.ModuleScript
local MathUtils = {}

function MathUtils.lerp(a, b, t)
    return a + (b - a) * t
end

function MathUtils.formatNumber(n)
    if n >= 1e6 then return string.format("%.1fM", n/1e6)
    elseif n >= 1e3 then return string.format("%.1fK", n/1e3)
    else return tostring(n) end
end

return MathUtils
```
Usage: `local MathUtils = require(game.ReplicatedStorage.Modules.MathUtils)`

## OOP Class with Metatables
```lua
local Enemy = {}
Enemy.__index = Enemy

function Enemy.new(name, health, damage)
    return setmetatable({
        name = name,
        health = health,
        maxHealth = health,
        damage = damage,
        alive = true,
    }, Enemy)
end

function Enemy:TakeDamage(amount)
    self.health = math.max(0, self.health - amount)
    if self.health <= 0 then
        self.alive = false
        self:OnDeath()
    end
end

function Enemy:OnDeath()
    print(self.name .. " died")
end

return Enemy
```

## Singleton
```lua
-- Only one instance ever, shared across all requires
local SoundManager = {}
local initialized = false

function SoundManager:Init()
    if initialized then return end
    initialized = true
    self.sounds = {}
end

function SoundManager:Play(name)
    -- play sound
end

return SoundManager
```

## Module Organization
```
ReplicatedStorage/
  Modules/           -- shared between client and server
    Config.lua       -- game constants, item data
    Utils.lua        -- math, string helpers
    Types.lua        -- type definitions

ServerScriptService/
  ServerModules/     -- server-only modules
    DataManager.lua
    ShopHandler.lua

StarterPlayerScripts/
  ClientModules/     -- client-only modules
    GUIManager.lua
    InputHandler.lua
```

## Avoid Circular Dependencies
Module A requires Module B which requires Module A = ERROR.
Fix: extract shared logic into a third module C that both A and B require.
